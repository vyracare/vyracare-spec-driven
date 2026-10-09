# Catalogo de Variaveis e Parametros por Repositorio

Este documento consolida os nomes de variaveis, parametros e referencias operacionais esperadas por repositorio.

Valores sensiveis nao sao exibidos aqui. Apenas nomes, contratos e finalidade.

## Regras gerais

- secrets do GitHub ficam no nivel do repositorio ou da organizacao
- parametros do AWS Systems Manager Parameter Store ficam nomeados por ambiente
- frontends nao devem versionar credenciais
- backends podem versionar apenas nomes de parametro e configuracao estrutural

## Repositorios Angular de produto

### `vyracare-app-shell`

GitHub:

- credenciais AWS para publicacao
- `PAT_TOKEN` para automacoes cross-repo quando necessario

Arquivos de ambiente:

- `src/environments/environments.ts`
- `src/environments/environments.dev.ts`
- `src/environments/environments.hml.ts`
- `src/environments/environments.prod.ts`

Configuracoes publicas esperadas:

- `apiUrl`
- `dashboardRemoteEntry`
- `userRemoteEntry`
- `profileRemoteEntry`
- `proceedingsRemoteEntry`

### `vyracare-app-user-mfe`

GitHub:

- credenciais AWS
- `PAT_TOKEN` quando a esteira precisar sincronizar com shell ou release flow

Arquivos de ambiente:

- `environments.ts`
- `environments.dev.ts`
- `environments.hml.ts`
- `environments.prod.ts`

Configuracoes publicas possiveis:

- `authApiUrl`
- `clientApiUrl`

### `vyracare-app-profile-mfe`

GitHub:

- credenciais AWS
- `PAT_TOKEN`

Arquivos de ambiente:

- `environments.ts`
- `environments.dev.ts`
- `environments.hml.ts`
- `environments.prod.ts`

Configuracoes publicas:

- URLs publicas de APIs consumidas pelo MFE

### `vyracare-app-dashboard-mfe`

GitHub:

- credenciais AWS
- `PAT_TOKEN`

Arquivos de ambiente:

- `environments.ts`
- `environments.dev.ts`
- `environments.hml.ts`
- `environments.prod.ts`

Configuracoes publicas esperadas:

- `appointmentsApiUrl`
- `financeApiUrl`

### `vyracare-app-proceedings-mfe`

GitHub:

- credenciais AWS
- `PAT_TOKEN`

Arquivos de ambiente:

- `environments.ts`
- `environments.dev.ts`
- `environments.hml.ts`
- `environments.prod.ts`

Configuracoes publicas possiveis:

- `apiUrl` da API de proceedings

### `vyracare-app-mobile`

Secrets GitHub:

- `OTA_SIGNING_PRIVATE_KEY`;
- `PAT_TOKEN`;
- `AWS_ROLE_ARN`.

Variables GitHub:

- `MOBILE_UPDATES_BUCKET`;
- `MOBILE_UPDATES_CDN_URL`;
- `MOBILE_UPDATES_CLOUDFRONT_ID`;
- `MOBILE_ANDROID_MIN_BUILD` e `MOBILE_ANDROID_MAX_BUILD`;
- `MOBILE_IOS_MIN_BUILD` e `MOBILE_IOS_MAX_BUILD`;
- `AWS_REGION`.

Configuracao publica versionada:

- chave publica RSA de OTA;
- identificador `br.com.vyracare.app`;
- schema do manifesto e canais.

A chave privada, certificados Apple e keystore Android nunca podem ser versionados.

## Design system

### `vyracare-design-system`

GitHub:

- credenciais AWS ou pipeline equivalente para:
  - pacote no CodeArtifact
  - playground publicado
  - Storybook publicado

Variaveis operacionais relevantes:

- dominio CodeArtifact
- repositorio CodeArtifact
- bucket do playground
- bucket do Storybook

## APIs .NET

### Regra comum das APIs

GitHub:

- credenciais AWS
- `PAT_TOKEN`

Runtime de Lambda:

- `Mongo__Database`
- `MONGO_PARAMETER_NAME`
- `JWT_PARAMETER_NAME`

Parameter Store por ambiente:

- `/vyracare/shared/mongo-dev`
- `/vyracare/shared/mongo-hml`
- `/vyracare/shared/mongo-prod`
- `/vyracare/shared/jwt-signing-dev`
- `/vyracare/shared/jwt-signing-hml`
- `/vyracare/shared/jwt-signing-prod`

Fallback ainda suportado:

- `MONGO_URI`
- `JWT_KEY`

### `vyracare-api-authentication`

Configuracao adicional:

- integra com Cognito
- pode sincronizar `apiUrl` no shell
- pode sincronizar `authApiUrl` no `vyracare-app-user-mfe`
- `TenancyApi__BaseUrl`
- `TenancyApi__InternalApiKey`
- `TENANCY_INTERNAL_API_KEY_PARAMETER_NAME`

Arquivo de metadata relevante:

- `.vyracare/mfe-consumer.json`

### `vyracare-api-client`

Configuracao adicional:

- pode sincronizar `clientApiUrl` em consumers configurados

Arquivo de metadata relevante:

- `.vyracare/mfe-consumer.json`

Seguranca multi-tenant:

- exige `tenant_id`, `membership_id` e `tenant_role` em tokens autenticados;
- indices unicos usam `tenantId + cpf` para pacientes e `tenantId + email` para funcionarios;
- registros legados sem tenant precisam de migracao explicita e nao entram automaticamente em nenhum tenant.

### `vyracare-api-tenancy`

Configuracao obrigatoria:

- `Mongo__ConnectionString` e `Mongo__Database`;
- `Jwt__Key`, `Jwt__Issuer` e `Jwt__Audience`;
- `Tenancy__InternalApiKey`;
- `Tenancy__TrialDays`, com padrao `30`;
- `/vyracare/tenancy/internal-api-key-<env>` no Parameter Store.

As rotas internas exigem `X-Internal-Api-Key`. O header e exclusivo para comunicacao backend-backend e nunca deve ser publicado em environment Angular.

### `vyracare-api-proceedings`

Configuracao adicional:

- pode sincronizar `apiUrl` da API de proceedings em consumers configurados

Arquivo de metadata relevante:

- `.vyracare/mfe-consumer.json`

### `vyracare-api-appointments`

Configuracao adicional:

- `Appointments__TimeZone`
- `Appointments__WeeklyAvailableMinutes`
- `Appointments__FollowUpWindowDays`
- sincroniza `appointmentsApiUrl` no dashboard por `.vyracare/mfe-consumer.json`

### `vyracare-api-finance`

Configuracao adicional:

- `Finance__TimeZone`
- sincroniza `financeApiUrl` no dashboard por `.vyracare/mfe-consumer.json`

## Templates

### `templates-angular`

Deve gerar projetos com:

- `environments.ts`
- `environments.dev.ts`
- `environments.hml.ts`
- `environments.prod.ts`

### `template-dot-net-api`

Deve gerar projetos com:

- nomes de parametros configuraveis por ambiente
- wiring com `PAT_TOKEN`
- suporte a `.vyracare/mfe-consumer.json` quando houver consumidor

## Organizacao

### `.github`

Repositorio especial da organizacao.

Conteudo principal esperado:

- `profile/README.md`

Nao depende de runtime sensivel, mas deve refletir:

- URLs publicadas
- arquitetura
- fluxo de promocao

## Regra operacional

Se um repositorio exigir uma variavel, parametro ou secret de GitHub fora deste catalogo, o novo contrato deve ser documentado aqui antes de virar dependencia invisivel.
