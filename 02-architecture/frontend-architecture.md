# Arquitetura Frontend

## Modelo adotado

O frontend do Vyracare segue uma arquitetura de:

- shell Angular
- micro-frontends Angular
- design system compartilhado
- publicacao em S3 + CloudFront

## Camadas principais

### Design system
Fornece o contrato visual e de componentes.

### App shell
Responsavel por:

- layout base
- navegacao
- autenticacao
- configuracao de remotos
- orquestracao dos MFEs

### MFEs
Cada MFE representa um dominio funcional.

Cada projeto segue o mesmo principio:

- `environments.ts` para desenvolvimento local
- `environments.dev.ts` para `dev`
- `environments.hml.ts` para `hml`
- `environments.prod.ts` para `prod`

## Cabecalho das telas internas

As telas funcionais autenticadas, com excecao da pagina inicial do Dashboard, seguem um cabecalho visual comum:

- breadcrumb iniciado por `Dashboard` para indicar o contexto de navegacao;
- bloco de apresentacao com fundo em gradiente suave, borda e cantos arredondados;
- selo superior em caixa alta para identificar o dominio da tela;
- titulo e descricao objetiva da atividade;

A pagina inicial do Dashboard permanece com o hero e os indicadores proprios. A regra do cabecalho comum se aplica atualmente a Funcionarios, Procedimentos, Pacientes, Cadastro de paciente, Prontuario do paciente e Atendimentos.

Acoes primarias de negocio, como `Cadastrar paciente` e `Cadastrar atendimento`, ficam na barra da secao de conteudo logo abaixo do cabecalho. O bloco branco lateral de mensagem foi removido dos cabecalhos padronizados para reduzir ruido visual e deixar a apresentacao concentrada no titulo e na descricao.

As paginas principais de Funcionarios, Procedimentos e Atendimentos seguem tambem o padrao de gestao de Pacientes: card de listagem, busca quando aplicavel, tabela e acao primaria na barra do card. As acoes de cadastro navegam para paginas dedicadas com breadcrumb, hero e formulario em card branco. Esse fluxo evita formularios principais em modal e reserva modais para tarefas auxiliares ou confirmacoes, como configurar uma notificacao. Os contratos, campos exibidos e cuidados de minimizacao de dados estao detalhados em `clinical-catalog-management.md` e `dashboard-indicators.md`.

Em larguras menores, o cabecalho e as barras de acao devem ser empilhados para evitar rolagem horizontal e preservar a legibilidade.

## Componentes de formulario compartilhados

O pacote `@vyracare/design-system` e a fonte dos controles compartilhados. A partir da versao `0.5.0`, alem dos controles base, ele fornece campos semanticos para os dados recorrentes dos dominios:

- `vc-autocomplete`: ControlValueAccessor com label, hint, erro, carregamento, vazio, navegacao por teclado, lista acessivel e eventos de pesquisa/selecao. Os resultados sao exibidos em um painel flutuante sobre o conteudo, ancorado na largura do campo e sem alterar a altura ou o fluxo do formulario;
- `vc-modal`: superficie acessivel e responsiva com regioes projetadas de cabecalho, corpo e rodape. O componente centraliza backdrop, limite de viewport, rolagem vertical, espacamento entre titulo e conteudo, organizacao das acoes, fechamento por backdrop ou Escape e restauracao de foco;
- `vc-checkbox`: checkbox visual padronizado com label, descricao, erro e integracao com Angular Forms;
- `vc-select`: listbox customizado, sem depender da aparencia nativa diferente entre navegadores;
- `vc-input`: inclui mascaras de telefone, e-mail, data, CPF e CEP e emite o valor mascarado no desfoque.
- `vc-phone-input`: fixa tipo telefonico, teclado adequado, placeholder nacional e mascara para dez ou onze digitos;
- `vc-email-input`: fixa tipo e teclado de e-mail e normaliza o valor sem espacos e em minusculas;
- `vc-date-time-input`: usa o controle nativo `datetime-local` dentro do mesmo layout, estados e contrato de Angular Forms;
- `vc-postal-code-input`: fixa teclado numerico, placeholder e mascara brasileira `00000-000`, preservando o evento de desfoque para consultas de endereco.

Os MFEs devem preferir os campos semanticos quando o dado corresponder a telefone, e-mail, data/hora ou CEP, deixando validacoes de negocio no formulario consumidor. Eles nao devem recriar mascaras, tipos nativos ou placeholders para esses casos. Tambem nao devem recriar paineis ou estilos de autocomplete localmente. Empilhamento, sombra, estados interativos, truncamento de textos extensos e responsividade pertencem ao componente compartilhado. O `vyracare-app-dashboard-mfe` usa `vc-autocomplete` nos seletores de funcionario e procedimento e os campos semanticos no telefone e nos horarios do atendimento, mantendo debounce e consultas de dominio no MFE. O `vyracare-app-user-mfe` usa os controles compartilhados no cadastro e na edicao de pacientes.

Inputs e selects desabilitados recebem fundo e borda acinzentados, texto atenuado e cursor `not-allowed` diretamente pelo estilo-base do Design System. Assim, o mesmo estado visual e aplicado aos controles base e aos campos semanticos em todos os MFEs.

## Acoes por icone e tooltip

O Design System fornece `vc-icon-button` para acoes compactas e `vc-tooltip` para sua descricao flutuante. O tooltip aceita posicionamento superior, inferior, esquerdo ou direito, abre por hover ou foco de teclado e fecha na saida, perda de foco, tecla Escape ou movimentacao da viewport. Sua superficie usa coordenadas fixas calculadas a partir do gatilho, ficando acima de containers com `overflow` sem aumentar a largura de tabelas ou criar barras de rolagem. O texto visivel do tooltip complementa o `ariaLabel` obrigatorio do botao de icone.

Tabelas com varias acoes por registro devem preferir essa combinacao para reduzir largura e ruido visual, mantendo nomes objetivos para leitores de tela. Os MFEs nao devem depender apenas do atributo nativo `title` nem recriar a superficie flutuante localmente.

## Feedback global por toast

O Design System fornece `VcToastService` e `vc-toast-container` para mensagens transitorias de sucesso, erro, alerta e informacao. O shell mantem uma unica viewport flutuante montada acima das rotas. MFEs publicam feedback pelo servico e nao devem criar banners locais para o mesmo resultado.

O estado usa Angular Signals e um evento de navegador namespaced para sincronizar o shell e remotos mesmo quando mais de uma instancia fisica do pacote tiver sido carregada. Cada mensagem possui identificador, variante semantica, titulo, descricao e duracao; mensagens podem ser fechadas manualmente e sao removidas automaticamente por padrao. Erros usam live region assertiva e os demais estados usam live region educada.

O pacote `@vyracare/design-system` deve ser compartilhado como singleton na configuracao de Module Federation. Isso evita instancias duplicadas dos componentes, elimina colisoes `NG0912` e garante um contrato visual unico entre shell e MFEs.

Como o shell fornece a instancia singleton em tempo de execucao, sua versao do Design System deve ser igual ou superior a versao exigida por todos os MFEs carregados. Uma entrega de MFE que utilize uma nova exportacao somente pode ser promovida depois que o shell estiver alinhado com essa versao. Em ambiente local, depois da atualizacao do pacote, o processo do shell deve ser reiniciado para descartar a instancia anterior mantida em memoria. A ausencia desse alinhamento pode deixar um componente remoto indefinido e causar erros Angular como acesso a `ɵcmp` durante a ativacao da rota.

## Documentacao do codigo TypeScript

Componentes TypeScript devem usar comentarios JSDoc objetivos para explicar a responsabilidade da classe e de seus metodos. A documentacao deve registrar a intencao e a regra coordenada pelo metodo, evitando apenas repetir seu nome ou descrever detalhes obvios de sintaxe. Metodos publicos, protegidos e privados criados ou alterados em uma entrega devem ser revisados como parte da mesma mudanca.

## Ambientes frontend

### Dev

- bucket com sufixo `-dev`
- sem blue/green
- deploy versionado por pasta na raiz do bucket
- CloudFront `dev` aponta para a pasta versionada ativa

### HML

- bucket sem `-dev`
- uso da pasta `blue/<timestamp>`
- CloudFront `hml` aponta para `blue`

### Prod

- bucket sem `-dev`
- uso da pasta `green/<timestamp>`
- CloudFront `prod` aponta para `green`

## Estrategia de build

### `develop`
Build Angular usa configuracao `dev`, com `fileReplacement` para `environments.dev.ts`.

### `release/*`
Build Angular usa configuracao `hml`, com `fileReplacement` para `environments.hml.ts`.

### `main`
Build Angular usa configuracao `production`, com `fileReplacement` para `environments.prod.ts`.

## Integracao com backends

As URLs de API podem ser atualizadas automaticamente pelas esteiras `.NET`.

A regra atual e:

- branch `develop` -> atualiza `environments.dev.ts`
- branch `release/*` -> atualiza `environments.hml.ts`
- branch `main` -> atualiza `environments.prod.ts`

Isso vale para:

- shell consumidor de auth
- MFEs consumidores de APIs especificas

### Dashboard

O `vyracare-app-dashboard-mfe` nao mantem valores demonstrativos nos cards integrados. Ele consulta:

- `appointmentsApiUrl` para os indicadores operacionais;
- `financeApiUrl` para os indicadores de saude financeira.

As duas chamadas enviam o JWT do usuario. O ambiente local aponta para as portas `5003` e `5004`; os arquivos de `dev`, `hml` e `prod` devem receber as URLs publicadas pelas esteiras das APIs.

## Risco conhecido

Quando automacoes alteram arquivos de environment, o maior risco e gerar:

- propriedades duplicadas
- virgulas duplicadas
- arquivo JSON/TS invalido

As pipes atuais ja possuem normalizacao para reduzir esse risco, mas isso continua sendo um ponto sensivel da arquitetura de sincronizacao entre repositorios.
