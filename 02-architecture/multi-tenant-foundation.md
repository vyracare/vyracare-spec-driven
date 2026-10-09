# Fundacao multi-tenant

## Status

Esta especificacao define o primeiro incremento executavel da evolucao SaaS. O objetivo e permitir que uma pessoa crie uma conta proprietaria, cadastre sua empresa e receba um contexto de tenant verificavel em todas as APIs, sem permitir associacao automatica por dominio de e-mail ou CNPJ.

## Decisoes vigentes

- uma identidade pode participar de mais de uma empresa;
- toda empresa nasce com uma membership `Owner` ativa;
- CNPJ e opcional para permitir profissionais autonomos;
- associacao a empresa existente exige convite explicito;
- o periodo de teste e de **30 dias corridos**, calculado no backend;
- o primeiro incremento usa uma base compartilhada para o plano de controle e `tenant_id` obrigatorio no plano de dados;
- separar database por tenant permanece como evolucao do modelo bridge, depois da validacao do piloto;
- `tenant_id` enviado pelo navegador nunca e fonte de autoridade.

## Modelo de dominio

### Tenant

- `id`: identificador imutavel;
- `legalName`: razao social ou nome do profissional;
- `tradeName`: nome exibido na plataforma;
- `document`: CNPJ opcional, normalizado somente com digitos;
- `status`: `Trialing`, `Active`, `PastDue`, `Suspended` ou `Cancelled`;
- `trialStartsAtUtc` e `trialEndsAtUtc`;
- `createdAtUtc` e `updatedAtUtc`.

### Membership

- `id`, `tenantId` e `userId`;
- `role`: inicialmente `Owner`, `Administrator` ou `Member`;
- `status`: `Active`, `Invited`, `Suspended` ou `Revoked`;
- `createdAtUtc` e `updatedAtUtc`.

O par `tenantId + userId` deve ser unico. Uma membership nunca e inferida pelo e-mail da pessoa.

### Invitation

- `id`, `tenantId`, e-mail normalizado, papel e status;
- token armazenado como hash, nunca em texto puro;
- expiracao, criador, data de aceite e membership resultante;
- estados `Pending`, `Accepted`, `Expired` e `Revoked`.

## Fluxo de primeiro acesso do proprietario

1. O shell coleta dados pessoais, credencial e dados minimos da empresa.
2. A API de autenticacao cria a identidade em estado valido.
3. A autenticacao chama o plano de controle por uma porta interna autenticada.
4. O plano de controle cria `Tenant` e `Membership Owner` de forma idempotente.
5. O tenant recebe `Trialing`, com fim exatamente 30 dias apos o inicio em UTC.
6. A autenticacao grava a projecao da membership padrao na identidade e emite JWT com contexto de tenant.
7. Se o provisionamento falhar, a criacao da identidade e compensada e o cadastro retorna falha; nao deve sobrar uma conta parcialmente ativa.

O endpoint interno recebe uma `idempotencyKey`. Repetir a mesma requisicao deve devolver o mesmo tenant e a mesma membership.

## Contrato do JWT

Claims obrigatorias para acessar o plano de dados:

- `sub`: usuario autenticado;
- `tenant_id`: tenant selecionado;
- `membership_id`: vinculo selecionado;
- `tenant_role`: papel no tenant;
- `access_level`: compatibilidade temporaria com a autorizacao atual;
- `plan`: inicialmente `trial`;
- `iss`, `aud`, `iat` e `exp`.

Quando o usuario possuir varias memberships, a autenticacao deve exigir selecao explicita do tenant ou usar uma preferencia previamente confirmada. Nunca deve combinar dados de memberships diferentes no mesmo token.

## TenantContext nas APIs

Depois da validacao criptografica do JWT, um middleware resolve um `TenantContext` imutavel. Requisicoes autenticadas sem `tenant_id`, `membership_id` ou `tenant_role` recebem `403`.

Repositorios do plano de dados:

- recebem `TenantContext` por injecao de dependencia;
- incluem `tenantId` em todos os documentos;
- iniciam cada filtro por `tenantId`;
- usam indices unicos compostos, por exemplo `tenantId + cpf`;
- nao oferecem metodos globais por padrao;
- incluem testes com dois tenants tentando ler e alterar o mesmo tipo de registro.

## APIs iniciais

### Plano de controle

- `POST /api/tenancy/internal/tenants`: provisionamento interno do tenant proprietario;
- `DELETE /api/tenancy/internal/tenants/{tenantId}`: compensacao interna restrita ao onboarding do proprietario;
- `GET /api/tenancy/internal/users/{userId}/memberships`: memberships ativas para emissao de token;
- `POST /api/tenancy/tenants/{tenantId}/invitations`: cria convite;
- `POST /api/tenancy/invitations/accept`: aceita convite autenticado.

### Autenticacao

O cadastro publico passa a aceitar um objeto `organization` com `legalName`, `tradeName` e `document` opcional. A resposta de sucesso inclui o token, tenant, membership e periodo de teste. O login emite o mesmo conjunto de claims usando a membership ativa selecionada.

## Seguranca entre servicos

No desenvolvimento local, a chamada interna usa chave configurada fora do codigo. Em AWS, a meta e substituir a chave por autorizacao IAM/SigV4 ou identidade de workload. A chave interna nunca deve ser exposta ao shell, aos MFEs ou aos logs.

## Criterios de aceite do piloto

- cadastro cria identidade, tenant e owner sem estado parcial;
- trial termina 30 dias apos a criacao;
- JWT contem todas as claims de tenant;
- API de clientes rejeita token sem tenant;
- paciente criado no tenant A nao pode ser listado, lido, alterado ou receber nota usando token do tenant B;
- CPF pode se repetir em tenants diferentes, mas nao dentro do mesmo tenant;
- logs tecnicos incluem `tenant_id` sem incluir dados clinicos;
- builds e testes unitarios permanecem verdes.

## Proximos incrementos

1. concluir convite e troca de tenant no shell;
2. migrar funcionarios da autenticacao para membership e perfil organizacional;
3. aplicar `TenantContext` a procedimentos, agenda e financeiro;
4. automatizar provisionamento de database dedicado por tenant/plano;
5. adicionar assinatura, entitlements, cobranca e suspensao controlada;
6. executar testes de penetracao e restauracao por tenant antes do piloto comercial.
