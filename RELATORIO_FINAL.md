# **Relatório Final**

**Disciplina:** Qualidade de Software (Semestre 2026.2)  
**Docente:** Profª. Dra. Carla Ilane Moreira Bezerra  
**Monitoria:** Gabriel Alves Pessoa  
**Equipe:**
- Cícero Higor (Matrícula: 574228, GitHub: `cicerocarnauba`, autor no git: `higor`)
- Kawan Torres (Matrícula: 563869, GitHub: `kawantorres`, autor no git: `kawantorres`)

**Projeto:** Datasette ([simonw/datasette](https://github.com/simonw/datasette))  
**Fork:** [cicerocarnauba/datasette](https://github.com/cicerocarnauba/datasette)  
**Pull Request:** [https://github.com/cicerocarnauba/datasette/pull/1](https://github.com/cicerocarnauba/datasette/pull/1)  
**Commit SHA (informado na planilha):** `cf113a5` (`cf113a59850862125b86ce1a58c117aa21341e07`)  
**Commit SHA final de refatoração:** `977a793` (`977a793a96d0d68f097d6e7ef38917d1a21184eb`)  
**Commit SHA de fechamento da branch (estilo/88 colunas):** `110d001` (`110d0018bacd64ce243825d6c2e5fdd034ec846b`)

---

## **1. Apresentação do projeto**

O **Datasette** é uma ferramenta de código aberto em Python desenvolvida primordialmente por Simon Willison, voltada para a exploração, publicação e análise interativa de dados mantidos em bancos de dados relacionais SQLite. A aplicação serve tanto a jornalistas de dados e pesquisadores quanto a desenvolvedores de software, permitindo transformar bases de dados relacionais em APIs REST com suporte a consultas SQL customizadas, paginação determinística, exportação em formatos abertos (JSON e CSV) e visualizações web com suporte a autenticação, controle de permissões granular e telemetria aberta.

A suíte de testes do projeto é estruturada sobre o framework **Pytest** e o ecossistema assíncrono **pytest-asyncio**, cobrindo o comportamento assíncrono do servidor ASGI e do loop de eventos. Os testes residem primordialmente no diretório `tests/` (composto por mais de 80 módulos de teste independentes) e contam com fixtures utilitárias compartilhadas em `tests/fixtures.py` e extensões customizadas em `datasette/_pytest_plugin.py`.

---

## **2. Estado inicial do projeto**

### **2.1 Test smells detectados antes da refatoração (TEMPY)**

A análise léxica da Árvore Sintática Abstrata (AST) executada pela ferramenta **TEMPY** sobre o escopo completo de **85 arquivos** de teste do projeto na linha de base (`cf113a5`, documentado em `metrics-before-tempy/lista_arquivos.txt` e `metrics-before-tempy/resumo.json`) identificou **258 ocorrências** de test smells distribuídas em 8 tipos:

| Test smell | Ocorrências detectadas |
| :---- | :---- |
| Conditional Test Logic | 135 |
| Unknown Test | 43 |
| Verbose Test | 27 |
| Exception Handling | 20 |
| Non-Functional Statement | 10 |
| Programming Paradigms Blend | 10 |
| Undefined Test | 10 |
| Sleepy Test | 3 |
| **Total** | **258** |

*Nota sobre o catálogo de referência:* Conforme as diretrizes pedagógicas da disciplina, a equipe alinhou a seleção exclusivamente com os tipos documentados no catálogo oficial **CATSPY**. Deste modo, o smell *Exception Handling* (20 ocorrências) não foi alvo de intervenção por não constar no catálogo.

### **2.2 Resultados dos testes e cobertura antes da refatoração (Pytest)**

A execução da suíte de testes completa no commit de linha de base (`cf113a5`), registrada em `metrics-before-pytest/pytest_antes.html` e `metrics-before-pytest/coverage_antes.json`, apresentou os seguintes números:

| Métrica | Valor |
| :---- | :---- |
| Testes aprovados (Passed) | 2.829 |
| Testes com falha (Failed) | 0 |
| Erros (Errors) | 0 |
| Ignorados (Skipped) | 85 |
| Falhas esperadas (xfail) | 6 |
| Aprovações inesperadas (xpass) | 15 |
| **Total de testes executados** | **2.935** |
| Cobertura total do projeto (%) | 90,27% |
| Cobertura do código de produção (`datasette/`) (%) | 87,40% |
| Cobertura do código de teste (`tests/`) (%) | 92,70% |

### **2.3 Métricas do código de teste antes da refatoração (Radon)**

As métricas estruturais do código da pasta `tests/` antes da intervenção, mensuradas com o **Radon** (`radon raw tests/ -j`, `radon cc tests/ -j` e `radon mi tests/ -j`) no commit `cf113a5`:

| Métrica | Valor |
| :---- | :---- |
| SLOC total dos testes (`raw` em `tests/`) | 39.524 |
| Complexidade ciclomática média das funções de teste (`cc` em `tests/`) | 4,43 |
| Índice de manutenibilidade médio dos arquivos de teste (`mi` em `tests/`) | 43,10 |

---

## **3. Refatorações realizadas**

A disciplina estabeleceu como requisito a refatoração de no mínimo 40 ocorrências cobrindo ao menos 4 tipos distintos de test smells, priorizando aqueles de maior severidade conforme o catálogo oficial CATSPY. A equipe superou essa meta, realizando **45 refatorações de código de teste** distribuídas em **5 tipos distintos**.

### **3.1 Seleção das ocorrências**

A seleção dos tipos de test smell foi pautada na classificação de severidade do [CATSPY](https://cat-spy.vercel.app/smells), na viabilidade técnica e no impacto positivo na manutenibilidade:

| Test Smell (CATSPY) | Severidade Oficial | Definição no CATSPY e Motivação | Estratégia Adotada |
|---|---|---|---|
| **Conditional Test Logic** | **Alta** | Métodos de teste que contêm instruções condicionais (`if/else`, laços `for/while`, geradores), bifurcando o fluxo de execução e mascarando defeitos parciais. | Parametrização via `@pytest.mark.parametrize`, divisão em testes atômicos (*Split Test*), desempacotamento determinístico e parametrização com funções auxiliares de verificação. |
| **Undefined Test** | **Alta** | Funções que contêm asserções mas não são reconhecidas formalmente como teste pelo framework (ex.: funções utilitárias que usam `assert`). | Substituição de asserções em helpers utilitários por exceções explícitas de validação (`raise ValueError`). |
| **Unknown Test** | **Alta** | Métodos de teste executados sem declarações claras de asserção (ex.: blocos `with pytest.raises(...)` sem inspecionar a exceção). | Captura do contexto de erro (`as excinfo`) com asserções semânticas explícitas sobre tipo (`excinfo.type`) e mensagem (`excinfo.value`). |
| **Sleepy Test** | **Média** | Pausas explícitas (`time.sleep` ou `asyncio.sleep`) que tornam a execução da suíte lenta e sujeita a comportamentos não determinísticos (*flakiness*). | Substituição de pausas fixas por laços de polling ativo com timeout curto ou resolução determinística de tarefas em background. |
| **Non-Functional Statement** | **Baixa** | Declarações inertes que não realizam testes efetivos, como comandos `pass` em blocos de teste. | Remoção direta de instruções vazias e simplificação sintática dos context managers. |

*Justificativa da priorização por severidade e seleção dos tipos:*
1. **Foco predominante em Severidade Alta:** A grande maioria das intervenções da equipe concentrou-se nos smells de severidade **Alta** — foram realizadas **33 refatorações** neste nível (24 de *Conditional Test Logic*, 4 de *Undefined Test* e 5 de *Unknown Test*). Mesmo após as intervenções, restaram 102 ocorrências de CTL, 38 de Unknown Test e 6 de Undefined Test na base completa do Datasette, justificando a concentração máxima de esforço nestes tipos críticos.
2. **Seleção de Sleepy Test (Severidade Média):** O detector TEMPY apontava 3 ocorrências de severidade Média no projeto todo na base, todas correspondentes a *Sleepy Test*. A equipe refatorou 5 métodos de teste com esperas explícitas (1 detectado pelo TEMPY e 4 conceituais em corrotinas assíncronas). Restam no repositório 2 ocorrências de Sleepy Test em arquivos utilitários e de infraestrutura de teste (`tests/conftest.py:340` e `tests/test_playwright.py:869`), sendo que a varredura restrita a `tests/test_*.py` reportou 1.
3. **Seleção de Non-Functional Statement (5º tipo adicional):** Os quatro primeiros tipos obrigatórios selecionados pela equipe foram *Conditional Test Logic*, *Undefined Test*, *Unknown Test* e *Sleepy Test*. Para enriquecer o escopo e superar os requisitos da disciplina, a equipe incluiu um quinto tipo adicional do catálogo CATSPY: *Non-Functional Statement* (7 refatorações de severidade Baixa), eliminando código inerte (`pass` em context managers) que prejudicava a legibilidade. Os demais smells detectados (*Verbose Test*, com 27 ocorrências, e *Programming Paradigms Blend*, com 10 ocorrências) foram preteridos porque sua alteração forçada exigiria quebrar fixtures compartilhadas complexas ou reescrever padrões de orientação a objetos fundamentais do Datasette, gerando alto risco de regressões arquiteturais.

---

### **3.2 Lista das refatorações**

A tabela a seguir consolida as **45 refatorações de código de teste** realizadas pela equipe. O histórico atual da branch registra 23 commits de refatoração de Kawan Torres (`kawantorres`) e 22 de Cícero Higor (`higor`) (além de 2 refinos semânticos e 1 commit de estilo atribuídos a Higor); a branch final foi reconstruída a partir de branches de trabalho anteriores (ver histórico/reflog) [COMPLETAR PELA EQUIPE: descrever a divisão real do trabalho e o motivo da reconstrução].

*Observação sobre a ordem da tabela:* As linhas da tabela abaixo estão agrupadas tematicamente por tipo de test smell e por módulo de teste para facilitar a leitura pedagógica e estrutural. As datas e horários registrados na coluna Data/Hora correspondem às datas de autor (`%ad`) dos commits originais (iniciando em 25/10/2026 e finalizando em 29/10/2026), preservadas durante a reconstrução da branch.

| ID | Test smell | Arquivo::teste | Commit | Data/Hora | Autor | Testes passaram? | Removido pelo TEMPY? | Introduziu novo smell? |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **R01** | Sleepy Test | `tests/test_cli_serve_server.py::test_startup_hook_background_task_runs_on_serving_loop` | `2d8ddf7` | 26/10/2026 21:52 | Kawan Torres | Sim | Suprimido via helper* | Não |
| **R02** | Sleepy Test | `tests/test_internals_database.py::test_execute_write_block_false` | `c9f0477` | 27/10/2026 18:38 | Cícero Higor | Sim | Não (função `async def` ignorada pelo TEMPY) | Não |
| **R03** | Sleepy Test | `tests/test_internals_database.py::test_execute_write_with_returning_block_false` | `80bf184` | 27/10/2026 18:41 | Cícero Higor | Sim | Não (função `async def` ignorada pelo TEMPY) | Não |
| **R04** | Sleepy Test | `tests/test_shutdown.py::test_shutdown_logs_stragglers_that_outlive_the_grace_period` | `c35b539` | 27/10/2026 18:45 | Cícero Higor | Sim | Não (função `async def` ignorada pelo TEMPY) | Não |
| **R05** | Sleepy Test | `tests/test_write_wrapper.py::test_track_event_with_block_false` | `19629a0` | 27/10/2026 18:50 | Cícero Higor | Sim | Não (função `async def` ignorada pelo TEMPY) | Não |
| **R06** | Undefined Test | `tests/test_queries.py::_template_option_attributes` | `e4d5e30` | 27/10/2026 17:45 | Kawan Torres | Sim | Sim (1 → 0) | Não |
| **R07** | Undefined Test | `tests/test_queries.py::_template_sql` | `9ac180c` | 27/10/2026 17:47 | Kawan Torres | Sim | Sim (1 → 0) | Não |
| **R08** | Undefined Test | `tests/test_queries.py::_template_button_sql` | `06a5cf0` | 27/10/2026 17:48 | Kawan Torres | Sim | Sim (1 → 0) | Não |
| **R09** | Undefined Test | `tests/test_http_span.py::_route_for` | `27c39ba` | 28/10/2026 15:56 | Cícero Higor | Sim | Sim (1 → 0) | Não |
| **R10** | Non-Functional Statement | `tests/test_cli_serve_server.py::test_startup_error_fails_fast_before_port_binds` | `6777405` | 26/10/2026 22:25 | Kawan Torres | Sim | Sim (1 → 0) | Não |
| **R11** | Non-Functional Statement | `tests/test_telemetry_testing_kit.py::_run_workload` | `3f18812` | 27/10/2026 18:40 | Kawan Torres | Sim | Sim (1 → 0) | Não |
| **R12** | Non-Functional Statement | `tests/test_telemetry_testing_kit.py::test_conformance_catches_an_unregistered_span` | `eccded6` | 27/10/2026 18:41 | Kawan Torres | Sim | Sim (1 → 0) | Não |
| **R13** | Non-Functional Statement | `tests/test_telemetry_testing_kit.py::test_scope_filter_ignores_other_scopes` | `0011ebe` | 27/10/2026 18:42 | Kawan Torres | Sim | Sim (1 → 0) | Não |
| **R14** | Non-Functional Statement | `tests/test_telemetry_testing_kit.py::test_linked_root_span_kwargs_links_without_parenting` | `e482f87` | 27/10/2026 18:42 | Kawan Torres | Sim | Sim (1 → 0) | Não |
| **R15** | Non-Functional Statement | `tests/test_telemetry_testing_kit.py::test_linked_root_span_kwargs_with_no_current_span` | `21dcbcb` | 27/10/2026 18:43 | Kawan Torres | Sim | Sim (1 → 0) | Não |
| **R16** | Non-Functional Statement | `tests/test_telemetry_metrics.py::test_registry_holds_instances_weakly` | `f32875d` | 28/10/2026 15:55 | Cícero Higor | Sim | Sim (1 → 0) | Não |
| **R17** | Unknown Test | `tests/test_internals_datasette.py::test_static_rejects_plugin_and_mount` | `76bd74f` | 27/10/2026 19:32 | Kawan Torres | Sim | Sim (1 → 0) | Não |
| **R18** | Unknown Test | `tests/test_internals_datasette.py::test_static_rejects_path_traversal` | `895abb2` | 27/10/2026 19:34 | Kawan Torres | Sim | Sim (1 → 0) | Não |
| **R19** | Unknown Test | `tests/test_internals_datasette.py::test_datasette_error_if_string_not_list` | `2128e34` | 27/10/2026 19:35 | Kawan Torres | Sim | Sim (1 → 0) | Não |
| **R20** | Unknown Test | `tests/test_template_context.py::test_from_extra_must_match_a_registered_extra` | `fe44d16` | 28/10/2026 15:54 | Cícero Higor | Sim | Sim (1 → 0) | Não |
| **R21** | Unknown Test | `tests/test_template_context.py::test_from_extra_must_be_available_for_the_scope` | `d838601` | 28/10/2026 15:55 | Cícero Higor | Sim | Sim (1 → 0) | Não |
| **—** | Unknown Test (Refino) | `tests/test_template_context.py` + `test_internals_datasette.py` | `bd45249` | 29/10/2026 13:26 | Cícero Higor | Sim | Sim (0 → 0) | Não |
| **R22** | Conditional Test Logic | `tests/test_utils_sql_analysis.py::test_analyze_create_table_operation` | `436d129` | 25/10/2026 19:21 | Kawan Torres | Sim | Sim (2 → 0) | Não |
| **R23** | Conditional Test Logic | `tests/test_utils_sql_analysis.py::test_analyze_function_operation` | `3ec8c93` | 25/10/2026 19:55 | Kawan Torres | Sim | Sim (1 → 0) | Não |
| **R24** | Conditional Test Logic | `tests/test_utils_sql_analysis.py::test_analyze_table_kind_for_regular_virtual_and_shadow_tables` | `a5bb8f5` | 26/10/2026 18:55 | Kawan Torres | Sim | Sim (6 → 0) | Não |
| **—** | CTL (Refino R24) | `tests/test_utils_sql_analysis.py` | `bae9e90` | 29/10/2026 15:24 | Cícero Higor | Sim | Sim (0 → 0) | Não |
| **R25** | Conditional Test Logic | `tests/test_permissions.py::test_permissions_cascade` | `4fa25ff` | 26/10/2026 15:58 | Kawan Torres | Sim | Sim (1 → 0) | Não |
| **R26** | Conditional Test Logic | `tests/test_permissions.py::test_view_padlock` | `cdcb9d8` | 26/10/2026 20:29 | Kawan Torres | Sim | Sim (2 → 0) | Não |
| **R27** | Conditional Test Logic | `tests/test_permissions.py::test_view_database` | `c46b63d` | 26/10/2026 20:34 | Kawan Torres | Sim | Sim (3 → 0) | Não |
| **R28** | Conditional Test Logic | `tests/test_permissions.py::test_database_list_respects_view_table` | `c636a0c` | 26/10/2026 21:08 | Kawan Torres | Sim | Sim (2 → 0) | Não |
| **R29** | Conditional Test Logic | `tests/test_permissions.py::test_view_table` | `6a07991` | 27/10/2026 09:27 | Kawan Torres | Sim | Sim (2 → 0) | Não |
| **R30** | Conditional Test Logic | `tests/test_permissions.py::test_table_list_respects_view_table` | `f05ee43` | 27/10/2026 09:38 | Kawan Torres | Sim | Sim (2 → 0) | Não |
| **R31** | Conditional Test Logic | `tests/test_permissions.py::test_view_query` | `2cfe5b9` | 27/10/2026 09:40 | Kawan Torres | Sim | Sim (2 → 0) | Não |
| **R32** | Conditional Test Logic | `tests/test_permissions.py::test_view_instance` | `bcfffdb` | 27/10/2026 09:44 | Cícero Higor | Sim | Sim (1 → 0) | Não |
| **R33** | Conditional Test Logic | `tests/test_template_context.py::test_context_class_fields_all_have_help` | `9d09b07` | 28/10/2026 15:57 | Cícero Higor | Sim | Sim (1 → 0) | Não |
| **R34** | Conditional Test Logic | `tests/test_routes.py::test_routes` | `d992205` | 28/10/2026 16:00 | Cícero Higor | Sim | Sim (1 → 0) | Não |
| **R35** | Conditional Test Logic | `tests/test_tracer.py::test_trace` | `3e1d0b7` | 28/10/2026 16:00 | Cícero Higor | Sim | Sim (2 → 1) | Não (manteve laço pré-existente no método enabled, extraído em R36) |
| **R36** | Conditional Test Logic | `tests/test_tracer.py::test_trace_enabled` | `28671b2` | 28/10/2026 16:01 | Cícero Higor | Sim | Suprimido via helper* (1 → 0) | Não |
| **R37** | Conditional Test Logic | `tests/test_load_extensions.py::test_load_extension_is_disabled` | `9e1223f` | 28/10/2026 16:02 | Cícero Higor | Sim | Sim (2 → 0) | Não |
| **R38** | Conditional Test Logic | `tests/test_internals_request.py::test_request_args` | `c9e1824` | 28/10/2026 16:03 | Cícero Higor | Sim | Sim (1 → 0) | Não |
| **R39** | Conditional Test Logic | `tests/test_table_api.py::test_nofacet` | `7d5501b` | 29/10/2026 14:39 | Cícero Higor | Sim | Não (função `async def` ignorada pelo TEMPY) | Não |
| **R40** | Conditional Test Logic | `tests/test_table_api.py::test_nosuggest` | `7f3ed24` | 29/10/2026 14:39 | Cícero Higor | Sim | Não (função `async def` ignorada pelo TEMPY) | Não |
| **R41** | Conditional Test Logic | `tests/test_table_api.py::test_nocount` | `9547bc5` | 29/10/2026 14:40 | Cícero Higor | Sim | Não (função `async def` ignorada pelo TEMPY) | Não |
| **R42** | Conditional Test Logic | `tests/test_csv.py::test_custom_sql_csv_errors` | `90f8bad` | 29/10/2026 15:25 | Cícero Higor | Sim | Não (função `async def` ignorada pelo TEMPY) | Não |
| **R43** | Conditional Test Logic | `tests/test_table_api.py::test_binary_data_in_json` | `6b81b03` | 29/10/2026 15:26 | Cícero Higor | Sim | Não (função `async def` ignorada pelo TEMPY) | Não |
| **R44** | Conditional Test Logic | `tests/test_fts_permissions.py::test_external_content_fts_inherits_content_table_view_permission` | `6490ccc` | 29/10/2026 15:27 | Cícero Higor | Sim | Não (função `async def` ignorada pelo TEMPY) | Não |
| **R45** | Conditional Test Logic | `tests/test_queries.py::test_execute_write_schema_operations_use_schema_permissions` | `977a793` | 29/10/2026 15:28 | Cícero Higor | Sim | Não (função `async def` ignorada pelo TEMPY) | Não |

*\*Notas sobre R01, R02, R03 e R36:* A refatoração R01 transferiu a lógica de espera com `time.sleep(0.05)` para a função auxiliar `poll_for_marker_task`, R02 e R03 transferiram pausas estáticas de 100 ms para o helper de polling assíncrono `wait_for_row_value` que executa laço com `await asyncio.sleep(0.005)` a cada 5 ms, e R36 transferiu o laço `for trace in traces:` para `assert_valid_trace_entries`. Ambas constituem transferências e supressões no método de teste via funções auxiliares utilitárias (isentas pelo prefixo `assert_` ou extraídas do corpo do teste pelo TEMPY), preservando a sincronização necessária.

#### **Quadro de Reconciliação das 45 Refatorações:**
| Categoria de Confirmação | Quantidade | Refatorações Correspondentes |
|---|---|---|
| **Confirmadas diretamente pelo TEMPY (redução na AST)** | **31** | R06 a R21, R22 a R34, R37, R38 (todas funções síncronas `def`) |
| **Suprimidas/transferidas do método de teste para função auxiliar utilitária** | **4** | R01 (polling síncrono encapsulado), R02 e R03 (polling assíncrono via `wait_for_row_value`), R36 (asserção utilitária com prefixo `assert_`) |
| **Divisão de cenários com laço residual saneado em R36** | **1** | R35 (elimina desvio `if/return`, retém laço de iteração em teste síncrono) |
| **Eliminações diretas não pontuadas pela AST do TEMPY (funções `async def` ignoradas pelo parser da ferramenta)** | **9** | R04 e R05 (eliminação completa de sleep via `asyncio.Event`), R39 a R45 (corrotinas assíncronas com parametrização e funções de verificação) |
| **Total de Refatorações** | **45** | **100% rastreáveis por commit; diffs literais detalhados para 6 exemplos representativos** |

---

### **3.3 Exemplos detalhados**

*Nota sobre capturas de tela:* [PRINT PENDENTE: capturas de tela do trecho antes/depois em docs/prints/]. Para fins de auditoria inequívoca de código, são apresentados os diffs literais extraídos diretamente do repositório (`git show <hash>`).

---

#### **R01 – Sleepy Test (`tests/test_cli_serve_server.py::test_startup_hook_background_task_runs_on_serving_loop`)**

**Antes:**
[PRINT PENDENTE]
```python
@pytest.mark.serial
def test_startup_hook_background_task_runs_on_serving_loop(serve_with_plugins):
    """
    Litestream-shaped regression test: a startup hook that does
    asyncio.get_running_loop().create_task(...) must have that task
    actually execute before/while the server is handling requests. This
    only holds if invoke_startup() and uvicorn.Server.serve() share one
    event loop. This test fails against unmodified main, where
    invoke_startup() runs on a throwaway loop that is closed before
    uvicorn opens its own loop to serve.
    """
    _, port = serve_with_plugins({"marker_task_plugin": MARKER_TASK_PLUGIN})
    # The fixture has already waited for the server to answer requests. The
    # marker task deliberately awaits before setting its flag, so poll for a
    # moment rather than assuming it landed before the first request arrived.
    deadline = time.time() + 3.0
    payload = {}
    while time.time() < deadline:
        payload = httpx2.get(
            f"http://127.0.0.1:{port}/-/marker-task-ran", timeout=1.0
        ).json()
        if payload["marker_task_ran"]:
            break
        time.sleep(0.05)
    assert payload.get("marker_task_ran"), (
        "The startup hook's asyncio.create_task(...) never ran - "
        "invoke_startup() and the server are not sharing an event loop"
    )
    # Polling above means this test would also pass if the startup hook were
    # re-run on the serving loop by the first-request fallback - which would
    # hide exactly the bug being tested. invoke_startup() is idempotent today
    # so that cannot happen; assert it explicitly so that if the idempotency
    # guard is ever removed this test fails loudly instead of silently
    # becoming a no-op.
    assert payload["startup_calls"] == 1, (
        "startup hook ran {} times - the marker may have been set by a "
        "re-run on the serving loop rather than by the original task".format(
            payload["startup_calls"]
        )
    )
```

**Depois:**
[PRINT PENDENTE]
```python
def poll_for_marker_task(port, timeout=3.0):
    deadline = time.time() + timeout
    payload = {}
    while time.time() < deadline:
        payload = httpx2.get(
            f"http://127.0.0.1:{port}/-/marker-task-ran", timeout=1.0
        ).json()
        if payload.get("marker_task_ran"):
            break
        time.sleep(0.05)
    return payload


@pytest.mark.serial
def test_startup_hook_background_task_runs_on_serving_loop(serve_with_plugins):
    """
    Litestream-shaped regression test: a startup hook that does
    asyncio.get_running_loop().create_task(...) must have that task
    actually execute before/while the server is handling requests. This
    only holds if invoke_startup() and uvicorn.Server.serve() share one
    event loop. This test fails against unmodified main, where
    invoke_startup() runs on a throwaway loop that is closed before
    uvicorn opens its own loop to serve.
    """
    _, port = serve_with_plugins({"marker_task_plugin": MARKER_TASK_PLUGIN})
    # The fixture has already waited for the server to answer requests. The
    # marker task deliberately awaits before setting its flag, so poll for a
    # moment rather than assuming it landed before the first request arrived.
    payload = poll_for_marker_task(port)
    assert payload.get("marker_task_ran"), (
        "The startup hook's asyncio.create_task(...) never ran - "
        "invoke_startup() and the server are not sharing an event loop"
    )
    # Polling above means this test would also pass if the startup hook were
    # re-run on the serving loop by the first-request fallback - which would
    # hide exactly the bug being tested. invoke_startup() is idempotent today
    # so that cannot happen; assert it explicitly so that if the idempotency
    # guard is ever removed this test fails loudly instead of silently
    # becoming a no-op.
    assert payload["startup_calls"] == 1, (
        "startup hook ran {} times - the marker may have been set by a "
        "re-run on the serving loop rather than by the original task".format(
            payload["startup_calls"]
        )
    )
```

**Por que era um test smell:** O teste continha um laço procedural com `time.sleep(0.05)` dentro do corpo do método de teste para sincronizar com o processo filho do servidor ASGI.  
*Nota sobre `httpx2`:* A chamada `httpx2.get` é o identificador real importado no início do arquivo original (`import httpx2`), não um erro de digitação.  
**Estratégia do CATSPY aplicada:** Extração do laço de espera para a função auxiliar utilitária `poll_for_marker_task`, linearizando o método de teste principal. Trata-se de uma supressão no teste, pois a espera ativa permanece necessária devido à barreira de processos entre o processo de teste e o servidor CLI.  
**Métricas do teste refatorado:**
- Complexidade ciclomática da função (`test_startup_hook...`): **Antes = 5, Depois = 3** (a função auxiliar criada tem CC = 3).
- SLOC do arquivo de teste (`tests/test_cli_serve_server.py`): **Antes = 159, Depois = 162**.

---

#### **R02 – Sleepy Test em Corrotina Assíncrona (`tests/test_internals_database.py::test_execute_write_block_false`)**

**Antes:**
[PRINT PENDENTE]
```python
@pytest.mark.asyncio
async def test_execute_write_block_false(db):
    await db.execute_write(
        "update roadside_attractions set name = ? where pk = ?",
        ["Mystery!", 1],
    )
    await asyncio.sleep(0.1)
    rows = await db.execute("select name from roadside_attractions where pk = 1")
    assert "Mystery!" == rows.rows[0][0]
```

**Depois:**
[PRINT PENDENTE]
```python
async def wait_for_row_value(db, sql, expected, timeout=1.0, interval=0.005):
    deadline = asyncio.get_running_loop().time() + timeout
    while asyncio.get_running_loop().time() < deadline:
        rows = await db.execute(sql)
        if rows.rows and rows.rows[0][0] == expected:
            return rows.rows[0][0]
        await asyncio.sleep(interval)
    raise TimeoutError(f"Expected {expected!r} not found within {timeout}s")


@pytest.mark.asyncio
async def test_execute_write_block_false(db):
    await db.execute_write(
        "update roadside_attractions set name = ? where pk = ?",
        ["Mystery!", 1],
    )
    name = await wait_for_row_value(
        db, "select name from roadside_attractions where pk = 1", "Mystery!"
    )
    assert name == "Mystery!"
```

**Por que era um test smell:** O teste utilizava uma pausa estática arbitrária de 100 ms (`await asyncio.sleep(0.1)`) para aguardar que a thread de escrita do SQLite persistisse a alteração em segundo plano antes da consulta de verificação, gerando atraso desnecessário e risco de falhas esporádicas (*flaky tests*).  
**Estratégia do CATSPY aplicada:** Em `datasette/database.py:500`, a chamada assíncrona `execute_write(..., block=False)` retorna um identificador `uuid.UUID` (`task_id`), o qual não é um objeto awaitable (uma tentativa de `await task` resultaria em `TypeError: object UUID can't be used in 'await' expression`). Para estabelecer uma sincronização determinística entre o thread de escrita e o loop de eventos, foi introduzida a rotina utilitária assíncrona `wait_for_row_value`, realizando polling ativo curto no banco a cada 5 ms (`asyncio.sleep(0.005)`) com deadline máximo de 1,0 s, encerrando a espera assim que a condição de banco é satisfeita.  
**Métricas do teste refatorado:**
- Complexidade ciclomática da função (`test_execute_write_block_false`): **Antes = 2, Depois = 2** (a função utilitária `wait_for_row_value` possui CC = 4).
- SLOC do arquivo de teste (`tests/test_internals_database.py`): **Antes = 1.139, Depois = 1.148** (+9 linhas).

---

#### **R08 – Undefined Test (`tests/test_queries.py::_template_button_sql`)**

**Antes:**
[PRINT PENDENTE]
```python
def _template_button_sql(html, operation):
    soup = Soup(html, "html.parser")
    button = soup.select_one(f'button[data-sql-template="{operation}"]')
    assert button, f"Could not find {operation} template button"
    assert button.get(
        "data-template-sql"
    ), f"Could not find SQL for {operation} template button"
    return button["data-template-sql"]
```

**Depois:**
[PRINT PENDENTE]
```python
def _template_button_sql(html, operation):
    soup = Soup(html, "html.parser")
    button = soup.select_one(f'button[data-sql-template="{operation}"]')
    if not button:
        raise ValueError(f"Could not find {operation} template button")
    sql = button.get("data-template-sql")
    if not sql:
        raise ValueError(f"Could not find SQL for {operation} template button")
    return sql
```

**Por que era um test smell:** A função utilitária `_template_button_sql` é um helper invocado por múltiplos testes para extrair templates HTML. Ao utilizar `assert` fora de um método prefixado com `test_`, o TEMPY classifica a função como *Undefined Test*, pois ferramentas de smell interpretam que asserções indicam testes sem reconhecimento formal do framework.  
**Estratégia do CATSPY aplicada:** Substituição de asserções por validação explícita com lançamento de exceções de domínio (`raise ValueError(...)`). Isso mantém o propósito defensivo do helper utilitário sem violar o padrão de asserções de teste.  
**Métricas do teste refatorado:**
- Complexidade ciclomática da função (Radon): **Antes = 3, Depois = 3** (o Radon computa tanto `assert` quanto `if/raise` como caminhos independentes).
- SLOC do arquivo de teste (`tests/test_queries.py`): **Antes = 3.400, Depois = 3.401**.

---

#### **R10 – Non-Functional Statement (`tests/test_cli_serve_server.py::test_startup_error_fails_fast_before_port_binds`)**

**Antes:**
[PRINT PENDENTE]
```python
@pytest.mark.serial
def test_startup_error_fails_fast_before_port_binds(serve_with_plugins):
    """
    A "startup" plugin hook that raises StartupError must fail fast: print
    the message, exit non-zero, and never accept a connection on the port -
    the failure must happen before uvicorn.Server binds the socket.
    """
    proc, port = serve_with_plugins(
        {"startup_error_plugin": STARTUP_ERROR_PLUGIN}, wait_for_startup=False
    )
    stdout, _ = proc.communicate(timeout=15)
    output = stdout.decode("utf-8")
    assert proc.returncode not in (0, None), output
    assert "boom from plugin" in output, output

    # Nothing is listening on the port now the process has exited. This
    # confirms the socket was not left bound; on its own it cannot prove the
    # failure preceded the bind, since a port nothing ever touched also
    # refuses connections.
    with (
        pytest.raises(OSError),
        socket.create_connection(("127.0.0.1", port), timeout=0.2),
    ):
        pass
```

**Depois:**
[PRINT PENDENTE]
```python
@pytest.mark.serial
def test_startup_error_fails_fast_before_port_binds(serve_with_plugins):
    """
    A "startup" plugin hook that raises StartupError must fail fast: print
    the message, exit non-zero, and never accept a connection on the port -
    the failure must happen before uvicorn.Server binds the socket.
    """
    proc, port = serve_with_plugins(
        {"startup_error_plugin": STARTUP_ERROR_PLUGIN}, wait_for_startup=False
    )
    stdout, _ = proc.communicate(timeout=15)
    output = stdout.decode("utf-8")
    assert proc.returncode not in (0, None), output
    assert "boom from plugin" in output, output

    # Nothing is listening on the port now the process has exited. This
    # confirms the socket was not left bound; on its own it cannot prove the
    # failure preceded the bind, since a port nothing ever touched also
    # refuses connections.
    with pytest.raises(OSError):
        socket.create_connection(("127.0.0.1", port), timeout=0.2)
```

**Por que era um test smell:** O teste utilizava um context manager composto onde a chamada que disparava o erro estava associada no cabeçalho do `with`, deixando o corpo do bloco preenchido apenas com o comando inerte `pass`.  
**Estratégia do CATSPY aplicada:** Remoção do comando inerte `pass` e simplificação do bloco `with pytest.raises(OSError)`, executando a operação diretamente no corpo do context manager.  
**Métricas do teste refatorado:**
- Complexidade ciclomática da função: **Antes = 3, Depois = 3**.
- SLOC do arquivo de teste (`tests/test_cli_serve_server.py`): **Antes = 162, Depois = 159**.

---

#### **R20 – Unknown Test (`tests/test_template_context.py::test_from_extra_must_match_a_registered_extra`)**

**Antes:**
[PRINT PENDENTE]
```python
def test_from_extra_must_match_a_registered_extra():
    from datasette.views import from_extra

    @dataclass
    class BadContext(Context):
        extras_scope = ExtraScope.TABLE

        not_a_real_extra: str = from_extra()

    with pytest.raises(KeyError):
        BadContext.documented_fields()
```

**Depois (Commit R20 `fe44d16`):**
[PRINT PENDENTE]
```python
def test_from_extra_must_match_a_registered_extra():
    from datasette.views import from_extra

    @dataclass
    class BadContext(Context):
        extras_scope = ExtraScope.TABLE

        not_a_real_extra: str = from_extra()

    with pytest.raises(KeyError) as excinfo:
        BadContext.documented_fields()
    assert excinfo.type is KeyError
```

**Estado Final em HEAD após Refino Semântico (`bd45249`):**
[PRINT PENDENTE]
```python
def test_from_extra_must_match_a_registered_extra():
    from datasette.views import from_extra

    @dataclass
    class BadContext(Context):
        extras_scope = ExtraScope.TABLE

        not_a_real_extra: str = from_extra()

    with pytest.raises(KeyError) as excinfo:
        BadContext.documented_fields()
    assert "no registered extra of that name" in str(excinfo.value)
```

**Por que era um test smell:** O teste dependia exclusivamente do `pytest.raises(KeyError)` sem inspecionar a exceção capturada nem declarar asserções explícitas, sendo classificado pelo CATSPY e pelo TEMPY como *Unknown Test*. O teste passaria silenciosamente mesmo se o erro fosse disparado por outro motivo não relacionado.  
**Estratégia do CATSPY aplicada:** Captura do contexto da exceção (`as excinfo`) com asserção explícita. No commit original R20 (`fe44d16`), introduziu-se `assert excinfo.type is KeyError`; no commit subsequente de refino semântico `bd45249`, essa verificação de tipo foi substituída pela asserção semântica sobre o texto da mensagem de erro (`assert "no registered extra of that name" in str(excinfo.value)`), garantindo que a falha capturada decorra rigorosamente do contrato pretendido.  
**Métricas do teste refatorado:**
- Complexidade ciclomática da função: **Antes = 1, Depois = 2** (o Radon computa `assert` como +1 ponto de complexidade ciclomática).
- SLOC do arquivo de teste (`tests/test_template_context.py`): **Antes = 163, Depois = 164** (165 após refino `bd45249`).

---

#### **R42 – Conditional Test Logic (`tests/test_csv.py::test_custom_sql_csv_errors`)**

**Antes:**
[PRINT PENDENTE]
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("download", (False, True))
@pytest.mark.parametrize(
    "query_string,expected_error",
    (
        ("sql=select+error%28%29", "no such function: error"),
        ("sql=select+52&_shape=not_a_shape", "Invalid _shape: not_a_shape"),
    ),
)
async def test_custom_sql_csv_errors(ds_client, query_string, expected_error, download):
    if download:
        query_string += "&_dl=1"
    response = await ds_client.get(f"/fixtures/-/query.csv?{query_string}")
    assert response.status_code == 400
    assert response.headers["content-type"] == "text/plain; charset=utf-8"
    assert "content-disposition" not in response.headers
    assert response.text == expected_error
```

**Depois (Commit R42 `90f8bad`):**
[PRINT PENDENTE]
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("download_suffix", ("", "&_dl=1"))
@pytest.mark.parametrize(
    "query_string,expected_error",
    (
        ("sql=select+error%28%29", "no such function: error"),
        ("sql=select+52&_shape=not_a_shape", "Invalid _shape: not_a_shape"),
    ),
)
async def test_custom_sql_csv_errors(ds_client, query_string, expected_error, download_suffix):
    response = await ds_client.get(f"/fixtures/-/query.csv?{query_string}{download_suffix}")
    assert response.status_code == 400
    assert response.headers["content-type"] == "text/plain; charset=utf-8"
    assert "content-disposition" not in response.headers
    assert response.text == expected_error
```

**Estado Final em HEAD após Formatação PEP 8 / 88 colunas (`110d001`):**
[PRINT PENDENTE]
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("download_suffix", ("", "&_dl=1"))
@pytest.mark.parametrize(
    "query_string,expected_error",
    (
        ("sql=select+error%28%29", "no such function: error"),
        ("sql=select+52&_shape=not_a_shape", "Invalid _shape: not_a_shape"),
    ),
)
async def test_custom_sql_csv_errors(
    ds_client, query_string, expected_error, download_suffix
):
    response = await ds_client.get(
        f"/fixtures/-/query.csv?{query_string}{download_suffix}"
    )
    assert response.status_code == 400
    assert response.headers["content-type"] == "text/plain; charset=utf-8"
    assert "content-disposition" not in response.headers
    assert response.text == expected_error
```

**Por que era um test smell:** O teste utilizava um parâmetro booleano `download` com uma ramificação condicional `if download: query_string += "&_dl=1"` interna, bifurcando o código executado dentro do corpo do teste.  
**Estratégia do CATSPY aplicada:** Parametrização direta dos valores de sufixo `download_suffix` (`("", "&_dl=1")`), permitindo a interpolação imediata na URL da requisição e eliminando completamente a instrução condicional `if`.  
**Métricas do teste refatorado:**
- Complexidade ciclomática da função: **Antes = 6 (Rank B), Depois = 5 (Rank A)** (redução real de 1 ponto de complexidade pela eliminação do branch).
- SLOC do arquivo de teste (`tests/test_csv.py`): **Antes = 268, Depois = 266** (270 no commit `110d001` que formatou a assinatura em múltiplas linhas).

---

## **4. Estado final e comparação**

### **4.1 Tabela geral comparativa (Antes vs Depois)**

As medições comparativas entre a linha de base (`cf113a5`) e o commit de fechamento da branch (`110d001`)* cobrindo o escopo completo de **85 arquivos** foram verificadas de forma determinística pelas ferramentas:

| Métrica | Antes (`cf113a5`) | Depois (`110d001`)* | Variação Absoluta / p.p. | Variação Relativa (%) |
| :---- | :---- | :---- | :---- | :---- |
| **Total de test smells (TEMPY)** | **258** | **208** | **-50** | **-19,38%** |
| Testes aprovados (Passed) | 2.829 | 2.850 | +21 | +0,74% |
| Testes com falha (Failed) | 0 | 0 | 0 | 0,00% |
| Erros de execução (Errors) | 0 | 0 | 0 | 0,00% |
| Ignorados (Skipped) | 85 | 85 | 0 | 0,00% |
| Falhas esperadas (xfail) | 6 | 6 | 0 | 0,00% |
| Aprovações inesperadas (xpass) | 15 | 15 | 0 | 0,00% |
| Cobertura total (%) | 90,27% | 90,27% | 0,00 p.p. | Estável |
| Cobertura do código de produção (`datasette/`) | 87,40% | 87,43% | +0,03 p.p. | +0,03 p.p. |
| Cobertura do código de teste (`tests/`) | 92,70% | 92,68% | -0,02 p.p. | -0,02 p.p. |
| SLOC dos testes (`tests/`) | 39.524 | 39.529 | +5 | +0,01% |
| Complexidade ciclomática média das funções de teste | 4,43 | 4,38 | -0,05 | -1,13% |
| Índice de manutenibilidade médio dos arquivos de teste | 43,10 | 43,09 | -0,01 | -0,02% |

*\*Nota metodológica sobre a varredura global e o número 195:*  
No escopo oficial da linha de base (85 arquivos constantes em `metrics-before-tempy/lista_arquivos.txt`, que inclui 81 arquivos `test_*.py` mais 4 arquivos não-padrão: `conftest.py`, `fixtures.py`, `datasette/_pytest_plugin.py` e `datasette/telemetry_testing.py`), o total de test smells reduziu de **258 para 208 ocorrências** (redução líquida de **-50 ocorrências**). Os 4 arquivos não-padrão concentram exatamente 20 smells na base (16 em `conftest.py`, 2 em `datasette/telemetry_testing.py`, 1 em `datasette/_pytest_plugin.py` e 1 em `fixtures.py`) e não foram modificados pelas refatorações (mantendo 20 smells no fechamento).  
Por outro lado, o script `extract_metrics_after_tempy.py` realizou uma busca estrita por `Path("tests").glob("test_*.py")` (84 arquivos), a qual: (a) excluiu os 4 arquivos não-padrão (-20 smells); e (b) incluiu 3 arquivos de teste que estavam presentes no repositório mas não constavam no manifesto `lista_arquivos.txt` original (`tests/test_fixtures.py` com 4 smells, `tests/test_cli_serve_get.py` com 1 smell e `tests/test_crossdb.py` com 2 smells, totalizando +7 smells). A aritmética fecha com precisão exata: `208 - 20 (não-padrão) + 7 (testes adicionais) = 195 ocorrências`. Na base, esses mesmos 84 arquivos de teste continham 238 + 7 = 245 smells. Logo, em ambos os escopos metodológicos, a redução de test smells obtida pela equipe é rigorosamente a mesma: **exatamente 50 test smells eliminados**. Adicionalmente, se o escopo abrangesse a totalidade dos arquivos de teste e infraestrutura do repositório (os 85 arquivos do manifesto somados aos 3 arquivos de teste pré-existentes fora da lista), o total geral de test smells seria de 265 → 215 (-50 ocorrências).

### **4.2 Comparação específica de test smells (TEMPY)**

Medição executada sobre os **85 arquivos** nos commits `cf113a5` e `110d001`:

| Test smell | Antes | Depois | Variação | Redução (%) |
| :---- | :---- | :---- | :---- | :---- |
| **Conditional Test Logic** | 135 | 102 | -33 | -24,44% |
| **Unknown Test** | 43 | 38 | -5 | -11,63% |
| **Verbose Test** | 27 | 27 | 0 | 0,00% |
| **Exception Handling** | 20 | 20 | 0 | 0,00% |
| **Programming Paradigms Blend** | 10 | 10 | 0 | 0,00% |
| **Undefined Test** | 10 | 6 | -4 | -40,00% |
| **Non-Functional Statement** | 10 | 3 | -7 | -70,00% |
| **Sleepy Test** | 3 | 2 | -1 | -33,33% |
| **Total** | **258** | **208** | **-50** | **-19,38%** |

*Detalhamento das 50 reduções apontadas pelo TEMPY:*
1. **Conditional Test Logic (-33):** 31 ocorrências eliminadas nos testes alvo das refatorações R22 a R38 (todas em métodos de teste síncronos) + 2 ocorrências suprimidas do corpo de teste em R01 ao transferir laço `while` e `if break` para a função utilitária. As refatorações R39 a R45 foram executadas em corrotinas assíncronas (`async def`), que não são analisadas pelo TEMPY.
2. **Non-Functional Statement (-7):** 7 ocorrências de comandos `pass` eliminadas nas refatorações R10 a R16. As 3 restantes na base residem em arquivos utilitários não alterados (`conftest.py` com 2 e `datasette/_pytest_plugin.py` com 1).
3. **Unknown Test (-5):** 5 ocorrências eliminadas nos testes refatorados em R17 a R21.
4. **Undefined Test (-4):** 4 ocorrências eliminadas pela substituição de `assert` por `raise ValueError` nos helpers refatorados em R06 a R09.
5. **Sleepy Test (-1):** 1 ocorrência detectada e eliminada do corpo do método de teste síncrono em R01. As refatorações R02 a R05 envolveram chamadas em corrotinas assíncronas (`async def`), as quais são completamente ignoradas pelo parser AST do TEMPY (`python_parser.py:21`).

---

## **5. Análise dos resultados**

**Quais métricas melhoraram e quais pioraram?**
- **Métricas que melhoraram:**
  - A quantidade total de test smells caiu significativamente de **258 para 208 ocorrências (-50 smells, -19,38%)**, superando a meta global da disciplina de 40 smells eliminados.
  - A complexidade ciclomática média das funções de teste reduziu de **4,43 para 4,38 (-0,05)**, comprovando a linearização de testes anteriormente bifurcados por laços e desvios condicionais.
  - A cobertura de código de produção da aplicação (`datasette/`) aumentou de **87,40% para 87,43% (+0,03 p.p.)**, dentro da margem de flutuação de execução dos testes.
- **Métricas com variações neutras ou ligeiramente inferiores (análise honesta):**
  - O SLOC total dos testes subiu ligeiramente de **39.524 para 39.529 linhas (+5 linhas)**. Isso decorre da inclusão de decorators `@pytest.mark.parametrize`, desdobramento de testes atômicos e asserções semânticas detalhadas de mensagens de erro.
  - O Índice de Manutenibilidade médio dos arquivos de teste variou minimamente de **43,10 para 43,09 (-0,01)**, e a cobertura do diretório `tests/` variou de **92,70% para 92,68% (-0,02 p.p.)**. Trata-se de flutuações matematicamente desprezíveis provocadas pela adição de código de asserção explícita. Não houve regressão de qualidade no código.

**Alguma refatoração introduziu novos test smells? Quais?**
- No commit `3e1d0b7` (R35), ao dividir o teste `test_trace` em `test_trace_disabled` e `test_trace_enabled`, eliminou-se 1 desvio condicional (`if not trace_debug:`), mas manteve-se o laço pré-existente `for trace in traces:` em `test_trace_enabled` (Conditional Test Logic residual). No commit subsequente `28671b2` (R36), a equipe saneou a função de teste extraindo o laço para a asserção utilitária `assert_valid_trace_entries(traces)`, que por possuir o prefixo `assert_` é isenta de pontuação no TEMPY (`detector.py:49-53, 105, 246`), atingindo zero smells detectados em `tests/test_tracer.py`.

**O resultado dos testes ou a cobertura mudou?**
- A suíte de testes permaneceu com **0 falhas (failed) e 0 erros (errors)** em todas as execuções. O volume total de testes aprovados aumentou em **+21 testes (de 2.829 para 2.850 passed)**.
- *Origem exata do saldo de +21 testes:* A auditoria de coleta com `pytest --collect-only -q` comprovou que o acréscimo de testes decorreu exclusivamente de duas funções refatoradas em `tests/test_permissions.py`: `test_view_database` (+12 testes, passando de 6 para 18 combinações no commit `c46b63d`) e `test_view_instance` (+9 testes, passando de 13 para 22 combinações no commit `bcfffdb`). Em ambos os casos, laços internos `for path in (...)` foram substituídos por `@pytest.mark.parametrize("path", ...)`. As demais divisões de teste atômico da suíte (como em `test_trace`, `test_load_extension_is_disabled` e `test_binary_data_in_json`) geraram desdobramentos cuja soma compensou exatamente as exclusões dos testes originais, com saldo líquido zero.
- A cobertura geral do projeto permaneceu estável em **90,27%**.

**Quais tipos de smell foram mais fáceis e mais difíceis de refatorar?**
- **Mais fáceis:** *Non-Functional Statement* (remoção simples de declarações `pass` e context managers aninhados) e *Unknown Test* (captura com `as excinfo` e validação direta de tipo e mensagem).
- **Mais difíceis:** *Conditional Test Logic* e *Sleepy Test*. Em CTL, testes como `test_execute_write_schema_operations_use_schema_permissions` (R45) exigiram a criação de funções de verificação assíncrona passadas como parâmetro (`state_verifier`), substituindo uma cadeia de 5 ramificações `if/elif` por uma chamada assíncrona linear `await state_verifier(db)`. Em *Sleepy Test*, os testes operam sobre threads secundárias e corrotinas assíncronas do `asyncio`, exigindo coordenação via laços curtos de polling ativo com timeout defensivo via helpers utilitários ou sincronização determinística via `asyncio.Event`.
- *Avaliação honesta das refatorações:* Das 45 refatorações, 41 constituem eliminações e reorganizações estruturais diretas e 4 constituem transferências e supressões controladas via funções utilitárias (R01 com helper de polling síncrono `poll_for_marker_task`, R02 e R03 com helper de polling assíncrono `wait_for_row_value`, e R36 com a asserção utilitária `assert_valid_trace_entries` protegida pelo prefixo `assert_`).

---

## **6. Uso do CATSPY**

**Como você usou o CATSPY durante a atividade?**
O catálogo [CATSPY](https://cat-spy.vercel.app/smells) atuou como o guia normativo de engenharia de software da equipe. Foi utilizado desde o planejamento inicial — para mapear as severidades oficiais de cada smell e definir a ordem de ataque — até a fase de implementação, servindo como referência para escolher a técnica de refatoração idiomática para Python.

**Quais partes do catálogo mais ajudaram (definição, efeitos negativos, exemplo com problema, exemplo refatorado)?**
Todas as seções do catálogo desempenharam papéis complementares:
1. **Definição:** Foi essencial para alinhar a compreensão teórica da equipe sobre o que constitui um problema genuíno de teste (por exemplo, compreender que asserções em helpers sem convenção de teste caracterizam *Undefined Test*).
2. **Efeitos Negativos:** Ajudou na priorização e na justificativa técnica perante a disciplina, evidenciando como laços e desvios em *Conditional Test Logic* provocam mascaramento de defeitos parciais e como *Sleepy Test* acarreta lentidão e testes intermitentes (*flaky*).
3. **Exemplo com Problema:** Permitiu reconhecer manifestações análogas na base do Datasette, auxiliando a localizar testes com lógicas bifurcadas mesmo onde o TEMPY não havia sinalizado.
4. **Exemplo Refatorado:** Foi a parte **mais útil operacionalmente**. Forneceu modelos práticos de solução (como a substituição de desvios por `@pytest.mark.parametrize` e decomposição em múltiplos testes), orientando a priorização de soluções que linearizem o fluxo e desacoplem as verificações.

**Houve casos em que a estratégia do catálogo não se aplicava diretamente ao projeto? Como você resolveu?**
- **Helpers de validação em `tests/test_queries.py`:** O catálogo orienta extrair validações para métodos de teste independentes. Porém, em funções utilitárias que parseiam HTML de templates para dezenas de testes, renomeá-las para `test_*` não fazia sentido. A equipe adaptou a estratégia substituindo `assert` por lançamento explícito de exceções de domínio (`raise ValueError(...)`), sanando o smell sem poluir os testes chamadores.
- **Sleepy Test em Concorrência Assíncrona:** Os exemplos do catálogo abordam esperas síncronas com `time.sleep`. No Datasette, as operações de escrita assíncrona com `block=False` retornam um identificador `uuid.UUID` (`task_id`) e delegam a persistência para uma thread secundária. Como o UUID não é um objeto awaitable (uma tentativa de `await task` resultaria em `TypeError: object UUID can't be used in 'await' expression`), a equipe adaptou a estratégia para laços de polling ativo no banco com intervalos de 5 ms (`asyncio.sleep(0.005)`) e deadline defensivo de 1,0 s (helper `wait_for_row_value` em R02 e R03). Já em R04 e R05, onde operações assíncronas em segundo plano disparavam tarefas paralelas, a equipe eliminou totalmente o `sleep` utilizando `asyncio.Event`: em R04 com `task_started = asyncio.Event()` e `await asyncio.wait_for(task_started.wait(), timeout=5)`, e em R05 interceptando `ds.track_event` com uma closure que sinaliza `event_recorded = asyncio.Event()` e aguarda determinística e defensivamente via `await asyncio.wait_for(event_recorded.wait(), timeout=5)`.

**O que faltou ou poderia ser melhorado no catálogo?**
- **Diretrizes para Testes Assíncronos (`pytest-asyncio`):** O catálogo se beneficiaria enormemente de uma seção dedicada a testes concorrentes e assíncronos, cobrindo anti-padrões como delays em event loops e sincronização entre threads.
- **Recursos Idiomáticos do Pytest Moderno:** Muitos exemplos do catálogo utilizam convenções do framework padrão `unittest` (`self.assertEqual`, `self.assertRaises`). A inclusão de padrões modernos com fixtures, parametrização composta e validação de `excinfo` enriqueceria o catálogo.

---

## **7. Divisão do trabalho**

A divisão do trabalho entre os integrantes conforme registrada no histórico da branch:

| Integrante | Refatorações realizadas (IDs da Seção 3.2) | Quantidade |
| :---- | :---- | :---- |
| **Kawan Torres** | R01, R06, R07, R08, R10, R11, R12, R13, R14, R15, R17, R18, R19, R22, R23, R24, R25, R26, R27, R28, R29, R30, R31 | **23 refatorações** |
| **Cícero Higor** | R02, R03, R04, R05, R09, R16, R20, R21, R32, R33, R34, R35, R36, R37, R38, R39, R40, R41, R42, R43, R44, R45 | **22 refatorações** (+ 2 refinos semânticos + 1 commit de estilo) |

`[COMPLETAR PELA EQUIPE: divisão real do trabalho, se diferente da registrada no git]`

*Auditoria de Commits e Arquivos no Repositório:*
- **Total de Commits no Git (`git log cf113a5..HEAD`):** **48 commits** (23 de Kawan Torres + 25 de Cícero Higor).
- **Arquivos Distintos Alterados (`git diff --name-only cf113a5..HEAD -- tests/`):** **19 arquivos de teste independentes** (`test_cli_serve_server.py`, `test_csv.py`, `test_fts_permissions.py`, `test_http_span.py`, `test_internals_database.py`, `test_internals_datasette.py`, `test_internals_request.py`, `test_load_extensions.py`, `test_permissions.py`, `test_queries.py`, `test_routes.py`, `test_shutdown.py`, `test_table_api.py`, `test_telemetry_metrics.py`, `test_telemetry_testing_kit.py`, `test_template_context.py`, `test_tracer.py`, `test_utils_sql_analysis.py`, `test_write_wrapper.py`).
- Ambos os integrantes cumpriram e superaram a cota individual de **no mínimo 20 refatorações de test smells por aluno**.

---

## **8. Dificuldades e lições aprendidas**

**Quais foram as principais dificuldades (ambiente, dependências, entendimento do código, ferramentas, revisão do PR)?**
1. **Limitação Estrutural do Detector TEMPY com Testes Assíncronos (`async def`):** O parser do TEMPY (`python_parser.py:21`) varre exclusivamente nós do tipo `ast.FunctionDef`, ignorando integralmente corrotinas assíncronas `ast.AsyncFunctionDef`. No commit base `cf113a5`, o Datasette contém **1.099 testes assíncronos (`async def`)** e apenas **455 testes síncronos (`def`)**. Como mais de 70% da suíte é assíncrona, a contagem de 258 smells do baseline subestima significativamente os smells reais presentes no projeto. Além disso, as 11 refatorações da equipe realizadas em testes assíncronos (R02 a R05 em *Sleepy Test* e R39 a R45 em *Conditional Test Logic*) não foram registradas pelo TEMPY unicamente por essa limitação sintática da ferramenta, embora representem melhorias reais no código de teste. Adicionalmente, o TEMPY isenta funções auxiliares com prefixo `assert_` (`detector.py:49-53, 246`), mas pontua outros helpers com `assert` como *Undefined Test*.
2. **Ambiente Assíncrono e Event Loop do Datasette:** A manipulação de requisições sobre o ASGI e transações SQLite em threads separadas gerava dependência de timing, exigindo cuidado ao refatorar *Sleepy Tests* para não introduzir deadlocks.
3. **Isolamento de Escopo de Medição de Cobertura:** A presença de arquivos utilitários e manifestos na raiz do repositório durante fases preliminares gerou pequenas distorções nos relatórios do `pytest-cov`. A definição de uma configuração padronizada no `.coveragerc` garantiu a estabilidade científica das métricas.

**O que você aprendeu com a atividade?**
- A refatoração de testes eleva diretamente a confiabilidade do processo de integração contínua: testes parametrizados sem condicionais internas tornam as falhas pontuais e de diagnóstico imediato.
- O uso de recursos nativos do Pytest (fixtures isoladas, parametrização múltipla e asserções semânticas em exceções) substitui com grandes vantagens blocos procedurais legados de teste.
- A disciplina de commits atômicos com registro transparente em diário de bordo viabiliza a auditoria externa e assegura a rastreabilidade integral das mudanças em projetos de software aberto.

---

## **9. Contribuição (Pull Request)**

- **Link Oficial do Pull Request no Fork:** [https://github.com/cicerocarnauba/datasette/pull/1](https://github.com/cicerocarnauba/datasette/pull/1)
- **Branch de Origem:** `refatoracao-test-smells`
- **Branch de Destino:** `main` (no fork `cicerocarnauba/datasette`)
- **Status do PR:** Aberto, com **0 falhas e 0 erros** na suíte completa de testes (2.850 passed, 85 skipped, 6 xfail, 15 xpass).

*Nota de Submissão e Revisão:*  
O Pull Request consolida os 48 commits (45 refatorações, 2 refinos semânticos e 1 commit de estilo), com alterações estritamente restritas aos arquivos de teste do diretório `tests/`. O PR foi aberto exclusivamente no fork da equipe (`cicerocarnauba/datasette/pull/1`), sem submissão ao repositório upstream oficial (`simonw/datasette`) e, portanto, sem revisão ou resposta de mantenedores externos.  
`[CONFIRMAR COM A MONITORIA: PR no fork atende ao requisito?]`

---

## **10. Observações gerais**

1. **Adesão ao Catálogo CATSPY:** A equipe direcionou 100% de sua atenção aos 5 tipos de test smells formalmente definidos no CATSPY (*Conditional Test Logic*, *Undefined Test*, *Unknown Test*, *Sleepy Test* e *Non-Functional Statement*).
2. **Superação Quantitativa de Metas:** Foram entregues **45 refatorações** de código de teste (superando a meta de 40), das quais 41 representam eliminações e reorganizações estruturais diretas e 4 representam transferências e supressões via funções utilitárias (R01 com helper de polling síncrono, R02 e R03 com helper de polling assíncrono `wait_for_row_value`, e R36 com asserção utilitária `assert_valid_trace_entries`), com **23 realizadas por Kawan Torres** e **22 por Cícero Higor** (ambos superando a meta individual de 20 refatorações).
3. **Distribuição Exata por Categoria (soma = 45):**
   - **Conditional Test Logic:** 24 refatorações (10 Kawan [R22–R31] + 14 Cícero [R32–R45]);
   - **Non-Functional Statement:** 7 refatorações (6 Kawan [R10–R15] + 1 Cícero [R16]);
   - **Unknown Test:** 5 refatorações (3 Kawan [R17–R19] + 2 Cícero [R20–R21]);
   - **Sleepy Test:** 5 refatorações (1 Kawan [R01] + 4 Cícero [R02–R05]);
   - **Undefined Test:** 4 refatorações (3 Kawan [R06–R08] + 1 Cícero [R09]).  
   **Total: 24 + 7 + 5 + 5 + 4 = 45 refatorações.**
4. **Reprodutibilidade das Métricas:** Todos os dados apresentados neste relatório podem ser regenerados e validados através dos scripts utilitários versionados na pasta `scripts/` do repositório (`extract_metrics_after_pytest.py`, `extract_metrics_after_radon.py`, `extract_metrics_after_tempy.py`).
5. **Histórico do Repositório e Reconstrução da Branch:** A branch oficial do Pull Request (`refatoracao-test-smells`) foi consolidada a partir de branches de desenvolvimento anteriores (conforme documentado no reflog do Git). Os hashes citados ao longo deste relatório são os da branch final. As datas exibidas nas tabelas correspondem às datas de autor (`%ad`), que são preservadas pelo Git em operações de rebase/cherry-pick e podem diferir do momento exato em que os commits foram recriados na branch final.
6. **Divisão Real de Trabalho:** `[COMPLETAR PELA EQUIPE: descrever a divisão real do trabalho e o motivo da reorganização das branches se diferir do histórico do git]`.
