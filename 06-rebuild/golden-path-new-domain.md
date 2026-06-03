# Golden Path para Novo Dominio

Este documento descreve o caminho recomendado para criar um novo dominio no ecossistema Vyracare.

O objetivo e evitar que cada novo produto seja montado de forma diferente.

## Quando usar este guia

Use este fluxo quando surgir:

- uma nova API de dominio
- um novo MFE Angular
- uma nova integracao shell <-> MFE
- um novo fluxo de negocio que precise nascer alinhado ao padrao da organizacao

## Resultado esperado

Ao final do processo, o novo dominio deve ter:

- repositorio criado a partir do template correto
- branch flow alinhado com `develop`, `release/*` e `main`
- naming consistente com os recursos AWS
- CI e CD funcionando
- ambientes `dev`, `hml` e `prod`
- documentacao minima do repositorio
- convencao de commits em portugues

## Etapa 1. Definir o tipo de dominio

Antes de criar repositorio, decida:

1. o dominio e frontend, backend ou ambos
2. qual nome final do repositorio
3. quais ambientes ele precisara publicar
4. se ele sera consumido pelo shell
5. se ele consumira alguma API existente

### Exemplos de naming

- API: `vyracare-api-appointments`
- MFE: `vyracare-app-appointments-mfe`

## Etapa 2. Escolher o template

### Se for Angular

Use:

- `templates-angular`

O projeto novo deve nascer com:

- `environments.ts`
- `environments.dev.ts`
- `environments.hml.ts`
- `environments.prod.ts`
- `pr-to-develop.yml`
- `pr-to-release.yml`
- `pr-to-main.yml`
- `publish.yml`

### Se for .NET

Use:

- `template-dot-net-api`

O projeto novo deve nascer com:

- estrutura da API
- projeto de testes
- `GlobalUsings.cs` no projeto de testes
- `pr-to-develop.yml`
- `pr-to-release.yml`
- `pr-to-main.yml`
- `publish.yml`

## Etapa 3. Nomear corretamente

### Frontend

- bucket `dev`: `<repo>-dev`
- bucket `hml/prod`: `<repo>`

### Backend

- Lambda `dev`: `<repo-normalizado>-dev`
- Lambda `hml`: `<repo-normalizado>-hml`
- Lambda `prod`: `<repo-normalizado>`
- API Gateway segue o mesmo padrao da Lambda

### Dados e secrets

- `vyracare_db_dev`
- `vyracare_db_hml`
- `vyracare_db`

Se o dominio precisar de novos secrets, nomeie por ambiente.

## Etapa 4. Criar o repositorio

### Sequencia recomendada

1. gerar o projeto pelo template
2. ajustar nome final do projeto
3. validar README
4. validar workflows gerados
5. validar convencao de commits

## Etapa 5. Ligar o repositorio as reusable workflows

### Angular

O repositorio novo deve usar:

- `vyracare-infra-pipes-angular/.github/workflows/ci-angular.yml`
- `vyracare-infra-pipes-angular/.github/workflows/cd-angular.yml`

### .NET

O repositorio novo deve usar:

- `vyracare-infra-pipes-dot-net/.github/workflows/ci-generic-dot-net.yml`
- `vyracare-infra-pipes-dot-net/.github/workflows/cd-generic-dot-net.yml`

Ou, se for autenticacao ou fluxo especial:

- `ci-auth-dot-net.yml`
- `cd-auth-dot-net.yml`

## Etapa 6. Definir ambientes

### Angular

- `environment.ts`: local
- `environment.dev.ts`: `develop`
- `environment.hml.ts`: `release/*`
- `environment.prod.ts`: `main`

### .NET

- `develop` publica `dev`
- `release/*` publica `hml`
- `main` publica `prod`

## Etapa 7. Integrar com consumers

### Se for um novo MFE

Atualize o shell para receber:

- novo `remoteEntry.js`
- nova rota
- novo item de navegacao quando fizer sentido

### Se for uma nova API

Decida se ela precisa atualizar automaticamente algum frontend consumidor.

Se sim:

1. criar `.vyracare/mfe-consumer.json`
2. mapear o repositorio consumidor
3. definir a chave de environment que sera atualizada

## Etapa 8. Implementar a primeira entrega minima

### Angular

Entrega minima recomendada:

- rota principal
- componente inicial
- README basico
- teste minimo

### .NET

Entrega minima recomendada:

- controller ou endpoint
- handler
- porta
- implementacao de infraestrutura
- teste unitario
- Swagger funcional

## Etapa 9. Publicar em dev

### Fluxo esperado

1. PR para `develop`
2. merge em `develop`
3. CI passa
4. publish em `dev`
5. criacao automatica de `release/<versionamento>`
6. PR automatica para release

### Validacao minima em dev

- frontend acessivel
- backend respondendo
- shell consumindo o endpoint ou o `remoteEntry`
- logs sem erro de bootstrap

## Etapa 10. Promover para hml

### Fluxo esperado

1. merge em `release/*`
2. CI roda novamente
3. publish em `hml`
4. PR automatica para `main`

### Validacao minima em hml

- `environment.hml.ts` correto
- banco `hml` correto
- secrets `hml` corretos
- Swagger acessivel no caso de APIs

## Etapa 11. Promover para prod

### Fluxo esperado

1. merge em `main`
2. publish em `prod`

### Validacao minima em prod

- URL publicada correta
- `remoteEntry.js` correto
- API correta
- banco `prod`
- secrets `prod`

## Etapa 12. Atualizar documentacao

Depois da primeira publicacao funcional:

1. atualizar README do repositorio
2. atualizar `vyracare-spec-driven` se o dominio introduzir novo contrato
3. atualizar o perfil da organizacao se houver nova URL publica relevante

## Erros que este golden path tenta evitar

- repositorio criado sem template
- pipeline divergente
- ambiente `dev` usando arquivo local
- frontend consumindo API do ambiente errado
- secret sem padrao por ambiente
- novo dominio sem README
- naming AWS inconsistente

## Definicao de pronto

Um novo dominio so deve ser considerado alinhado ao ecossistema quando:

- usa o template correto
- usa as reusable workflows corretas
- publica em `dev`, `hml` e `prod`
- respeita naming e secrets por ambiente
- tem README minimamente util
- tem commits em portugues
