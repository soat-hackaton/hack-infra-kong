# Hack-Infra-Kong

## 📌 O que é o projeto
Este repositório é responsável pela infraestrutura do **Kong API Gateway** para o projeto do Hackathon. Ele contém os manifestos Kubernetes necessários para implantar o Kong Ingress Controller em um cluster Amazon EKS. O Kong atua como o principal ponto de entrada (gateway) para a comunicação externa e roteamento para os microsserviços internos, aplicando políticas de forma unificada.

## 🚀 Principais tecnologias utilizadas
- **Kubernetes (K8s)**: Orquestração dos recursos da infraestrutura.
- **Kong API Gateway**: Utilizado como Ingress Controller e gerenciador de API.
- **AWS EKS**: Plataforma de Kubernetes gerenciada pela AWS.
- **GitHub Actions**: Automação da pipeline de deploy (CI/CD).

## 📋 Pré-requisitos / Variáveis de Ambiente
Para que o deploy da infraestrutura ocorra corretamente via GitHub Actions, os seguintes **Secrets** e **Variáveis** devem estar configurados no repositório:

- `AWS_ACCESS_KEY_ID`: Credencial de acesso da AWS.
- `AWS_SECRET_ACCESS_KEY`: Chave secreta de acesso da AWS.
- `AWS_SESSION_TOKEN`: Token de sessão (especialmente necessário se utilizar conta AWS Academy labs).
- `AWS_REGION`: Região da AWS (Ex: `us-west-2` - configurado como variável na pipeline).
- `EKS_CLUSTER_NAME`: Nome do cluster EKS alvo (Ex: `hack-infra-eks` - configurado na pipeline).

Um cluster Amazon EKS correspondente já deve estar provisionado e online.

## 🧪 Testes e Qualidade
Dado que o foco deste repositório é puramente em configuração de infraestrutura (`.yaml`), não existem testes unitários tradicionais baseados em código (como no SonarQube). A qualidade é garantida por meio de:
- **Gestão Declarativa**: Todos os manifestos são versionados no Git, seguindo os princípios de GitOps.
- **Garantia de Ordem no Deploy**: A pipeline foi desenhada para aplicar primeiro os CRDs (definições base do Kong) e aguardar (`sleep 10`) antes de aplicar as rotas e plugins, evitando falhas de sincronismo do K8s.

## 🔄 Pipeline CI/CD
A pipeline configurada no `.github/workflows/deploy-infra.yml` funciona de forma totalmente automatizada em cada aprovação/merge na branch `main`:
1. **Checkout Code**: Baixa as configurações.
2. **AWS Configure**: Autentica na conta da AWS.
3. **Kube Config Update**: Estabelece a conexão do `kubectl` com o cluster EKS através das credenciais.
4. **Deploy**:
   - Aplica a base (`kong-installation.yaml`).
   - Aguarda 10 segundos para a criação dos CRDs do Kong no cluster.
   - Aplica o diretório `plugins/` (acionando `cors.yaml` e `rate-limit.yaml` de forma global).

## 🛡️ Melhores Práticas Implementadas
- **Infraestrutura como Código (IaC)**: Rastreabilidade, versionamento e reversão facilitada.
- **Automação de Deploy**: Intervenção manual ausente na rotina de update do API Gateway.
- **Configuração Global e Centralizada (Plugins)**:
  - **CORS Global**: O arquivo `plugins/cors.yaml` garante as regras para as origens, headers e métodos suportados para todos os serviços, unificando a política do gateway.
  - **Rate Limit Global**: O arquivo `plugins/rate-limit.yaml` restringe em até `100` requisições por minuto no gateway, protegendo o ambiente interno de picos ou ataques volumétricos.
- **Proteção da Branch Base**: Branch `main` protegida, exigindo Pull Requests para mudanças na infraestrutura de roteamento.