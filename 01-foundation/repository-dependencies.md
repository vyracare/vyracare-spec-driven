# Matriz de Dependencias entre Repositorios

Este documento descreve quem depende de quem no ecossistema Vyracare.

O objetivo e evitar reconstrucoes ou evolucoes fora de ordem.

## Leitura rapida

### Dependencias de fundacao

| Repositorio | Depende de | Motivo |
| --- | --- | --- |
| `.github` | nenhum | perfil da organizacao e documentacao publica |
| `vyracare-spec-driven` | nenhum | base de especificacao e operacao |
| `vyracare-infra-pipes-angular` | nenhum | reusable workflows Angular |
| `vyracare-infra-pipes-dot-net` | nenhum | reusable workflows .NET |
| `vyracare-infra-pipes-mobile` | nenhum | reusable workflows Android, iOS e live update |

### Dependencias de template

| Repositorio | Depende de | Motivo |
| --- | --- | --- |
| `templates-angular` | `vyracare-infra-pipes-angular` | gera projetos Angular ja ligados as esteiras |
| `template-dot-net-api` | `vyracare-infra-pipes-dot-net` | gera APIs .NET ja ligadas as esteiras |

### Dependencias compartilhadas de frontend

| Repositorio | Depende de | Motivo |
| --- | --- | --- |
| `vyracare-design-system` | CodeArtifact e AWS de publicacao | pacote compartilhado e Storybook |
| `vyracare-app-shell` | `vyracare-design-system`, `templates-angular`, `vyracare-infra-pipes-angular` | shell consome design system e esteiras |
| `vyracare-app-user-mfe` | `vyracare-design-system`, `templates-angular`, `vyracare-infra-pipes-angular`, `vyracare-app-shell` | MFE publicado e carregado pelo shell |
| `vyracare-app-profile-mfe` | `vyracare-design-system`, `templates-angular`, `vyracare-infra-pipes-angular`, `vyracare-app-shell` | MFE publicado e carregado pelo shell |
| `vyracare-app-dashboard-mfe` | `vyracare-design-system`, `templates-angular`, `vyracare-infra-pipes-angular`, `vyracare-app-shell` | MFE publicado e carregado pelo shell |
| `vyracare-app-proceedings-mfe` | `vyracare-design-system`, `templates-angular`, `vyracare-infra-pipes-angular`, `vyracare-app-shell` | MFE publicado e carregado pelo shell |
| `vyracare-app-mobile` | `vyracare-app-shell`, MFEs Angular, `vyracare-infra-pipes-angular`, `vyracare-infra-pipes-mobile`, Capacitor | compoe o bundle web validado e gera Android/iOS |

### Dependencias de backend

| Repositorio | Depende de | Motivo |
| --- | --- | --- |
| `vyracare-api-authentication` | `template-dot-net-api`, `vyracare-infra-pipes-dot-net`, AWS, MongoDB Atlas | auth publica Lambda, Gateway e Cognito |
| `vyracare-api-tenancy` | `template-dot-net-api`, `vyracare-infra-pipes-dot-net`, AWS, MongoDB Atlas | plano de controle de tenants, memberships, convites e trial |
| `vyracare-api-client` | `template-dot-net-api`, `vyracare-infra-pipes-dot-net`, AWS, MongoDB Atlas, API Busca CEP dos Correios | API de clientes e fachada de consulta de CEP |
| `vyracare-api-proceedings` | `template-dot-net-api`, `vyracare-infra-pipes-dot-net`, AWS, MongoDB Atlas | API de procedimentos |
| `vyracare-api-appointments` | `template-dot-net-api`, `vyracare-infra-pipes-dot-net`, AWS, MongoDB Atlas | API de agenda e indicadores operacionais |
| `vyracare-api-finance` | `template-dot-net-api`, `vyracare-infra-pipes-dot-net`, AWS, MongoDB Atlas | API financeira e indicadores mensais |

## Dependencias cruzadas entre produto

### Shell e MFEs

- o shell depende dos `remoteEntry.js` publicados pelos MFEs
- os MFEs nao dependem do shell em runtime local, mas dependem do shell no fluxo de integracao publicada
- quando um MFE publica, a esteira pode atualizar o shell consumidor

### Shell e APIs

- o shell depende da API de autenticacao
- o cadastro publico do shell envia os dados minimos da empresa para o onboarding coordenado pela autenticacao
- `environment.dev.ts`, `environment.hml.ts` e `environment.prod.ts` apontam para a auth por ambiente

### MFEs e APIs

- `vyracare-app-user-mfe` depende da API client para pacientes e das claims emitidas pela auth para controlar o modo administrador
- `vyracare-app-profile-mfe` pode depender de auth e de backend proprio conforme evolucao
- `vyracare-app-proceedings-mfe` depende da API de proceedings quando o fluxo remoto esta ativo
- `vyracare-app-dashboard-mfe` depende das APIs de appointments e finance para compor seus indicadores
- `vyracare-app-dashboard-mfe` depende tambem de authentication e proceedings para pesquisar e selecionar funcionarios e procedimentos no cadastro de atendimento
- shell e MFEs Angular exigem `@vyracare/design-system` `^0.10.0`, compartilhado como singleton, para navegacao, feedback, formularios, autocomplete, modais, icones e controles responsivos
- o app mobile nao duplica os MFEs; recebe o artefato testado do shell e injeta apenas o runtime de atualizacao Capacitor

### APIs e consumers

- a auth pode atualizar o shell e o `vyracare-app-user-mfe`
- a auth depende da API de tenancy para provisionar a empresa proprietaria e inclui a projecao da membership no JWT
- a API client recusa tokens sem contexto de tenant e filtra pacientes e funcionarios pelo `tenant_id` validado
- APIs genericas podem atualizar MFEs consumidores configurados em `.vyracare/mfe-consumer.json`

## Ordem segura para subir do zero

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

## Regra operacional

Se um repositorio depende de outro para:

- pipeline
- pacote
- URL publicada
- sincronizacao de environment

entao a dependencia precisa estar estavel antes da publicacao do dependente.
