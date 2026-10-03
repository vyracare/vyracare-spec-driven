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

As APIs devem aplicar autorizacao no backend para operacoes privilegiadas. A ocultacao ou desabilitacao de controles no frontend nao substitui essa validacao. Na gestao de pacientes:

- leitura e notas exigem usuario autenticado;
- atualizacao integral da ficha exige role `Administrador`;
- tokens emitidos antes da inclusao das claims precisam ser renovados por novo login.

## Logging

Diagnostico principal de backend fica em:

- `/aws/lambda/<function-name>`

## Acesso entre repositorios

Automacoes que criam branch, PR ou sincronizam arquivo em outro repositorio usam:

- `PAT_TOKEN`

Evitar depender do token default do `github-actions[bot]` quando houver `push` cross-repo ou branch creation.
