# Diário de Bordo Unificado — Test Smells em Projetos OpenSource
## Universidade Federal do Ceará — Qualidade de Software

- **Disciplina:** Qualidade de Software (Semestre 2026.2)
- **Docente:** Profª. Dra. Carla Ilane Moreira Bezerra
- **Monitoria:** Gabriel Alves Pessoa
- **Equipe:**
  - Cícero Higor (login/git: `ocarnauba_`)
  - Kawan Torres (login/git: `kawantorres`)
- **Projeto:** [simonw/datasette](https://github.com/simonw/datasette)
- **Fork:** [cicerocarnauba/datasette](https://github.com/cicerocarnauba/datasette)
- **Commit Base Inicial:** `cf113a5` (`cf113a59850862125b86ce1a58c117aa21341e07`)
- **Commit Final de Refatoração de Código:** `746720f` (`746720f83737b5c8795da3cb0a542b8faae2f46b`)
- **Branch de Trabalho:** `refatoracao-test-smells`
- **Link Oficial no Google Docs da Disciplina:** https://docs.google.com/document/d/1jvLB-C7dT7S-WHd-vc8-YLf6qjfu3JX3XeOdKnqxBc8/edit?usp=sharing
- **Catálogo de Referência Oficial:** [CATSPY](https://cat-spy.vercel.app/smells) (UFC Quixadá)

---

## 📊 Resumo Executivo das Refatorações da Equipe (45 Registros | 47 Commits)

| Integrante | Smells Refatorados (CATSPY) | Qtd. Registros | Commits | Arquivos Atribuídos |
|---|---|---|---|---|
| **Kawan Torres** | Sleepy Test (1)<br>Undefined Test (3)<br>Non-Functional Statement (6)<br>Unknown Test (3)<br>Conditional Test Logic (3) | **16 registros** | `9bce5bb` a `bfe137a` (16 commits) | `tests/test_cli_serve_server.py`<br>`tests/test_queries.py`<br>`tests/test_telemetry_testing_kit.py`<br>`tests/test_internals_datasette.py`<br>`tests/test_utils_sql_analysis.py` |
| **Cícero Higor** | Sleepy Test (4)<br>Undefined Test (1)<br>Non-Functional Statement (1)<br>Unknown Test (2)<br>Conditional Test Logic (21) | **29 registros** | `8b7144b` a `746720f` (31 commits)* | `tests/test_internals_database.py`<br>`tests/test_shutdown.py`<br>`tests/test_write_wrapper.py`<br>`tests/test_http_span.py`<br>`tests/test_telemetry_metrics.py`<br>`tests/test_template_context.py`<br>`tests/test_permissions.py`<br>`tests/test_routes.py`<br>`tests/test_tracer.py`<br>`tests/test_load_extensions.py`<br>`tests/test_internals_request.py`<br>`tests/test_table_api.py`<br>`tests/test_csv.py`<br>`tests/test_fts_permissions.py` |
| **Total Dupla** | **5 tipos distintos do CATSPY** | **45 registros** | **47 commits atômicos** | **18 arquivos de teste independentes** |

*\*Nota de auditoria de commits:* Cícero Higor realizou 29 refatorações atômicas de testes e 2 commits complementares de refinamento (`3172976` para desempacotamento determinístico em R24 e `92ae493` para validação de mensagem em Unknown Test R20/R21), totalizando 31 commits. A equipe descartou refatorações de *Exception Handling* por ausência desse smell no catálogo de referência CATSPY.

---

# Parte 1: Registros de Refatoração — Kawan Torres (Registros #01 a #16)

---

### Registro #01 — Sleepy Test
- **Responsável:** Kawan Torres
- **Commit:** `9bce5bb`
- **Arquivo e Teste:** `tests/test_cli_serve_server.py::test_startup_hook_background_task_runs_on_serving_loop` (linha 107)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Sleepy Test. O teste continha uma chamada a `time.sleep(0.05)` dentro de um laço de polling com deadline para aguardar que uma tarefa de inicialização em segundo plano executasse no servidor ASGI, tornando a suíte de testes artificialmente lenta.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Desacoplar a lógica de espera ativa da função de teste sem perder o tempo de tolerância (deadline de 3.0s) necessário para a tarefa assíncrona registrar sua conclusão via HTTP.

3. **Usei o CATSPY para:**
   Consultar a seção de Sleepy Test no CATSPY sobre o impacto de pausas e técnicas para delegar polling ativo para funções utilitárias dedicadas. Link: https://cat-spy.vercel.app/smells/Sleepy%20Test

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Extração da rotina de consulta com timeout para a função auxiliar `poll_for_marker_task(port, timeout=3.0)`, removendo a chamada direta de `time.sleep` do corpo do teste e simplificando o método para `payload = poll_for_marker_task(port)`.

---

### Registro #02 — Undefined Test
- **Responsável:** Kawan Torres
- **Commit:** `7a4c7a2`
- **Arquivo e Teste:** `tests/test_queries.py::_template_option_attributes` (linhas 18-20)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Undefined Test. A função utilitária `_template_option_attributes` continha asserções (`assert`) em seu corpo, mas não possui o prefixo canônico `test_` e não é reconhecida como um teste pelo framework pytest, configurando asserções em método utilitário.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Garantir a integridade da extração de atributos HTML de elementos `<option>` sem depender de `assert` para controle de fluxo e sem quebrar os múltiplos testes que consomem essa função.

3. **Usei o CATSPY para:**
   Consultar a definição de Undefined Test no CATSPY sobre como helpers não devem conter declarações de teste soltas, devendo lançar exceções canônicas de validação. Link: https://cat-spy.vercel.app/smells/Undefined%20Test

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Substituição das instruções `assert` por validações explícitas com lançamento de `raise ValueError(...)`.

---

### Registro #03 — Undefined Test
- **Responsável:** Kawan Torres
- **Commit:** `aca7a06`
- **Arquivo e Teste:** `tests/test_queries.py::_template_sql` (linhas 31-33)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Undefined Test. A função auxiliar `_template_sql` utilizava `assert` internamente para validar se o botão de template SQL continha o atributo `data-template-sql`.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Diferenciar falhas de extração do elemento DOM de falhas de asserção de teste, assegurando que o utilitário retorne a string SQL ou falhe explicitamente.

3. **Usei o CATSPY para:**
   Consultar as diretrizes do CATSPY para a refatoração de Undefined Test, focando em substituir asserções internas por exceções de controle explícitas. Link: https://cat-spy.vercel.app/smells/Undefined%20Test

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Substituição dos `assert` pela verificação condicional defensiva lançando `raise ValueError(f"Could not find SQL for {operation} template button")`.

---

### Registro #04 — Undefined Test
- **Responsável:** Kawan Torres
- **Commit:** `86fd342`
- **Arquivo e Teste:** `tests/test_queries.py::_template_button_sql` (linhas 39-42)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Undefined Test. A função helper `_template_button_sql` realizava asserções com `assert button` e `assert button.get(...)` sem ser um teste formal da suíte.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Assegurar que a busca pelo botão de template via BeautifulSoup retorne o atributo correto com tratamento de ausência robusto.

3. **Usei o CATSPY para:**
   Consultar a definição de Undefined Test no CATSPY e as orientações para transformar checagens informais em tratamento de erro em Python. Link: https://cat-spy.vercel.app/smells/Undefined%20Test

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Conversão dos blocos `assert` em verificações explícitas `if not button: raise ValueError(...)`, eliminando o smell e mantendo a função estritamente utilitária.

---

### Registro #05 — Non-Functional Statement
- **Responsável:** Kawan Torres
- **Commit:** `475ffb1`
- **Arquivo e Teste:** `tests/test_cli_serve_server.py::test_startup_error_fails_fast_before_port_binds` (linhas 150-154)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Non-Functional Statement. O teste utilizava um context manager composto com parênteses contendo uma instrução inerte `pass` no corpo do bloco `with`.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Reorganizar o context manager para que a chamada de conexão com socket ocorra como corpo efetivo do context manager `pytest.raises(OSError)`, sem manter comandos supérfluos.

3. **Usei o CATSPY para:**
   Consultar a seção de Non-Functional Statement no CATSPY sobre instruções redundantes e blocos inertes em testes. Link: https://cat-spy.vercel.app/smells/Non-Functional%20Statement

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Simplificação do context manager para `with pytest.raises(OSError): socket.create_connection(("127.0.0.1", port), timeout=0.2)`, eliminando a instrução `pass`.

---

### Registro #06 — Non-Functional Statement
- **Responsável:** Kawan Torres
- **Commit:** `a116702`
- **Arquivo e Teste:** `tests/test_telemetry_testing_kit.py::_run_workload` (linha 55)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Non-Functional Statement. A função utilitária `_run_workload` continha uma instrução inerte `pass` dentro do context manager de tracer.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Garantir a abertura e fechamento do span de telemetria sem manter uma instrução inerte.

3. **Usei o CATSPY para:**
   Consultar a definição de Non-Functional Statement no CATSPY para remoção de declarações inertes. Link: https://cat-spy.vercel.app/smells/Non-Functional%20Statement

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Substituição da instrução `pass` por atribuição descritiva mínima de identificador contextual.

---

### Registro #07 — Non-Functional Statement
- **Responsável:** Kawan Torres
- **Commit:** `b5603f9`
- **Arquivo e Teste:** `tests/test_telemetry_testing_kit.py::test_conformance_catches_an_unregistered_span` (linha 69)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Non-Functional Statement. O teste de conformidade de spans continha um bloco com a declaração `pass` isolada dentro do context manager de span não registrado.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Manter a geração do span de telemetria para verificação de conformidade sem deixar o corpo do bloco `with` vazio.

3. **Usei o CATSPY para:**
   Consultar estratégias para refatoração de Non-Functional Statement no CATSPY. Link: https://cat-spy.vercel.app/smells/Non-Functional%20Statement

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Eliminação do comando `pass` inerte através da reestruturação do escopo do context manager.

---

### Registro #08 — Non-Functional Statement
- **Responsável:** Kawan Torres
- **Commit:** `ccdb305`
- **Arquivo e Teste:** `tests/test_telemetry_testing_kit.py::test_scope_filter_ignores_other_scopes` (linha 94)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Non-Functional Statement. Presença de `pass` inerte dentro do context manager de teste de filtro de escopo de telemetria.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Remover o `pass` mantendo o acionamento do span para o escopo sob validação.

3. **Usei o CATSPY para:**
   Verificar recomendações do CATSPY sobre clareza e remoção de código morto em testes. Link: https://cat-spy.vercel.app/smells/Non-Functional%20Statement

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Remoção da instrução `pass` e compactação do corpo do context manager.

---

### Registro #09 — Non-Functional Statement
- **Responsável:** Kawan Torres
- **Commit:** `2d896ac`
- **Arquivo e Teste:** `tests/test_telemetry_testing_kit.py::test_linked_root_span_kwargs_links_without_parenting` (linha 115)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Non-Functional Statement. O teste utilizava `pass` dentro de um context manager de span raiz vinculado.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Garantir a execução da vinculação de spans sem instruções supérfluas.

3. **Usei o CATSPY para:**
   Consultar a documentação do CATSPY sobre eliminação de declarações inertes. Link: https://cat-spy.vercel.app/smells/Non-Functional%20Statement

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Substituição do `pass` por atribuição explícita de identificador de span.

---

### Registro #10 — Non-Functional Statement
- **Responsável:** Kawan Torres
- **Commit:** `0781592`
- **Arquivo e Teste:** `tests/test_telemetry_testing_kit.py::test_linked_root_span_kwargs_with_no_current_span` (linha 134)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Non-Functional Statement. Uso de declaração inerte `pass` em bloco de teste de span sem span pai corrente.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Estruturar o teste de forma compacta e funcional sem declarações inúteis.

3. **Usei o CATSPY para:**
   Consultar o catálogo CATSPY para eliminação de Non-Functional Statements. Link: https://cat-spy.vercel.app/smells/Non-Functional%20Statement

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Remoção definitiva do `pass`, encerrando as ocorrências desse smell no arquivo.

---

### Registro #11 — Unknown Test
- **Responsável:** Kawan Torres
- **Commit:** `4c73f6e`
- **Arquivo e Teste:** `tests/test_internals_datasette.py::test_static_rejects_plugin_and_mount` (linhas 150-154)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Unknown Test. O teste verificava a rejeição de argumentos conflitantes usando apenas `with pytest.raises(ValueError):` sem asserção explícita (`assert`).

2. **Minhas principais dificuldades na remoção do code smell são:**
   Capturar o contexto da exceção e verificar as propriedades de erro.

3. **Usei o CATSPY para:**
   Consultar a seção de Unknown Test no CATSPY sobre a necessidade de asserções explícitas. Link: https://cat-spy.vercel.app/smells/Unknown%20Test

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Captura do contexto da exceção com `with pytest.raises(ValueError) as excinfo:` e adição da asserção explícita sobre a mensagem de erro.

---

### Registro #12 — Unknown Test
- **Responsável:** Kawan Torres
- **Commit:** `7531827`
- **Arquivo e Teste:** `tests/test_internals_datasette.py::test_static_rejects_path_traversal` (linhas 159-163)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Unknown Test. O teste usava o argumento `match` em `pytest.raises` sem uma declaração `assert` explícita no corpo da função.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Tornar a validação da exceção explícita no corpo do teste.

3. **Usei o CATSPY para:**
   Consultar a definição de Unknown Test no CATSPY sobre asserções explícitas para rastreabilidade de defeitos. Link: https://cat-spy.vercel.app/smells/Unknown%20Test

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Captura com `as excinfo` e inclusão da asserção explícita `assert "cannot escape static root" in str(excinfo.value)`.

---

### Registro #13 — Unknown Test
- **Responsável:** Kawan Torres
- **Commit:** `c8889c1`
- **Arquivo e Teste:** `tests/test_internals_datasette.py::test_datasette_error_if_string_not_list` (linhas 272-276)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Unknown Test. O teste validava erro de inicialização com string apenas via context manager de exceção, sem asserção explícita.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Garantir a verificação exata da mensagem de validação de argumentos da classe Datasette.

3. **Usei o CATSPY para:**
   Verificar recomendações do CATSPY para Unknown Test. Link: https://cat-spy.vercel.app/smells/Unknown%20Test

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Adição de asserção explícita `assert "files= must be a list of paths, not a string" in str(excinfo.value)`.

---

### Registro #14 — Conditional Test Logic
- **Responsável:** Kawan Torres
- **Commit:** `040ae2c`
- **Arquivo e Teste:** `tests/test_utils_sql_analysis.py::test_analyze_create_table_operation` (linhas 124-125)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Conditional Test Logic. O teste continha estruturas de controle e filtragem (`for` e `if`) embutidas na asserção final para validar se nenhuma operação não interna afetou as tabelas de catálogo.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Isolar o predicado de filtragem sem alterar a verificação semântica do teste de análise SQL.

3. **Usei o CATSPY para:**
   Consultar a seção de Conditional Test Logic no CATSPY sobre como estruturas de controle em testes obscurecem a intenção da asserção. Link: https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Extração da lógica de compreensão e filtragem para a função auxiliar declarativa `non_internal_system_table_operations(analysis)`, simplificando a asserção no corpo do teste para `assert not non_internal_system_table_operations(analysis)`.

---

### Registro #15 — Conditional Test Logic
- **Responsável:** Kawan Torres
- **Commit:** `3e2bcc8`
- **Arquivo e Teste:** `tests/test_utils_sql_analysis.py::test_analyze_function_operation` (linha 224)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Conditional Test Logic. O teste utilizava um loop `for` na compreensão de conjunto para construir tuplas de operações e comparar com o conjunto esperado.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Preservar a correspondência exata de tuplas de 5 elementos sem manter compreensões com controle de iteração no corpo do teste.

3. **Usei o CATSPY para:**
   Consultar práticas de Conditional Test Logic no CATSPY para extração de processamento de tuplas. Link: https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Extração da construção do conjunto para a função auxiliar declarativa `function_operation_tuples(analysis)`, tornando a asserção linear e direta.

---

### Registro #16 — Conditional Test Logic
- **Responsável:** Kawan Torres (Refino Cícero Higor: `3172976`)
- **Commit:** `bfe137a` (Refino: `3172976`)
- **Arquivo e Teste:** `tests/test_utils_sql_analysis.py::test_analyze_table_kind_for_regular_virtual_and_shadow_tables` (linhas 283-297)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Conditional Test Logic. O teste continha expressões geradoras consecutivas com `for` e `if` em chamadas `next(...)` para localizar operações de inserção em três tabelas.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Eliminar os geradores com filtros condicionais de forma genuína, sem apenas transferir a condicional para um helper intermediário.

3. **Usei o CATSPY para:**
   Consultar a seção de Conditional Test Logic no CATSPY sobre linearidade em asserções de teste. Link: https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Inicialmente extraído para helper e, no refinamento definitivo `3172976`, substituído por desempacotamento direto determinístico `[regular_insert] = regular_analysis.operations`, eliminando 100% de qualquer lógica condicional ou geradora.

---

# Parte 2: Registros de Refatoração — Cícero Higor (Registros #17 a #45)

---

### Registro #17 — Sleepy Test
- **Responsável:** Cícero Higor
- **Commit:** `8b7144b`
- **Arquivo e Teste:** `tests/test_internals_database.py::test_execute_write_block_false` (linha 115)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Sleepy Test. O teste assíncrono continha uma chamada a `asyncio.sleep(0.1)` com pausa arbitrária para aguardar a execução de uma escrita em segundo plano na fila do SQLite.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Substituir o delay fixo por sincronização determinística sem depender de tempos de espera empíricos.

3. **Usei o CATSPY para:**
   Consultar a seção de Sleepy Test no CATSPY para compreender estratégias de sincronização baseadas em eventos e awaiting determinístico. Link: https://cat-spy.vercel.app/smells/Sleepy%20Test

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Substituição do delay arbitrário pelo aguardo determinístico do objeto `asyncio.Future` da tarefa de escrita.

---

### Registro #18 — Sleepy Test
- **Responsável:** Cícero Higor
- **Commit:** `d059a5c`
- **Arquivo e Teste:** `tests/test_internals_database.py::test_execute_write_with_returning_block_false` (linha 142)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Sleepy Test. Uso de `asyncio.sleep(0.1)` arbitrário para aguardar o retorno de uma escrita SQLite assíncrona com cláusula `RETURNING`.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Garantir a captura determinística do resultado retornado pela transação sem introduzir esperas temporais cegas.

3. **Usei o CATSPY para:**
   Verificar recomendações do CATSPY para erradicação de Sleepy Test em fluxos assíncronos. Link: https://cat-spy.vercel.app/smells/Sleepy%20Test

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Eliminação do `asyncio.sleep` através da resolução determinística via awaiting da future submetida à thread de escrita.

---

### Registro #19 — Sleepy Test
- **Responsável:** Cícero Higor
- **Commit:** `d8b827c`
- **Arquivo e Teste:** `tests/test_shutdown.py::test_shutdown_logs_stragglers_that_outlive_the_grace_period` (linhas 45-50)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Sleepy Test. O teste utilizava `asyncio.sleep` com valores arbitrários para forçar tarefas a ultrapassarem o período de carência (*grace period*) de encerramento do servidor.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Coordenar o ciclo de vida das tarefas pendentes sem depender de timeouts fixos extensos.

3. **Usei o CATSPY para:**
   Consultar orientações do CATSPY sobre testes de encerramento e tolerância temporal. Link: https://cat-spy.vercel.app/smells/Sleepy%20Test

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Ajuste dos tempos limites e uso de sinalizadores de eventos assíncronos (`asyncio.Event`) para desbloqueio coordenado durante o shutdown.

---

### Registro #20 — Sleepy Test
- **Responsável:** Cícero Higor
- **Commit:** `9533ed8`
- **Arquivo e Teste:** `tests/test_write_wrapper.py::test_track_event_with_block_false` (linha 62)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Sleepy Test. Uso de delay arbitrário para aguardar que um evento de telemetria fosse gravado de forma não bloqueante.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Validar que o evento foi processado sem atrasar desnecessariamente o encerramento do teste.

3. **Usei o CATSPY para:**
   Consultar estratégias de refatoração para Sleepy Test. Link: https://cat-spy.vercel.app/smells/Sleepy%20Test

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Substituição do delay por polling com timeout agressivo e verificação ativa do buffer de eventos gravados.

---

### Registro #21 — Undefined Test
- **Responsável:** Cícero Higor
- **Commit:** `7daa8b0`
- **Arquivo e Teste:** `tests/test_http_span.py::_route_for` (linhas 25-28)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Undefined Test. A função utilitária interna `_route_for` continha declarações `assert` em seu corpo, gerando asserções fora de métodos formais de teste.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Remover as asserções sem perder a garantia de que as rotas HTTP de telemetria foram resolvidas corretamente.

3. **Usei o CATSPY para:**
   Consultar no CATSPY a estratégia de conversão de asserções em helpers para tratamento idiomático de exceções. Link: https://cat-spy.vercel.app/smells/Undefined%20Test

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Substituição de `assert` por lançamento formal de `raise ValueError(f"Route for {path} not found")`.

---

### Registro #22 — Non-Functional Statement
- **Responsável:** Cícero Higor
- **Commit:** `4e944db`
- **Arquivo e Teste:** `tests/test_telemetry_metrics.py::test_registry_holds_instances_weakly` (linha 78)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Non-Functional Statement. O teste utilizava uma instrução inerte `pass` dentro de um context manager de supressão de referências fracas.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Garantir a coleta de lixo pelo garbage collector sem manter declarações inúteis no código.

3. **Usei o CATSPY para:**
   Consultar a seção de Non-Functional Statement no CATSPY sobre simplificação de blocos inertes. Link: https://cat-spy.vercel.app/smells/Non-Functional%20Statement

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Remoção limpa da instrução `pass` e estruturação compacta da liberação de referências.

---

### Registro #23 — Unknown Test
- **Responsável:** Cícero Higor
- **Commit:** `667b9c5` (Refino: `92ae493`)
- **Arquivo e Teste:** `tests/test_template_context.py::test_from_extra_must_match_a_registered_extra` (linha 82)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Unknown Test. O teste usava apenas `with pytest.raises(KeyError): BadContext.documented_fields()` sem nenhuma declaração `assert` explícita.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Capturar o contexto de erro e assegurar que a mensagem do erro corresponda exatamente à regra de extras não registrados.

3. **Usei o CATSPY para:**
   Consultar a definição de Unknown Test no CATSPY sobre como asserções explícitas de mensagem garantem a intenção do teste. Link: https://cat-spy.vercel.app/smells/Unknown%20Test

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Captura com `as excinfo` e inclusão da asserção explícita `assert "no registered extra of that name" in str(excinfo.value)`.

---

### Registro #24 — Unknown Test
- **Responsável:** Cícero Higor
- **Commit:** `4fdd31f` (Refino: `92ae493`)
- **Arquivo e Teste:** `tests/test_template_context.py::test_from_extra_must_be_available_for_the_scope` (linha 97)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Unknown Test. O teste validava escopo incorreto apenas com `with pytest.raises(ValueError):` sem asserção explícita.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Inspecionar o erro de escopo de template e adicionar asserção legível.

3. **Usei o CATSPY para:**
   Verificar recomendações do CATSPY para Unknown Test. Link: https://cat-spy.vercel.app/smells/Unknown%20Test

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Adição de asserção explícita `assert "is not available for scope" in str(excinfo.value)`.

---

### Registro #25 — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Commit:** `167c563`
- **Arquivo e Teste:** `tests/test_permissions.py::test_permissions_cascade` (linhas 110-120)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Conditional Test Logic. O teste continha um laço `for` com condicionais internas para iterar sobre múltiplos níveis de permissão em cascata.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Separar os níveis de cascata sem duplicar o setup complexo de permissões da fixture.

3. **Usei o CATSPY para:**
   Consultar a recomendação do CATSPY de aplicar parametrização (`@pytest.mark.parametrize`) para decompor loops de teste em casos independentes. Link: https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Conversão do loop de validação em casos parametrizados declarativos com `@pytest.mark.parametrize`.

---

### Registro #26 — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Commit:** `5cd9f1a`
- **Arquivo e Teste:** `tests/test_permissions.py::test_view_padlock` (linhas 145-155)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Conditional Test Logic. O teste utilizava ramificações condicionais para validar a exibição de ícones de cadeado em tabelas protegidas.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Isolar os cenários de visibilidade de cadeado em entradas tabulares limpas.

3. **Usei o CATSPY para:**
   Consultar o catálogo CATSPY para refatoração de Conditional Test Logic. Link: https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Parametrização direta dos pares `(permission_setting, expected_padlock)` via pytest.

---

### Registro #27 — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Commit:** `28bd370`
- **Arquivo e Teste:** `tests/test_permissions.py::test_view_database` (linhas 180-195)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Conditional Test Logic. Presença de branches `if/else` no corpo do teste para verificar respostas de acesso ao banco com diferentes atores autenticados.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Tratar atores anônimos e autenticados sem lógica condicional bifurcada no teste.

3. **Usei o CATSPY para:**
   Consultar a técnica de *Split Test* e *Parametrize* no CATSPY. Link: https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Decomposição da verificação em parâmetros de entrada com status HTTP esperado por perfil de ator.

---

### Registro #28 — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Commit:** `d5df3e9`
- **Arquivo e Teste:** `tests/test_permissions.py::test_database_list_respects_view_table` (linhas 230-240)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Conditional Test Logic. Laço iterativo sobre a lista de bancos para checagem condicional de visibilidade de tabelas.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Validar a visibilidade de múltiplas tabelas em uma única asserção declarativa.

3. **Usei o CATSPY para:**
   Consultar técnicas de asserção de coleções no CATSPY. Link: https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Substituição do loop condicional por comparação direta de conjuntos de nomes esperados.

---

### Registro #29 — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Commit:** `3c3ede7`
- **Arquivo e Teste:** `tests/test_permissions.py::test_view_table` (linhas 260-275)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Conditional Test Logic. Verificações de permissão de visualização de tabela estruturadas com condicionais sobre tipos de resposta.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Garantir a cobertura dos branches de permissão sem código procedural.

3. **Usei o CATSPY para:**
   Consultar o catálogo CATSPY para Conditional Test Logic. Link: https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Parametrização declarativa do conjunto de testes.

---

### Registro #30 — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Commit:** `9942a6b`
- **Arquivo e Teste:** `tests/test_permissions.py::test_table_list_respects_view_table` (linhas 300-315)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Conditional Test Logic. Loop iterativo com filtragem condicional para inspecionar itens da lista de tabelas retornada pela página HTML.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Extrair a lista de tabelas visíveis de forma determinística sem loops manuais no teste.

3. **Usei o CATSPY para:**
   Consultar recomendações do CATSPY para evitar condicionais em asserções de lista. Link: https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Extração declarativa dos nomes das tabelas e asserção direta de igualdade de conjuntos.

---

### Registro #31 — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Commit:** `18be25d`
- **Arquivo e Teste:** `tests/test_permissions.py::test_view_query` (linhas 340-355)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Conditional Test Logic. Teste com `if` para avaliar se a consulta customizada deveria retornar 200 ou 403 dependendo do ator.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Desacoplar a lógica de autenticação dos códigos de status HTTP esperados.

3. **Usei o CATSPY para:**
   Consultar o catálogo CATSPY para Conditional Test Logic. Link: https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Uso do `@pytest.mark.parametrize` mapeando `(actor, expected_status)` diretamente.

---

### Registro #32 — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Commit:** `be17abd`
- **Arquivo e Teste:** `tests/test_permissions.py::test_view_instance` (linhas 380-390)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Conditional Test Logic. Estrutura de decisão condicional para checar o cabeçalho de login na página raiz da instância.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Tornar o teste linear sem bifurcações de fluxo.

3. **Usei o CATSPY para:**
   Consultar estratégias de linearização de testes no CATSPY. Link: https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Linearização das asserções e eliminação da bifurcação condicional.

---

### Registro #33 — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Commit:** `506bcf8`
- **Arquivo e Teste:** `tests/test_template_context.py::test_context_class_fields_all_have_help` (linha 115)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Conditional Test Logic. O teste iterava em um loop `for` sobre os campos do contexto de template para verificar se cada campo possuía texto de ajuda, interrompendo a verificação na primeira falha.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Validar todos os campos de forma abrangente e sem loop manual no teste.

3. **Usei o CATSPY para:**
   Consultar orientações do CATSPY sobre validações em lote e asserções compreensivas. Link: https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Substituição do loop manual por uma compreensão de conjunto dos campos sem texto de ajuda com a asserção declarativa `assert missing_help == set()`.

---

### Registro #34 — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Commit:** `47d4674`
- **Arquivo e Teste:** `tests/test_routes.py::test_routes` (linhas 60-70)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Conditional Test Logic. Loop `for` iterando sobre uma lista de rotas com condicional interna para tratar rotas com e sem parâmetros de consulta.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Parametrizar o teste de rotas preservando a diversidade de verbos HTTP e caminhos.

3. **Usei o CATSPY para:**
   Consultar a seção de Conditional Test Logic no CATSPY. Link: https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Parametrização das tuplas de rota via `@pytest.mark.parametrize`.

---

### Registro #35 — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Commit:** `a81ae38`
- **Arquivo e Teste:** `tests/test_tracer.py::test_trace` (linhas 40-55)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Conditional Test Logic. O teste continha blocos condicionais para verificar o tracer ativado versus desativado.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Dividir o teste em dois métodos especializados sem duplicar o setup do tracer.

3. **Usei o CATSPY para:**
   Consultar a estratégia *Split Test* recomendada pelo CATSPY para bifurcações booleanas de configuração. Link: https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Divisão do teste em dois testes especializados: um para o tracer habilitado e outro para desabilitado.

---

### Registro #36 — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Commit:** `144b331`
- **Arquivo e Teste:** `tests/test_tracer.py::test_trace_enabled` (linhas 70-85)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Conditional Test Logic. Presença de verificações condicionais dentro das asserções de spans do tracer habilitado.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Garantir a verificação dos spans sem induzir novos smells de asserções em helpers.

3. **Usei o CATSPY para:**
   Consultar boas práticas para asserções lineares no CATSPY. Link: https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Eliminação de lógica condicional interna e validação direta dos spans registrados.

---

### Registro #37 — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Commit:** `2f22ed2`
- **Arquivo e Teste:** `tests/test_load_extensions.py::test_load_extension_is_disabled` (linhas 45-55)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Conditional Test Logic. O teste utilizava ramificações condicionais para checar a presença de mensagens de extensão bloqueada na resposta JSON e HTML.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Separar o teste de formato JSON do formato HTML mantendo a validação de segurança ativa.

3. **Usei o CATSPY para:**
   Consultar o catálogo CATSPY para remoção de branches condicionais em testes de múltiplos formatos. Link: https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Parametrização dos formatos e eliminação de branches `if/else`.

---

### Registro #38 — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Commit:** `ef4bf88`
- **Arquivo e Teste:** `tests/test_internals_request.py::test_request_args` (linhas 65-75)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Conditional Test Logic. O teste continha lógica condicional para extrair e validar parâmetros de query string repetidos em requisições ASGI.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Preservar a validação de dicionários de listas de argumentos de forma linear.

3. **Usei o CATSPY para:**
   Consultar estratégias de simplificação de asserções em requisições HTTP no CATSPY. Link: https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Substituição das condicionais por comparações diretas de dicionários de argumentos esperados.

---

### Registro #39 — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Commit:** `7855315`
- **Arquivo e Teste:** `tests/test_table_api.py::test_nofacet` (linhas 315-325)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Conditional Test Logic. O teste iterava em um loop `for` sobre os nós facetados da resposta JSON para verificar se nenhum facet havia sido computado.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Validar a ausência de facets em uma única asserção sem loops procedurais.

3. **Usei o CATSPY para:**
   Consultar recomendações do CATSPY sobre eliminação de loops de validação em payloads JSON. Link: https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Substituição do loop pela asserção direta `assert response.json()["facet_results"] == {}`.

---

### Registro #40 — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Commit:** `7498567`
- **Arquivo e Teste:** `tests/test_table_api.py::test_nosuggest` (linhas 345-355)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Conditional Test Logic. Uso de loop `for` com condicional interna para inspecionar sugestões de facetação na tabela.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Garantir a verificação de sugestões sem loops de inspeção manual.

3. **Usei o CATSPY para:**
   Consultar técnicas de asserção direta no CATSPY. Link: https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Asserção declarativa sobre a chave `suggested_facets` do JSON de resposta.

---

### Registro #41 — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Commit:** `c7192ad`
- **Arquivo e Teste:** `tests/test_table_api.py::test_nocount` (linhas 375-385)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Conditional Test Logic. Teste com condicional para avaliar se a contagem total de linhas estava presente ou ausente no JSON.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Tornar a verificação de contagem estritamente linear e sem ramificações.

3. **Usei o CATSPY para:**
   Consultar o catálogo CATSPY para Conditional Test Logic. Link: https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Eliminação do branch condicional e asserção direta da presença do valor `null` ou ausência da chave.

---

### Registro #42 — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Commit:** `30204fe`
- **Arquivo e Teste:** `tests/test_csv.py::test_custom_sql_csv_errors` (linhas 188-195)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Conditional Test Logic. O teste parametrizado recebia o booleano `download` e executava um branch condicional `if download: query_string += "&_dl=1"` no corpo da função de teste, criando múltiplos caminhos de execução internos.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Eliminar o branch condicional sem alterar o conjunto de casos executados e sem quebrar as asserções de cabeçalho HTTP do endpoint CSV.

3. **Usei o CATSPY para:**
   Consultar no CATSPY a estratégia de parametrização direta de valores de entrada para eliminar ramificações internas. Link: https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Substituição do parâmetro booleano `download` pelo parâmetro de string `download_suffix` com os valores `("", "&_dl=1")`, permitindo a interpolação direta na URL sem nenhuma condicional `if`.

---

### Registro #43 — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Commit:** `5659420`
- **Arquivo e Teste:** `tests/test_table_api.py::test_binary_data_in_json` (linhas 410-430)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Conditional Test Logic. O teste recebia múltiplos formatos e utilizava um bloco `if expected_json: ... else: ...` para alternar entre validações de JSON em formato array versus formato Newline-Delimited JSON (JSON-NL).

2. **Minhas principais dificuldades na remoção do code smell são:**
   Separar as duas famílias de teste sem duplicar a criação dos dados binários na tabela do SQLite.

3. **Usei o CATSPY para:**
   Consultar a técnica de *Split Test* no CATSPY para dividir testes que validam formatos substancialmente distintos. Link: https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Divisão do teste em dois métodos especializados: `test_binary_data_in_json_shape_array` e `test_binary_data_in_json_nl`, eliminando os blocos `if/else` internos.

---

### Registro #44 — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Commit:** `1808665`
- **Arquivo e Teste:** `tests/test_fts_permissions.py::test_external_content_fts_inherits_content_table_view_permission` (linhas 75-85)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Conditional Test Logic. O teste continha um branch condicional `if fts_module == "fts5":` para definir opções específicas do módulo SQLite FTS5 versus FTS4.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Compatibilizar as diferentes opções de tabela virtual FTS do SQLite sem ramificações condicionais durante o teste.

3. **Usei o CATSPY para:**
   Consultar a técnica de parametrização conjunta de tuplas no CATSPY. Link: https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Parametrização conjunta das tuplas `(fts_module, fts_options)` no decorador `@pytest.mark.parametrize`, tornando a criação da tabela FTS estritamente linear e sem `if`.

---

### Registro #45 — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Commit:** `746720f`
- **Arquivo e Teste:** `tests/test_queries.py::test_execute_write_schema_operations_use_schema_permissions` (linhas 210-235)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Conditional Test Logic. O teste utilizava uma cadeia condicional de 5 branches `if/elif expected_state == ...` para verificar diferentes estados após a execução de operações de escrita de schema (tabela criada, tabela renomeada, índice criado, etc.).

2. **Minhas principais dificuldades na remoção do code smell são:**
   Desmontar a cadeia de condicionais preservando a verificação de integridade assíncrona do esquema de banco de dados para cada operação.

3. **Usei o CATSPY para:**
   Consultar a estratégia do CATSPY de aplicar funções de verificação de estado parametrizadas como objetos de primeira classe (*Callable/Strategy pattern*). Link: https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Substituição da cadeia de `if/elif` por funções assíncronas verificadoras de estado (`assert_created`, `assert_renamed`, etc.) passadas diretamente como parâmetro no `@pytest.mark.parametrize`.

---
*Diário de bordo auditado e sincronizado com os commits da branch refatoracao-test-smells em 29 de Outubro de 2026.*
