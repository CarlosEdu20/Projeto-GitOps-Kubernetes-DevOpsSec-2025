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

O princípio fundamental do **GitOps** é ter o **Git como a única fonte da verdade**.  
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
Nesse repositório ficarão apenas os arquivos YAML de configuração do Kubernetes.

**Passos:**
1. No GitHub, clique em **New repository**.  
2. Nomeie como preferir (ex: `online-boutique-gitops`).  
3. Deixe o repositório **público** para que o ArgoCD possa fazer a sicronização mais pra frente.  
4. Crie o repositório.

---

## 1.3. Estrutura do repositório de manifestos

Dentro do repositório recém-criado, crie a seguinte estrutura de pastas e arquivos:

<img width="1213" height="180" alt="image" src="https://github.com/user-attachments/assets/2d50fec3-ffed-4349-82ef-81ef4c428548" />

**Considerações:**

- O arquivo pode ter outro nome, mas recomendo **`online-boutique.yaml`** para manter consistência de acordo com o documento do projeto.
- O conteúdo desse arquivo yaml deve ser exatamente o do arquivo `release/kubernetes-manifests.yaml` presente no repositório oficial que você forkou.  












   

 




