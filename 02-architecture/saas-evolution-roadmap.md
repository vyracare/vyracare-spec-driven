# Evolucao do Vyracare para SaaS

## Status e objetivo

Este documento e uma direcao arquitetural futura, nao uma descricao de capacidade ja entregue. O objetivo e transformar o Vyracare em um servico hospedado multiempresa sem distribuir o codigo-fonte, os pipelines, os manifests de infraestrutura ou os artefatos internos aos clientes.

No modelo comercial proposto, a Vyracare opera uma unica linha de produto e concede acesso ao servico por assinatura. A clinica recebe credenciais, configuracao e suporte; nao recebe repositorios, pacotes, imagens de runtime ou acesso administrativo a AWS.

## Limite entre produto e conteudo educacional

Os repositorios do produto permanecem privados. Videos, artigos e cursos devem usar uma edicao demonstrativa separada, com dados sinteticos, nomes genericos, contas descartaveis e infraestrutura isolada da producao.

A edicao educacional pode demonstrar conceitos e pequenos trechos, mas nao deve conter:

- credenciais, tokens, cookies, arquivos `.env` ou valores de Parameter Store;
- identificadores de contas AWS, URLs internas, dominios administrativos ou configuracoes reais de CI/CD;
- dumps de banco, dados de pacientes, logs reais ou payloads que permitam reidentificacao;
- o conjunto completo dos servicos, regras comerciais, provisionamento e controles antifraude do produto;
- chaves de assinatura, algoritmos internos de autorizacao ou procedimentos operacionais de recuperacao.

Antes de gravar, deve existir um checklist de tela limpa, perfil de navegador exclusivo, ambiente `demo`, base sintetica e revisao quadro a quadro de terminais e DevTools. Publicar um video torna o que aparece nele copiavel; marca d'agua e aviso de copyright ajudam na atribuicao, mas nao substituem controle de acesso ao codigo.

## Proposta de arquitetura

### Plano de controle compartilhado

Criar um plano de controle central para capacidades SaaS:

- `Tenant` representando a clinica;
- `Membership` ligando usuario, tenant e papel;
- onboarding, convite, ativacao, suspensao e offboarding;
- plano comercial, assinatura, limites e `entitlements`;
- catalogo de tenants e resolucao segura da particao de dados;
- administracao interna, suporte auditavel e provisionamento automatizado.

Autenticacao continua centralizada. Cada identidade autenticada deve ser vinculada a um tenant e receber `tenant_id`, `membership_id`, papeis e permissoes no JWT. O backend deriva o tenant exclusivamente da identidade validada; `tenant_id` recebido em body, query string ou header do navegador nunca e fonte de autoridade.

### Plano de dados

Os MFEs e APIs atuais formam o plano de dados. Toda operacao deve exigir um `TenantContext` imutavel e aplicar o escopo antes de consultar, alterar, exportar ou excluir dados.

A recomendacao inicial e o modelo `bridge`:

- shell, MFEs, API Gateway, observabilidade e parte do compute permanecem compartilhados;
- autenticacao, onboarding e cobranca permanecem compartilhados;
- dados clinicos adotam separacao logica forte por clinica, preferencialmente database dedicado por tenant no inicio;
- tenants ou planos com requisitos especiais podem evoluir para recursos dedicados sem criar outra versao do produto.

Silo, pool e bridge sao modelos validos e a escolha deve considerar custo, compliance e risco. Particionar dados nao garante isolamento por si so; autorizacao e isolamento de tenant precisam de controles explicitos em todas as APIs.

### Contratos obrigatorios dos backends

- middleware comum resolve `TenantContext` depois da validacao do token;
- repositorios exigem tenant no construtor ou em cada operacao e nao oferecem consulta global por padrao;
- chaves, indices, caches, filas, arquivos e idempotency keys incluem o tenant;
- logs e metricas incluem `tenant_id` tecnico, sem incluir dados clinicos;
- tarefas assincronas carregam contexto assinado e validado, sem confiar em campos livres;
- endpoints administrativos usam autorizacao separada, menor privilegio e trilha de auditoria;
- testes automatizados tentam acesso cruzado entre pelo menos dois tenants em cada dominio.

## Seguranca, privacidade e operacao

Dados de saude sao dados pessoais sensiveis na LGPD. Antes do uso comercial, a plataforma precisa de inventario de tratamento, definicao documentada dos papeis de controlador e operador, base legal revisada, politica de retencao, atendimento aos direitos do titular e processo de incidente.

Baseline minimo:

- criptografia em transito e em repouso, rotacao de secrets e menor privilegio;
- MFA para administradores e suporte;
- auditoria de login, exportacao, alteracao clinica e acesso administrativo;
- backup por tenant, restauracao testada e plano de continuidade;
- rate limit, quotas e protecao contra noisy neighbor por tenant/plano;
- observabilidade com SLOs, alertas e custo atribuido por tenant;
- verificacao de dependencias, SAST, secret scanning e push protection;
- politica de resposta e comunicacao de incidentes alinhada a ANPD;
- ambientes de demo, desenvolvimento, homologacao e producao sem dados compartilhados.

## Protecao do ativo de software

- manter produto, infraestrutura e documentacao operacional em repositorios privados com acesso minimo;
- ativar protecao de branch, revisao obrigatoria, logs de auditoria, secret scanning e push protection quando disponiveis;
- fornecer aos clientes somente o servico hospedado e contratos de uso, privacidade, nivel de servico e tratamento de dados;
- separar formalmente codigo produzido para videos do codigo proprietario;
- revisar titularidade de contribuicoes de terceiros e contratos de colaboradores;
- avaliar registro das versoes relevantes do software no INPI para reforcar prova de autoria e titularidade;
- submeter licenca, termos e estrutura societaria a assessoria juridica especializada antes da comercializacao.

O registro no INPI aumenta a seguranca probatoria, mas nao substitui segredo operacional, controle de repositorio, contratos nem medidas tecnicas. Este documento nao constitui aconselhamento juridico.

## Roadmap incremental

### Fase 0 - proteger o que ja existe

1. Tornar privados os repositorios proprietarios e revisar acessos.
2. Criar ambiente e identidade visual de demonstracao para os videos.
3. Habilitar protecoes contra vazamento de secrets.
4. Definir o que pode e o que nao pode aparecer no conteudo.
5. Registrar um baseline versionado e avaliar deposito no INPI.

### Fase 1 - fundacao multi-tenant

1. Introduzir `Tenant`, `Membership` e `TenantContext`.
2. Incluir tenant e papeis no token.
3. Adaptar uma API piloto, recomendada `vyracare-api-client`.
4. Criar testes de isolamento cruzado e auditoria.
5. Migrar os demais dominios somente depois da validacao do piloto.

Os contratos executaveis, o onboarding de empresa e o periodo de teste de 30 dias estao detalhados em [Fundacao multi-tenant](./multi-tenant-foundation.md).

### Fase 2 - operacao comercial

1. Automatizar onboarding, convite e provisionamento.
2. Implementar planos, entitlements, limites e cobranca.
3. Criar console interno de suporte sem acesso irrestrito a dados.
4. Adicionar notificacoes de ciclo de assinatura e inadimplencia.
5. Medir uso, custo e margem por tenant.

### Fase 3 - prontidao para producao

1. Executar threat modeling e teste de invasao.
2. Validar backup e restauracao por tenant.
3. Formalizar privacidade, retencao, incidente e continuidade.
4. Realizar piloto fechado com poucas clinicas e dados controlados.
5. Liberar venda ampla somente depois de medir isolamento, suporte e confiabilidade.

## Decisoes que devem ser tomadas antes da implementacao

- publico inicial e quantidade esperada de clinicas;
- database dedicado, colecoes compartilhadas ou modelo hibrido por plano;
- papeis e permissoes de cada perfil da clinica;
- provedor de cobranca e regras de inadimplencia;
- responsabilidades da Vyracare e da clinica no tratamento de dados;
- RPO, RTO, retencao, exportacao e encerramento de conta;
- quais partes, se houver, terao uma edicao educacional publica.

## Referencias oficiais

- [AWS SaaS Lens: modelos silo, pool e bridge](https://docs.aws.amazon.com/wellarchitected/latest/saas-lens/silo-pool-and-bridge-models.html)
- [AWS: autorizacao e controle de acesso multi-tenant](https://docs.aws.amazon.com/prescriptive-guidance/latest/saas-multitenant-api-access-authorization/introduction.html)
- [AWS: prevencao de acesso entre tenants](https://docs.aws.amazon.com/wellarchitected/latest/saas-lens/preventing-cross-tenant-access.html)
- [ANPD: guia de seguranca da informacao](https://www.gov.br/anpd/pt-br/centrais-de-conteudo/materiais-educativos-e-publicacoes/guia_seguranca_da_informacao_para_atpps___defeso_eleitoral.pdf)
- [ANPD: comunicacao de incidente de seguranca](https://www.gov.br/anpd/pt-br/canais_atendimento/agente-de-tratamento/comunicado-de-incidente-de-seguranca-cis)
- [INPI: registro de programa de computador](https://www.gov.br/inpi/pt-br/assuntos/programas-de-computador/guia-completo-de-programa-de-computador)
- [GitHub: visibilidade e seguranca de repositorios](https://docs.github.com/en/enterprise-cloud@latest/repositories/creating-and-managing-repositories/about-repositories)
