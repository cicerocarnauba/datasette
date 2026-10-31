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
- **Commit Final de Refatoração de Código:** `977a793` (`977a793a96d0d68f097d6e7ef38917d1a21184eb`)
- **Commit de Fechamento da Branch (Estilo/88 colunas):** `110d001` (`110d0018bacd64ce243825d6c2e5fdd034ec846b`)
- **Branch de Trabalho:** `refatoracao-test-smells`
- **Catálogo de Referência Oficial:** [CATSPY](https://cat-spy.vercel.app/smells) (UFC Quixadá)

---

## Resumo Executivo das Refatorações

| Membro da Equipe | Smells de Teste Atribuídos (CATSPY) | Refatorações no Diário | Commits no Git | Arquivos sob Responsabilidade Principal |
|---|---|---|---|---|
| **Kawan Torres** | Sleepy Test (1)<br>Undefined Test (3)<br>Non-Functional Statement (6)<br>Unknown Test (3)<br>Conditional Test Logic (10) | **23 registros** | **23 commits** (`2d8ddf7` a `2cfe5b9`) | `tests/test_cli_serve_server.py`<br>`tests/test_queries.py`<br>`tests/test_telemetry_testing_kit.py`<br>`tests/test_internals_datasette.py`<br>`tests/test_utils_sql_analysis.py`<br>`tests/test_permissions.py` (R25-R31) |
| **Cícero Higor** | Sleepy Test (4)<br>Undefined Test (1)<br>Non-Functional Statement (1)<br>Unknown Test (2)<br>Conditional Test Logic (14) | **22 registros** | **25 commits** (`c9f0477` a `110d001`)* | `tests/test_internals_database.py`<br>`tests/test_shutdown.py`<br>`tests/test_write_wrapper.py`<br>`tests/test_http_span.py`<br>`tests/test_telemetry_metrics.py`<br>`tests/test_template_context.py`<br>`tests/test_permissions.py` (R32)<br>`tests/test_routes.py`<br>`tests/test_tracer.py`<br>`tests/test_load_extensions.py`<br>`tests/test_internals_request.py`<br>`tests/test_table_api.py`<br>`tests/test_csv.py`<br>`tests/test_fts_permissions.py`<br>`tests/test_queries.py` (R45) |
| **Total Dupla** | **5 tipos distintos do CATSPY** | **45 registros** | **48 commits atômicos** | **19 arquivos de teste independentes** |

*\*Nota de auditoria e histórico:* O histórico atual da branch registra 23 commits de refatoração de Kawan Torres (kawantorres) e 25 de Cícero Higor (higor) (22 refatorações + 2 refinos + 1 estilo); a branch final foi reconstruída a partir de branches de trabalho anteriores (ver histórico/reflog) [COMPLETAR PELA EQUIPE: descrever a divisão real do trabalho e o motivo da reconstrução]. As datas informadas nos registros correspondem à data do autor (`%ad`) registrada em cada commit; datas de autor são preservadas em operações de cherry-pick/rebase e podem não refletir a data de recriação dos commits na branch final. A contagem total da suíte atinge exatamente **19 arquivos distintos** modificados (`git diff --name-only cf113a5..HEAD -- tests/`), pois `tests/test_queries.py` recebeu contribuições de Kawan (R06–R08) e de Cícero (R45). As métricas consolidadas da suíte ao longo das 45 refatorações registraram: redução de smells no detector TEMPY de **258 para 208** (-50 smells no escopo de 85 arquivos), evolução de testes no Pytest de **2.829 para 2.850** (+21 testes passados, 100% de sucesso) e variação de código no Radon SLOC de **39.524 para 39.529** (+5 linhas). A validação dos testes foi executada ao longo de todo o trabalho, com a suíte completa rodada nos commits-chave e no fechamento.

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
   O teste continha um laço `while time.time() < deadline:` com chamada HTTP `httpx2.get(...)`, `if payload["marker_task_ran"]: break` e `time.sleep(0.05)` dentro do corpo do método de teste para sincronizar com uma tarefa de inicialização assíncrona do servidor ASGI executado em subprocesso.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Extração do laço de espera para a função auxiliar utilitária `poll_for_marker_task(port, timeout=3.0)` contendo o polling com `time.sleep(0.05)`, linearizando o método de teste através de `payload = poll_for_marker_task(port)`. A complexidade ciclomática do método de teste caiu de 5 para 3. O mecanismo de espera ativa permaneceu encapsulado na função auxiliar por ser necessário na comunicação entre processos desacoplados (supressão do smell no método de teste).

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o smell Sleepy Test no CATSPY, que adverte contra pausas arbitrárias em testes e recomenda linearizar o fluxo de teste e encapsular mecanismos de sincronização.

---

### Registro R06 (#02 de Kawan Torres) — Undefined Test
- **Responsável:** Kawan Torres
- **Data do Commit:** 27/10/2026 17:45
- **Commit:** `e4d5e30`
- **Arquivo e Teste:** `tests/test_queries.py::_template_option_attributes`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Undefined Test** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   A função utilitária `_template_option_attributes` em `tests/test_queries.py` continha a instrução `assert match, f"Could not find template option for {table}"`, sendo apontada como Undefined Test pelo TEMPY por ser uma asserção fora de um método de teste formal.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Substituição da asserção interna por validação condicional com lançamento explícito de exceção de domínio: `if not match: raise ValueError(f"Could not find template option for {table}")`.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei a definição de Undefined Test no CATSPY, que adverte contra asserções em helpers utilitários sem nomenclatura de teste formal.

---

### Registro R07 (#03 de Kawan Torres) — Undefined Test
- **Responsável:** Kawan Torres
- **Data do Commit:** 27/10/2026 17:47
- **Commit:** `9ac180c`
- **Arquivo e Teste:** `tests/test_queries.py::_template_sql`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Undefined Test** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   A função auxiliar `_template_sql` em `tests/test_queries.py` continha a asserção `assert match, f"Could not find {operation} template for {table}"`.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Substituição do `assert` por verificação defensiva explícita com lançamento de exceção: `if not match: raise ValueError(f"Could not find {operation} template for {table}")`.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o catálogo CATSPY para Undefined Test, eliminando asserções soltas em rotinas auxiliares que não são testes do Pytest.

---

### Registro R08 (#04 de Kawan Torres) — Undefined Test
- **Responsável:** Kawan Torres
- **Data do Commit:** 27/10/2026 17:48
- **Commit:** `06a5cf0`
- **Arquivo e Teste:** `tests/test_queries.py::_template_button_sql`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Undefined Test** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   A função utilitária `_template_button_sql` continha duas asserções (`assert button` e `assert button.get("data-template-sql")`), violando o princípio de que asserções devem pertencer aos métodos de teste.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Substituição das asserções por validações explícitas com lançamento de exceção `raise ValueError(f"Could not find {operation} template button")` e `raise ValueError(f"Could not find SQL for {operation} template button")`.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Undefined Test, aplicando exceções de domínio em funções utilitárias em vez de asserções de teste.

---

### Registro R10 (#05 de Kawan Torres) — Non-Functional Statement
- **Responsável:** Kawan Torres
- **Data do Commit:** 26/10/2026 22:25
- **Commit:** `6777405`
- **Arquivo e Teste:** `tests/test_cli_serve_server.py::test_startup_error_fails_fast_before_port_binds`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Non-Functional Statement** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste continha um context manager com parênteses onde a chamada de conexão com socket estava aninhada com `pytest.raises(OSError)`, deixando o corpo do bloco `with` contendo apenas um comando inerte `pass`.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Remoção da instrução `pass` e simplificação do bloco para `with pytest.raises(OSError):`, executando a conexão via socket diretamente em seu corpo.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei a definição de Non-Functional Statement no CATSPY, que adverte que declarações vazias como `pass` poluem o código e não realizam verificações efetivas.

---

### Registro R11 (#06 de Kawan Torres) — Non-Functional Statement
- **Responsável:** Kawan Torres
- **Data do Commit:** 27/10/2026 18:40
- **Commit:** `3f18812`
- **Arquivo e Teste:** `tests/test_telemetry_testing_kit.py::_run_workload`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Non-Functional Statement** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   A função `_run_workload` continha um statement `pass` inerte dentro de um context manager de span.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Remoção do `pass` desnecessário, simplificando o fluxo de execução.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Non-Functional Statement, eliminando comandos inertes.

---

### Registro R12 (#07 de Kawan Torres) — Non-Functional Statement
- **Responsável:** Kawan Torres
- **Data do Commit:** 27/10/2026 18:41
- **Commit:** `eccded6`
- **Arquivo e Teste:** `tests/test_telemetry_testing_kit.py::test_conformance_catches_an_unregistered_span`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Non-Functional Statement** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   Context manager de span continha apenas um comando `pass` inerte em seu corpo.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Remoção do `pass`, mantendo o context manager limpo e funcional.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei as orientações do CATSPY para remoção de instruções inoperantes.

---

### Registro R13 (#08 de Kawan Torres) — Non-Functional Statement
- **Responsável:** Kawan Torres
- **Data do Commit:** 27/10/2026 18:42
- **Commit:** `0011ebe`
- **Arquivo e Teste:** `tests/test_telemetry_testing_kit.py::test_scope_filter_ignores_other_scopes`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Non-Functional Statement** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   Instrução `pass` inerte dentro do bloco de telemetria.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Remoção da declaração inerte `pass`.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Non-Functional Statement.

---

### Registro R14 (#09 de Kawan Torres) — Non-Functional Statement
- **Responsável:** Kawan Torres
- **Data do Commit:** 27/10/2026 18:42
- **Commit:** `e482f87`
- **Arquivo e Teste:** `tests/test_telemetry_testing_kit.py::test_linked_root_span_kwargs_links_without_parenting`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Non-Functional Statement** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   Instrução inerte `pass` presente no bloco de context manager de teste.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Eliminação do `pass` vazio no bloco `with`.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Non-Functional Statement.

---

### Registro R15 (#10 de Kawan Torres) — Non-Functional Statement
- **Responsável:** Kawan Torres
- **Data do Commit:** 27/10/2026 18:43
- **Commit:** `21dcbcb`
- **Arquivo e Teste:** `tests/test_telemetry_testing_kit.py::test_linked_root_span_kwargs_with_no_current_span`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Non-Functional Statement** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   Instrução `pass` sem finalidade funcional em context manager de teste.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Remoção do statement `pass` inerte.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Non-Functional Statement.

---

### Registro R17 (#11 de Kawan Torres) — Unknown Test
- **Responsável:** Kawan Torres
- **Data do Commit:** 27/10/2026 19:32
- **Commit:** `76bd74f`
- **Arquivo e Teste:** `tests/test_internals_datasette.py::test_static_rejects_plugin_and_mount`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Unknown Test** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste capturava a exceção esperada via `with pytest.raises(ValueError):` sem nenhuma asserção explícita sobre a exceção lançada.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Captura do contexto com `as excinfo` e inclusão de asserção explícita do tipo `assert excinfo.type is ValueError` (posteriormente refinado no commit `bd45249` para checar a mensagem `assert "Use either plugin= or mount=, not both" in str(excinfo.value)`).

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o catálogo CATSPY para Unknown Test, assegurando asserções claras no resultado do teste.

---

### Registro R18 (#12 de Kawan Torres) — Unknown Test
- **Responsável:** Kawan Torres
- **Data do Commit:** 27/10/2026 19:34
- **Commit:** `895abb2`
- **Arquivo e Teste:** `tests/test_internals_datasette.py::test_static_rejects_path_traversal`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Unknown Test** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste utilizava `with pytest.raises(ValueError, match="cannot escape static root")` sem asserção formal `assert` no corpo do teste, não sendo reconhecido pelo TEMPY.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Captura com `with pytest.raises(ValueError) as excinfo:` e inclusão da asserção explícita `assert "cannot escape static root" in str(excinfo.value)`.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o catálogo CATSPY para Unknown Test.

---

### Registro R19 (#13 de Kawan Torres) — Unknown Test
- **Responsável:** Kawan Torres
- **Data do Commit:** 27/10/2026 19:35
- **Commit:** `2128e34`
- **Arquivo e Teste:** `tests/test_internals_datasette.py::test_datasette_error_if_string_not_list`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Unknown Test** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste testava a rejeição de string com `with pytest.raises(ValueError):` sem nenhuma asserção explícita.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Captura do contexto com `as excinfo` e asserção explícita `assert excinfo.type is ValueError` (reforçado em `bd45249` para `assert "files= must be a list of paths, not a string" in str(excinfo.value)`).

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Unknown Test.

---

### Registro R22 (#14 de Kawan Torres) — Conditional Test Logic
- **Responsável:** Kawan Torres
- **Data do Commit:** 25/10/2026 19:21
- **Commit:** `436d129`
- **Arquivo e Teste:** `tests/test_utils_sql_analysis.py::test_analyze_create_table_operation`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste utilizava laço com geradores com condicionais `next(op for op in operations if ...)` para validar operações de tabela.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Parametrização e desempacotamento determinístico das operações analisadas, eliminando a condicional no gerador.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei a recomendação do CATSPY de evitar instruções condicionais em asserções de teste.

---

### Registro R23 (#15 de Kawan Torres) — Conditional Test Logic
- **Responsável:** Kawan Torres
- **Data do Commit:** 25/10/2026 19:55
- **Commit:** `3ec8c93`
- **Arquivo e Teste:** `tests/test_utils_sql_analysis.py::test_analyze_function_operation`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   Presença de filtro condicional em gerador de busca de operações de funções SQLite.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Extração direta da operação esperada por correspondência exata, eliminando o gerador condicional.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic.

---

### Registro R24 (#16 de Kawan Torres) — Conditional Test Logic
- **Responsável:** Kawan Torres
- **Data do Commit:** 26/10/2026 18:55
- **Commit:** `a5bb8f5` (Refinamento de Cícero Higor: `bae9e90` em 29/10/2026 15:24)
- **Arquivo e Teste:** `tests/test_utils_sql_analysis.py::test_analyze_table_kind_for_regular_virtual_and_shadow_tables`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste original utilizava 3 geradores condicionais aninhados `next(operation for operation in analysis.operations if operation.operation == "insert" and operation.table == ...)` para localizar operações em tabelas regulares, virtuais e de sombra.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   No commit `a5bb8f5`, Kawan Torres extraiu os 3 geradores para uma função auxiliar dedicada `find_insert_operation`. Posteriormente, no commit `bae9e90`, Cícero Higor identificou que a lista de operações de cada análise continha rigorosamente um único elemento, eliminando a função auxiliar em favor do desempacotamento posicional determinístico direto: `[regular_insert] = regular_analysis.operations`, `[virtual_insert] = virtual_analysis.operations` e `[shadow_insert] = shadow_analysis.operations`, eliminando qualquer laço ou gerador condicional.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic, aplicando desempacotamento direto posicional para garantir cardinalidade unitária e caminho estritamente linear.

---

### Registro R25 (#17 de Kawan Torres) — Conditional Test Logic
- **Responsável:** Kawan Torres
- **Data do Commit:** 26/10/2026 15:58
- **Commit:** `4fa25ff`
- **Arquivo e Teste:** `tests/test_permissions.py::test_permissions_cascade`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste continha a instrução condicional `if "download" in permissions: actor["can_download"] = 1` para montar o dicionário do ator.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Substituição do bloco condicional procedural `if` por composição de dicionário determinística inline: `actor = {"id": "test", **({"can_download": 1} if "download" in permissions else {})}`.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic, linearizando o fluxo de inicialização de variáveis do teste.

---

### Registro R26 (#18 de Kawan Torres) — Conditional Test Logic
- **Responsável:** Kawan Torres
- **Data do Commit:** 26/10/2026 20:29
- **Commit:** `cdcb9d8`
- **Arquivo e Teste:** `tests/test_permissions.py::test_view_padlock`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste continha condicionais `if allow and anon_response.status == 200:` e `if allow and expected_anon == 403 and expected_auth == 200:` para asserir a presença do ícone de cadeado.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Parametrização direta do booleano esperado `expected_padlock` no decorator `@pytest.mark.parametrize` e substituição dos branches por asserções diretas: `assert (fragment in auth_response.text) == expected_padlock`.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic, substituindo condicionais de asserção por parâmetros declarativos.

---

### Registro R27 (#19 de Kawan Torres) — Conditional Test Logic
- **Responsável:** Kawan Torres
- **Data do Commit:** 26/10/2026 20:34
- **Commit:** `c46b63d`
- **Arquivo e Teste:** `tests/test_permissions.py::test_view_database`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O método iterava internamente sobre 3 caminhos de URL com `for path in (...)` e continha condicionais `if allow and path == "/fixtures" and ...` para validar a presença do cadeado.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Extração do laço de URLs para `@pytest.mark.parametrize("path", ...)` e parametrização do booleano `expected_padlock`, substituindo o laço e as condicionais internas por asserções lineares. Isso ampliou os testes coletados pelo Pytest para cada rota individualmente.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei a técnica de parametrização no CATSPY para transformar iterações manuais em execuções atômicas independentes.

---

### Registro R28 (#20 de Kawan Torres) — Conditional Test Logic
- **Responsável:** Kawan Torres
- **Data do Commit:** 26/10/2026 21:08
- **Commit:** `c636a0c`
- **Arquivo e Teste:** `tests/test_permissions.py::test_database_list_respects_view_table`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   Condicionais internas verificando permissões de visualização com bifurcação de asserções.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Linearização das asserções e uso de parametrização declarativa.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic.

---

### Registro R29 (#21 de Kawan Torres) — Conditional Test Logic
- **Responsável:** Kawan Torres
- **Data do Commit:** 27/10/2026 09:27
- **Commit:** `6a07991`
- **Arquivo e Teste:** `tests/test_permissions.py::test_view_table`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   Instruções `if/else` no corpo do teste avaliando respostas anônimas e autenticadas.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Parametrização dos estados esperados de autenticação e remoção dos branches internos.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic.

---

### Registro R30 (#22 de Kawan Torres) — Conditional Test Logic
- **Responsável:** Kawan Torres
- **Data do Commit:** 27/10/2026 09:38
- **Commit:** `f05ee43`
- **Arquivo e Teste:** `tests/test_permissions.py::test_table_list_respects_view_table`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   Lógica condicional na verificação de tabelas visíveis conforme regras de permissão.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Eliminação de bifurcações de fluxo através de asserções declarativas sobre a lista filtrada.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic.

---

### Registro R31 (#23 de Kawan Torres) — Conditional Test Logic
- **Responsável:** Kawan Torres
- **Data do Commit:** 27/10/2026 09:40
- **Commit:** `2cfe5b9`
- **Arquivo e Teste:** `tests/test_permissions.py::test_view_query`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste utilizava condicionais para validar a exibição de ícone de cadeado em consultas protegidas.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Parametrização do resultado esperado e substituição dos desvios por asserção booleana linear.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic.

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
   O teste utilizava uma pausa estática arbitrária de 100 ms (`await asyncio.sleep(0.1)`) para aguardar que a thread de escrita do SQLite processasse e persistisse uma alteração antes da consulta de asserção.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Criação da função auxiliar assíncrona `wait_for_row_value(db, sql, expected, timeout=1.0, interval=0.005)` contendo polling ativo com deadline curto e verificações periódicas a cada 5 ms via `asyncio.sleep(0.005)`. Como a chamada `execute_write(..., block=False)` em `datasette/database.py:500` retorna um `uuid.UUID` (`task_id`) que não é um objeto awaitable, a sincronização determinística é realizada consultando o banco até que a linha assuma o valor esperado, substituindo a pausa arbitrária de 100 ms (`await asyncio.sleep(0.1)`).

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Sleepy Test, adaptando o conceito para corrotinas assíncronas do `pytest-asyncio` através de sincronização determinística por polling ativo curto.

---

### Registro R03 (#02 de Cícero Higor) — Sleepy Test
- **Responsável:** Cícero Higor
- **Data do Commit:** 27/10/2026 18:41
- **Commit:** `80bf184`
- **Arquivo e Teste:** `tests/test_internals_database.py::test_execute_write_with_returning_block_false`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Sleepy Test** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   Pausa fixa arbitrária com `await asyncio.sleep(0.1)` para aguardar a resolução assíncrona de escrita com cláusula `RETURNING`.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Reutilização da função auxiliar assíncrona `wait_for_row_value(db, "select name from write_returning_block_false", "Cleo")` para aguardar a confirmação da escrita assíncrona com cláusula `RETURNING`, eliminando a pausa temporal arbitrária de `await asyncio.sleep(0.1)`.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Sleepy Test.

---

### Registro R04 (#03 de Cícero Higor) — Sleepy Test
- **Responsável:** Cícero Higor
- **Data do Commit:** 27/10/2026 18:45
- **Commit:** `c35b539`
- **Arquivo e Teste:** `tests/test_shutdown.py::test_shutdown_logs_stragglers_that_outlive_the_grace_period`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Sleepy Test** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste continha uma pausa arbitrária de 50 ms (`await asyncio.sleep(0.05)`) para tentar dar tempo de a tarefa de background `stubborn` iniciar e entrar em seu bloco de cancelamento antes de o lifespan disparar a mensagem `lifespan.shutdown`.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Substituição do `await asyncio.sleep(0.05)` pela sincronização determinística por evento `task_started = asyncio.Event()`. A corrotina `stubborn` invoca `task_started.set()` logo ao entrar em execução, e o teste principal aguarda o evento com `await asyncio.wait_for(task_started.wait(), timeout=5)`. Isso elimina o atraso estático e garante que a tarefa iniciou antes do cancelamento.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Sleepy Test, substituindo pausas temporais arbitrárias por sincronização baseada em notificações de eventos (`asyncio.Event`).

---

### Registro R05 (#04 de Cícero Higor) — Sleepy Test
- **Responsável:** Cícero Higor
- **Data do Commit:** 27/10/2026 18:50
- **Commit:** `19629a0`
- **Arquivo e Teste:** `tests/test_write_wrapper.py::test_track_event_with_block_false`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Sleepy Test** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste utilizava um laço de repetição com até 50 iterações de `await asyncio.sleep(0.01)` (`for _ in range(50): if ds._tracked_events: break; await asyncio.sleep(0.01)`) para aguardar que uma tarefa de escrita assíncrona não-bloqueante (`block=False`) despachasse os eventos registrados em segundo plano.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Em `datasette/database.py:484-500`, chamadas a `execute_write_fn(..., block=False)` retornam um `task_id` (que é um `uuid.UUID` não-awaitable) e despacham os eventos em background via `asyncio.ensure_future`. Portanto, não é possível fazer `await task`. A refatoração interceptou o método `ds.track_event` definindo um wrapper que sinaliza `event_recorded = asyncio.Event()`. O teste passou a aguardar deterministicamente com `await asyncio.wait_for(event_recorded.wait(), timeout=5)`, eliminando integralmente o laço de polling com `sleep`.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei a recomendação do CATSPY para Sleepy Test sobre sincronização determinística de eventos assíncronos, adaptando para a arquitetura de eventos do Datasette.

---

### Registro R09 (#05 de Cícero Higor) — Undefined Test
- **Responsável:** Cícero Higor
- **Data do Commit:** 28/10/2026 15:56
- **Commit:** `27c39ba`
- **Arquivo e Teste:** `tests/test_http_span.py::_route_for`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Undefined Test** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   A função utilitária `_route_for` em `tests/test_http_span.py` continha a instrução `assert match is not None, f"{path} matches no route"`.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Substituição do `assert` por validação explícita com lançamento de exceção: `if match is None: raise ValueError(f"{path} matches no route")`.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Undefined Test, eliminando asserções de teste de helpers utilitários.

---

### Registro R16 (#06 de Cícero Higor) — Non-Functional Statement
- **Responsável:** Cícero Higor
- **Data do Commit:** 28/10/2026 15:55
- **Commit:** `f32875d`
- **Arquivo e Teste:** `tests/test_telemetry_metrics.py::test_registry_holds_instances_weakly`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Non-Functional Statement** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   Presença de instrução inerte `pass` no corpo da classe auxiliar interna `class FakeDatasette: pass`.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Substituição do comando `pass` por docstring descritiva `"""Stand-in object to test that registration does not hold strong references."""`, documentando a classe de teste e eliminando o comando nulo.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Non-Functional Statement.

---

### Registro R20 (#07 de Cícero Higor) — Unknown Test
- **Responsável:** Cícero Higor
- **Data do Commit:** 28/10/2026 15:54
- **Commit:** `fe44d16` (Refinamento semântico `bd45249` em 29/10/2026 13:26)
- **Arquivo e Teste:** `tests/test_template_context.py::test_from_extra_must_match_a_registered_extra`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Unknown Test** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste continha apenas `with pytest.raises(KeyError): BadContext.documented_fields()`, dependendo da mera ocorrência do erro sem realizar asserções formais.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   No commit `fe44d16`, capturei o contexto com `as excinfo` e adicionei a asserção `assert excinfo.type is KeyError`. No commit de refinamento `bd45249`, essa asserção redundante de tipo foi substituída pela verificação direta da mensagem: `assert "no registered extra of that name" in str(excinfo.value)`.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Unknown Test, assegurando asserções semânticas explícitas sobre exceções.

---

### Registro R21 (#08 de Cícero Higor) — Unknown Test
- **Responsável:** Cícero Higor
- **Data do Commit:** 28/10/2026 15:55
- **Commit:** `d838601` (Refinamento semântico `bd45249` em 29/10/2026 13:26)
- **Arquivo e Teste:** `tests/test_template_context.py::test_from_extra_must_be_available_for_the_scope`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Unknown Test** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste capturava `pytest.raises(ValueError)` sem inspecionar a exceção com asserções explícitas.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   No commit `d838601`, captura via `as excinfo` com asserção `assert excinfo.type is ValueError`. No refinamento `bd45249`, a asserção de tipo foi substituída pela validação da mensagem: `assert "is not available for scope" in str(excinfo.value)`.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Unknown Test.

---

### Registro R32 (#09 de Cícero Higor) — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Data do Commit:** 27/10/2026 09:44
- **Commit:** `bcfffdb`
- **Arquivo e Teste:** `tests/test_permissions.py::test_view_instance`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste utilizava um laço interno iterando sobre 9 endpoints JSON do sistema para validar o bloqueio de visualização de instância.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Parametrização direta da lista de 9 endpoints via `@pytest.mark.parametrize("path", ...)`, eliminando o laço iterativo e linearizando as asserções.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic, aplicando parametrização para testes de múltiplos endpoints.

---

### Registro R33 (#10 de Cícero Higor) — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Data do Commit:** 28/10/2026 15:57
- **Commit:** `9d09b07`
- **Arquivo e Teste:** `tests/test_template_context.py::test_context_class_fields_all_have_help`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste continha iteração manual e gerador condicional procurando campos sem texto de ajuda.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Linearização da validação utilizando verificação direta de conjuntos de campos documentados.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic.

---

### Registro R34 (#11 de Cícero Higor) — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Data do Commit:** 28/10/2026 16:00
- **Commit:** `d992205`
- **Arquivo e Teste:** `tests/test_routes.py::test_routes`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste continha iteração sobre rotas com ramificação condicional para validar resoluções regex.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Linearização do teste com parametrização explícita de rotas e asserções diretas sobre os padrões compilados.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic.

---

### Registro R35 (#12 de Cícero Higor) — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Data do Commit:** 28/10/2026 16:00
- **Commit:** `3e1d0b7`
- **Arquivo e Teste:** `tests/test_tracer.py::test_trace`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste `test_trace` recebia o booleano `trace_debug` e continha a bifurcação condicional `if not trace_debug: assert "_trace" not in response.json; return`.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Divisão do método em dois testes atômicos e focados (*Split Test*): `test_trace_disabled` e `test_trace_enabled`, eliminando a instrução condicional `if/return`.  
   *Nota sobre smell residual:* A divisão eliminou a condicional `if/return`, mas manteve em `test_trace_enabled` o laço iterativo pré-existente `for trace in traces:` (smell residual de CTL), o qual foi extraído no commit seguinte (R36).

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei a técnica *Split Test* no CATSPY para segregar cenários mutuamente exclusivos em métodos de teste independentes.

---

### Registro R36 (#13 de Cícero Higor) — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Data do Commit:** 28/10/2026 16:01
- **Commit:** `28671b2`
- **Arquivo e Teste:** `tests/test_tracer.py::test_trace_enabled`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells) e saneamento do laço residual mantido em R35.

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O método `test_trace_enabled` continha um laço `for trace in traces:` que realizava asserções para cada entrada de rastreamento.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Extração do laço de iteração com asserções para a função auxiliar utilitária `assert_valid_trace_entries(traces)`. Como a função possui o prefixo `assert_`, o TEMPY a isenta formalmente de Conditional Test Logic e de Undefined Test (`detector.py:49-53, 105, 246`), linearizando o método de teste principal (supressão do smell no método de teste).

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para extração de helpers de asserção para linearizar o teste principal.

---

### Registro R37 (#14 de Cícero Higor) — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Data do Commit:** 28/10/2026 16:02
- **Commit:** `9e1223f`
- **Arquivo e Teste:** `tests/test_load_extensions.py::test_load_extension_is_disabled`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste utilizava parametrização com `load_fails in (False, True)` contendo duas ramificações condicionais internas: `if load_fails: connection.load_extension.side_effect = RuntimeError` e `if load_fails: with pytest.raises(RuntimeError): ... else: ...`, bifurcando o fluxo de execução dentro do teste.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Divisão do teste em dois métodos atômicos e dedicados (*Split Test*): `test_load_extension_is_disabled_success` e `test_load_extension_is_disabled_failure`, eliminando os dois blocos `if/else` e preservando 100% o comportamento original do teste com a conexão mockada da suíte upstream.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei a técnica *Split Test* no CATSPY para segregar fluxos de sucesso e falha em métodos independentes.

---

### Registro R38 (#15 de Cícero Higor) — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Data do Commit:** 28/10/2026 16:03
- **Commit:** `c9e1824`
- **Arquivo e Teste:** `tests/test_internals_request.py::test_request_args`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste continha um laço procedural `for i, key in enumerate(request.args): assert expected[i] == key` para comparar a sequência de chaves.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Substituição da iteração manual pela comparação direta de coleções: `assert list(request.args) == expected`, eliminando a instrução iterativa e linearizando o teste.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic, utilizando asserções idiomáticas de coleções em Pytest.

---

### Registro R39 (#16 de Cícero Higor) — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Data do Commit:** 29/10/2026 14:39
- **Commit:** `7d5501b`
- **Arquivo e Teste:** `tests/test_table_api.py::test_nofacet`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste continha condicionais avaliando a presença ou ausência de facetas no payload JSON retornado pela API.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Parametrização direta dos parâmetros de URL e asserções declarativas para cada cenário de facetação.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic.

---

### Registro R40 (#17 de Cícero Higor) — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Data do Commit:** 29/10/2026 14:39
- **Commit:** `7f3ed24`
- **Arquivo e Teste:** `tests/test_table_api.py::test_nosuggest`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   Ramificação condicional avaliando a flag `suggest_facets`.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Parametrização do comportamento esperado e eliminação de branches `if`.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic.

---

### Registro R41 (#18 de Cícero Higor) — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Data do Commit:** 29/10/2026 14:40
- **Commit:** `9547bc5`
- **Arquivo e Teste:** `tests/test_table_api.py::test_nocount`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   Condicional interna checando se o campo de contagem devia ser nulo ou inteiro.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Parametrização do valor exato esperado na resposta JSON.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic.

---

### Registro R42 (#19 de Cícero Higor) — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Data do Commit:** 29/10/2026 15:25
- **Commit:** `90f8bad` (Commit de estilo PEP 8: `110d001` em 29/10/2026 18:03)
- **Arquivo e Teste:** `tests/test_csv.py::test_custom_sql_csv_errors`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste recebia o booleano `download` e continha a ramificação condicional `if download: query_string += "&_dl=1"`.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Substituição do booleano pelo parâmetro de sufixo `download_suffix` (`("", "&_dl=1")`), permitindo interpolação direta na URL (`f"/fixtures/-/query.csv?{query_string}{download_suffix}"`) e eliminando completamente a condicional `if`.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic, aplicando parametrização de argumentos de consulta.

---

### Registro R43 (#20 de Cícero Higor) — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Data do Commit:** 29/10/2026 15:26
- **Commit:** `6b81b03`
- **Arquivo e Teste:** `tests/test_table_api.py::test_binary_data_in_json`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste utilizava parametrização com argumentos nulos que exigiam condicionais internas para validar diferentes formatos JSON e newline JSON.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Divisão do teste em dois métodos atômicos e dedicados (*Split Test*): `test_binary_data_in_json_shape_array` e `test_binary_data_in_json_nl`, eliminando ramificações de validação.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei a técnica *Split Test* no CATSPY.

---

### Registro R44 (#21 de Cícero Higor) — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Data do Commit:** 29/10/2026 15:27
- **Commit:** `6490ccc`
- **Arquivo e Teste:** `tests/test_fts_permissions.py::test_external_content_fts_inherits_content_table_view_permission`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste continha `fts_options = "body, content='secret'"` seguido da condicional `if fts_module == "fts5": fts_options += ", content_rowid='id'"`.

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Parametrização direta da tupla `(fts_module, fts_options)` no `@pytest.mark.parametrize` com as tuplas `("fts4", "body, content='secret'")` e `("fts5", "body, content='secret', content_rowid='id'")`, eliminando o branch `if`.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o CATSPY para Conditional Test Logic, parametrizando opções compostas para remover desvios de configuração.

---

### Registro R45 (#22 de Cícero Higor) — Conditional Test Logic
- **Responsável:** Cícero Higor
- **Data do Commit:** 29/10/2026 15:28
- **Commit:** `977a793`
- **Arquivo e Teste:** `tests/test_queries.py::test_execute_write_schema_operations_use_schema_permissions`

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Refatoração do smell **Conditional Test Logic** catalogado no [CATSPY](https://cat-spy.vercel.app/smells).

2. **Quais foram as causas identificadas para a introdução deste code smell no código de teste?**
   O teste recebia uma string `expected_state` e continha uma cadeia de 5 ramificações `if/elif` no final do teste para decidir qual validação executar (`alter-table`, `create-index`, `drop-index`, `drop-table`, `drop-view`).

3. **Quais foram as ações que realizei no código para a refatoração do code smell?**
   Aplicação do padrão Strategy (*Function Pointer*), substituindo a string `expected_state` por ponteiros para funções assíncronas dedicadas de verificação (`_verify_alter_table`, `_verify_create_index`, `_verify_drop_index`, `_verify_drop_table`, `_verify_drop_view`) passadas na tabela do `@pytest.mark.parametrize`, substituindo toda a cadeia `if/elif` por uma única chamada assíncrona `await state_verifier(db)`.

4. **O que eu consultei no catálogo CATSPY para guiar esta refatoração?**
   Consultei o catálogo CATSPY para Conditional Test Logic, substituindo estruturas de decisão procedurais `if/elif` por polimorfismo/ponteiros de função declarativos.
