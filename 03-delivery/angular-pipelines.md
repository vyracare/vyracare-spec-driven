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

- Node.js `24.15.0`, alinhado ao `.nvmrc` e a `engines.node` dos projetos;
- `build` em um job
- `test` em job separado que depende do `build`
- upload/download de artifact do dist
- suporte a CodeArtifact para o design system
- retry na instalacao do npm

Quando existe `package-lock.json`, CI e CD instalam de forma deterministica com `npm ci`. Consumidores do CodeArtifact usam `npm ci --legacy-peer-deps` para respeitar o lockfile e a matriz de peers ja validada; o fallback `npm install --legacy-peer-deps --package-lock=false` fica restrito a repositorios legados que ainda nao possuem lockfile. Uma esteira nao deve ignorar um lockfile apenas porque consome o Design System privado.

O registry padrao dos projetos deve ser `https://registry.npmjs.org/`, e somente o escopo `@vyracare` deve apontar para o CodeArtifact. Isso impede que configuracoes globais do AWS CLI tentem resolver pacotes publicos no repositorio privado.

O baseline deve permanecer na linha Node `24` LTS enquanto Angular 22 for a versao-alvo. Alteracoes de Node precisam atualizar em conjunto os projetos, templates e reusable workflows.

## Publicacao do Design System

O workflow do `vyracare-design-system` consulta no CodeArtifact a versao publicada mais recentemente antes de calcular o proximo incremento semantico. Essa versao remota, e nao somente a versao registrada no checkout, deve ser usada como base para evitar a tentativa de republicar uma versao imutavel quando o pacote tiver sido publicado, mas o commit automatico de versionamento nao tiver chegado ao repositorio.

Depois da publicacao, os consumidores devem atualizar o `package-lock.json` para a nova versao do `@vyracare/design-system`; manter apenas um intervalo compativel no `package.json` nao atualiza instalacoes feitas com `npm ci`.

### Compatibilidade Angular 22

A major `2.x` do Design System usa Angular `22.2.6`, TypeScript `6.0.3` e peer dependencies Angular `^22.0.0`. O primeiro commit da onda deve usar prefixo `chore`, que a automacao de versionamento interpreta como incremento major, publicando `2.0.0` a partir da linha `1.x`.

Enquanto shell e MFEs ainda estiverem em Angular 20, seus lockfiles devem permanecer na major `1.x`. A major `2.x` somente deve ser adotada por um consumidor junto com sua migracao para Angular 22 e com validacao integrada de Module Federation.

O shell e o profile foram os primeiros consumidores promovidos para Angular 22 e Design System `2.0.0`. Os demais MFEs continuam na major compativel com seu Angular ate a respectiva onda; misturar Angular 20 e a major `2.x` do pacote nao e suportado.

O pipeline do Design System deve executar com Node `24.15.0`, instalar pelo lockfile, aprovar testes, build da biblioteca, playground e Storybook antes da publicacao. O commit automatico de versao gerado pela esteira deve ser sincronizado localmente antes da proxima mudanca.

No Angular 22, o pacote gerado expõe os tipos em `types/vyracare-design-system.d.ts`; a validacao anterior de `index.d.ts`, usada pelo empacotamento Angular 20, nao deve ser mantida. O gate de publicacao valida manifest, bundle FESM, declaracao de tipos no caminho exportado e entrada SCSS publica.

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
- pode publicar o artefato do shell e notificar o orquestrador mobile quando `publish-mobile-update` estiver habilitado

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

## Integracao mobile

O hook mobile e opcional e fica desligado por padrao. Sua ativacao exige que bucket, CDN, OIDC, chave de assinatura e environments protegidos estejam validados conforme [Esteiras Mobile](./mobile-pipelines.md). A falha de infraestrutura mobile nao deve impedir o deploy web antes dessa ativacao controlada.
