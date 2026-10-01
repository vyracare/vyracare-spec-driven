# Indicadores do Dashboard

Este documento define os contratos que substituem os valores estaticos do `vyracare-app-dashboard-mfe`.

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
