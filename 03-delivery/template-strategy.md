# Templates e Scaffolding

## Angular

Template:

- `templates-angular`

O template Angular deve gerar:

- estrutura de projeto Angular
- `environments.ts`
- `environments.dev.ts`
- `environments.hml.ts`
- `environments.prod.ts`
- `angular.json` com configuracoes `dev` e `hml`
- workflows:
  - `pr-to-develop.yml`
  - `pr-to-release.yml`
  - `pr-to-main.yml`
  - `publish.yml`

## .NET

Template:

- `template-dot-net-api`

O template .NET deve gerar:

- API .NET baseada no padrao atual
- projeto de testes
- `GlobalUsings.cs` no projeto de testes com `global using Xunit;`
- workflows:
  - `pr-to-develop.yml`
  - `pr-to-release.yml`
  - `pr-to-main.yml`
  - `publish.yml`

## Papel dos scripts de rename

Os templates dependem de scripts para transformar placeholders em codigo real:

### Angular
- `rename-angular-project.sh`

### .NET
- `rename-dotnet-project.sh`

Esses scripts precisam manter coerencia com:

- nomes de repositos
- nomes de workflows
- nome dos projetos
- caminhos dos `.csproj`
- referencias de testes

## Regras que o template deve preservar

- branch flow `develop -> release/* -> main`
- alinhamento com reusable workflows
- suporte a `dev`, `hml` e `prod`
- convencao de naming dos recursos AWS
- suporte a sincronizacao com consumers quando aplicavel

## O que nao deve acontecer

- workflow de template tentar publicar infra propria na AWS
- gerar projeto sem arquivo `environments.dev.ts`
- gerar projeto sem arquivo `environments.hml.ts`
- gerar .NET sem `test-project-path`
- gerar pipelines com credenciais padrao do `github-actions[bot]` em vez de `PAT_TOKEN`
