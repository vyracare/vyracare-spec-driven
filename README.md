# Vyracare Spec-Driven

Esta pasta consolida o estado atual da arquitetura, do processo de entrega e das convencoes operacionais do ecossistema Vyracare.

O objetivo nao e substituir o codigo-fonte nem os READMEs dos repositorios. O objetivo e manter uma base de especificacao navegavel, orientada a AI-DLC, para:

- alinhar arquitetura e operacao entre frontend, backend, templates e infra
- documentar o fluxo real de ambientes `dev`, `hml` e `prod`
- registrar convencoes de naming, secrets, buckets, lambdas, gateways e branches
- registrar padroes de exposicao de Swagger e observabilidade operacional
- servir como baseline para agentes, automacoes, revisoes tecnicas e onboarding

## Estrutura

### Fundacao
- [AI-DLC](./01-foundation/ai-dlc.md)
- [Panorama de Repositorios](./01-foundation/repository-landscape.md)

### Arquitetura
- [Arquitetura Frontend](./02-architecture/frontend-architecture.md)
- [Arquitetura Backend](./02-architecture/backend-architecture.md)
- [Estrategia de Ambientes](./02-architecture/environment-strategy.md)

### Entrega
- [Esteiras Angular](./03-delivery/angular-pipelines.md)
- [Esteiras .NET](./03-delivery/dotnet-pipelines.md)
- [Templates e Scaffolding](./03-delivery/template-strategy.md)

### Seguranca
- [Secrets e Configuracao](./04-security/secrets-and-config.md)
- [Runtime, IAM e Integracoes](./04-security/runtime-and-access.md)

### Operacao
- [Runbooks e Diagnostico](./05-operations/runbooks.md)

## Escopo atual

Esta base reflete o que esta implementado ate o momento nos seguintes grupos:

- `vyracare-app-shell`
- `vyracare-app-user-mfe`
- `vyracare-app-profile-mfe`
- `vyracare-app-dashboard-mfe`
- `vyracare-app-proceedings-mfe`
- `vyracare-api-authentication`
- `vyracare-api-client`
- `vyracare-api-proceedings`
- `templates-angular`
- `template-dot-net-api`
- `vyracare-infra-pipes-angular`
- `vyracare-infra-pipes-dot-net`
- `vyracare-design-system`

## Leitura recomendada

1. Comecar por [AI-DLC](./01-foundation/ai-dlc.md)
2. Ler [Panorama de Repositorios](./01-foundation/repository-landscape.md)
3. Ler [Estrategia de Ambientes](./02-architecture/environment-strategy.md)
4. Ler [Esteiras Angular](./03-delivery/angular-pipelines.md)
5. Ler [Esteiras .NET](./03-delivery/dotnet-pipelines.md)
6. Consultar [Runbooks e Diagnostico](./05-operations/runbooks.md) durante incidentes

## Principios desta pasta

- descrever o estado real antes do estado ideal
- manter nomes, fluxos e recursos como estao implementados
- documentar tradeoffs quando a solucao atual nao for a mais elegante
- separar claramente arquitetura, processo e operacao
