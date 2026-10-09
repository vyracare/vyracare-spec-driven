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

### `vyracare-app-mobile`
Container Capacitor unico para Android e iOS. Incorpora o shell Angular, aplica atualizacoes web assinadas e preserva um bundle local para fallback.

## Backend

### `vyracare-api-authentication`
API de autenticacao, primeiro acesso e recuperacao de senha.

### `vyracare-api-client`
API de clientes e primeira API do plano de dados com `TenantContext` obrigatorio.

### `vyracare-api-tenancy`
Plano de controle para empresas, memberships, convites e ciclo do trial de 30 dias.

### `vyracare-api-proceedings`
API de procedimentos.

### `vyracare-api-appointments`
API de agenda clinica e dos indicadores operacionais exibidos no dashboard.

### `vyracare-api-finance`
API de lancamentos, boletos e dos indicadores mensais de saude financeira exibidos no dashboard.

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

### `vyracare-infra-pipes-mobile`
Pipelines reutilizaveis de CI nativa e publicacao de bundles web assinados para o aplicativo.

## Relacao entre grupos

- o shell consome MFEs por `remoteEntry.js`
- os MFEs consomem APIs publicadas em API Gateway
- as APIs rodam em Lambda e persistem em MongoDB
- os templates geram novos repositorios com o mesmo padrao
- as pipes encapsulam build, deploy, promocao e sincronizacao entre repositorios
- o app mobile reutiliza o build do shell e dos MFEs; alteracoes nativas continuam sendo publicadas nas lojas

## Ordem recomendada de criacao do ecossistema

Para reconstruir a plataforma do zero, a ordem mais segura e:

1. `.github`
2. `vyracare-spec-driven`
3. `vyracare-infra-pipes-angular`
4. `vyracare-infra-pipes-dot-net`
5. `vyracare-infra-pipes-mobile`
6. `templates-angular`
7. `template-dot-net-api`
8. `vyracare-design-system`
9. `vyracare-app-shell`
10. MFEs Angular
11. APIs .NET
12. `vyracare-app-mobile`

Motivo:

- templates dependem das reusable workflows
- projetos de produto dependem de templates, environment strategy e naming ja estabilizados
- shell e APIs viram ponto de integracao para os demais repositorios
