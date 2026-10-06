# Runtime, IAM e Integracoes

## AWS principal

O ecossistema atual usa:

- S3
- CloudFront
- Lambda
- API Gateway HTTP
- Systems Manager Parameter Store
- Cognito
- IAM

## Lambdas

Cada API roda em Lambda `dotnet10`.

As Lambdas recebem via environment variables:

- `Mongo__Database`
- `MONGO_PARAMETER_NAME`
- `JWT_PARAMETER_NAME`
- outros valores de runtime conforme a API

## IAM

As roles de Lambda precisam de:

- `AWSLambdaBasicExecutionRole`
- leitura dos parametros usados pelo ambiente

Usuarios locais usados para operacao manual precisam de permissao explicita para:

- `ssm:GetParameter`
- `ssm:GetParameters`
- `ssm:PutParameter` quando houver manutencao manual
- `ssm:DeleteParameter` quando houver limpeza
- `ssm:DescribeParameters` quando a operacao exigir descoberta por nome

## API Gateway

As APIs atuais usam HTTP API com Lambda integration.

Ponto critico:

- `PayloadFormatVersion` deve ser `2.0` para o runtime atual
- as rotas de Swagger precisam estar provisionadas no gateway para cada ambiente

Rotas atualmente adotadas para Swagger:

- `ANY /swagger`
- `ANY /swagger/{proxy+}`

## Cognito

`vyracare-api-authentication` tambem provisiona recursos de Cognito no fluxo de auth.

## Autorizacao por nivel de acesso

O JWT emitido pela autenticacao carrega o `AccessLevel` como role e como claim `access_level`. O cargo funcional permanece separado em `job_role`.

O nome do nivel de acesso exibido pelo shell deve ser lido dessas claims; ele nao pode ser um texto estatico nem servir como fonte de autorizacao. Quando a claim estiver ausente, o shell informa que o perfil nao foi definido e as operacoes administrativas permanecem bloqueadas. Depois de atribuir ou alterar `AccessLevel` no cadastro do usuario, e necessario realizar um novo login para emitir outro JWT.

As APIs devem aplicar autorizacao no backend para operacoes privilegiadas. A ocultacao ou desabilitacao de controles no frontend nao substitui essa validacao. Na gestao de pacientes:

- leitura e notas exigem usuario autenticado;
- atualizacao integral da ficha exige role `Administrador`;
- tokens emitidos antes da inclusao das claims precisam ser renovados por novo login.

Na gestao de funcionarios, listagem administrativa, consulta individual, edicao e alteracao de status exigem role `Administrador`. A API nunca devolve senha ou hash. A inativacao bloqueia novos logins e remove o funcionario das consultas operacionais; a autoinativacao e rejeitada. Tokens ja emitidos continuam sujeitos ao tempo de expiracao configurado, pois nao existe revogacao central de sessao nesta etapa.

O registro publico nao pode definir role funcional, departamento, telefone, status ou nivel administrativo. Esses valores sao neutralizados pela API e o acesso inicial fica restrito a `Leitura`. Somente `POST /api/auth/employees`, protegido por `Administrador`, cria perfis completos de funcionarios.

## Auditoria de dependencias

Builds e entregas devem executar as auditorias nativas dos gerenciadores de pacote, incluindo dependencias transitivas. Alertas conhecidos nao podem ser ocultados nem tratados como falha funcional da alteracao em revisao; eles devem ser registrados e corrigidos por atualizacao compativel e validada.

Na revisao de 2026-10-06, `vyracare-api-authentication` ainda recebeu alertas transitivos do NuGet para `SharpCompress 0.30.1` (moderado) e `Snappier 1.0.0` (alto), trazidos pela versao atual do driver MongoDB. A gestao de funcionarios nao introduziu esses pacotes e nao utiliza diretamente suas APIs, mas a atualizacao do driver e a regressao dos fluxos de persistencia permanecem divida tecnica prioritaria. A existencia desses alertas deve continuar visivel nos pipelines ate a remediacao, sem uso de supressao de auditoria como solucao.

Na mesma revisao, `npm audit --omit=dev` do `vyracare-app-profile-mfe` apontou vulnerabilidades na linha Angular 20.3 instalada e em dependencias transitivas do Module Federation. A correcao automatica sem alteracao principal foi bloqueada pelos pinos divergentes de peer dependencies; a alternativa sugerida com `--force` introduziria mudanca principal do Module Federation. A remediacao deve ser coordenada entre shell e MFEs, mantendo todas as bibliotecas Angular na mesma versao de patch e validando o compartilhamento singleton antes da promocao. Nao se deve aplicar `npm audit fix --force` isoladamente em um unico MFE.

## Logging

Diagnostico principal de backend fica em:

- `/aws/lambda/<function-name>`

## Acesso entre repositorios

Automacoes que criam branch, PR ou sincronizam arquivo em outro repositorio usam:

- `PAT_TOKEN`

Evitar depender do token default do `github-actions[bot]` quando houver `push` cross-repo ou branch creation.
