# Checklists de Validacao

Este documento consolida as validacoes minimas para afirmar que a reconstrucao do ecossistema chegou a um estado utilizavel.

## Checklist de fundacao

- `.github` publicado com README de organizacao
- `vyracare-spec-driven` publicado
- reusable workflows publicados
- templates publicados

## Checklist de design system

- pacote publicado no CodeArtifact
- major `2.0.0` publicada, compativel com Angular `22.2.6` e peers Angular `^22.0.0`
- playground e testes executam em modo zoneless
- 43 suites e 153 testes aprovados na migracao Angular 22
- build da biblioteca, playground e Storybook aprovados
- auditoria das dependencias de producao sem vulnerabilidades conhecidas
- playground acessivel
- Storybook acessivel
- shell e MFEs conseguem instalar `@vyracare/design-system`
- componentes macro reutilizaveis existem no Design System, com testes, acessibilidade e responsividade
- shell e MFEs nao mantem copias locais de macros compartilhados

## Checklist da onda Angular 22 - shell e profile

- [x] migrations oficiais executadas em sequencia Angular 20 -> 21 -> 22
- [x] shell e profile usam Design System `2.0.0`
- [x] Module Federation atualizado para `22.0.0`
- [x] Zone.js removido do runtime e dos testes
- [x] instalacao limpa pelo lockfile aprovada
- [x] builds de desenvolvimento e producao aprovados
- [x] 143 testes aprovados entre shell e profile
- [x] `remoteEntry.js` responde e expoe `./Routes` e `./App`
- [x] auditoria de dependencias de producao sem vulnerabilidades conhecidas
- [x] pipeline do shell usa reusable workflow e instalacao deterministica
- [ ] migrar Webpack Module Federation para Application Builder/Native Federation apos prova em MFE piloto
- [ ] restaurar extracao agregada de licencas na migracao do builder
- [ ] corrigir fallback local de URLs profundas na migracao do dev-server
- [ ] reduzir o SCSS do cadastro de funcionario abaixo do warning budget

## Checklist de frontend

- shell acessivel em `dev`, `hml` e `prod`
- cada MFE publica `remoteEntry.js`
- shell resolve os remotos corretos por ambiente
- `environment.ts` e apenas local
- `environment.dev.ts` representa `develop`
- `environment.hml.ts` representa `release/*`
- `environment.prod.ts` representa `main`
- viewport de `320px`, `375px` e `430px` sem rolagem horizontal
- navbar compacto, busca global e menus flutuantes contidos na viewport
- barra inferior com `Inicio`, `Agenda`, `Pacientes` e `Menu`, com estado ativo e areas seguras
- `Menu` abre o sidebar como bottom sheet e permite fechamento por botao, backdrop e tecla Escape
- tabelas operacionais convertidas em cartoes rotulados no celular
- formularios em uma coluna e alvos de toque com no minimo `44px`
- modais mobile limitados a viewport e com rolagem interna

## Checklist de backend

- Lambdas criadas nos tres ambientes
- API Gateways criados nos tres ambientes
- rotas principais respondem `200`, `201`, `401` ou `409` conforme o caso de uso
- Swagger UI acessivel nas APIs publicadas
- `PayloadFormatVersion` das integrations em `2.0`

## Checklist de dados e parametros

- `vyracare_db_dev` existente
- `vyracare_db_hml` existente
- `vyracare_db` existente
- parametros `/vyracare/shared/mongo-*` existentes
- parametros `/vyracare/shared/jwt-signing-*` existentes
- JSON dos parametros valido e sem BOM

## Checklist de pipelines

### Toolchains

- projetos Node possuem `.nvmrc` em `24.15.0` e `engines.node` em `>=24.15.0 <25`
- workflows Angular e mobile instalam Node `24.15.0`
- APIs e template .NET possuem `global.json` em `10.0.404`
- workflows .NET instalam SDK `10.0.404`
- instalacoes limpas usam os lockfiles versionados sem divergencia

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
- `.terraform.lock.hcl` versionado nos modulos Terraform executaveis
- nenhum diretorio `bin`, `obj`, `.terraform` ou `publish*` versionado
- especificacao atualizada para refletir o estado entregue e suas decisoes
- testes e builds proporcionais ao risco aprovados
- commits criados e enviados em todos os repositorios alterados
- impedimentos de push registrados explicitamente quando um remoto ou permissao nao existir
