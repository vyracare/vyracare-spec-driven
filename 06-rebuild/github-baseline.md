# Baseline de GitHub

Este documento descreve o que precisa existir no GitHub para a organizacao funcionar como o Vyracare atual.

## Repositorios minimos

### Organizacao e especificacao

- `.github`
- `vyracare-spec-driven`

### Reusable workflows

- `vyracare-infra-pipes-angular`
- `vyracare-infra-pipes-dot-net`

### Templates

- `templates-angular`
- `template-dot-net-api`

### Frontend

- `vyracare-design-system`
- `vyracare-app-shell`
- `vyracare-app-user-mfe`
- `vyracare-app-profile-mfe`
- `vyracare-app-dashboard-mfe`
- `vyracare-app-proceedings-mfe`

### Backend

- `vyracare-api-authentication`
- `vyracare-api-client`
- `vyracare-api-proceedings`

## Branches esperadas

Todo repositorio de produto deve considerar:

- `main`
- `develop`
- `release/*` como branch efemera e versionada

Templates e reusable workflows devem nascer alinhados com esse fluxo.

## Secrets e variables do GitHub

Cada repositorio que publica na AWS precisa de configuracoes e secrets de GitHub para:

- credenciais AWS
- `PAT_TOKEN` para branch creation, PR opening e update cross-repo

Sem `PAT_TOKEN`, o `github-actions[bot]` nao resolve sozinho cenarios como:

- criar `release/<versionamento>`
- abrir PR automaticamente
- atualizar arquivo em outro repositorio

## Protecoes recomendadas

### `main`

- exigir PR
- impedir push direto para nao administradores
- exigir checks obrigatorios

### `develop`

- exigir PR quando o time quiser o mesmo gate do fluxo principal
- permitir automacoes controladas se houver justificativa

### `release/*`

- tratar como branch de promocao
- manter CI obrigatoria antes do merge em `main`

## Workflows minimos por tipo de repositorio

### Angular

- `pr-to-develop.yml`
- `pr-to-release.yml`
- `pr-to-main.yml`
- `publish.yml`

### .NET

- `pr-to-develop.yml`
- `pr-to-release.yml`
- `pr-to-main.yml`
- `publish.yml`

## Convencoes obrigatorias

- commits em portugues
- naming consistente de repositorios
- uso de reusable workflows, nao copia divergente
- `environment.ts` local
- `environment.dev.ts`, `environment.hml.ts`, `environment.prod.ts` quando Angular
