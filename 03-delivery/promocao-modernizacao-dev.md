# Preparacao da promocao da modernizacao para dev

Registro da verificacao de 27 de novembro de 2026. Commits, CI, aprovacoes e conteudo dos PRs podem mudar; conferir novamente antes de qualquer merge. Os PRs abaixo apontam para `develop`. CI aprovado nao substitui revisao de codigo nem validacao de ambiente.

| Repositorio | PR | Head verificado | Evidencia e atencao |
| --- | --- | --- | --- |
| API autenticacao | [#78](https://github.com/vyracare/vyracare-api-authentication/pull/78) | `eb2e75a` | Teste e build CI aprovados; SDK e dependencias. |
| API tenancy | [#3](https://github.com/vyracare/vyracare-api-tenancy/pull/3) | `82cc5de` | Teste e build CI aprovados; inclui correcao do escopo do inicializador. |
| API financeiro | [#6](https://github.com/vyracare/vyracare-api-finance/pull/6) | `e8bc02c` | Teste e build CI aprovados; inclui operacoes e permissoes financeiras anteriores a atualizacao. |
| API pacientes | [#40](https://github.com/vyracare/vyracare-api-client/pull/40) | `b606c03` | Teste e build CI aprovados; GitHub exige revisao. |
| API agendamentos | [#6](https://github.com/vyracare/vyracare-api-appointments/pull/6) | `e773ea8` | Teste e build CI aprovados; inclui notificacoes anteriores a atualizacao. |
| API procedimentos | [#29](https://github.com/vyracare/vyracare-api-proceedings/pull/29) | `1281e7b` | Teste e build CI aprovados; GitHub exige revisao; inclui historico de release, busca, isolamento por tenant e inventario. |
| Shell | [#107](https://github.com/vyracare/vyracare-app-shell/pull/107) | `7dde53c` | Teste e build CI aprovados; GitHub exige revisao; inclui financeiro, notificacoes e navegacao alem de Angular 22. |
| Dashboard | [#51](https://github.com/vyracare/vyracare-app-dashboard-mfe/pull/51) | `c0940b1` | Teste e build CI aprovados; GitHub exige revisao. |
| MFE funcionarios | [#44](https://github.com/vyracare/vyracare-app-profile-mfe/pull/44) | `08f633e` | Teste e build CI aprovados. |
| MFE pacientes | [#51](https://github.com/vyracare/vyracare-app-user-mfe/pull/51) | `3a08716` | Teste e build CI aprovados. |
| MFE procedimentos | [#39](https://github.com/vyracare/vyracare-app-proceedings-mfe/pull/39) | `f09f398` | Teste e build CI aprovados. |
| MFE financeiro | [#1](https://github.com/vyracare/vyracare-app-finance-mfe/pull/1) | `0d00e67` | Sem checks no PR: Actions desativadas; infraestrutura e URL do remoto ainda nao configuradas. |

Ausencia de `REVIEW_REQUIRED` na consulta nao prova que um PR recebeu aprovacao. Conferir regras e estado diretamente no GitHub, sem contornar protecoes. PRs antigos de release, inclusive alguns com conflitos, nao fazem parte desta promocao e exigem triagem propria.

O push em `develop` dos frontends inicia publicacao automatica em dev. No shell, o workflow tambem habilita publicacao de atualizacao mobile; confirmar o destino e a reversao desse fluxo antes da janela. Shell e remotos compartilham Angular com `strictVersion`; coordenar os artefatos e seus rollbacks para evitar uma combinacao Angular 20/22 incompativel. Configurar e validar CORS por origem em dev, sem transportar para publicacao o wildcard usado no teste local.

Antes de promover procedimentos, repetir o [inventario de dev](procedimentos-inventario-legado.md), pois a consulta anterior e apenas um retrato. Se houver documentos antigos, concluir atribuicao auditada antes da mudanca. A validacao local cobriu casos de autenticacao, tenant e financeiro com Mongo isolado, mas nao comprova Lambda, integracao completa em AWS ou experiencia mobile. Definir responsaveis, sequencia, verificacoes e rollback antes de publicar.
