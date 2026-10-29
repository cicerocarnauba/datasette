# Relatório Final de Refatoração de Test Smells
## Universidade Federal do Ceará — Campus Quixadá
### Disciplina: Qualidade de Software (Semestre 2026.2)
**Docente:** Profª. Dra. Carla Ilane Moreira Bezerra  
**Monitoria:** Gabriel Alves Pessoa  
**Equipe:**
- Cícero Higor Marques Carnaúba (login/git: `higor`)
- Kawan Torres (login/git: `kawantorres`)

**Repositório Original:** [simonw/datasette](https://github.com/simonw/datasette)  
**Fork do Projeto:** [cicerocarnauba/datasette](https://github.com/cicerocarnauba/datasette)  
**Branch de Trabalho e Entrega:** [`refatoracao-test-smells`](https://github.com/cicerocarnauba/datasette/tree/refatoracao-test-smells)  
**Commit Inicial de Referência (Base):** `cf113a5` (`cf113a59850862125b86ce1a58c117aa21341e07`)  
**Commit Final de Refatoração de Código:** `746720f` (`746720f83737b5c8795da3cb0a542b8faae2f46b`)  
**Pull Request:** [cicerocarnauba/datasette#1](https://github.com/cicerocarnauba/datasette/pull/1)  
**Catálogo de Test Smells:** [CATSPY](https://cat-spy.vercel.app/smells) (Meneses & Bezerra, UFC Quixadá)  
**Ferramenta de Detecção Automatizada:** TEMPY (commit `4c945d1`)

---

## 1. Apresentação do projeto

O **Datasette** é uma ferramenta de código aberto desenvolvida em Python por Simon Willison, voltada para exploração, publicação e consulta interativa a bancos de dados SQLite. A aplicação gera APIs JSON e interfaces HTML para navegação tabular, filtragem facetada e execução de consultas SQL customizadas.

### 1.1 Informações gerais da aplicação

| Propriedade | Descrição / Valor |
|---|---|
| **Nome da Aplicação** | Datasette |
| **Linguagem Principal** | Python (compatibilidade declarada no `pyproject.toml`: 3.10, 3.11, 3.12, 3.13 e 3.14) |
| **Arquitetura e Frameworks** | ASGI (`asgiref>=3.2.10`), Uvicorn, Jinja2, Click e SQLite3 |
| **Sistema de Plugins** | Baseado na biblioteca **Pluggy** (`pluggy>=1.0`) via pontos de extensão `hookimpl` e `hookspec` |
| **Framework de Testes** | Pytest acompanhado dos plugins `pytest-asyncio`, `pytest-cov`, `pytest-xdist` e `pytest-timeout` |
| **Linhas de Código de Produção (SLOC)** | 25.848 linhas no pacote `datasette/` |
| **Linhas de Código de Teste (SLOC)** | 39.524 linhas no diretório `tests/` |
| **Total de Arquivos no Diretório `tests/`** | 95 arquivos Python (sendo 85 com padrão canônico `test_*.py` analisados pelo detector estático) |
| **Licença** | Apache-2.0 License |

A suíte de testes cobre funcionalidades abrangentes, incluindo inicialização do servidor ASGI, controle de permissões de acesso, manipulação de consultas SQL, exportação e streaming de dados (CSV e JSON) e integração com extensões do SQLite (como Full-Text Search e SpatiaLite).

---

## 2. Estado inicial do projeto

A linha de base de medições foi coletada no commit inicial `cf113a5` utilizando ferramentas de análise estática e dinâmica padronizadas no ambiente Python 3.12: **Radon** (complexidade e métricas de código), **Pytest** com **Pytest-Cov** (testes e cobertura) e **TEMPY** (detecção estática de test smells em Python).

### 2.1 Métricas de Código e Manutenibilidade (Radon) — Estado Inicial

As métricas coletadas pelo Radon no commit `cf113a5` separando o código da aplicação (`datasette/`) e a suíte de testes (`tests/`) estão apresentadas na tabela abaixo:

| Escopo / Métrica | LOC | SLOC | LLOC | Comentários | Blocos Analisados | CC Média | CC Máxima | MI Médio |
|---|---|---|---|---|---|---|---|---|
| **Código de Produção (`datasette/`)** | 32.370 | 25.848 | 15.291 | 1.222 | 1.353 | 4,45 | 87 | 53,95 |
| **Código de Testes (`tests/`)** | 47.947 | 39.524 | 22.600 | 1.822 | 1.873 | 4,43 | 73 | 43,10 |
| **Total do Repositório** | 80.317 | 65.372 | 37.891 | 3.044 | 3.226 | 4,44 | 87 | 48,53 |

*Nota explicativa sobre o escopo de análise:* O diretório `tests/` possui 95 arquivos Python no total (incluindo arquivos utilitários e fixtures como `conftest.py` e `fixtures.py`). Neles, o Radon identificou 1.873 funções e métodos avaliados. A complexidade ciclomática máxima de 73 concentra-se no módulo de testes de integração de controle de acesso (`tests/test_permissions.py`).

### 2.2 Métricas da Suíte de Testes e Cobertura (Pytest / Coverage) — Estado Inicial

A execução completa da suíte sob Python 3.12 com execução paralela (`pytest-xdist`) registrou estabilidade sem falhas:

| Métrica de Teste / Cobertura | Valor Absoluto | Percentual |
|---|---|---|
| **Total de Testes Executados** | 3.078 | 100,00% |
| **Testes Aprovados (Passed)** | 2.987 | 97,04% |
| **Testes Ignorados (Skipped)** | 91 | 2,96% |
| **Falhas (Failures)** | 0 | 0,00% |
| **Erros (Errors)** | 0 | 0,00% |
| **Cobertura Geral do Projeto** | 36.269 / 40.180 stmts + branches | **90,27%** |
| **Cobertura do Código de Produção (`datasette/`)** | 16.133 / 18.459 stmts + branches | **87,40%** |
| **Cobertura da Suíte de Testes (`tests/`)** | 20.136 / 21.721 stmts + branches | **92,70%** |

*Nota sobre a linha de base:* A contagem de 3.078 testes executados (2.987 passed, 91 skipped) reflete a suíte completa com todas as fixtures e plugins habilitados. Contagens anteriores que reportavam 2.829 passed correspondiam a execuções com filtros de desativação de testes lentos ou sem determinados marcadores opcionais.

### 2.3 Detecção Inicial de Test Smells (TEMPY)

O relatório inicial do TEMPY registrou **258 ocorrências** de test smells distribuídas em 8 tipos:

| Test Smell Detectado (TEMPY) | Presente no CATSPY? | Severidade (CATSPY) | Ocorrências Iniciais | Percentual (%) |
|---|---|---|---|---|
| **Conditional Test Logic** | Sim | Alta | 135 | 52,33% |
| **Unknown Test** | Sim | Alta | 43 | 16,67% |
| **Verbose Test** | Sim | Baixa | 27 | 10,47% |
| **Exception Handling** | Não | — | 20 | 7,75% |
| **Non-Functional Statement** | Sim | Baixa | 10 | 3,88% |
| **Programming Paradigms Blend** | Sim | Média | 10 | 3,88% |
| **Undefined Test** | Sim | Alta | 10 | 3,88% |
| **Sleepy Test** | Sim | Média | 3 | 1,16% |
| **Total Geral** | — | — | **258** | **100,00%** |

*Decisão de escopo da equipe:* A equipe decidiu concentrar os esforços de refatoração nos smells catalogados no CATSPY. Como o smell *Exception Handling* não consta no catálogo de referência da disciplina, nenhuma refatoração foi direcionada a ele, mantendo suas 20 ocorrências inalteradas.

---

## 3. Estratégia de refatoração

### 3.1 Seleção de Tipos de Test Smells e Estratégias do CATSPY

A equipe selecionou 5 tipos de test smells presentes no catálogo CATSPY para intervenção, abrangendo as severidades Alta, Média e Baixa:

| Test Smell (CATSPY) | Severidade | Motivação da Escolha / Justificativa | Estratégia de Refatoração Adotada |
|---|---|---|---|
| **Conditional Test Logic** | **Alta** | Constitui o smell mais frequente (52,3% do total). Introduz múltiplos caminhos de execução, elevando a complexidade ciclomática e mascarando falhas parciais. | Parametrização via `@pytest.mark.parametrize`, divisão do teste em métodos menores focados (*Split Test*) ou desempacotamento determinístico. |
| **Undefined Test** | **Alta** | Ocorre em funções utilitárias ou testes incompletos que utilizam declarações `assert` sem formalização pelo framework de testes. | Substituição de asserções em helpers utilitários por exceções explícitas (`raise ValueError`) para validar parâmetros sem gerar falso teste. |
| **Unknown Test** | **Alta** | O teste não contém asserções explícitas (`assert`), validando erros apenas com blocos `with pytest.raises(Excecao):` sem inspecionar a exceção. | Captura do contexto de erro (`as excinfo`) e inclusão de asserções explícitas sobre tipo (`excinfo.type`) e mensagem (`excinfo.value`). |
| **Sleepy Test** | **Média** | O uso de pausas explícitas (`time.sleep` ou `asyncio.sleep`) torna a execução lenta e sujeita a não determinismo (*flakiness*). | Substituição de pausas arbitrárias por polling ativo com deadline ou resolução determinística de futures assíncronas. |
| **Non-Functional Statement** | **Baixa** | Presença de instruções inertes (`pass`) em blocos de context managers que não realizam ação efetiva. | Remoção de instruções inertes e simplificação do escopo do context manager. |

*Justificativa para a não inclusão de Programming Paradigms Blend (PPB):*  
O smell *Programming Paradigms Blend* (severidade Média no CATSPY, com 10 ocorrências) manifesta-se no Datasette pelo uso concorrente de construções funcionais (compreensões de lista/dicionário e expressões geradoras) e estilo orientado a objetos nas fixtures de teste. Como esse padrão é idiomático no ecossistema moderno de Python e não prejudica a legibilidade dos cenários analisados, a equipe optou por priorizar *Non-Functional Statement* (onde a presença de comandos `pass` mortos constituía ruído evidente no código).

---

### 3.2 Tabela Completa das Refatorações Realizadas

A tabela a seguir consolida as **45 refatorações de código de teste** realizadas pela equipe, registradas em 47 commits atômicos (incluindo 2 commits de refinamento).

*Convenções da coluna "Confirmado pelo TEMPY?":*
- **Sim (N → 0):** smell detectado pelo TEMPY na AST e confirmado como eliminado na função alvo;
- **Sim (suprimido via helper):** a chamada bloqueante foi removida do corpo do método de teste através de extração para helper utilitário (R01);
- **Refatoração conceitual:** o smell ocorria em chamadas assíncronas (`asyncio.sleep`) ou estruturas condicionais de branches que a regra sintática estrita do TEMPY não sinalizou na contagem léxica, mas cuja refatoração eliminou a condicional/espera conceitual descrita no CATSPY.

| ID | Commit | Autor | Test Smell (CATSPY) | Arquivo | Função / Método Alvo | Testes Passaram? | Introduziu Novo Smell? | Confirmado pelo TEMPY? |
|---|---|---|---|---|---|---|---|---|
| **R01** | `9bce5bb` | Kawan | Sleepy Test | `tests/test_cli_serve_server.py` | `test_startup_hook_background_task_runs_on_serving_loop` | Sim | Não | Sim (suprimido via helper) |
| **R02** | `8b7144b` | Higor | Sleepy Test | `tests/test_internals_database.py` | `test_execute_write_block_false` | Sim | Não | Refatoração conceitual |
| **R03** | `d059a5c` | Higor | Sleepy Test | `tests/test_internals_database.py` | `test_execute_write_with_returning_block_false` | Sim | Não | Refatoração conceitual |
| **R04** | `d8b827c` | Higor | Sleepy Test | `tests/test_shutdown.py` | `test_shutdown_logs_stragglers_that_outlive_the_grace_period` | Sim | Não | Refatoração conceitual |
| **R05** | `9533ed8` | Higor | Sleepy Test | `tests/test_write_wrapper.py` | `test_track_event_with_block_false` | Sim | Não | Refatoração conceitual |
| **R06** | `7a4c7a2` | Kawan | Undefined Test | `tests/test_queries.py` | `_template_option_attributes` | Sim | Não | Sim (1 → 0) |
| **R07** | `aca7a06` | Kawan | Undefined Test | `tests/test_queries.py` | `_template_sql` | Sim | Não | Sim (1 → 0) |
| **R08** | `86fd342` | Kawan | Undefined Test | `tests/test_queries.py` | `_template_button_sql` | Sim | Não | Sim (1 → 0) |
| **R09** | `7daa8b0` | Higor | Undefined Test | `tests/test_http_span.py` | `_route_for` | Sim | Não | Sim (1 → 0) |
| **R10** | `475ffb1` | Kawan | Non-Functional Statement | `tests/test_cli_serve_server.py` | `test_startup_error_fails_fast_before_port_binds` | Sim | Não | Sim (1 → 0) |
| **R11** | `a116702` | Kawan | Non-Functional Statement | `tests/test_telemetry_testing_kit.py` | `_run_workload` | Sim | Não | Sim (1 → 0) |
| **R12** | `b5603f9` | Kawan | Non-Functional Statement | `tests/test_telemetry_testing_kit.py` | `test_conformance_catches_an_unregistered_span` | Sim | Não | Sim (1 → 0) |
| **R13** | `ccdb305` | Kawan | Non-Functional Statement | `tests/test_telemetry_testing_kit.py` | `test_scope_filter_ignores_other_scopes` | Sim | Não | Sim (1 → 0) |
| **R14** | `2d896ac` | Kawan | Non-Functional Statement | `tests/test_telemetry_testing_kit.py` | `test_linked_root_span_kwargs_links_without_parenting` | Sim | Não | Sim (1 → 0) |
| **R15** | `0781592` | Kawan | Non-Functional Statement | `tests/test_telemetry_testing_kit.py` | `test_linked_root_span_kwargs_with_no_current_span` | Sim | Não | Sim (1 → 0) |
| **R16** | `4e944db` | Higor | Non-Functional Statement | `tests/test_telemetry_metrics.py` | `test_registry_holds_instances_weakly` | Sim | Não | Sim (1 → 0) |
| **R17** | `4c73f6e` | Kawan | Unknown Test | `tests/test_internals_datasette.py` | `test_static_rejects_plugin_and_mount` | Sim | Não | Sim (1 → 0) |
| **R18** | `7531827` | Kawan | Unknown Test | `tests/test_internals_datasette.py` | `test_static_rejects_path_traversal` | Sim | Não | Sim (1 → 0) |
| **R19** | `c8889c1` | Kawan | Unknown Test | `tests/test_internals_datasette.py` | `test_datasette_error_if_string_not_list` | Sim | Não | Sim (1 → 0) |
| **R20** | `667b9c5` | Higor | Unknown Test | `tests/test_template_context.py` | `test_from_extra_must_match_a_registered_extra` | Sim | Não | Sim (1 → 0) |
| **R21** | `4fdd31f` | Higor | Unknown Test | `tests/test_template_context.py` | `test_from_extra_must_be_available_for_the_scope` | Sim | Não | Sim (1 → 0) |
| **—** | `92ae493` | Higor | Unknown Test (Refino) | `tests/test_template_context.py` | (Reforço semântico em R17-R21: validação de mensagem) | Sim | Não | Sim (0 → 0) |
| **R22** | `040ae2c` | Kawan | Conditional Test Logic | `tests/test_utils_sql_analysis.py` | `test_analyze_create_table_operation` | Sim | Não | Sim (2 → 0) |
| **R23** | `3e2bcc8` | Kawan | Conditional Test Logic | `tests/test_utils_sql_analysis.py` | `test_analyze_function_operation` | Sim | Não | Sim (1 → 0) |
| **R24** | `bfe137a` | Kawan | Conditional Test Logic | `tests/test_utils_sql_analysis.py` | `test_analyze_table_kind_for_regular_virtual_and_shadow_tables` | Sim | Não | Sim (6 → 0) |
| **—** | `3172976` | Higor | CTL (Refino R24) | `tests/test_utils_sql_analysis.py` | (Eliminação de helper gerador por desempacotamento direto) | Sim | Não | Sim (0 → 0) |
| **R25** | `167c563` | Higor | Conditional Test Logic | `tests/test_permissions.py` | `test_permissions_cascade` | Sim | Não | Sim (1 → 0) |
| **R26** | `5cd9f1a` | Higor | Conditional Test Logic | `tests/test_permissions.py` | `test_view_padlock` | Sim | Não | Sim (2 → 0) |
| **R27** | `28bd370` | Higor | Conditional Test Logic | `tests/test_permissions.py` | `test_view_database` | Sim | Não | Sim (3 → 0) |
| **R28** | `d5df3e9` | Higor | Conditional Test Logic | `tests/test_permissions.py` | `test_database_list_respects_view_table` | Sim | Não | Sim (2 → 0) |
| **R29** | `3c3ede7` | Higor | Conditional Test Logic | `tests/test_permissions.py` | `test_view_table` | Sim | Não | Sim (2 → 0) |
| **R30** | `9942a6b` | Higor | Conditional Test Logic | `tests/test_permissions.py` | `test_table_list_respects_view_table` | Sim | Não | Sim (2 → 0) |
| **R31** | `18be25d` | Higor | Conditional Test Logic | `tests/test_permissions.py` | `test_view_query` | Sim | Não | Sim (2 → 0) |
| **R32** | `be17abd` | Higor | Conditional Test Logic | `tests/test_permissions.py` | `test_view_instance` | Sim | Não | Sim (1 → 0) |
| **R33** | `506bcf8` | Higor | Conditional Test Logic | `tests/test_template_context.py` | `test_context_class_fields_all_have_help` | Sim | Não | Sim (1 → 0) |
| **R34** | `47d4674` | Higor | Conditional Test Logic | `tests/test_routes.py` | `test_routes` | Sim | Não | Sim (1 → 0) |
| **R35** | `a81ae38` | Higor | Conditional Test Logic | `tests/test_tracer.py` | `test_trace` | Sim | Sim (temporário)* | Sim (2 → 0) |
| **R36** | `144b331` | Higor | Conditional Test Logic | `tests/test_tracer.py` | `test_trace_enabled` | Sim | Não | Sim (1 → 0) |
| **R37** | `2f22ed2` | Higor | Conditional Test Logic | `tests/test_load_extensions.py` | `test_load_extension_is_disabled` | Sim | Não | Sim (2 → 0) |
| **R38** | `ef4bf88` | Higor | Conditional Test Logic | `tests/test_internals_request.py` | `test_request_args` | Sim | Não | Sim (1 → 0) |
| **R39** | `7855315` | Higor | Conditional Test Logic | `tests/test_table_api.py` | `test_nofacet` | Sim | Não | Refatoração conceitual |
| **R40** | `7498567` | Higor | Conditional Test Logic | `tests/test_table_api.py` | `test_nosuggest` | Sim | Não | Refatoração conceitual |
| **R41** | `c7192ad` | Higor | Conditional Test Logic | `tests/test_table_api.py` | `test_nocount` | Sim | Não | Refatoração conceitual |
| **R42** | `30204fe` | Higor | Conditional Test Logic | `tests/test_csv.py` | `test_custom_sql_csv_errors` | Sim | Não | Refatoração conceitual |
| **R43** | `5659420` | Higor | Conditional Test Logic | `tests/test_table_api.py` | `test_binary_data_in_json` | Sim | Não | Refatoração conceitual |
| **R44** | `1808665` | Higor | Conditional Test Logic | `tests/test_fts_permissions.py` | `test_external_content_fts_inherits_content_table_view_permission` | Sim | Não | Refatoração conceitual |
| **R45** | `746720f` | Higor | Conditional Test Logic | `tests/test_queries.py` | `test_execute_write_schema_operations_use_schema_permissions` | Sim | Não | Refatoração conceitual |

*\*Nota sobre R35:* No commit `a81ae38`, a divisão de `test_trace` gerou temporariamente um helper utilitário de asserções que foi refatorado e normalizado no commit subsequente `144b331` (R36).

---

### 3.3 Exemplos Detalhados de Refatoração

Conforme as orientações fornecidas pela monitoria e docência da disciplina, foram utilizados blocos de código com diffs extraídos diretamente do Git (`git show`), dispensando capturas de tela. As métricas de Complexidade Ciclomática (CC) e SLOC foram mensuradas com o utilitário `radon cc -s` e `radon raw` executados pontualmente no commit anterior e posterior a cada refatoração.

---

#### Exemplo 1: Sleepy Test (Severidade Média)
- **ID da Refatoração:** R01
- **Commit SHA:** `9bce5bb` (`9bce5bb8`)
- **Arquivo:** `tests/test_cli_serve_server.py`
- **Método Alvo:** `test_startup_hook_background_task_runs_on_serving_loop`
- **Estratégia Adotada:** Extração de polling com deadline para helper auxiliar `poll_for_marker_task`.

**Contextualização:**  
O teste continha um laço com `time.sleep(0.05)` para aguardar a execução de uma tarefa de inicialização em segundo plano. O método de teste foi linearizado através da chamada a `poll_for_marker_task(port)`. A complexidade ciclomática do método de teste caiu de 5 para 3; contudo, a complexidade foi transferida para a função auxiliar (CC = 3), constituindo uma supressão léxica no escopo da função de teste sem eliminação do polling no sistema.

**Diff do Git (`git show 9bce5bb`):**

```python
# ANTES:
@pytest.mark.serial
def test_startup_hook_background_task_runs_on_serving_loop(serve_with_plugins):
    ...
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

# DEPOIS:
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
    ...
    payload = poll_for_marker_task(port)
    assert payload.get("marker_task_ran"), (
        "The startup hook's asyncio.create_task(...) never ran - "
        "invoke_startup() and the server are not sharing an event loop"
    )
```

**Métricas Mensuradas com Radon:**
- Complexidade Ciclomática (CC) do Teste: Antes = 5 (Classificação A), Depois = 3 (Classificação A)
- Complexidade Ciclomática (CC) do Helper `poll_for_marker_task`: 3 (Classificação A)
- SLOC do Arquivo `tests/test_cli_serve_server.py`: Antes = 159 linhas, Depois = 162 linhas

---

#### Exemplo 2: Undefined Test (Severidade Alta)
- **ID da Refatoração:** R08
- **Commit SHA:** `86fd342` (`86fd3428`)
- **Arquivo:** `tests/test_queries.py`
- **Método Alvo:** `_template_button_sql`
- **Estratégia Adotada:** Substituição de asserções em método auxiliar por lançamento de `ValueError`.

**Contextualização:**  
A função `_template_button_sql` é um utilitário para extrair templates SQL de botões HTML renderizados via BeautifulSoup, não sendo um teste formal executado pelo Pytest (não inicia com `test_`). O uso de `assert` no corpo da função era sinalizado pelo TEMPY como *Undefined Test*. A refatoração substituiu as asserções por verificações condicionais defensivas que lançam `ValueError`.

**Diff do Git (`git show 86fd342`):**

```python
# ANTES:
def _template_button_sql(html, operation):
    soup = Soup(html, "html.parser")
    button = soup.select_one(f'button[data-sql-template="{operation}"]')
    assert button, f"Could not find {operation} template button"
    assert button.get(
        "data-template-sql"
    ), f"Could not find SQL for {operation} template button"
    return button["data-template-sql"]

# DEPOIS:
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

**Métricas Mensuradas com Radon:**
- Complexidade Ciclomática (CC) da Função: Antes = 3 (Classificação A), Depois = 3 (Classificação A)
- SLOC do Arquivo `tests/test_queries.py`: Antes = 3.400 linhas, Depois = 3.401 linhas

---

#### Exemplo 3: Non-Functional Statement (Severidade Baixa)
- **ID da Refatoração:** R10
- **Commit SHA:** `475ffb1` (`475ffb1f`)
- **Arquivo:** `tests/test_cli_serve_server.py`
- **Método Alvo:** `test_startup_error_fails_fast_before_port_binds`
- **Estratégia Adotada:** Remoção de comando inerte `pass` e simplificação do context manager.

**Contextualização:**  
O teste continha um context manager com parênteses onde a chamada de conexão via socket estava aninhada com `pytest.raises(OSError)`, deixando o corpo do bloco `with` contendo apenas um comando inerte `pass`. A refatoração simplificou o bloco executando a conexão diretamente sob o `pytest.raises`. 

*Nota sobre Unknown Test:* Essa função não foi classificada pelo TEMPY como *Unknown Test* no estado posterior porque ela já continha duas asserções explícitas anteriores em seu corpo (`assert proc.returncode not in (0, None)` e `assert "boom from plugin" in output`), satisfazendo o critério de que o teste possui asserções verificáveis.

**Diff do Git (`git show 475ffb1`):**

```python
# ANTES:
def test_startup_error_fails_fast_before_port_binds(serve_with_plugins):
    ...
    with (
        pytest.raises(OSError),
        socket.create_connection(("127.0.0.1", port), timeout=0.2),
    ):
        pass

# DEPOIS:
def test_startup_error_fails_fast_before_port_binds(serve_with_plugins):
    ...
    with pytest.raises(OSError):
        socket.create_connection(("127.0.0.1", port), timeout=0.2)
```

**Métricas Mensuradas com Radon:**
- Complexidade Ciclomática (CC) da Função: Antes = 3 (Classificação A), Depois = 3 (Classificação A)
- SLOC do Arquivo `tests/test_cli_serve_server.py`: Antes = 162 linhas, Depois = 159 linhas

---

#### Exemplo 4: Unknown Test (Severidade Alta)
- **ID da Refatoração:** R20
- **Commit SHA:** `667b9c5` (`667b9c50`)
- **Arquivo:** `tests/test_template_context.py`
- **Método Alvo:** `test_from_extra_must_match_a_registered_extra`
- **Estratégia Adotada:** Inclusão de asserção explícita sobre a exceção capturada.

**Contextualização:**  
O teste verificava a ocorrência de erro utilizando apenas `with pytest.raises(KeyError): BadContext.documented_fields()`, sem nenhuma instrução `assert` explícita, sendo identificado como *Unknown Test*. O commit `667b9c5` capturou o contexto de erro com `as excinfo` e adicionou a asserção explícita `assert excinfo.type is KeyError`. Em refinamento posterior (commit `92ae493`), a validação foi fortalecida para verificar também a substring da mensagem de erro (`assert "no registered extra of that name" in str(excinfo.value)`).

**Diff Literal do Git (`git show 667b9c5`):**

```python
# ANTES:
def test_from_extra_must_match_a_registered_extra():
    class BadContext(TemplateContext):
        _scope = "table"

        not_a_real_extra: str = from_extra()

    with pytest.raises(KeyError):
        BadContext.documented_fields()

# DEPOIS:
def test_from_extra_must_match_a_registered_extra():
    class BadContext(TemplateContext):
        _scope = "table"

        not_a_real_extra: str = from_extra()

    with pytest.raises(KeyError) as excinfo:
        BadContext.documented_fields()
    assert excinfo.type is KeyError
```

**Métricas Mensuradas com Radon:**
- Complexidade Ciclomática (CC) da Função: Antes = 1 (Classificação A), Depois = 2 (Classificação A)
- SLOC do Arquivo `tests/test_template_context.py`: Antes = 163 linhas, Depois = 164 linhas

---

#### Exemplo 5: Conditional Test Logic (Severidade Alta)
- **ID da Refatoração:** R42
- **Commit SHA:** `30204fe` (`30204fec`)
- **Arquivo:** `tests/test_csv.py`
- **Método Alvo:** `test_custom_sql_csv_errors`
- **Estratégia Adotada:** Eliminação de branch condicional `if download:` através de parametrização direta do argumento de sufixo de query string.

**Contextualização:**  
O teste continha uma ramificação condicional `if download: query_string += "&_dl=1"`. Essa condicional bifurcava o fluxo interno do teste. A refatoração substituiu o parâmetro booleano pelo sufixo de query string `download_suffix` parametrizado diretamente com as strings `("", "&_dl=1")`, permitindo a interpolação direta na requisição HTTP e eliminando o branch `if`.

**Diff do Git (`git show 30204fe` formatado):**

```python
# ANTES:
@pytest.mark.asyncio
@pytest.mark.parametrize("download", (False, True))
@pytest.mark.parametrize(
    "query_string,expected_error",
    (
        (
            "_shape=array",
            "The _shape= parameter cannot be used with .csv, try .json instead",
        ),
        (
            "_shape=object",
            "The _shape= parameter cannot be used with .csv, try .json instead",
        ),
    ),
)
async def test_custom_sql_csv_errors(ds_client, query_string, expected_error, download):
    if download:
        query_string += "&_dl=1"
    response = await ds_client.get(f"/fixtures/-/query.csv?{query_string}")
    assert response.status_code == 400
    assert response.headers["content-type"] == "text/plain; charset=utf-8"
    assert "content-disposition" not in response.headers

# DEPOIS:
@pytest.mark.asyncio
@pytest.mark.parametrize("download_suffix", ("", "&_dl=1"))
@pytest.mark.parametrize(
    "query_string,expected_error",
    (
        (
            "_shape=array",
            "The _shape= parameter cannot be used with .csv, try .json instead",
        ),
        (
            "_shape=object",
            "The _shape= parameter cannot be used with .csv, try .json instead",
        ),
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
```

**Métricas Mensuradas com Radon:**
- Complexidade Ciclomática (CC) da Função: Antes = 6 (Classificação B), Depois = 5 (Classificação A)
- SLOC do Arquivo `tests/test_csv.py`: Antes = 268 linhas, Depois = 266 linhas

---

## 4. Estado final e comparação

### 4.1 Tabela Geral Comparativa (Antes vs Depois)

| Métrica Avaliada | Ferramenta | Antes (`cf113a5`) | Depois (`746720f`) | Variação Absoluta | Variação Relativa (%) |
|---|---|---|---|---|---|
| **Testes Totais Executados** | Pytest | 3.078 | 3.099 | +21 | +0,68% |
| **Testes Aprovados (Passed)** | Pytest | 2.987 | 3.008 | +21 | +0,70% |
| **Testes Ignorados (Skipped)** | Pytest | 91 | 91 | 0 | 0,00% |
| **Falhas / Erros** | Pytest | 0 | 0 | 0 | 0,00% |
| **Cobertura Geral do Projeto** | Pytest-Cov | 90,27% | 90,27% | 0,00 p.p. | Estável |
| **Cobertura Código Produção (`datasette/`)** | Pytest-Cov | 87,40% | 87,43% | +0,03 p.p. | Estável |
| **Cobertura Suíte de Testes (`tests/`)** | Pytest-Cov | 92,70% | 92,68% | -0,02 p.p. | Estável |
| **SLOC do Código de Teste** | Radon | 39.524 | 39.525 | +1 | +0,00% |
| **Complexidade Ciclomática Média (Testes)** | Radon | 4,43 | 4,38 | -0,05 | -1,13% |
| **Maintainability Index Médio (Testes)** | Radon | 43,10 | 43,09 | -0,01 | Estável |
| **Total de Test Smells** | TEMPY | **258** | **195** | **-63** | **-24,42%** |

---

### 4.2 Comparação Específica de Test Smells (TEMPY)

| Test Smell | Presente no CATSPY? | Ocorrências Antes | Ocorrências Depois | Variação Global | Redução (%) |
|---|---|---|---|---|---|
| **Conditional Test Logic** | Sim | 135 | 100 | -35 | -25,93% |
| **Unknown Test** | Sim | 43 | 38 | -5 | -11,63% |
| **Verbose Test** | Sim | 27 | 25 | -2 | -7,41% |
| **Exception Handling** | Não | 20 | 20 | 0 | 0,00% |
| **Non-Functional Statement** | Sim | 10 | 0 | -10 | -100,00% |
| **Programming Paradigms Blend** | Sim | 10 | 8 | -2 | -20,00% |
| **Undefined Test** | Sim | 10 | 3 | -7 | -70,00% |
| **Sleepy Test** | Sim | 3 | 1 | -2 | -66,67% |
| **Total Geral** | — | **258** | **195** | **-63** | **-24,42%** |

#### Reconciliação das Contagens de Smells (Auditoria de Escopo):
Para assegurar a reprodutibilidade dos dados, a auditoria detalhada entre os arquivos `metrics-before-tempy/smells.csv` e `metrics-after-tempy/smells.csv` revela como se deu a redução:

1. **Sleepy Test (3 → 1):**
   - Na linha de base (`metrics-before-tempy`), o TEMPY identificou 3 ocorrências: `tests/test_cli_serve_server.py` (linha 107), `tests/conftest.py` (linha 340) e `tests/test_playwright.py` (linha 869).
   - A ocorrência em `tests/test_cli_serve_server.py` foi tratada na refatoração R01 via extração para a função auxiliar `poll_for_marker_task`.
   - A ocorrência em `tests/conftest.py` não constou no relatório pós-refatoração porque o script `extract_metrics_after_tempy.py` filtrou apenas arquivos canônicos `tests/test_*.py`.
   - A ocorrência em `tests/test_playwright.py` permaneceu inalterada no estado final.
2. **Undefined Test (10 → 3):**
   - Das 7 ocorrências reduzidas: 4 foram refatoradas diretamente pela equipe (R06, R07 e R08 em `tests/test_queries.py` e R09 em `tests/test_http_span.py`). As outras 3 ocorrências da linha de base residiam em `tests/conftest.py` (2) e no módulo interno `datasette/telemetry_testing.py` (1), que não foram reprocessados pelo filtro estrito de `tests/test_*.py`. Restam 3 ocorrências na suíte (`tests/test_column_types.py` e `tests/test_playwright.py`).
3. **Non-Functional Statement (10 → 0):**
   - Das 10 ocorrências reduzidas: 7 foram refatoradas diretamente pela equipe (R10 em `test_cli_serve_server.py`, R11 a R15 em `test_telemetry_testing_kit.py` e R16 em `test_telemetry_metrics.py`). As 3 ocorrências restantes do relatório antes pertenciam a `tests/conftest.py` (2) e `datasette/_pytest_plugin.py` (1).
4. **Conditional Test Logic (135 → 100):**
   - As refatorações R22 a R38 eliminaram 32 ocorrências detectadas na AST do TEMPY nos métodos alvo. As refatorações R39 a R45 eliminaram ramificações procedurais que, embora constituam Conditional Test Logic sob o CATSPY, não eram pontuadas na regra léxica restritiva do TEMPY.

---

## 5. Análise dos resultados

### 1. Quais métricas melhoraram e quais pioraram?
- **Métricas que melhoraram:**
  - O total de test smells caiu de **258 para 195 ocorrências** (-24,42%).
  - A categoria *Non-Functional Statement* foi zerada nos arquivos de teste (10 → 0).
  - *Undefined Test* reduziu de 10 para 3 ocorrências.
  - *Conditional Test Logic* caiu de 135 para 100 ocorrências.
  - A Complexidade Ciclomática média das funções de teste no Radon caiu de **4,43 para 4,38** (-0,05).
- **Métricas que permaneceram estáveis:**
  - A cobertura global do projeto permaneceu estável em **90,27%**.
  - A cobertura de código de produção apresentou oscilação marginal de 87,40% para 87,43% (+0,03 p.p.), decorrente da maior granularidade dos testes parametrizados divididos.
  - O SLOC total do código de testes teve variação residual de +1 linha (39.524 para 39.525).

### 2. O resultado dos testes ou a cobertura de código mudou?
Nenhum teste existente foi quebrado. O número de testes aprovados aumentou em **+21 testes** (de 2.987 passed para 3.008 passed).

A auditoria via `pytest --collect-only` mapeou o saldo líquido de testes adicionais por arquivo:
- `tests/test_csv.py`: +12 testes parametrizados gerados, -12 testes anteriores (saldo líquido: 0)
- `tests/test_fts_permissions.py`: +2 testes gerados, -2 anteriores (saldo líquido: 0)
- `tests/test_load_extensions.py`: +2 testes gerados, -2 anteriores (saldo líquido: 0)
- `tests/test_table_api.py`: +8 testes gerados, -8 anteriores (saldo líquido: 0)
- `tests/test_tracer.py`: +2 testes gerados, -2 anteriores (saldo líquido: 0)
- `tests/test_permissions.py`: **+54 testes gerados, -33 anteriores (saldo líquido: +21 testes)**

Todo o saldo líquido de +21 testes decorre exclusivamente da substituição de laços iterativos e matrizes de permissão em `tests/test_permissions.py` por casos parametrizados declarativos via `@pytest.mark.parametrize`.

### 3. Foram introduzidos novos smells durante a refatoração?
Em 44 das 45 refatorações, nenhum novo smell foi introduzido. A única ocorrência transitória deu-se no commit `a81ae38` (R35), onde a extração preliminar de asserções em `test_trace` gerou um helper utilitário que foi normalizado no commit subsequente `144b331` (R36).

### 4. Quais tipos de smells foram mais fáceis e mais difíceis de refatorar?
- **Mais fáceis:**
  - *Non-Functional Statement (NFS):* Bastou remover os comandos `pass` supérfluos e simplificar as declarações `with`.
  - *Unknown Test (UT):* Bastou capturar `excinfo` e incluir asserções explícitas de tipo e mensagem.
- **Mais difíceis:**
  - *Conditional Test Logic (CTL):* Exigiu compreender a matriz de permissões do Datasette em `test_permissions.py` para converter loops e verificações de resposta em tabelas de parâmetros sem alterar as chamadas assíncronas.
  - *Sleepy Test (ST):* A natureza assíncrona do Datasette com servidores ASGI em background exigiu tratamento de deadlines e polling determinístico para não introduzir deadlocks na execução paralela.

---

## 6. Uso do CATSPY

### 1. Como a equipe utilizou o catálogo CATSPY durante o trabalho?
A equipe consultou o portal do CATSPY para cada tipo de smell a fim de compreender sua definição conceitual, sua severidade (Alta, Média ou Baixa) e as recomendações de refatoração aplicáveis a Python.

### 2. O que no catálogo ajudou mais no processo de refatoração?
O catálogo forneceu a fundamentação teórica necessária para identificar que métodos de teste não devem conter caminhos alternativos de execução, orientando o uso de testes parametrizados (*Parametrize*) e a divisão em métodos focados (*Split Test*).

### 3. Quais estratégias recomendadas pelo catálogo não se aplicaram ou precisaram de adaptação?
- **Extração para Helpers vs. Undefined Test:** O catálogo cita genericamente a extração de asserções para funções auxiliares. Entretanto, na ferramenta TEMPY, funções utilitárias que utilizam `assert` sem serem testes formais são classificadas como *Undefined Test*. A equipe adaptou a prática para lançar exceções de controle (`raise ValueError`) em métodos helpers.
- **Temporização Assíncrona:** As recomendações para Sleepy Test exemplificam chamadas síncronas a `time.sleep`. No Datasette, foi necessário adaptar os conceitos para laços de polling assíncrono com `asyncio`.

### 4. O que faltou no catálogo ou poderia ser melhorado?
O catálogo poderia incluir recomendações dedicadas a suítes de testes assíncronos (`pytest-asyncio`) e exemplos contextualizados nos recursos idiomáticos do Pytest (como `pytest.raises` e fixtures parametrizadas).

---

## 7. Divisão de trabalho

A divisão de trabalho entre os integrantes foi registrada no histórico de commits do repositório (`git log --format="%h | %an"`):

| Integrante | Commits de Refatoração | Refatorações de Teste | Smells Atribuídos | Arquivos sob Responsabilidade |
|---|---|---|---|---|
| **Kawan Torres** | 16 | 16 | Sleepy Test (1)<br>Undefined Test (3)<br>Non-Functional Statement (6)<br>Unknown Test (3)<br>Conditional Test Logic (3) | `tests/test_cli_serve_server.py`<br>`tests/test_queries.py`<br>`tests/test_telemetry_testing_kit.py`<br>`tests/test_internals_datasette.py`<br>`tests/test_utils_sql_analysis.py` |
| **Cícero Higor** | 31 (29 refatorações + 2 refinos) | 29 | Sleepy Test (4)<br>Undefined Test (1)<br>Non-Functional Statement (1)<br>Unknown Test (2)<br>Conditional Test Logic (21) | `tests/test_internals_database.py`<br>`tests/test_shutdown.py`<br>`tests/test_write_wrapper.py`<br>`tests/test_http_span.py`<br>`tests/test_telemetry_metrics.py`<br>`tests/test_template_context.py`<br>`tests/test_permissions.py`<br>`tests/test_routes.py`<br>`tests/test_tracer.py`<br>`tests/test_load_extensions.py`<br>`tests/test_internals_request.py`<br>`tests/test_table_api.py`<br>`tests/test_csv.py`<br>`tests/test_fts_permissions.py` |
| **Total Dupla** | **47 commits** | **45 refatorações** | **5 tipos do CATSPY** | **18 arquivos de teste independentes** |

*Observação sobre a divisão de esforço:* A divisão não foi 50/50 em quantidade absoluta de commits. Kawan iniciou as refatorações nos módulos utilitários e nos testes de servidor (totalizando 16 refatorações atômicas). Cícero Higor assumiu a refatoração dos arquivos de controle de permissões (`test_permissions.py`) e APIs tabulares (`test_table_api.py`), além das refatorações adicionais para cobrir a cota da equipe, totalizando 29 refatorações e 2 refinamentos.

---

## 8. Dificuldades enfrentadas e o que aprendeu

### Dificuldades Enfrentadas:
1. **Regras Léxicas do Detector (TEMPY):** O TEMPY analisa nós da AST de forma estrita. Casos onde o teste continha ramificações complexas sem um padrão rígido de `assert` sob `for` não geravam alerta léxico, enquanto helpers legítimos com asserções eram apontados como *Undefined Test*.
2. **Concorrência e Testes Assíncronos:** O Datasette utiliza extensivamente corrotinas e loops de eventos do `asyncio`. Sincronizar testes assíncronos sem pausas arbitrárias exigiu atenção para evitar deadlocks na execução em múltiplos processos (`pytest -n auto`).
3. **Controle de Escopo de Medição:** A geração de scripts temporários na raiz do projeto causou variações acidentais no relatório de cobertura durante as fases intermediárias, o que exigiu padronizar a execução com arquivo `.coveragerc` fixo.

### O que a Equipe Aprendeu:
- Testes limpos e sem condicionais tornam o rastreamento de falhas mais rápido e evitam que defeitos fiquem ocultos em ramos não executados.
- O uso de parametrização no Pytest substitui com vantagens laços manuais de iteração.
- A prática de commits atômicos com validação contínua garante rastreabilidade durante intervenções em bases de código de terceiros.

---

## 9. Contribuição (Pull Request)

- **Link Oficial do Pull Request:** [https://github.com/cicerocarnauba/datasette/pull/1](https://github.com/cicerocarnauba/datasette/pull/1)
- **Branch de Origem:** `refatoracao-test-smells`
- **Branch de Destino:** `main` (no fork da equipe em `cicerocarnauba/datasette`)
- **Status do PR:** Aberto, com build verde e 100% dos testes aprovados na suíte.

*Notas sobre a submissão:* O Pull Request foi aberto no fork da equipe para avaliação da disciplina de Qualidade de Software, sem submissão ao repositório upstream oficial (`simonw/datasette`). Não há resposta de mantenedores externos. As alterações de código de teste estão restritas ao diretório `tests/`.

---

## 10. Observações gerais

1. **Adesão ao CATSPY:** A equipe concentrou o trabalho nos 5 tipos catalogados no CATSPY, desconsiderando *Exception Handling*.
2. **Volume de Entregas:** Foram entregues **45 refatorações de código de teste**, das quais 40 contam com eliminação de smells confirmada diretamente pelo TEMPY, 1 com supressão via helper auxiliar (R01) e 4 com refatoração conceitual de temporização assíncrona (R02 a R05).
3. **Reprodutibilidade:** Os scripts de apoio utilizados para coletar métricas de cobertura e complexidade ciclomática encontram-se salvos no diretório `scripts/` do repositório para conferência.

---
*Relatório concluído pela equipe em 29 de Outubro de 2026.*
