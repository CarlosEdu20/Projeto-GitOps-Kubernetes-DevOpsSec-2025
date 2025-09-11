# GitOps na Prática: Implantando uma aplicação web com Kubernetes e ArgoCD

## Objetivo:
No cenário atual da programação, o desenvolvimento moderno de aplicações exige demandas de entregas cada vez mais rápidas, seguras e que possam ser escaláveis. Empresas como Netflix e Nubank  utilizam plataformas como kubernetes para orquestrar centenas (ou milhares) de containers de forma eficiente, automatizada e resiliente.
Ao mesmo tempo, surgiu a necessidade de tornar os processos de deploy mais auditáveis, previsíveis e versionados, e é nesse cenário que surge o GitOps, uma prática que usa o Git como 
fonte de verdade para toda a infraestrutura e aplicações.

Aprender Kubernetes permite entender como aplicações são executadas em ambientes distríbuidos, como escalar cargas, lidar com falhas e automatizar o ciclo de vida dos serviços. Já o GitOps, com ferramentas como o ArgoCD, permite fazer deploys de forma 
automatizada e segura apenas com um git push, trazendo mais controle e rastreabilidade 
para os times de desenvolvimento e operações. 


Para isso, utilizaremos a aplicação de exemplo **Online Boutique**, um conhecido conjunto de microserviços mantido pelo Google Cloud. A fonte da verdade para nossa implantação será um repositório GitOps contendo os manifestos Kubernetes necessários.

## Tecnologias Utilizadas:
Para todos os sistemas operacionais:
  - Kubernetes
  - ArgoCD
  - Docker
  - Git & Github
  - Kubectl
  - k3d 

Ambiente Windows:
  - Rancher Desktop
  - WSL2
  - Chocolatey

## Pré-requisitos
Antes de começar a seguir as etapas, garanta que você possua todas essas ferramentas instaladas.

**Para todos os sistemas:**
- **Git** ([Instalação](https://git-scm.com/downloads)): Essencial para o versionamento de código.  
- **Conta no GitHub** ([Criar conta](https://github.com/)): Necessária para criar um repositório público para os manifestos da aplicação.  
- **Docker** ([Instalação](https://docs.docker.com/get-docker/)): A plataforma de contêineres deve estar funcionando localmente.  
  - No Windows, o Rancher Desktop gerenciará isso.  
  - No Linux, pode ser necessário instalá-lo separadamente.  
- **Kubectl** ([Instalação](https://kubernetes.io/docs/tasks/tools/)): A ferramenta de linha de comando para interagir com o Kubernetes.  
- **ArgoCD** ([Instalação](https://argo-cd.readthedocs.io/en/stable/getting_started/)): O ArgoCD deverá ser instalado no cluster.  
- **k3d** ([Instalação](https://k3d.io/)): Uma ferramenta leve para criar clusters k3s (Kubernetes) locais.  

**Para o ambiente Windows:**
- **Rancher Desktop** ([Instalação](https://rancherdesktop.io/)): Fornece um ambiente Kubernetes integrado via WSL2. Certifique-se de que o Kubernetes esteja habilitado nas configurações.  
- **Chocolatey (Opcional, mas recomendado)** ([Instalação](https://chocolatey.org/install)): Um gerenciador de pacotes para instalar facilmente ferramentas de linha de comando.  


**Requisitos de hardware:**
Rodar um cluster Kubernetes localmente consome uma quantidade significativa de recursos computacionais. Esta recomendação visa garantir uma experiência fluida durante o projeto.
- **Sistema Operacional:**
   - **Windows:** Windows 10 (versão 2004 ou superior) ou Windows 11, com suporte a WSL2.
   - **Linux:** Qualquer distro moderna.

- **Processador (CPU):**
  - **Minímo:** 2 núcleos.
  - **Recomendado:** 4 núcleos ou mais

- **Memória (RAM):**
  - **Mínimo:** 8 GB.
  - **Recomendado:** 16 GB ou mais.

 - **Armazenamento (Disco):** 30 GB de espaço livre para acomodar as ferramentas, as imagens dos contêineres e os dados do cluster.


# Etapa 1: Preparação do repositório GitHub

O princípio fundamental do processo de **GitOps** é ter o **Git como a única fonte da verdade**.  
Por isso, o primeiro passo prático deste projeto é preparar um repositório que conterá a configuração declarativa da aplicação.  
É importante que você já possua uma conta no GitHub para seguir esta etapa.

---

## 1.1. Fork do repositório da aplicação

Para ter acesso aos manifestos Kubernetes da aplicação **Online Boutique**, primeiro vamos fazer um **fork** (uma cópia) do repositório oficial do Google Cloud Platform para a sua conta.  

🔗 Repositório oficial: [GoogleCloudPlatform/microservices-demo](https://github.com/GoogleCloudPlatform/microservices-demo)  

**Passos:**
1. Acesse o link acima.  
2. Clique no botão **Fork** (canto superior direito).  
3. Escolha a sua conta do GitHub como destino.  

Isso criará uma cópia completa do repositório na sua conta.

---

## 1.2. Criação do repositório de manifestos (GitOps)

Para manter o projeto organizado, vamos criar **outro repositório** que será usado pelo **ArgoCD** como fonte da verdade.  
Neste repositório ficarão apenas os arquivos YAML de configuração do Kubernetes.

**Passos:**
1. No GitHub, clique em **New repository**.  
2. Nomeie como preferir (ex: `online-boutique-gitops`).  
3. Deixe o repositório **público** para que o ArgoCD possa fazer a sincronização mais pra frente.  
4. Crie o repositório.

---

## 1.3. Estrutura do repositório de manifestos

Dentro do repositório recém-criado, crie a seguinte estrutura de pastas e arquivos:

<img width="1213" height="180" alt="image" src="https://github.com/user-attachments/assets/2d50fec3-ffed-4349-82ef-81ef4c428548" />

1. Clone o repositório para sua máquina local usando o `git clone`.
2. Dentro do repositório, crie uma pasta usando o comando `mkdir k8s` e entre na mesma usando o comando `cd k8s`.
3. Dentro da mesma, crie o arquivo yaml usando o comando `touch online-boutique.yaml`.

**Considerações:**

- O arquivo pode ter outro nome, mas recomendo **`online-boutique.yaml`** para manter consistência de acordo com o documento do projeto.
- O conteúdo desse arquivo yaml deve ser exatamente o do arquivo `release/kubernetes-manifests.yaml` presente no repositório oficial que você forkou.

---

## 1.4. Entendendo um pouco de Kubernetes e Clusters

Antes de prosseguirmos com a criação do cluster, é importante entender, de forma simples, o que é o **Kubernetes** e qual o papel de um **cluster**.

- **Kubernetes**: É uma plataforma open-source criada pelo Google usada para **orquestrar contêineres** (como os do Docker).  
  Ele automatiza tarefas importantes, como:
  - Implantação de aplicações.
  - Escalonamento automático (aumentar ou reduzir instâncias conforme a demanda).
  - Atualizações sem tempo de inatividade (rolling updates).
  - Monitoramento e recuperação automática em caso de falhas.

- **Cluster Kubernetes**: É o **conjunto de máquinas (nós)** que executam o Kubernetes.  
  Ele é dividido em:
  - **Control Plane (plano de controle)**: responsável por gerenciar o estado do cluster e decidir onde e como os contêineres devem rodar.  
  - **Workers (nós de trabalho)** → onde de fato os contêineres e aplicações são executados.

O Kubernetes é o “cérebro” que gerencia aplicações em contêineres, e o cluster é a infraestrutura (as máquinas) onde tudo roda.
Para este projeto, irei montar um cluster dentro da minha própria máquina, em um ambiente de produção, normalmente se usa os clusters disponíveis na nuvem junto ao kubernetes para orquestrá-los.

## 1.5. Criando o cluster usando o k3d

Agora que já expliquei um pouco dos conceitos básicos de Kubernetes e clusters, irei criar um cluster **local** usando o **k3d**.  
O `k3d` é uma ferramenta que facilita a execução do **k3s** (uma versão leve do Kubernetes) dentro de contêineres Docker.

Execute o comando abaixo para criar um cluster chamado `gitops-cluster` com **1 servidor (control plane)** e **2 nós workers**:

`k3d cluster create gitops-cluster --servers 1 --agents 2`

Criei 2 nós workers no cluster para simular um ambiente mais próximo da realidade e demonstrar como o Kubernetes distribui a carga de trabalho.

Após a criação, verifique se o **kubectl** está conectado ao cluster usando o comando abaixo:

`kubectl cluster-info`

O mesmo deverá retornar a seguinte imagem.

<img width="1446" height="134" alt="image" src="https://github.com/user-attachments/assets/66e45aa7-3ca1-43e8-8ddd-b987e132055f" />

Liste os nós do cluster para confirmar que temos 1 servidor de control-plane e 2 workers usando o comando:

`kubectl get nodes`

A saida esperada deve ser essa:

<img width="1918" height="124" alt="image" src="https://github.com/user-attachments/assets/3a97f7b4-8c12-491d-9b47-2977e63bc969" />













   

 





