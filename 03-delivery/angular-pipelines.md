# Esteiras Angular

## Repositorio base

As esteiras reutilizaveis Angular vivem em:

- `vyracare-infra-pipes-angular`

Workflows principais:

- `ci-angular.yml`
- `cd-angular.yml`
- `rollback-generic.yml`

## Fluxo por repositorio Angular

Cada projeto Angular possui, em geral:

- `pr-to-develop.yml`
- `pr-to-release.yml`
- `pr-to-main.yml`
- `publish.yml`

## Fluxo detalhado

### PR to develop

Mantem o fluxo de qualidade antes da promocao para `develop`.

### Push em `develop`

Workflow:

- publica em `dev`
- cria ou reaproveita `release/<versionamento>`
- abre PR de `develop` para `release/<versionamento>`

No app shell, isso esta materializado em:

- `.github/workflows/pr-to-release.yml`

### Push em `release/*`

Workflow:

- detecta se a release esta a frente da `main`
- roda CI de novo
- publica em `hml`
- abre PR para `main`

No app shell, isso esta materializado em:

- `.github/workflows/pr-to-main.yml`

### Push em `main`

Workflow:

- publica em `prod`

## CI Angular

Arquivo base:

- `vyracare-infra-pipes-angular/.github/workflows/ci-angular.yml`

Comportamento atual:

- `build` em um job
- `test` em job separado que depende do `build`
- upload/download de artifact do dist
- suporte a CodeArtifact para o design system
- retry na instalacao do npm

## Publicacao do Design System

O workflow do `vyracare-design-system` consulta no CodeArtifact a versao publicada mais recentemente antes de calcular o proximo incremento semantico. Essa versao remota, e nao somente a versao registrada no checkout, deve ser usada como base para evitar a tentativa de republicar uma versao imutavel quando o pacote tiver sido publicado, mas o commit automatico de versionamento nao tiver chegado ao repositorio.

Depois da publicacao, os consumidores devem atualizar o `package-lock.json` para a nova versao do `@vyracare/design-system`; manter apenas um intervalo compativel no `package.json` nao atualiza instalacoes feitas com `npm ci`.

## CD Angular

Arquivo base:

- `vyracare-infra-pipes-angular/.github/workflows/cd-angular.yml`

Comportamento atual:

- resolve contexto por branch
- detecta bucket e CloudFront
- provisiona infra quando necessario
- builda o projeto
- publica em pasta de ambiente com timestamp
- faz cleanup de versoes antigas
- troca `OriginPath` no CloudFront
- invalida cache
- pode atualizar `remoteEntry` no shell orquestrador

## Mapeamento branch -> build configuration

### `develop`
- `BUILD_CONFIGURATION=dev`
- ambiente funcional: `dev`

### `release/*`
- `BUILD_CONFIGURATION=hml`
- ambiente funcional: `hml`

### `main`
- `BUILD_CONFIGURATION=production`
- ambiente funcional: `prod`

## Ponto sensivel

As automacoes que atualizam `environment` e `remoteEntry` entre repositorios precisam manter:

- arquivo correto por ambiente
- sintaxe TS valida
- apenas uma propriedade por chave

Esse ponto ja recebeu varias correcoes e precisa continuar sendo tratado como area de risco nas futuras evolucoes.
