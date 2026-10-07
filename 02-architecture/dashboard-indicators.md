# Indicadores do Dashboard

Este documento define os contratos que substituem os valores estaticos do `vyracare-app-dashboard-mfe`.

## Acoes principais

No dashboard mobile, os botoes `Novo atendimento` e `Novo paciente` ocupam toda a largura disponivel e possuem altura minima de toque. Em telas maiores, permanecem compactos e lado a lado para preservar a hierarquia do hero.

## Indicadores operacionais

Fonte: `vyracare-api-appointments`.

Endpoint:

- `GET /api/appointments/dashboard/summary`

Resposta:

```json
{
  "appointmentsToday": 18,
  "confirmedLastTwoHours": 4,
  "pendingFollowUps": 6,
  "weeklyOccupancyRate": 82
}
```

Regras:

- `appointmentsToday`: agendamentos do dia que nao estejam cancelados;
- `confirmedLastTwoHours`: agendamentos de hoje confirmados nas duas horas anteriores;
- `pendingFollowUps`: retornos com vencimento dentro da janela configurada e ainda nao agendados;
- `weeklyOccupancyRate`: minutos nao cancelados da semana divididos pelos minutos semanais disponiveis.

## Indicadores financeiros

Fonte: `vyracare-api-finance`.

Endpoint:

- `GET /api/finance/dashboard/summary?month=yyyy-MM`

Quando `month` nao e informado, a API usa o mes corrente no fuso configurado.

Resposta:

```json
{
  "expectedRevenue": 124800,
  "revenueVariationPercentage": 12,
  "operationalExpenses": 41200,
  "expenseVariationPercentage": -6,
  "pendingInvoicesCount": 8,
  "pendingInvoicesAmount": 9600
}
```

Regras:

- receita e despesa consideram apenas lancamentos confirmados do mes;
- as variacoes comparam o mes consultado com o mes imediatamente anterior;
- boletos pendentes agregam quantidade e valor de todos os boletos com status pendente;
- valores monetarios sao retornados como numero decimal e formatados no frontend em `pt-BR`/BRL.

## Seguranca e tempo

- os endpoints de resumo exigem JWT;
- o dashboard envia `Authorization: Bearer <token>` pelo interceptor HTTP;
- datas sao persistidas em UTC;
- limites de dia, semana e mes usam `America/Sao_Paulo` por padrao.

## Integracao por ambiente

| Ambiente | Appointments | Finance |
| --- | --- | --- |
| Local | `http://localhost:5003/api/appointments` | `http://localhost:5004/api/finance` |
| Dev | preenchido pela esteira apos deploy | preenchido pela esteira apos deploy |
| HML | preenchido pela esteira apos deploy | preenchido pela esteira apos deploy |
| Prod | preenchido pela esteira apos deploy | preenchido pela esteira apos deploy |

Enquanto nao houver dados cadastrados, os cards exibem zero. Falhas de rede ou autenticacao sao tratadas pelo dashboard sem restaurar valores ficticios.

## Agenda de atendimentos

Rotas no shell:

- `/dashboard/agenda`: listagem de atendimentos e entrada do menu lateral;
- `/dashboard/agenda/novo`: pagina dedicada ao cadastro.

A tela pertence ao `vyracare-app-dashboard-mfe` e permite:

- informar nome e telefone do paciente;
- pesquisar e selecionar o profissional cadastrado por nome, e-mail ou telefone;
- pesquisar e selecionar o procedimento ativo por nome ou codigo;
- definir inicio e termino do atendimento;
- configurar antecedencia em horas ou dias por modal;
- listar todos os agendamentos com estado de proximidade.

Na pagina de cadastro, telefone usa `vc-phone-input`, com a mesma mascara nacional aplicada ao cadastro de pacientes. Inicio e termino usam `vc-date-time-input`. Ambos pertencem ao Design System e integram o formulario reativo por `ControlValueAccessor`, evitando configuracoes e estilos divergentes entre MFEs.

O conteudo principal segue o mesmo padrao visual das demais telas internas. A listagem apresenta breadcrumb `Dashboard / Atendimentos`, cabecalho em gradiente e tabela. O botao `Cadastrar atendimento`, localizado na barra da tabela, navega para `/dashboard/agenda/novo`. A pagina de cadastro usa breadcrumb `Dashboard / Atendimentos / Cadastrar`, hero em gradiente e formulario em card branco; ao salvar, retorna para a listagem. Somente a configuracao da antecedencia da notificacao permanece em modal. O item lateral `Atendimentos` aponta para `/dashboard/agenda`.

Ao criar um atendimento, a confirmacao da API publica o toast global de sucesso antes do retorno para a agenda. Falhas de gravacao, pesquisa dos autocompletes, carregamento da agenda ou consulta dos resumos do dashboard publicam toast de erro com descricao segura; validacoes locais do formulario permanecem inline e nao disparam feedback global antes de uma requisicao.

O breadcrumb e o hero da listagem de atendimentos reutilizam os mesmos tokens globais, dimensoes, tipografia, bordas, gradiente e comportamento responsivo das paginas de Pacientes, Funcionarios e Procedimentos. O MFE nao deve sobrescrever localmente as cores de texto, destaque ou descricao desse cabecalho.

### Autocomplete de funcionario e procedimento

Os campos `Funcionario responsavel` e `Procedimento` nao aceitam texto livre como referencia final. A interface inicia a pesquisa apos dois caracteres, aplica debounce de 250 ms e exige que o usuario selecione uma opcao retornada pelas APIs.

A lista de resultados usa o `vc-autocomplete` do Design System e deve abrir como uma camada flutuante sobre as linhas seguintes do formulario. O painel acompanha a largura do campo, preserva a altura do grid, possui destaque de hover/foco e mantem navegacao por teclado e atributos de acessibilidade.

Na pagina de cadastro, o bloco de notificacao deve manter espacamento vertical proprio antes da barra de acoes, evitando contato visual entre o card de lembrete e os botoes `Cancelar` e `Salvar atendimento`.

Contratos consumidos:

- `GET /api/auth/employees?search={texto}&limit=20`: retorna funcionarios ativos cujo nome, e-mail ou telefone corresponde ao texto. A resposta contem somente `id`, `fullName`, `email`, `phone` e `role`, sem credenciais ou hash de senha;
- `GET /api/proceedings?search={texto}&activeOnly=true&limit=20`: retorna procedimentos ativos cujo nome ou codigo corresponde ao texto.

Ao salvar, o agendamento persiste o identificador e o nome da opcao selecionada em `employeeId`/`employeeName` e `proceedingId`/`proceedingName`. Alterar o texto depois de uma selecao invalida a referencia anterior e obriga uma nova selecao.

O `vyracare-app-dashboard-mfe` possui URLs separadas para Authentication e Proceedings em `local`, `dev`, `hml` e `prod`, alem das URLs de Appointments e Finance ja utilizadas.

O backend devolve um `scheduleStatus` calculado:

- `Today`: atendimento no dia corrente;
- `Approaching`: atendimento nas proximas 24 horas;
- `Scheduled`: atendimento futuro fora da janela de 24 horas;
- `Overdue`: horario encerrado sem conclusao;
- `Completed`, `Cancelled` e `NoShow`: estados finais do atendimento.

## Contrato de notificacao

Ao criar o agendamento, a API transforma a antecedencia em `reminderAt` UTC. O dashboard consulta a cada minuto:

- `GET /api/appointments/notifications/due`

Quando o navegador concedeu permissao, o frontend exibe uma notificacao nativa e confirma a entrega em:

- `POST /api/appointments/{id}/notifications/acknowledge`

A confirmacao grava `notificationSentAt` e impede repeticao. Agendamentos cancelados, concluidos ou ja iniciados nao sao retornados. Esta fase cobre notificacao interna/nativa do navegador; SMS, WhatsApp e e-mail exigem integracao futura com provedor externo.
