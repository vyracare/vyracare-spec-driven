# Gestao de Pacientes

Este documento define a navegacao, os contratos e as regras de acesso da gestao de pacientes.

## Rotas do frontend

O `vyracare-app-shell` monta o `vyracare-app-user-mfe` em `/pacientes`.

| Rota | Responsabilidade |
| --- | --- |
| `/pacientes` | consulta e busca de pacientes |
| `/pacientes/cadastro` | cadastro da ficha inicial |
| `/pacientes/editar/{id-paciente}` | ficha preenchida em modo de edicao ou leitura |

A rota anterior `/cadastro/pacientes` redireciona para `/pacientes/cadastro` para preservar links existentes. O menu lateral aponta para `/pacientes`.

## Consulta

A tela principal exibe nome, CPF, telefone, e-mail, ultima atualizacao e acoes. A busca usa um unico termo, sem diferenciar maiusculas e minusculas, sobre:

- nome completo;
- telefone;
- e-mail.

Contrato:

- `GET /api/client/patients?search={termo}`

Sem `search`, a API devolve todos os pacientes ordenados por nome.

## Ficha e autorizacao

Contrato de leitura:

- `GET /api/client/patients/{id}`

Contrato de atualizacao integral:

- `PUT /api/client/patients/{id}`

Somente tokens com role `Administrador` podem executar o `PUT`. A interface desabilita os campos para os demais niveis, mas a API e a autoridade final e responde `403` para tentativa nao autorizada.

O `vyracare-api-authentication` inclui no JWT:

- claim de role derivada de `AccessLevel`;
- `access_level` com o mesmo valor para consumo do frontend;
- `job_role` com o cargo funcional.

Depois dessa alteracao, usuarios ja autenticados precisam entrar novamente para receber as novas claims.

## Notas profissionais

Qualquer funcionario autenticado pode registrar notas, inclusive quando nao puder alterar a ficha cadastral.

Contratos:

- `POST /api/client/patients/{id}/notes`;
- `GET /api/client/patients/{id}/notes`.

Entrada de criacao:

```json
{
  "content": "Evolucao observada apos o procedimento.",
  "procedureName": "Peeling"
}
```

A API acrescenta `id`, `authorId`, `authorName` e `createdAt` em UTC a partir do usuario autenticado e do relogio do backend. O historico e devolvido do mais recente para o mais antigo. Notas sao anexadas e nao substituem registros anteriores.

## Responsabilidades por repositorio

- `vyracare-app-user-mfe`: tabela, busca, cadastro, ficha, modal de nota e historico;
- `vyracare-api-client`: persistencia, busca, atualizacao autorizada e notas;
- `vyracare-api-authentication`: claims de autorizacao;
- `vyracare-app-shell`: rotas e navegacao lateral.
