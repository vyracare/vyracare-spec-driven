# Panorama de Repositorios

## Frontend

### `vyracare-design-system`
Biblioteca compartilhada de UI e estilo para os projetos Angular.

Responsabilidades:

- componentes reutilizaveis
- estilos compartilhados
- contrato visual entre shell e MFEs
- publicacao para consumo via pacote

### `vyracare-app-shell`
Host principal da plataforma.

Responsabilidades:

- layout base
- navegacao
- registro de `remoteEntry`
- integracao com API de autenticacao

### `vyracare-app-user-mfe`
MFE orientado ao dominio de usuario.

### `vyracare-app-profile-mfe`
MFE orientado ao dominio de perfil.

### `vyracare-app-dashboard-mfe`
MFE orientado ao dashboard.

### `vyracare-app-proceedings-mfe`
MFE orientado ao dominio de procedimentos.

## Backend

### `vyracare-api-authentication`
API de autenticacao, primeiro acesso e recuperacao de senha.

### `vyracare-api-client`
API de clientes.

### `vyracare-api-proceedings`
API de procedimentos.

## Templates

### `templates-angular`
Template base para novos projetos Angular.

### `template-dot-net-api`
Template base para novas APIs .NET.

## Infra reutilizavel

### `vyracare-infra-pipes-angular`
Pipelines reutilizaveis de CI/CD e rollback para Angular.

### `vyracare-infra-pipes-dot-net`
Pipelines reutilizaveis de CI/CD para APIs .NET.

## Relacao entre grupos

- o shell consome MFEs por `remoteEntry.js`
- os MFEs consomem APIs publicadas em API Gateway
- as APIs rodam em Lambda e persistem em MongoDB
- os templates geram novos repositorios com o mesmo padrao
- as pipes encapsulam build, deploy, promocao e sincronizacao entre repositorios

## Ordem recomendada de criacao do ecossistema

Para reconstruir a plataforma do zero, a ordem mais segura e:

1. `.github`
2. `vyracare-spec-driven`
3. `vyracare-infra-pipes-angular`
4. `vyracare-infra-pipes-dot-net`
5. `templates-angular`
6. `template-dot-net-api`
7. `vyracare-design-system`
8. `vyracare-app-shell`
9. MFEs Angular
10. APIs .NET

Motivo:

- templates dependem das reusable workflows
- projetos de produto dependem de templates, environment strategy e naming ja estabilizados
- shell e APIs viram ponto de integracao para os demais repositorios
