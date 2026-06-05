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

Cada API roda em Lambda `dotnet8`.

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

- `GetSecretValue`
- `PutSecretValue`
- `DescribeSecret`
- `DeleteSecret` quando houver limpeza
- `ListSecrets` com `Resource: "*"` se a listagem for necessaria

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

## Logging

Diagnostico principal de backend fica em:

- `/aws/lambda/<function-name>`

## Acesso entre repositorios

Automacoes que criam branch, PR ou sincronizam arquivo em outro repositorio usam:

- `PAT_TOKEN`

Evitar depender do token default do `github-actions[bot]` quando houver `push` cross-repo ou branch creation.
