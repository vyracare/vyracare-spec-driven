# Governanca de Desenvolvimento

Este documento registra regras permanentes para qualquer entrega no ecossistema Vyracare. Elas se aplicam a trabalho manual, automacoes e agentes de desenvolvimento.

## Design System como origem de componentes reutilizaveis

Todo componente macro com potencial de reutilizacao entre duas ou mais telas, dominios ou MFEs deve ser criado primeiro no `vyracare-design-system`.

Sao componentes macro, entre outros:

- estruturas de navegacao, menus expansivos, barras de acao e cabecalhos;
- modais, paineis, cards compostos, estados vazios e feedback;
- padroes de busca, filtros, paginacao e apresentacao de listas;
- composicoes de formulario que resolvam comportamento visual ou acessibilidade recorrente;
- qualquer bloco que replique estrutura, responsividade, estados ou interacao ja existente em outra aplicacao.

O componente do Design System deve ser generico, orientado por entradas e eventos, acessivel, responsivo e livre de regras de negocio. Shell e MFEs ficam responsaveis por composicao, dados, permissoes, navegacao e regras do dominio.

O fluxo obrigatorio e:

1. verificar se o componente ja existe no catalogo;
2. criar ou evoluir o componente generico no Design System;
3. cobrir comportamento, acessibilidade e estados responsivos com testes;
4. publicar uma nova versao do pacote no CodeArtifact;
5. atualizar o shell antes dos MFEs quando o pacote compartilhado em runtime ganhar uma nova exportacao;
6. consumir o componente publicado nas aplicacoes, sem copiar sua implementacao localmente.

Uma implementacao local somente e aceita quando for estritamente especifica de um dominio e nao tiver potencial real de reutilizacao. A excecao precisa ser justificada na especificacao da funcionalidade.

## Spec-driven como criterio de entrada e saida

Nenhuma alteracao funcional ou arquitetural deve existir apenas no codigo. Antes ou durante a implementacao, a especificacao aplicavel deve registrar objetivo, ownership, contratos, permissoes, responsividade, seguranca e criterios de aceite.

Ao final de cada incremento ou interacao de desenvolvimento:

1. confrontar o resultado com o `vyracare-spec-driven`;
2. atualizar a documentacao quando o estado implementado, uma decisao ou um tradeoff tiver mudado;
3. executar testes e builds proporcionais ao risco;
4. inspecionar o diff e preservar alteracoes nao relacionadas;
5. criar commits coerentes em todos os repositorios modificados;
6. enviar os commits aos respectivos remotos;
7. registrar explicitamente qualquer impedimento de push, como repositorio sem remoto, permissao ou esteira indisponivel.

Uma interacao nao e considerada concluida enquanto houver codigo alterado sem documentacao correspondente, validacao pendente ou commit local que poderia ter sido enviado.

## Definicao de pronto

Uma entrega esta pronta quando:

- segue a arquitetura e os contratos documentados;
- mantem regras de negocio fora do Design System;
- reutiliza componentes publicados em vez de duplicar macros nos MFEs;
- preserva isolamento por tenant e autorizacao no backend quando aplicavel;
- possui testes e build aprovados;
- atualiza a especificacao e os runbooks afetados;
- nao inclui artefatos locais, segredos ou mudancas mecanicas sem intencao;
- possui commits e pushes realizados em cada repositorio alterado, ou impedimento documentado.

## Revisao de dependencias e runtimes

"Mais recente" nao significa automaticamente "mais adequado para producao". Atualizacoes devem considerar suporte do provedor, ciclo LTS/STS, compatibilidade de infraestrutura, bibliotecas compartilhadas e estrategia de rollback.

Versoes de framework sao revistas periodicamente e sempre que uma versao em uso se aproxima do fim de suporte. A decisao vigente esta registrada em [Modernizacao de Runtimes e Frameworks](../02-architecture/runtime-modernization.md).
