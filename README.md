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

## Pré-requisitos:
Antes de começar a seguir as etapas, garanta que você possua todas essas ferramentas instaladas.

**Para todos os sistemas:**
- **Git**: Essencial para o vesionamento de código.
- **Conta no GitHub:** Necessária para criar um repositório público para os manifestos da aplicação.
- **Docker:** A plataforma de contêineres deve estar funcionando localmente. No Windows, o Rancher Desktop gerenciará isso; no Linux, pode ser necessário instalá-lo separadamente.
- **Kubectl:** A ferramenta de linha de comando para interagir com o Kubernetes. 
- **ArgoCD:** O argoCD deverá ser instalado no cluster.
- **k3d:** Uma ferramenta leve para criar clusters k3s (Kubernetes) locais. Siga o guia de instalação oficial.

**Para o ambiente windows:**
- **Rancher Desktop:** Instale o Rancher Desktop, o mesmo fornece um ambiente Kubernetes integrado via WSL2. Certifique-se de que o Kubernetes esteja habilitado nas configurações.
- **Chocolatey (Opcional, mas recomendado):** Um gerenciador de pacotes para instalar facilmente ferramentas de linha de comando.

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







   

 
