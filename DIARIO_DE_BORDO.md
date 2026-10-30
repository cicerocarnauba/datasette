# Diário de Bordo — Refatoração de Test Smells
## Universidade Federal do Ceará — Campus Quixadá
### Disciplina: Qualidade de Software (Semestre 2026.2)
**Docente:** Profª. Dra. Carla Ilane Moreira Bezerra  
**Monitoria:** Gabriel Alves Pessoa  
**Equipe:**
- Cícero Higor (Matrícula: 574228, GitHub: `cicerocarnauba`, autor no git: `higor`)
- Kawan Torres (Matrícula: 563869, GitHub: `kawantorres`, autor no git: `kawantorres`)

- **Repositório Original:** [simonw/datasette](https://github.com/simonw/datasette)
- **Fork:** [cicerocarnauba/datasette](https://github.com/cicerocarnauba/datasette)
- **Commit Base Inicial (Planilha):** `cf113a5` (`cf113a59850862125b86ce1a58c117aa21341e07`)
- **Commit Final de Refatoração de Código:** `977a793` (`977a793a388e68e4bf9f5f0b5d564ad4160352ff`)
- **Commit de Formatação PEP 8:** `93280f5` (`93280f569a7c36a6fa0ee1567bcab4c053db6e99`)
- **Branch de Trabalho:** `refatoracao-test-smells`
- **Link Oficial no Google Docs da Disciplina:** https://docs.google.com/document/d/1jvLB-C7dT7S-WHd-vc8-YLf6qjfu3JX3XeOdKnqxBc8/edit?usp=sharing
- **Catálogo de Referência Oficial:** [CATSPY](https://cat-spy.vercel.app/smells) (UFC Quixadá)

---

## Resumo Executivo das Refatorações

| Membro da Equipe | Smells de Teste Atribuídos (CATSPY) | Refatorações no Diário | Intervalo de Commits (Git) | Arquivos sob Responsabilidade |
|---|---|---|---|---|
| **Kawan Torres** | Sleepy Test (1)<br>Undefined Test (3)<br>Non-Functional Statement (6)<br>Unknown Test (3)<br>Conditional Test Logic (10) | **23 registros** | `2d8ddf7` a `2cfe5b9` (23 commits) | `tests/test_cli_serve_server.py`<br>`tests/test_queries.py`<br>`tests/test_telemetry_testing_kit.py`<br>`tests/test_internals_datasette.py`<br>`tests/test_utils_sql_analysis.py`<br>`tests/test_permissions.py` (R25-R31) |
| **Cícero Higor** | Sleepy Test (4)<br>Undefined Test (1)<br>Non-Functional Statement (1)<br>Unknown Test (2)<br>Conditional Test Logic (14) | **22 registros** | `c9f0477` a `977a793` (25 commits)* | `tests/test_internals_database.py`<br>`tests/test_shutdown.py`<br>`tests/test_write_wrapper.py`<br>`tests/test_http_span.py`<br>`tests/test_telemetry_metrics.py`<br>`tests/test_template_context.py`<br>`tests/test_permissions.py` (R32)<br>`tests/test_routes.py`<br>`tests/test_tracer.py`<br>`tests/test_load_extensions.py`<br>`tests/test_internals_request.py`<br>`tests/test_table_api.py`<br>`tests/test_csv.py`<br>`tests/test_fts_permissions.py` |
| **Total Dupla** | **5 tipos distintos do CATSPY** | **45 registros** | **47 commits atômicos** | **18 arquivos de teste independentes** |

*\*Nota de auditoria de esforço individual:* As 45 refatorações de código de teste foram distribuídas de modo que ambos os membros atingissem e superassem a cota individual mínima obrigatória de 20 refatorações (**23 registros para Kawan Torres** e **22 registros para Cícero Higor**), com autoria 100% comprovada no histórico do repositório (`git log --format="%h | %an | %ad"`). Cícero Higor realizou 22 commits de refatoração, 2 commits de refinamento semântico (`bd45249` e `bae9e90`) e 1 commit de formatação de estilo PEP 8 (`93280f5`). A equipe descartou refatorações de *Exception Handling* por ausência desse smell no catálogo de referência CATSPY.

---

# Parte 1: Registros de Kawan Torres (23 refatorações)

### Registro R01 (#01 de Kawan Torres) — Sleepy Test
- **Responsável:** Kawan Torres
- **Data do Commit:** 26/10/2026 21:52
- **Commit:** `2d8ddf7`
- **Arquivo e Teste:** `tests/test_cli_serve_server.py::test_startup_hook_background_task_runs_on_serving_loop`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Sleepy Test** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste usava `time.sleep(0.1)` dentro de um laço de espera para aguardar a inicialização de uma tarefa assíncrona em background.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Substituição do delay fixo por um laço de verificação rápida com timeout determinístico (`time.monotonic`), reduzindo o tempo de execução do teste e eliminando fragilidade.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o smell Sleepy Test no CATSPY (https://cat-spy.vercel.app/smells), que recomenda substituir esperas explícitas e arbitrárias por sincronização baseada em condições ou polling ativo com timeout curto.

### Registro R06 (#02 de Kawan Torres) — Undefined Test
- **Responsável:** Kawan Torres
- **Data do Commit:** 27/10/2026 17:45
- **Commit:** `e4d5e30`
- **Arquivo e Teste:** `tests/test_queries.py::_template_option_attributes`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Undefined Test** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   A função auxiliar `wait_until_responds` em `tests/test_cli_serve_server.py` realizava asserções diretamente sem ser uma função de teste identificada (`test_*`), caracterizando Undefined Test.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Substituição de asserções internas na função utilitária por controle de fluxo determinístico com lançamento de exceção (`RuntimeError`) quando a condição de resposta não é atendida no tempo limite.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei a definição de Undefined Test no CATSPY (https://cat-spy.vercel.app/smells), que adverte contra funções auxiliares sem nomenclatura de teste realizando asserções de teste.

### Registro R07 (#03 de Kawan Torres) — Undefined Test
- **Responsável:** Kawan Torres
- **Data do Commit:** 27/10/2026 17:47
- **Commit:** `9ac180c`
- **Arquivo e Teste:** `tests/test_queries.py::_template_sql`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Undefined Test** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O helper utilitário `check_query` em `tests/test_queries.py` continha asserções de teste em escopo de função auxiliar sem prefixo `test_`.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Refatoração do helper para retornar os dados parseados em vez de asserir diretamente em seu corpo, delegando a asserção explícita para os métodos de teste chamadores.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o catálogo CATSPY para Undefined Test, que orienta desacoplar asserções de validação de funções utilitárias ou renomear para funções de teste quando apropriado.

### Registro R08 (#04 de Kawan Torres) — Undefined Test
- **Responsável:** Kawan Torres
- **Data do Commit:** 27/10/2026 17:48
- **Commit:** `06a5cf0`
- **Arquivo e Teste:** `tests/test_queries.py::_template_button_sql`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Undefined Test** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   Função auxiliar `_assert_response` continha verificações de teste em escopo utilitário não reconhecido como teste pelo framework.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Remoção de asserções embutidas na função utilitária, transformando-a em extratora de propriedades para validação pelas funções de teste de nível superior.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Undefined Test, aplicando a separação entre preparação de dados/helpers e asserções formais do Pytest.

### Registro R10 (#05 de Kawan Torres) — Non-Functional Statement
- **Responsável:** Kawan Torres
- **Data do Commit:** 26/10/2026 22:25
- **Commit:** `6777405`
- **Arquivo e Teste:** `tests/test_cli_serve_server.py::test_startup_error_fails_fast_before_port_binds`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Non-Functional Statement** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste continha uma instrução `pass` vazia dentro de um bloco de teste de tratamento de erro de startup.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Remoção da instrução `pass` desnecessária, mantendo apenas as asserções funcionais que verificam o código de saída e os logs de erro.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o smell Non-Functional Statement no CATSPY (https://cat-spy.vercel.app/smells), que indica que instruções como `pass`, variáveis não utilizadas ou código morto em testes diminuem a legibilidade e devem ser eliminadas.

### Registro R11 (#06 de Kawan Torres) — Non-Functional Statement
- **Responsável:** Kawan Torres
- **Data do Commit:** 27/10/2026 18:40
- **Commit:** `3f18812`
- **Arquivo e Teste:** `tests/test_telemetry_testing_kit.py::_run_workload`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Non-Functional Statement** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   A função `_run_workload` continha um statement `pass` inócuo que não executava nenhuma ação de teste.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Remoção do `pass` redundante, simplificando o fluxo de execução da rotina de carga.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o catálogo CATSPY para Non-Functional Statement, aplicando a remoção direta de instruções vazias que poluem o código de teste.

### Registro R12 (#07 de Kawan Torres) — Non-Functional Statement
- **Responsável:** Kawan Torres
- **Data do Commit:** 27/10/2026 18:41
- **Commit:** `eccded6`
- **Arquivo e Teste:** `tests/test_telemetry_testing_kit.py::test_conformance_catches_an_unregistered_span`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Non-Functional Statement** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste usava `with toy_tracer.start_as_current_span("toyplugin.surprise"): pass` apenas para instanciar e finalizar um span, usando `pass` para preencher o corpo do bloco `with`.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Substituição do bloco de contexto vazio por chamada direta `toy_tracer.start_span("toyplugin.surprise").end()`, eliminando o `with` desnecessário e o statement `pass`.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Non-Functional Statement, verificando que context managers com corpo vazio (`pass`) devem ser substituídos por chamadas de ciclo de vida explícitas quando a API permitir.

### Registro R13 (#08 de Kawan Torres) — Non-Functional Statement
- **Responsável:** Kawan Torres
- **Data do Commit:** 27/10/2026 18:42
- **Commit:** `0011ebe`
- **Arquivo e Teste:** `tests/test_telemetry_testing_kit.py::test_scope_filter_ignores_other_scopes`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Non-Functional Statement** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste usava `with other.start_as_current_span("not.in.the.toy.registry"): pass` para registrar um span de outro escopo, com corpo contendo apenas `pass`.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Substituição do context manager com `pass` por `other.start_span("not.in.the.toy.registry").end()`, emitindo e fechando o span em uma única linha limpa.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei a recomendação de Non-Functional Statement no CATSPY, simplificando a invocação do OpenTelemetry sem comandos inócuos.

### Registro R14 (#09 de Kawan Torres) — Non-Functional Statement
- **Responsável:** Kawan Torres
- **Data do Commit:** 27/10/2026 18:42
- **Commit:** `e482f87`
- **Arquivo e Teste:** `tests/test_telemetry_testing_kit.py::test_linked_root_span_kwargs_links_without_parenting`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Non-Functional Statement** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste usava `with toy_tracer.start_as_current_span("toyplugin.effect", **kwargs): pass` para criar o span de efeito com links sem corpo de bloco.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Substituição do bloco de contexto vazio por chamada direta `toy_tracer.start_span("toyplugin.effect", **kwargs).end()`, removendo o `pass`.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Non-Functional Statement, padronizando a criação de spans pontuais sem blocos de context manager vazios.

### Registro R15 (#10 de Kawan Torres) — Non-Functional Statement
- **Responsável:** Kawan Torres
- **Data do Commit:** 27/10/2026 18:43
- **Commit:** `21dcbcb`
- **Arquivo e Teste:** `tests/test_telemetry_testing_kit.py::test_linked_root_span_kwargs_with_no_current_span`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Non-Functional Statement** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste usava `with toy_tracer.start_as_current_span("toyplugin.orphanless", **kwargs): pass` para gerar um span órfão, com statement `pass`.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Substituição por `toy_tracer.start_span("toyplugin.orphanless", **kwargs).end()`, erradicando o `pass` e simplificando o teste.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Non-Functional Statement, concluindo o saneamento de context managers vazios na suíte de telemetria.

### Registro R17 (#11 de Kawan Torres) — Unknown Test
- **Responsável:** Kawan Torres
- **Data do Commit:** 27/10/2026 19:32
- **Commit:** `76bd74f`
- **Arquivo e Teste:** `tests/test_internals_datasette.py::test_static_rejects_plugin_and_mount`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Unknown Test** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste utilizava asserções indiretas que verificavam apenas que exceções genéricas eram capturadas sem inspecionar propriedades da mensagem ou tipo concreto.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Refatoração para asserções semânticas diretas sobre a resposta e estrutura de erro, garantindo validação explícita do comportamento de rejeição de plugins.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o smell Unknown Test no CATSPY (https://cat-spy.vercel.app/smells), que define o problema como testes sem asserções explícitas ou com asserções fracas que não atestam o comportamento correto.

### Registro R18 (#12 de Kawan Torres) — Unknown Test
- **Responsável:** Kawan Torres
- **Data do Commit:** 27/10/2026 19:34
- **Commit:** `895abb2`
- **Arquivo e Teste:** `tests/test_internals_datasette.py::test_static_rejects_path_traversal`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Unknown Test** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste verificava rejeição de path traversal mas realizava apenas asserções implícitas sobre o status HTTP sem verificar se o arquivo não foi exposto.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Reforço de asserções explícitas validando o status de erro e a ausência de vazamento de conteúdo sensível no corpo da resposta.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei as diretrizes do CATSPY para Unknown Test, adicionando asserções que validam explicitamente os efeitos esperados do caso de borda de segurança.

### Registro R19 (#13 de Kawan Torres) — Unknown Test
- **Responsável:** Kawan Torres
- **Data do Commit:** 27/10/2026 19:35
- **Commit:** `2128e34`
- **Arquivo e Teste:** `tests/test_internals_datasette.py::test_datasette_error_if_string_not_list`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Unknown Test** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste conferia o comportamento de erro de string não-lista sem asserir o payload exato da mensagem de erro.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Inclusão de asserção explícita sobre a estrutura do erro retornado pela API, tornando a validação precisa e determinística.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Unknown Test, assegurando asserções completas no teste unitário.

### Registro R22 (#14 de Kawan Torres) — Conditional Test Logic
- **Responsável:** Kawan Torres
- **Data do Commit:** 25/10/2026 19:21
- **Commit:** `436d129`
- **Arquivo e Teste:** `tests/test_utils_sql_analysis.py::test_analyze_create_table_operation`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste continha ramificações condicionais para validar diferentes tabelas analisadas dentro do mesmo método.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Eliminação dos branches internos por meio de asserções diretas nos dicionários de operações retornados, assegurando caminho de execução único.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o smell Conditional Test Logic no CATSPY (https://cat-spy.vercel.app/smells), que indica que testes com condicionais possuem múltiplos fluxos de execução, aumentando a probabilidade de asserções não serem executadas.

### Registro R23 (#15 de Kawan Torres) — Conditional Test Logic
- **Responsável:** Kawan Torres
- **Data do Commit:** 25/10/2026 19:55
- **Commit:** `3ec8c93`
- **Arquivo e Teste:** `tests/test_utils_sql_analysis.py::test_analyze_function_operation`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste `test_analyze_function_operation` continha laço/condicional para filtrar operações de função.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Substituição da lógica condicional por asserção direta nos metadados da operação de função capturada pelo analisador SQL.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic, aplicando asserções diretas sem caminhos alternativos no teste.

### Registro R24 (#16 de Kawan Torres) — Conditional Test Logic
- **Responsável:** Kawan Torres
- **Data do Commit:** 26/10/2026 18:55
- **Commit:** `a5bb8f5` (Refinamento Cícero Higor: `bae9e90`)
- **Arquivo e Teste:** `tests/test_utils_sql_analysis.py::test_analyze_table_kind_for_regular_virtual_and_shadow_tables`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste usava gerador com condicional `next(op for op in operations if op.operation == ...)` para localizar operações de inserção em diferentes tabelas.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Substituição do gerador com filtro condicional por desempacotamento determinístico direto `[regular_insert] = regular_analysis.operations`, `[virtual_insert] = virtual_analysis.operations` e `[shadow_insert] = shadow_analysis.operations`, garantindo tamanho exato e ausência de laços ou geradores condicionais.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic. Como geradores com cláusula `if` representam caminhos condicionais implícitos, o desempacotamento direto de listas de elemento único foi a solução ideal para garantir cardinalidade e caminho determinístico.

### Registro R25 (#17 de Kawan Torres) — Conditional Test Logic
- **Responsável:** Kawan Torres
- **Data do Commit:** 26/10/2026 15:58
- **Commit:** `4fa25ff`
- **Arquivo e Teste:** `tests/test_permissions.py::test_permissions_cascade`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste `test_permissions_cascade` continha condicionais aninhadas avaliando diferentes combinações de permissões.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Reestruturação do teste para validações sequenciais e determinísticas sem blocos condicionais de decisão.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei as estratégias de remoção de Conditional Test Logic no CATSPY, linearizando o fluxo de execução dos testes.

### Registro R26 (#18 de Kawan Torres) — Conditional Test Logic
- **Responsável:** Kawan Torres
- **Data do Commit:** 26/10/2026 20:29
- **Commit:** `cdcb9d8`
- **Arquivo e Teste:** `tests/test_permissions.py::test_view_padlock`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste avaliava a visibilidade do ícone de cadeado de permissão através de condicionais que variavam pelo ator autenticado.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Substituição das condicionais por verificação determinística e asserções explícitas de presença/ausência do fragmento HTML.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic, isolando as expectativas em asserções diretas sem condicionais.

### Registro R27 (#19 de Kawan Torres) — Conditional Test Logic
- **Responsável:** Kawan Torres
- **Data do Commit:** 26/10/2026 20:34
- **Commit:** `c46b63d`
- **Arquivo e Teste:** `tests/test_permissions.py::test_view_database`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste continha bifurcação condicional ao validar listas de bancos de dados para atores anônimos e autenticados.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Linearização do teste com asserções diretas para a resposta anônima e autenticada, sem if/else no corpo do teste.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic, eliminando desvios de fluxo no teste.

### Registro R28 (#20 de Kawan Torres) — Conditional Test Logic
- **Responsável:** Kawan Torres
- **Data do Commit:** 26/10/2026 21:08
- **Commit:** `c636a0c`
- **Arquivo e Teste:** `tests/test_permissions.py::test_database_list_respects_view_table`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste continha dois laços `for html_fragment in html_fragments:` iterando sobre fragmentos HTML de tabelas e views para validar sua ausência (anônimo) e presença (autenticado).

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Desenrolamento dos laços em 4 asserções literais diretas: `assert ">names</a> 🔒" not in anon_response_text`, `assert ">v</a> 🔒" not in anon_response_text`, e os respectivos `in` para a resposta autenticada, eliminando o laço iterativo de asserções.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic, que desaconselha laços de iteração em testes quando o conjunto de elementos é pequeno e fixo, favorecendo asserções pontuais e legíveis.

### Registro R29 (#21 de Kawan Torres) — Conditional Test Logic
- **Responsável:** Kawan Torres
- **Data do Commit:** 27/10/2026 09:27
- **Commit:** `6a07991`
- **Arquivo e Teste:** `tests/test_permissions.py::test_view_table`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste `test_view_table` continha lógica condicional `if/else` no corpo do teste para calcular se o ícone de cadeado (`🔒`) deveria estar presente ou ausente de acordo com o parâmetro de permissão.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Inclusão do parâmetro explícito `expected_padlock` (True/False) na tabela do `@pytest.mark.parametrize` e asserção direta `assert (">padlock</a> 🔒" in response.text) == expected_padlock`, erradicando a condicional interna.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei a recomendação do CATSPY para Conditional Test Logic sobre transformar bifurcações internas em colunas adicionais de testes parametrizados.

### Registro R30 (#22 de Kawan Torres) — Conditional Test Logic
- **Responsável:** Kawan Torres
- **Data do Commit:** 27/10/2026 09:38
- **Commit:** `f05ee43`
- **Arquivo e Teste:** `tests/test_permissions.py::test_table_list_respects_view_table`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste `test_table_list_respects_view_table` continha laços `for html_fragment in html_fragments:` para verificar a presença e ausência de cadeados em tabelas e views protegidas.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Desenrolamento dos laços em asserções literais individuais para `>compound_three_primary_keys</a> 🔒` e `>paginated_view</a> 🔒`, tornando as falhas imediatamente rastreáveis por linha específica.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic, aplicando desenrolamento de laço (loop unrolling) para asserções atômicas e claras.

### Registro R31 (#23 de Kawan Torres) — Conditional Test Logic
- **Responsável:** Kawan Torres
- **Data do Commit:** 27/10/2026 09:40
- **Commit:** `2cfe5b9`
- **Arquivo e Teste:** `tests/test_permissions.py::test_view_query`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste `test_view_query` realizava checagem condicional com `if` no corpo do teste para decidir se o fragmento do cadeado deveria ser checado.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Adição de `expected_padlock` no `@pytest.mark.parametrize` do teste, parametrizando a expectativa do cadeado junto com as respostas anônima e autenticada sem bifurcação de código.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o catálogo CATSPY para Conditional Test Logic, movendo a variação do teste inteiramente para a matriz de parâmetros do Pytest.

---

# Parte 2: Registros de Cícero Higor (22 refatorações)

### Registro R02 (#01 de Cícero Higor) — Sleepy Test
- **Responsável:** Cícero Higor
- **Data do Commit:** 27/10/2026 18:38
- **Commit:** `c9f0477`
- **Arquivo e Teste:** `tests/test_internals_database.py::test_execute_write_block_false`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Sleepy Test** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste executava uma escrita no banco de dados com `block=False` e aguardava com `await asyncio.sleep(0.1)` para a gravação persistir no SQLite.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Criação da função auxiliar assíncrona `wait_for_row_value(db, sql, expected, timeout=1.0, interval=0.005)` em `tests/test_internals_database.py`. O teste passou a fazer polling com micro-intervalo (5ms) até o valor desejado ser retornado ou estourar o timeout determinístico, eliminando o delay arbitrário de 100ms.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei as diretrizes de Sleepy Test no CATSPY. Como o catálogo foca em atrasos síncronos, adaptamos o princípio de polling determinístico com deadline para o loop de eventos assíncrono do asyncio.

### Registro R03 (#02 de Cícero Higor) — Sleepy Test
- **Responsável:** Cícero Higor
- **Data do Commit:** 27/10/2026 18:41
- **Commit:** `80bf184`
- **Arquivo e Teste:** `tests/test_internals_database.py::test_execute_write_with_returning_block_false`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Sleepy Test** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste `test_execute_write_with_returning_block_false` também usava `await asyncio.sleep(0.1)` após disparar escrita assíncrona não-bloqueante.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Reutilização do helper assíncrono `wait_for_row_value`, substituindo o `await asyncio.sleep(0.1)` por uma chamada determinística que aguarda a linha com valor 'Cleo' ser persistida.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY em busca de padrões para esperas determinísticas, reaproveitando a estratégia de polling ativo com timeout implementada no R02.

### Registro R04 (#03 de Cícero Higor) — Sleepy Test
- **Responsável:** Cícero Higor
- **Data do Commit:** 27/10/2026 18:45
- **Commit:** `c35b539`
- **Arquivo e Teste:** `tests/test_shutdown.py::test_shutdown_logs_stragglers_that_outlive_the_grace_period`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Sleepy Test** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste de shutdown usava `await asyncio.sleep(0.05)` para tentar dar tempo de a tarefa stubborn iniciar antes de disparar o cancelamento do lifespan.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Substituição do `await asyncio.sleep(0.05)` por um evento determinístico `task_started = asyncio.Event()`. A corrotina stubborn sinaliza `task_started.set()` logo ao entrar em execução, e o teste aguarda exatamente o momento de início com `await asyncio.wait_for(task_started.wait(), timeout=5)`.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o smell Sleepy Test no CATSPY e as recomendações de sincronização por eventos de conclusão/notificação em vez de pausas arbitrárias.

### Registro R05 (#04 de Cícero Higor) — Sleepy Test
- **Responsável:** Cícero Higor
- **Data do Commit:** 27/10/2026 18:50
- **Commit:** `19629a0`
- **Arquivo e Teste:** `tests/test_write_wrapper.py::test_track_event_with_block_false`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Sleepy Test** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste continha um laço com 50 repetições de `await asyncio.sleep(0.01)` aguardando a gravação de eventos assíncronos em `ds._tracked_events`.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Substituição do laço de espera por interceptação com `asyncio.Event()`. Foi definido `event_recorded = asyncio.Event()` e encapsulado `ds.track_event` para chamar `.set()` quando o evento for processado, aguardando deterministicamente com `await asyncio.wait_for(event_recorded.wait(), timeout=5)`.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei a definição de Sleepy Test no CATSPY, aplicando sincronização direta por notificação de evento assíncrono para eliminar esperas artificiais.

### Registro R09 (#05 de Cícero Higor) — Undefined Test
- **Responsável:** Cícero Higor
- **Data do Commit:** 28/10/2026 15:56
- **Commit:** `27c39ba`
- **Arquivo e Teste:** `tests/test_http_span.py::_route_for`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Undefined Test** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   A função auxiliar `_route_for` em `tests/test_internals_datasette.py` continha `assert match is not None` em um helper utilitário de resolução de rotas.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Substituição do `assert match is not None` por `if match is None: raise ValueError(...)`, transformando o helper em uma função utilitária limpa que lança exceção descritiva de domínio em caso de erro, deixando os asserts exclusivamente para os métodos de teste.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Undefined Test. A recomendação de substituir asserções internas em helpers por exceções de domínio evitou que o TEMPY interpretasse o utilitário como teste incompleto/sem asserção.

### Registro R16 (#06 de Cícero Higor) — Non-Functional Statement
- **Responsável:** Cícero Higor
- **Data do Commit:** 28/10/2026 15:55
- **Commit:** `f32875d`
- **Arquivo e Teste:** `tests/test_telemetry_metrics.py::test_registry_holds_instances_weakly`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Non-Functional Statement** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   A classe local `FakeDatasette` declarada dentro de `test_registry_holds_instances_weakly` continha apenas a declaração `pass`.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Substituição do `pass` por uma docstring descritiva `"""Stand-in object to test that registration does not hold strong references."""`, tornando a finalidade da classe auxiliar autoexplicativa e eliminando o comando nulo `pass`.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Non-Functional Statement, que adverte contra instruções que não afetam o comportamento do teste. Substituir o `pass` vazio por docstring documenta o teste e sana o smell.

### Registro R20 (#07 de Cícero Higor) — Unknown Test
- **Responsável:** Cícero Higor
- **Data do Commit:** 28/10/2026 15:54
- **Commit:** `fe44d16`
- **Arquivo e Teste:** `tests/test_template_context.py::test_from_extra_must_match_a_registered_extra`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Unknown Test** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste `test_from_extra_must_match_a_registered_extra` usava `with pytest.raises(KeyError):` sem validar o conteúdo da chave faltante, deixando a asserção incompleta (Unknown Test).

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Adição de asserção explícita na mensagem de erro com `assert excinfo.value.args[0] == "unregistered_extra"` logo após o bloco `pytest.raises`, garantindo que a exceção ocorreu exatamente pelo motivo esperado.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Unknown Test, que recomenda verificar detalhadamente a mensagem e os atributos da exceção capturada para garantir que ela não é um falso positivo acidental.

### Registro R21 (#08 de Cícero Higor) — Unknown Test
- **Responsável:** Cícero Higor
- **Data do Commit:** 28/10/2026 15:55
- **Commit:** `d838601`
- **Arquivo e Teste:** `tests/test_template_context.py::test_from_extra_must_be_available_for_the_scope`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Unknown Test** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste `test_from_extra_must_be_available_for_the_scope` capturava `KeyError` com `pytest.raises` mas não asseria qual escopo causou a falha.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Adição da asserção explícita `assert excinfo.value.args[0] == "database"` no objeto `excinfo`, atestando que a exceção foi lançada especificamente pelo escopo inválido.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei as orientações de Unknown Test no CATSPY, inspecionando detalhadamente a exceção capturada para certificar a causa da falha.

### Registro R32 (#09 de Cícero Higor) — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Data do Commit:** 27/10/2026 09:44
- **Commit:** `bcfffdba`
- **Arquivo e Teste:** `tests/test_permissions.py::test_view_instance`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste `test_view_instance` possuía lógica condicional interna `if expected_padlock:` para asserir sobre o texto da resposta.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Substituição do `if` pela asserção booleana direta `assert (">fixtures</a> 🔒" in response.text) == expected_padlock`, eliminando o desvio condicional.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic, substituindo a ramificação condicional por asserção direta de igualdade booleana.

### Registro R33 (#10 de Cícero Higor) — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Data do Commit:** 28/10/2026 15:57
- **Commit:** `9d09b07`
- **Arquivo e Teste:** `tests/test_template_context.py::test_context_class_fields_all_have_help`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste `test_context_class_fields_all_have_help` continha um laço `for context_field in klass.documented_fields(): assert context_field.help` dentro de um teste já parametrizado por classe.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Extração da validação para a função auxiliar `fields_missing_help(klass)` que filtra campos sem documentação, permitindo ao teste asserir diretamente `assert fields_missing_help(klass) == []`. Se houver campo sem ajuda, a mensagem do teste exibe exatamente a lista dos campos faltantes.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei a recomendação do CATSPY para Conditional Test Logic sobre extrair verificação agregada para funções auxiliares de coleção, mantendo o método de teste limpo com uma única asserção clara.

### Registro R34 (#11 de Cícero Higor) — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Data do Commit:** 28/10/2026 16:00
- **Commit:** `d992205`
- **Arquivo e Teste:** `tests/test_routes.py::test_routes`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste `test_routes` (já parametrizado) continha uma bifurcação condicional morta `if expected_name is None: assert match is None else: ...`, sendo que nenhum dos parâmetros passava `expected_name=None`.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Remoção do bloco `if/else`, substituindo-o por asserções diretas e lineares: `assert match is not None` seguido das verificações de nome da view e dicionário de grupos.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic, que orienta remover código morto ou condicionais desnecessárias que criam caminhos de execução artificiais em testes.

### Registro R35 (#12 de Cícero Higor) — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Data do Commit:** 28/10/2026 16:00
- **Commit:** `3e1d0b7`
- **Arquivo e Teste:** `tests/test_tracer.py::test_trace`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste `test_trace` recebia o parâmetro booleano `trace_debug` e continha a bifurcação `if not trace_debug: assert "_trace" not in data; return`, executando verificações completamente distintas para True e False dentro do mesmo corpo.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Divisão do teste em dois métodos independentes focados (Split Test): `test_trace_disabled()`, que valida a ausência de `_trace` quando a funcionalidade está desativada, e `test_trace_enabled()`, que valida o payload completo quando a depuração está ativada.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei a estratégia de refatoração 'Split Test' recomendada pelo CATSPY para Conditional Test Logic, separando cenários mutuamente exclusivos com asserções distintas em métodos de teste dedicados.

### Registro R36 (#13 de Cícero Higor) — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Data do Commit:** 28/10/2026 16:01
- **Commit:** `28671b2`
- **Arquivo e Teste:** `tests/test_tracer.py::test_trace_enabled`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O método `test_trace_enabled` (gerado na divisão do R35) continha um laço `for trace in traces:` que realizava 8 asserções sobre cada entrada de rastreamento.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Extração do laço de validação para a função utilitária `assert_valid_trace_entries(traces)`, mantendo a verificação estrutural completa das entradas de trace sem poluir o corpo principal do teste com laço iterativo.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei as diretrizes do CATSPY para Conditional Test Logic, extraindo asserções iterativas complexas para funções de validação bem nomeadas.

### Registro R37 (#14 de Cícero Higor) — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Data do Commit:** 28/10/2026 16:02
- **Commit:** `9e1223f`
- **Arquivo e Teste:** `tests/test_load_extensions.py::test_load_extension_is_disabled`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste `test_load_extension_is_disabled` continha condicional `if/else` testando alternadamente o carregamento síncrono e assíncrono.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Linearização do teste com asserções diretas sobre o comportamento esperado de bloqueio de extensão.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic, simplificando as asserções para um único fluxo determinístico.

### Registro R38 (#15 de Cícero Higor) — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Data do Commit:** 28/10/2026 16:03
- **Commit:** `c9e1824`
- **Arquivo e Teste:** `tests/test_internals_request.py::test_request_args`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste continha condicionais para tratar casos com e sem argumentos na requisição HTTP.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Parametrização do teste para cobrir os diferentes conjuntos de argumentos sem condicionais dentro do método de teste.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei a técnica de parametrização no CATSPY para substituir ramificações condicionais.

### Registro R39 (#16 de Cícero Higor) — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Data do Commit:** 29/10/2026 14:39
- **Commit:** `7d5501b`
- **Arquivo e Teste:** `tests/test_table_api.py::test_nofacet`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste `test_nofacet` usava `@pytest.mark.parametrize("nofacet", (True, False))` e continha duas condicionais internas: `if nofacet: path += "&_nofacet=1"` e `if nofacet: assert ... == [] else: assert ... != []`.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Substituição do booleano e das condicionais pela parametrização direta das tuplas de parâmetros e expectativas: `("&_nofacet=1", False, False)` e `("", True, True)` nas variáveis `query_param, has_suggested_facets, has_facet_results`, eliminando completamente ambos os `if`s do teste.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic, aplicando parametrização completa de dados de entrada e saídas esperadas para erradicar bifurcações em testes parametrizados.

### Registro R40 (#17 de Cícero Higor) — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Data do Commit:** 29/10/2026 14:39
- **Commit:** `7f3ed24`
- **Arquivo e Teste:** `tests/test_table_api.py::test_nosuggest`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste `test_nosuggest` usava `@pytest.mark.parametrize("nosuggest", (True, False))` e continha concatenação condicional `if nosuggest: path += ...` e asserção bifurcada `if nosuggest: ... else: ...`.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Parametrização direta de `query_param, has_suggested_facets` com `("&_nosuggest=1", False)` e `("", True)`, com asserção booleana direta `assert (len(data["suggested_facets"]) > 0) is has_suggested_facets`, removendo todas as condicionais.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o catálogo CATSPY para Conditional Test Logic, mapeando as variações da query string e as asserções booleanas na tabela de parametrização.

### Registro R41 (#18 de Cícero Higor) — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Data do Commit:** 29/10/2026 14:40
- **Commit:** `9547bc5`
- **Arquivo e Teste:** `tests/test_table_api.py::test_nocount`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste `test_nocount` usava `if nocount: path += "&_nocount=1"` dentro do teste parametrizado.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Parametrização direta de `query_param, expected_count` com `("&_nocount=1", None)` e `("", 15)`, eliminando o bloco `if nocount:` na montagem da URL.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic, eliminando a condicional de construção de parâmetros através de parametrização limpa.

### Registro R42 (#19 de Cícero Higor) — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Data do Commit:** 29/10/2026 15:25
- **Commit:** `90f8bad` (Ajuste de estilo PEP 8: `93280f5`)
- **Arquivo e Teste:** `tests/test_csv.py::test_custom_sql_csv_errors`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste `test_custom_sql_csv_errors` continha laço/condicional validando múltiplos cenários de erro de consulta customizada de CSV.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Parametrização do teste por cenário de erro e asserção direta da mensagem de erro retornada, sem desvios condicionais.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic, aplicando `@pytest.mark.parametrize` para dividir os casos de erro em execuções atômicas isoladas.

### Registro R43 (#20 de Cícero Higor) — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Data do Commit:** 29/10/2026 15:26
- **Commit:** `6b81b03`
- **Arquivo e Teste:** `tests/test_table_api.py::test_binary_data_in_json`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste `test_binary_data_in_json` continha condicional tratando diferentes formatos de representação binária dentro do mesmo método.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Divisão do teste em dois métodos de teste atômicos (Split Test), cada um cobrindo uma variante específica de codificação binária.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei a técnica de Split Test do CATSPY para Conditional Test Logic, separando os caminhos alternativos em testes dedicados.

### Registro R44 (#21 de Cícero Higor) — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Data do Commit:** 29/10/2026 15:27
- **Commit:** `6490ccc`
- **Arquivo e Teste:** `tests/test_fts_permissions.py::test_external_content_fts_inherits_content_table_view_permission`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste `test_external_content_fts_inherits_content_table_view_permission` possuía lógica condicional `if/else` ao validar permissões de FTS e tabelas de conteúdo externo.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Linearização do fluxo com asserções explícitas e sequenciais para cada nível de permissão avaliado, sem desvios de execução.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic, eliminando a lógica de decisão dentro do teste de permissão.

### Registro R45 (#22 de Cícero Higor) — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Data do Commit:** 29/10/2026 15:28
- **Commit:** `977a793`
- **Arquivo e Teste:** `tests/test_queries.py::test_execute_write_schema_operations_use_schema_permissions`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste `test_execute_write_schema_operations_use_schema_permissions` possuía lógica condicional interna para tratar diferentes operações de esquema do SQLite.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Refatoração para asserções diretas e parametrização explícita das operações, garantindo que cada caso de teste execute um caminho único e determinístico.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o catálogo CATSPY para Conditional Test Logic, assegurando que o último teste refatorado da suíte tivesse caminho único e determinístico.
