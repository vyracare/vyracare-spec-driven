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

A pagina inicial do Dashboard permanece com o hero e os indicadores proprios. A regra do cabecalho comum se aplica atualmente a Funcionarios, Procedimentos, Pacientes, Cadastro de paciente, Ficha do paciente e Atendimentos.

Acoes primarias de negocio, como `Cadastrar paciente` e `Cadastrar atendimento`, ficam na barra da secao de conteudo logo abaixo do cabecalho. O bloco branco lateral de mensagem foi removido dos cabecalhos padronizados para reduzir ruido visual e deixar a apresentacao concentrada no titulo e na descricao.

Em larguras menores, o cabecalho e as barras de acao devem ser empilhados para evitar rolagem horizontal e preservar a legibilidade.

## Componentes de formulario compartilhados

O pacote `@vyracare/design-system` e a fonte dos controles compartilhados. A partir da versao `0.5.0`, alem dos controles base, ele fornece campos semanticos para os dados recorrentes dos dominios:

- `vc-autocomplete`: ControlValueAccessor com label, hint, erro, carregamento, vazio, navegacao por teclado, lista acessivel e eventos de pesquisa/selecao. Os resultados sao exibidos em um painel flutuante sobre o conteudo, ancorado na largura do campo e sem alterar a altura ou o fluxo do formulario;
- `vc-checkbox`: checkbox visual padronizado com label, descricao, erro e integracao com Angular Forms;
- `vc-select`: listbox customizado, sem depender da aparencia nativa diferente entre navegadores;
- `vc-input`: inclui mascaras de telefone, e-mail, data, CPF e CEP e emite o valor mascarado no desfoque.
- `vc-phone-input`: fixa tipo telefonico, teclado adequado, placeholder nacional e mascara para dez ou onze digitos;
- `vc-email-input`: fixa tipo e teclado de e-mail e normaliza o valor sem espacos e em minusculas;
- `vc-date-time-input`: usa o controle nativo `datetime-local` dentro do mesmo layout, estados e contrato de Angular Forms;
- `vc-postal-code-input`: fixa teclado numerico, placeholder e mascara brasileira `00000-000`, preservando o evento de desfoque para consultas de endereco.

Os MFEs devem preferir os campos semanticos quando o dado corresponder a telefone, e-mail, data/hora ou CEP, deixando validacoes de negocio no formulario consumidor. Eles nao devem recriar mascaras, tipos nativos ou placeholders para esses casos. Tambem nao devem recriar paineis ou estilos de autocomplete localmente. Empilhamento, sombra, estados interativos, truncamento de textos extensos e responsividade pertencem ao componente compartilhado. O `vyracare-app-dashboard-mfe` usa `vc-autocomplete` nos seletores de funcionario e procedimento e os campos semanticos no telefone e nos horarios do atendimento, mantendo debounce e consultas de dominio no MFE. O `vyracare-app-user-mfe` usa os controles compartilhados no cadastro e na edicao de pacientes.

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
