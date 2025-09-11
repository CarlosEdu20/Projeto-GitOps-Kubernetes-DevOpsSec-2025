# GitOps na Prática: Implantando uma aplicação web com Kubernetes e ArgoCD

## Objetivo:
No cenário atual da programação, o desenvolvimento moderno de aplicações exige demandas de entregas cada vez mais rápidas, seguras e que possam ser escaláveis. Empresas como Netflix e Nubank utilizam plataformas como kubernetes para orquestrar centenas (ou milhares) de containers de forma eficiente, automatizada e resiliente. Ao mesmo tempo, surgiu a necessidade de tornar os processos de deploy mais auditáveis, previsíveis e versionados, e é nesse cenário que surge o GitOps, uma prática que usa o Git como fonte de verdade para toda a infraestrutura e aplicações.

Aprender Kubernetes permite entender como aplicações são executadas em ambientes distríbuidos, como escalar cargas, lidar com falhas e automatizar o ciclo de vida dos serviços. Já o GitOps, com ferramentas como o ArgoCD, permite fazer deploys de forma automatizada e segura apenas com um git push, trazendo mais controle e rastreabilidade para os times de desenvolvimento e operações. 

Para este projeto, utilizarei a aplicação de exemplo **Online Boutique**, um sistema de demonstração mantido pelo **Google Cloud**, amplamente usado para estudos de Kubernetes e microserviços.  

A aplicação é composta por diversos microserviços que simulam uma loja virtual completa, sendo um ótimo exemplo prático para observar como o Kubernetes gerencia aplicações distribuídas.  

Neste cenário, a **fonte da verdade** para a implantação será um **repositório GitOps**, onde estão todos os **manifestos Kubernetes** necessários para que o ArgoCD realize o deploy e mantenha o estado do cluster sempre sincronizado com o que está versionado no Git.


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

## 1.5. Criando um cluster usando o k3d
Agora que já expliquei um pouco dos conceitos básicos de Kubernetes e clusters, irei criar um cluster **local** usando o **k3d**.  
O `k3d` é uma ferramenta que facilita a execução do **k3s** (uma versão leve do Kubernetes) dentro de contêineres Docker.


#### 1.5.1. Instalação do k3d

Antes de criar o cluster, você precisa instalar o `k3d`. O método varia conforme o seu sistema operacional.

* **Para Linux e macOS (via script):**
    ```bash
    curl -s [https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh](https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh) | bash
    ```

* **Para Windows (via Chocolatey):**
    ```powershell
    choco install k3d
    ```


#### 1.5.2. Criação do Cluster Multi-Nó

Agora, com o k3d instalado, execute o comando abaixo para criar um cluster chamado `gitops-cluster`. Ele terá **1 nó de controle (server)** e **2 nós de trabalho (workers)** para simular um ambiente mais próximo da realidade e demonstrar como o Kubernetes distribui a carga de trabalho.

```bash
k3d cluster create gitops-cluster --servers 1 --agents 2

````

#### 1.5.3. Verificação do Cluster
Liste os nós do cluster para confirmar que temos 1 servidor de control-plane e 2 workers usando o comando:

`kubectl get nodes`

A saida esperada deve ser essa:

<img width="1918" height="124" alt="image" src="https://github.com/user-attachments/assets/3a97f7b4-8c12-491d-9b47-2977e63bc969" />



## Etapa 2: Instalação do ArgoCD
Antes de prosseguirmos com a instalação, é importante entender **o que é o ArgoCD** e **por que vamos utilizá-lo neste projeto**.  

O **ArgoCD** é uma ferramenta **open source** voltada para **GitOps** e **Continuous Delivery (CD)** em ambientes Kubernetes.  
O mesmo foi desenvolvido pela comunidade do projeto **Argo** e tem como principal objetivo **automatizar a implantação e o gerenciamento de aplicações em clusters Kubernetes**.

Diferente de pipelines tradicionais de CI/CD, o ArgoCD segue o conceito de **GitOps**, onde o **repositório Git é a única fonte de verdade** (*single source of truth*). Isso significa que:

- O estado desejado da aplicação (manifests YAML, Helm Charts ou Kustomize) fica armazenado no Git.  
- O ArgoCD monitora continuamente esse repositório.  
- Se houver alguma mudança (commit, merge, atualização), ele aplica automaticamente no cluster.  
- Se o estado atual do cluster divergir do estado descrito no Git, o ArgoCD identifica essa diferença e pode **sincronizar** automaticamente ou sob demanda.

### Por que usar o ArgoCD?
- **Automatiza deploys:** reduz a necessidade de aplicar `kubectl apply` manualmente.  
- **Observabilidade:** fornece uma interface gráfica (UI) e linha de comando para visualizar o status das aplicações.  
- **Confiabilidade:** garante que o cluster esteja sempre em conformidade com o que está versionado no Git.  
- **Rollback simplificado:** em caso de falha, é possível voltar para um commit anterior com facilidade.  
- **Escalabilidade:** pode gerenciar múltiplos clusters e aplicações de forma centralizada.  

No contexto deste projeto, o ArgoCD será usado para **implantar aplicações de forma declarativa e automatizada**, tornando o processo de entrega mais seguro, auditável e próximo do que acontece em ambientes de produção reais.

---

## 2.1. Criação do namespace do ArgoCD
É uma boa prática antes de instalar o ArgoCD, criar seu próprio namespace para manter o cluster organizado. Isso evita que todos os componentes do ArgoCD sejam instalados no namespace "default" do Kubernetes.
 ```
    kubectl create namespace argocd
 ```


### 2.2. Instalação via Manifesto Oficial

A forma mais comum de instalar o ArgoCD é aplicando um manifesto YAML que contém todos os recursos necessários (Deployments, Services, ConfigMaps, etc.) para que a ferramenta funcione corretamente. Para isso, vamos utilizar o seguinte comando:

    ```
    kubectl apply -n argocd -f [https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml](https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml)
    ```

Após executar este comando, o Kubernetes começará a baixar as imagens e a criar os pods do ArgoCD. Isso pode levar alguns minutos. Você pode acompanhar o status com o comando `kubectl get pods -n argocd`.


### 2.3. Instalação do ArgoCD CLI (Ferramenta de Linha de Comando)
Para interagir com o ArgoCD via terminal (além da interface gráfica), precisamos instalar sua ferramenta de linha de comando (argocd). O método de instalação varia conforme o sistema operacional.

* **Para Linux e macOS:**
```
    # Baixa o binário
    curl -sSL -o argocd-linux-amd64 [https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64](https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64)
```
```
    # Instala o binário
    sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
```

```
    # Remove o arquivo baixado
    rm argocd-linux-amd64
```

* **Para Windows (via Chocolatey):**
O método mais simples no Windows é usar o gerenciador de pacotes Chocolatey.
```
    choco install argocd-cli
```

Após a instalação, verifique se foi bem-sucedida com o comando `argocd version --client.`

---

## Etapa 3: Acessar o ArgoCD localmente
Com o ArgoCD já instalado e rodando dentro do cluster, o próximo passo é acessar sua interface gráfica através do navegador de internet. Por padrão, o ArgoCD é instalado como um serviço do tipo `ClusterIP`, o que significa que ele não é exposto diretamente para fora do cluster. Para acessá-lo a partir do nosso navegador, é necessário fazer um túnel de rede conhecido como `port-forward`.

O comando `port-forward` cria uma ponte segura entre uma porta do seu computador local (`localhost`), normalmente essa porta é a 8080, e a porta do serviço do ArgoCD rodando dentro do cluster. Com isso em mente, vamos utilizar o seguinte comando:

```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Nota: Este terminal deve permanecer aberto enquanto você quiser acessar a interface do ArgoCD.

Agora vamos acessar o navegador usando o seguinte link:

 **[https://localhost:8080](https://localhost:8080)**

Nota: Você verá um aviso de segurança sobre um certificado inválido. Isso é normal e esperado. Clique em "Avançado" e depois em "Continuar para localhost (não seguro)" para prosseguir.

## 3.1. Login na interface
Para o primeiro acesso, o ArgoCD gera uma senha inicial. O nome de usuário é sempre `admin`. O método para obter a senha varia conforme o sistema operacional.

**Para Linux e macOS (via `base64`):**
Execute o comando abaixo para obter a senha e decodificá-la automaticamente.
```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

**Para Windows (via PowerShell):**
No Windows, o comando `base64` não é nativo. Por causa disso, o processo é feito em dois passos:

1. Primeiro passo, obtenha a senha codificada:

```
 kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"
```
Copie a longa cadeia de caracteres que aparecerá na tela.

2.  Segundo passo, decodifique a senha:
```
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String("SUA_SENHA_CODIFICADA_AQUI"))
```

Substitua `"SUA_SENHA_CODIFICADA_AQUI"` pela sequência que você copiou.

Após obter a senha, utilize as credenciais abaixo para entrar na interface do ArgoCD:
- **Username:** `admin`
- **Password:** A senha que você acabou de decodificar.

























   

 








