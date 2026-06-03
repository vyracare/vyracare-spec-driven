# AI-DLC

AI-DLC, neste contexto, e o ciclo de vida de entrega orientado por especificacao, automacao e verificacao assistida por IA.

No Vyracare, isso significa que a evolucao do ecossistema nao depende apenas de implementacao manual. Ela depende de:

- especificacoes operacionais claras
- templates de projeto
- esteiras reutilizaveis
- convencoes fortes de ambientes
- documentacao navegavel
- automacao de promocao entre `develop`, `release/*` e `main`

## Objetivos

- reduzir variacao entre repositorios
- evitar configuracoes ad hoc
- permitir que agentes e desenvolvedores trabalhem com o mesmo contrato tecnico
- manter frontend, backend e infra promovidos por fluxo previsivel
- tornar falhas investigaveis com base em naming, secrets e conventions

## Ciclo adotado

### 1. Specification
A mudanca comeca por contrato, nao por codigo solto. Isso inclui:

- nome do repositorio
- nome do ambiente
- nome de bucket, lambda, gateway e cloudfront
- estrategia de secrets
- estrategia de banco por ambiente
- estrategia de branch e promocao

### 2. Scaffolding
Novos projetos nascem por template:

- Angular via `templates-angular`
- .NET via `template-dot-net-api`

Os templates ja embutem:

- convencoes de branch
- workflows
- wiring com reusable workflows
- estrutura de codigo esperada
- convencao de commits em portugues

### 3. Verification
A promocao entre ambientes passa por CI antes de CD.

Hoje isso se traduz em:

- Angular:
  - CI de `build`
  - CI de `test`
- .NET:
  - job de `test-dotnet`
  - job de `build-dotnet`
  - build nao roda se os testes falharem

### 4. Promotion
O fluxo de promocao atual segue o modelo:

- `feature/*` ou `fix/*` -> PR para `develop`
- `develop`:
  - publica `dev`
  - cria `release/<versionamento>`
  - abre PR para release
- `release/*`:
  - roda CI novamente
  - publica `hml`
  - abre PR para `main`
- `main`:
  - publica `prod`

### 5. Runtime feedback
O ciclo fecha com observabilidade e diagnostico operacional:

- CloudWatch para Lambdas
- resposta do API Gateway
- recursos provisionados na AWS
- ambiente e secret corretos por branch

## Papel da especificacao

Esta pasta existe porque o ecossistema ja passou do ponto em que "lembrar de cabeca" e seguro.

Sem especificacao centralizada, os riscos aumentam:

- drift entre repositorios
- naming inconsistente
- secrets de ambiente errados
- pipelines divergentes
- contratos quebrados entre APIs e MFEs

Com especificacao centralizada, o objetivo e que qualquer novo repositorio siga o mesmo comportamento sem reinterpretacao local.

## Convencao de commit

Os commits da organizacao devem ser escritos em portugues.

Objetivo:

- manter consistencia de historico
- facilitar leitura operacional pelo time
- reduzir variacao semantica entre repositorios

Exemplos esperados:

- `feat: adiciona environment dev ao shell`
- `fix: corrige rotas de swagger da autenticacao`
- `docs: atualiza matriz de ambientes e acessos`
