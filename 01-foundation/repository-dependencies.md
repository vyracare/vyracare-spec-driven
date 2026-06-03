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

### Dependencias de backend

| Repositorio | Depende de | Motivo |
| --- | --- | --- |
| `vyracare-api-authentication` | `template-dot-net-api`, `vyracare-infra-pipes-dot-net`, AWS, MongoDB Atlas | auth publica Lambda, Gateway e Cognito |
| `vyracare-api-client` | `template-dot-net-api`, `vyracare-infra-pipes-dot-net`, AWS, MongoDB Atlas | API de clientes |
| `vyracare-api-proceedings` | `template-dot-net-api`, `vyracare-infra-pipes-dot-net`, AWS, MongoDB Atlas | API de procedimentos |

## Dependencias cruzadas entre produto

### Shell e MFEs

- o shell depende dos `remoteEntry.js` publicados pelos MFEs
- os MFEs nao dependem do shell em runtime local, mas dependem do shell no fluxo de integracao publicada
- quando um MFE publica, a esteira pode atualizar o shell consumidor

### Shell e APIs

- o shell depende da API de autenticacao
- `environment.dev.ts`, `environment.hml.ts` e `environment.prod.ts` apontam para a auth por ambiente

### MFEs e APIs

- `vyracare-app-user-mfe` depende das URLs de auth e client
- `vyracare-app-profile-mfe` pode depender de auth e de backend proprio conforme evolucao
- `vyracare-app-proceedings-mfe` depende da API de proceedings quando o fluxo remoto esta ativo

### APIs e consumers

- a auth pode atualizar o shell e o `vyracare-app-user-mfe`
- APIs genericas podem atualizar MFEs consumidores configurados em `.vyracare/mfe-consumer.json`

## Ordem segura para subir do zero

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

## Regra operacional

Se um repositorio depende de outro para:

- pipeline
- pacote
- URL publicada
- sincronizacao de environment

entao a dependencia precisa estar estavel antes da publicacao do dependente.
