# Checklists de Validacao

Este documento consolida as validacoes minimas para afirmar que a reconstrucao do ecossistema chegou a um estado utilizavel.

## Checklist de fundacao

- `.github` publicado com README de organizacao
- `vyracare-spec-driven` publicado
- reusable workflows publicados
- templates publicados

## Checklist de design system

- pacote publicado no CodeArtifact
- playground acessivel
- Storybook acessivel
- shell e MFEs conseguem instalar `@vyracare/design-system`

## Checklist de frontend

- shell acessivel em `dev`, `hml` e `prod`
- cada MFE publica `remoteEntry.js`
- shell resolve os remotos corretos por ambiente
- `environment.ts` e apenas local
- `environment.dev.ts` representa `develop`
- `environment.hml.ts` representa `release/*`
- `environment.prod.ts` representa `main`

## Checklist de backend

- Lambdas criadas nos tres ambientes
- API Gateways criados nos tres ambientes
- rotas principais respondem `200`, `201`, `401` ou `409` conforme o caso de uso
- Swagger UI acessivel nas APIs publicadas
- `PayloadFormatVersion` das integrations em `2.0`

## Checklist de dados e secrets

- `vyracare_db_dev` existente
- `vyracare_db_hml` existente
- `vyracare_db` existente
- secrets `mongo-*` existentes
- secrets `jwt-signing-*` existentes
- JSON dos secrets valido e sem BOM

## Checklist de pipelines

### Angular

- PR para `develop` roda CI
- `develop` publica `dev` e abre PR para `release/<versionamento>`
- `release/*` roda CI, publica `hml` e abre PR para `main`
- `main` publica `prod`

### .NET

- PR para `develop` roda CI
- job de teste falha antes do build quando necessario
- `develop` publica `dev` e abre PR para `release/<versionamento>`
- `release/*` roda CI, publica `hml` e abre PR para `main`
- `main` publica `prod`

## Checklist funcional minimo

- cadastro na autenticacao funciona
- login funciona
- shell consome a auth correta por ambiente
- shell carrega os MFEs corretos por ambiente
- APIs de client e proceedings respondem por gateway

## Checklist de governanca

- commits em portugues
- naming dos repositorios consistente
- naming dos recursos AWS consistente
- nenhuma credencial sensivel versionada
- READMEs dos repositorios centrais atualizados
