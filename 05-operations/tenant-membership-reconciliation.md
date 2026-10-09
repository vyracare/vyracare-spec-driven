# Runbook de Reconciliacao de Membership

## Objetivo

Diagnosticar e corrigir contas autenticadas que sao direcionadas para `/onboarding/empresa` mesmo pertencendo a uma empresa existente. O redirecionamento ocorre quando o JWT nao possui `tenant_id`, normalmente porque a identidade nao tem a projecao `tenantAccess` e o tenancy nao possui uma membership ativa correspondente.

Este runbook nao autoriza associar usuarios por dominio de e-mail, CNPJ ou semelhanca de nome. O tenant correto precisa ser confirmado por evidencia administrativa.

## Sintomas

- login e senha sao aceitos;
- o shell direciona para `/onboarding/empresa`;
- o usuario consta na gestao antiga de funcionarios;
- o JWT nao contem `tenant_id`, `membership_id` ou `tenant_role`;
- criar uma nova empresa seria incorreto porque a pessoa e funcionaria de um tenant existente.

## Causa observada

O endpoint administrativo antigo reutilizava o cadastro generico de identidade. Ele criava o usuario no auth, mas nao recebia o tenant do administrador, nao criava membership no plano de controle e nao persistia `tenantAccess`. O login apenas refletia essa projecao ausente no token.

Uma segunda falha observada no ambiente local fazia auth e tenancy detectarem `TENANCY_INTERNAL_API_KEY` sem copia-la para a configuracao tipada. A chamada interna recebia `401`, que o auth convertia em `503`. Os dois bootstraps agora mapeiam explicitamente o fallback e possuem teste unitario.

## Verificacao segura

1. Confirmar a identidade pelo e-mail somente para localizacao, sem registrar senha, hash ou token em logs.
2. Obter o `userId` persistido no auth.
3. Consultar memberships ativas pelo endpoint interno `GET /api/tenancy/internal/users/{userId}/memberships`.
4. Confirmar o tenant correto por evidencia administrativa independente.
5. Verificar se o tenant esta `Trialing` ou `Active` e se a conta proprietaria possui membership `Owner` ativa.

Interpretacao:

- membership ativa e `tenantAccess` ausente: sair e entrar novamente aciona reconciliacao automatica;
- nenhuma membership: criar o vinculo somente depois de confirmar o tenant correto;
- multiplas memberships: nao escolher automaticamente; usar o futuro seletor de empresa;
- membership suspensa ou revogada: nao reativar como correcao tecnica sem decisao administrativa.

## Correcao de uma conta legada

1. Chamar `POST /api/tenancy/internal/tenants/{tenantId}/memberships` com `userId` e papel `Administrator` ou `Member`.
2. Repetir a mesma chamada para validar idempotencia; o `membershipId` deve permanecer igual.
3. Consultar novamente as memberships e confirmar exatamente um vinculo ativo no cenario simples.
4. Solicitar logout e novo login. O auth consulta tenancy, persiste `tenantAccess` e emite um novo JWT.
5. Validar as claims `tenant_id`, `membership_id` e `tenant_role` sem registrar o token completo.
6. Confirmar que o shell segue para a aplicacao e nao para o onboarding de empresa.

Nunca editar o JWT, copiar `tenantAccess` de outro usuario ou inserir membership diretamente no Mongo como primeira opcao. Os endpoints internos preservam validacoes e idempotencia.

## Cadastro novo de funcionario

O fluxo corrigido e automatico:

1. administrador autenticado chama `POST /api/auth/employees`;
2. auth obtem `tenant_id` exclusivamente do principal autenticado;
3. auth cria a identidade;
4. tenancy cria a membership idempotente;
5. auth persiste a projecao `tenantAccess`;
6. o primeiro login ja emite JWT com contexto empresarial.

Se o provisionamento falhar, o auth remove a identidade criada. Se a membership ja tiver sido criada antes da falha da projecao, chama `DELETE /api/tenancy/internal/tenants/{tenantId}/memberships/{userId}` antes de remover a identidade.

## Validacao local

Servicos esperados:

- auth: `127.0.0.1:5000`;
- tenancy: `127.0.0.1:5006`;
- ambos usam o mesmo valor backend-only de `TENANCY_INTERNAL_API_KEY`;
- tenancy usa `Mongo__Database=vyracare_tenancy_dev`;
- auth usa `Mongo__Database=vyracare_db_dev`.

Validacoes minimas:

- Swagger das duas APIs responde;
- chamada interna sem chave responde `401`;
- chamada com chave e payload vazio responde `400`, provando que a autenticacao interna foi aceita;
- suites do auth e tenancy passam;
- pipelines de teste e build passam antes do merge.

Em Windows, `0x800711C7` indica bloqueio de DLL pela politica de Controle de Aplicativo, frequente em diretorios sincronizados pelo OneDrive. Nesse caso, executar os testes nos containers Linux e confirmar novamente no GitHub Actions; nao tratar o bloqueio como falha funcional do teste.

## Rollback e compensacao

Se um vinculo incorreto for criado e ainda nao houver dados operacionais associados:

1. confirmar exatamente `tenantId` e `userId`;
2. chamar o endpoint interno de exclusao da membership;
3. invalidar a sessao atual por logout;
4. verificar que o login seguinte nao emite claims do tenant removido;
5. registrar a decisao administrativa que justificou a remocao.

Nao excluir o tenant para corrigir apenas um funcionario. A exclusao de tenant e uma compensacao exclusiva do onboarding de proprietario e remove todas as memberships associadas.

## Evidencias da entrega

- auth: commit `85983b0`;
- tenancy: commit `3b0af22`;
- documentacao arquitetural inicial: commit `f184c56`;
- testes aprovados: 20 em auth e 5 em tenancy;
- membership legada validada com repeticao idempotente e papel `Administrator`.

Dados pessoais e identificadores reais usados na verificacao operacional nao devem ser registrados nesta especificacao.
