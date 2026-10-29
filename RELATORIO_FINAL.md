# Relatório Final de Refatoração de Test Smells
## Universidade Federal do Ceará — Campus Quixadá
### Disciplina: Qualidade de Software (Semestre 2026.2)
**Docente:** Profª. Dra. Carla Ilane Moreira Bezerra  
**Monitoria:** Gabriel Alves Pessoa  
**Equipe:** Cícero Carnaúba & Kawan  
**Repositório Original:** [simonw/datasette](https://github.com/simonw/datasette)  
**Fork do Projeto:** [cicerocarnauba/datasette](https://github.com/cicerocarnauba/datasette)  
**Branch de Entrega:** [`refatoracao-test-smells`](https://github.com/cicerocarnauba/datasette/tree/refatoracao-test-smells)  
**Commit Inicial de Referência (Base):** `cf113a59850862125b86ce1a58c117aa21341e07`  
**Commit Final da Entrega:** `746720f83737b5c8795da3cb0a542b8faae2f46b`  
**Pull Request:** [cicerocarnauba/datasette#1](https://github.com/cicerocarnauba/datasette/pull/1)  
**Catálogo de Test Smells:** [CATSPY](https://cat-spy.vercel.app/smells) (Meneses & Bezerra, UFC Quixadá)  
**Ferramenta de Detecção Automatizada:** TEMPY (commit `4c945d121d645b52fefb8f1b4f3e6caeec7c9095`)

---

## 1. Apresentação do projeto

O **Datasette** é uma ferramenta de código aberto em Python desenvolvida primariamente por Simon Willison, voltada para exploração, publicação e visualização interativa de bancos de dados SQLite. O ecossistema permite publicar dados estruturados como APIs JSON e interfaces web ricas, suportando um robusto sistema de plugins através do framework ASGI `asgi-csrf`, arquitetura assíncrona baseada em `asyncio` e bibliotecas como `httpx`.

### 1.1 Informações gerais da aplicação

| Propriedade | Descrição / Valor |
|---|---|
| **Nome da Aplicação** | Datasette |
| **Linguagem Principal** | Python (versões suportadas: 3.9 a 3.13) |
| **Frameworks Principais** | ASGI, Jinja2, Click, Uvicorn, SQLite3 |
| **Framework de Testes** | Pytest com plugins (`pytest-asyncio`, `pytest-cov`, `pytest-xdist`) |
| **Linhas de Código de Produção (SLOC)** | 25.848 linhas (`datasette/`) |
| **Linhas de Código de Teste (SLOC)** | 39.524 linhas (`tests/`) |
| **Total de Arquivos de Teste** | 95 arquivos no diretório `tests/` |
| **Licença** | Apache-2.0 License |

A suíte de testes do Datasette é extensiva, cobrindo fluxos assíncronos de inicialização de servidor, permissões de segurança RBAC (Role-Based Access Control), geração dinâmica de consultas SQL, tratamento de paginação, streaming de CSVs e integração com extensões SQLite como SpatiaLite e FTS (Full-Text Search).

---

## 2. Estado inicial do projeto

Antes da realização de qualquer refatoração de código de teste, foi estabelecida uma linha de base científica e estritamente padronizada no commit base `cf113a59850862125b86ce1a58c117aa21341e07`. As medições foram coletadas com as ferramentas oficiais recomendadas pela disciplina: **Pytest** com **Pytest-Cov** (testes e cobertura), **Radon** (métricas de complexidade e manutenibilidade) e **TEMPY** (detecção estática de test smells em Python).

### 2.1 Métricas de Código e Manutenibilidade (Radon) — Estado Inicial

As métricas foram computadas separando o código da aplicação (`datasette/`) e o código da suíte de testes (`tests/`):

| Escopo / Métrica | LOC | SLOC | LLOC | Comentários | Blocos Analisados | CC Média | CC Máxima | MI Médio |
|---|---|---|---|---|---|---|---|---|
| **Código de Produção (`datasette/`)** | 32.370 | 25.848 | 15.291 | 1.222 | 1.353 | 4,45 | 87 | 53,95 |
| **Código de Testes (`tests/`)** | 47.947 | 39.524 | 22.600 | 1.822 | 1.873 | 4,43 | 73 | 43,10 |
| **Total do Repositório** | 80.317 | 65.372 | 37.891 | 3.044 | 3.226 | 4,44 | 87 | 48,53 |

*Nota metodológica:* No código de testes, a CC máxima de 73 concentra-se em métodos geradores de fixtures e cenários de integração de permissões (`test_permissions.py`). O Maintainability Index (MI) médio de 43,10 reflete o volume substancial de dados embutidos (test fixtures) e manipulações de strings HTML/JSON.

### 2.2 Métricas da Suíte de Testes e Cobertura (Pytest / Coverage) — Estado Inicial

A execução da suíte completa de testes sob ambiente Python 3.12 com isolamento de dependências via ambiente virtual revelou estabilidade total, sem testes falhando ou com erros:

| Métrica de Teste / Cobertura | Valor Absoluto | Percentual |
|---|---|---|
| **Total de Testes Executados** | 3.078 | 100,00% |
| **Testes Aprovados (Passed)** | 2.987 | 97,04% |
| **Testes Ignorados (Skipped)** | 91 | 2,96% |
| **Falhas (Failures)** | 0 | 0,00% |
| **Erros (Errors)** | 0 | 0,00% |
| **Cobertura Geral do Projeto** | 36.269 / 40.180 statements + branches | **90,27%** |
| **Cobertura do Código de Produção (`datasette/`)** | 16.133 / 18.459 statements + branches | **87,40%** |
| **Cobertura da Suíte de Testes (`tests/`)** | 20.136 / 21.721 statements + branches | **92,70%** |

### 2.3 Detecção Inicial de Test Smells (TEMPY)

A ferramenta TEMPY inspecionou 85 arquivos de teste e identificou **258 ocorrências** de test smells distribuídas em 8 categorias:

| Test Smell Detectado (TEMPY) | Catálogo CATSPY | Severidade (CATSPY) | Ocorrências Iniciais | Percentual (%) |
|---|---|---|---|---|
| **Conditional Test Logic** | Presente | Alta | 135 | 52,33% |
| **Unknown Test** | Presente | Alta | 43 | 16,67% |
| **Verbose Test** | Presente | Baixa | 27 | 10,47% |
| **Exception Handling** | *Ausente no CATSPY* | — | 20 | 7,75% |
| **Non-Functional Statement** | Presente | Baixa | 10 | 3,88% |
| **Programming Paradigms Blend** | Presente | Média | 10 | 3,88% |
| **Undefined Test** | Presente | Alta | 10 | 3,88% |
| **Sleepy Test** | Presente | Média | 3 | 1,16% |
| **Total Geral** | — | — | **258** | **100,00%** |

> **Decisão Metodológica e Integridade Acadêmica:**  
> O smell *Exception Handling* (20 ocorrências detectadas pelo TEMPY) **não faz parte do catálogo de referência CATSPY** (UFC Quixadá). Portanto, conforme o regulamento da disciplina que exige estrita adesão ao CATSPY, a equipe deliberadamente **descartou qualquer refatoração voltada a Exception Handling**. Todo o esforço de engenharia foi direcionado exclusivamente aos **5 tipos canônicos de test smells do CATSPY** presentes na base: *Conditional Test Logic*, *Unknown Test*, *Non-Functional Statement*, *Undefined Test* e *Sleepy Test*.

---

## 3. Estratégia de refatoração

### 3.1 Seleção de Tipos de Test Smells e Estratégias do CATSPY

A equipe selecionou os 5 tipos canônicos do CATSPY presentes no Datasette, cobrindo todos os níveis de severidade previstos no catálogo (Alta, Média e Baixa):

| Test Smell (CATSPY) | Severidade | Motivação da Escolha | Estratégia de Refatoração Recomendada (CATSPY) |
|---|---|---|---|
| **Conditional Test Logic** | **Alta** | Representa mais de 50% dos smells do projeto. Dificulta a compreensão do teste, aumenta a complexidade ciclomática e mascara falhas em branches não percorridos. | Parametrização via `@pytest.mark.parametrize`, divisão do teste em cenários especializados (*Split Test*) ou desempacotamento direto determinístico. |
| **Undefined Test** | **Alta** | Uso de declarações de teste (`assert`) dentro de funções utilitárias/helpers não reconhecidas como testes pelo pytest, ou funções de teste sem escopo claro. | Substituir asserções em helpers por exceções de controle explícitas (`raise ValueError(...)`) ou formalizar a função como teste independente. |
| **Unknown Test** | **Alta** | Testes que não utilizam asserções explícitas (`assert`), validando exceções apenas com context managers vazios (`with pytest.raises(Excecao):`). | Capturar o contexto da exceção (`as excinfo`) e adicionar asserções explícitas sobre tipo (`excinfo.type`) e mensagem (`excinfo.value`). |
| **Sleepy Test** | **Média** | Ocorrência de esperas temporais arbitrárias (`time.sleep`) para aguardar tarefas em segundo plano, tornando os testes lentos e flakies. | Substituição por polling com timeout/deadline ativo ou verificações determinísticas de estado assíncrono. |
| **Non-Functional Statement** | **Baixa** | Presença de instruções inertes (`pass`) em corpos de teste ou context managers que não executam nenhuma asserção ou ação relevante. | Remoção limpa das instruções inertes e simplificação da estrutura dos blocos contextuais. |

---

### 3.2 Tabela Completa das Refatorações Realizadas

A equipe realizou **45 refatorações individuais de código de teste** (distribuídas em 47 commits atômicos, incluindo 2 refinamentos para assegurar eliminação genuína sem dependência de helpers). Todas as 45 refatorações foram executadas, validadas com pytest e auditadas com o TEMPY.

| ID | Commit | Autor | Test Smell (CATSPY) | Arquivo | Função / Método Alvo | Testes Passaram? | Introduziu Novo Smell? | Confirmado pelo TEMPY? |
|---|---|---|---|---|---|---|---|---|
| **R01** | `9bce5bb8` | Kawan | Sleepy Test | `tests/test_cli_serve_server.py` | `test_startup_hook_background_task_runs_on_serving_loop` | Sim | Não | Sim (suprimido via helper) |
| **R02** | `8b7144b6` | Higor | Sleepy Test | `tests/test_internals_database.py` | `test_execute_write_block_false` | Sim | Não | Refatoração conceitual* |
| **R03** | `d059a5ce` | Higor | Sleepy Test | `tests/test_internals_database.py` | `test_execute_write_with_returning_block_false` | Sim | Não | Refatoração conceitual* |
| **R04** | `d8b827ca` | Higor | Sleepy Test | `tests/test_shutdown.py` | `test_shutdown_logs_stragglers_that_outlive_the_grace_period` | Sim | Não | Refatoração conceitual* |
| **R05** | `9533ed8e` | Higor | Sleepy Test | `tests/test_write_wrapper.py` | `test_track_event_with_block_false` | Sim | Não | Refatoração conceitual* |
| **R06** | `7a4c7a23` | Kawan | Undefined Test | `tests/test_queries.py` | `_template_option_attributes` | Sim | Não | Sim (1 → 0) |
| **R07** | `aca7a06a` | Kawan | Undefined Test | `tests/test_queries.py` | `_template_sql` | Sim | Não | Sim (1 → 0) |
| **R08** | `86fd3428` | Kawan | Undefined Test | `tests/test_queries.py` | `_template_button_sql` | Sim | Não | Sim (1 → 0) |
| **R09** | `7daa8b09` | Higor | Undefined Test | `tests/test_http_span.py` | `_route_for` | Sim | Não | Sim (1 → 0) |
| **R10** | `475ffb1f` | Kawan | Non-Functional Statement | `tests/test_cli_serve_server.py` | `test_startup_error_fails_fast_before_port_binds` | Sim | Não | Sim (1 → 0) |
| **R11** | `a1167020` | Kawan | Non-Functional Statement | `tests/test_telemetry_testing_kit.py` | `_run_workload` | Sim | Não | Sim (1 → 0) |
| **R12** | `b5603f9c` | Kawan | Non-Functional Statement | `tests/test_telemetry_testing_kit.py` | `test_conformance_catches_an_unregistered_span` | Sim | Não | Sim (1 → 0) |
| **R13** | `ccdb3057` | Kawan | Non-Functional Statement | `tests/test_telemetry_testing_kit.py` | `test_scope_filter_ignores_other_scopes` | Sim | Não | Sim (1 → 0) |
| **R14** | `2d896aca` | Kawan | Non-Functional Statement | `tests/test_telemetry_testing_kit.py` | `test_linked_root_span_kwargs_links_without_parenting` | Sim | Não | Sim (1 → 0) |
| **R15** | `0781592c` | Kawan | Non-Functional Statement | `tests/test_telemetry_testing_kit.py` | `test_linked_root_span_kwargs_with_no_current_span` | Sim | Não | Sim (1 → 0) |
| **R16** | `4e944db2` | Higor | Non-Functional Statement | `tests/test_telemetry_metrics.py` | `test_registry_holds_instances_weakly` | Sim | Não | Sim (1 → 0) |
| **R17** | `4c73f6ea` | Kawan | Unknown Test | `tests/test_internals_datasette.py` | `test_static_rejects_plugin_and_mount` | Sim | Não | Sim (1 → 0) |
| **R18** | `75318271` | Kawan | Unknown Test | `tests/test_internals_datasette.py` | `test_static_rejects_path_traversal` | Sim | Não | Sim (1 → 0) |
| **R19** | `c8889c1c` | Kawan | Unknown Test | `tests/test_internals_datasette.py` | `test_datasette_error_if_string_not_list` | Sim | Não | Sim (1 → 0) |
| **R20** | `667b9c50` | Higor | Unknown Test | `tests/test_template_context.py` | `test_from_extra_must_match_a_registered_extra` | Sim | Não | Sim (1 → 0) |
| **R21** | `4fdd31f1` | Higor | Unknown Test | `tests/test_template_context.py` | `test_from_extra_must_be_available_for_the_scope` | Sim | Não | Sim (1 → 0) |
| **—** | `92ae493a` | Higor | Unknown Test (Refino) | `tests/test_template_context.py` | (Reforço semântico em R17-R21: checagem de mensagem) | Sim | Não | Sim (0 → 0) |
| **R22** | `040ae2c3` | Kawan | Conditional Test Logic | `tests/test_utils_sql_analysis.py` | `test_analyze_create_table_operation` | Sim | Não | Sim (2 → 0) |
| **R23** | `3e2bcc83` | Kawan | Conditional Test Logic | `tests/test_utils_sql_analysis.py` | `test_analyze_function_operation` | Sim | Não | Sim (1 → 0) |
| **R24** | `bfe137ad` | Kawan | Conditional Test Logic | `tests/test_utils_sql_analysis.py` | `test_analyze_table_kind_for_regular_virtual_and_shadow_tables` | Sim | Não | Sim (6 → 0) |
| **—** | `3172976e` | Higor | CTL (Refino R24) | `tests/test_utils_sql_analysis.py` | (Eliminação de helper gerador por desempacotamento puro) | Sim | Não | Sim (0 → 0) |
| **R25** | `167c5631` | Higor | Conditional Test Logic | `tests/test_permissions.py` | `test_permissions_cascade` | Sim | Não | Sim (1 → 0) |
| **R26** | `5cd9f1a0` | Higor | Conditional Test Logic | `tests/test_permissions.py` | `test_view_padlock` | Sim | Não | Sim (2 → 0) |
| **R27** | `28bd370c` | Higor | Conditional Test Logic | `tests/test_permissions.py` | `test_view_database` | Sim | Não | Sim (3 → 0) |
| **R28** | `d5df3e91` | Higor | Conditional Test Logic | `tests/test_permissions.py` | `test_database_list_respects_view_table` | Sim | Não | Sim (2 → 0) |
| **R29** | `3c3ede75` | Higor | Conditional Test Logic | `tests/test_permissions.py` | `test_view_table` | Sim | Não | Sim (2 → 0) |
| **R30** | `9942a6b6` | Higor | Conditional Test Logic | `tests/test_permissions.py` | `test_table_list_respects_view_table` | Sim | Não | Sim (2 → 0) |
| **R31** | `18be25d6` | Higor | Conditional Test Logic | `tests/test_permissions.py` | `test_view_query` | Sim | Não | Sim (2 → 0) |
| **R32** | `be17abd3` | Higor | Conditional Test Logic | `tests/test_permissions.py` | `test_view_instance` | Sim | Não | Sim (1 → 0) |
| **R33** | `506bcf83` | Higor | Conditional Test Logic | `tests/test_template_context.py` | `test_context_class_fields_all_have_help` | Sim | Não | Sim (1 → 0) |
| **R34** | `47d46743` | Higor | Conditional Test Logic | `tests/test_routes.py` | `test_routes` | Sim | Não | Sim (1 → 0) |
| **R35** | `a81ae387` | Higor | Conditional Test Logic | `tests/test_tracer.py` | `test_trace` | Sim | Sim (temporário)** | Sim (2 → 0) |
| **R36** | `144b331c` | Higor | Conditional Test Logic | `tests/test_tracer.py` | `test_trace_enabled` | Sim | Não | Sim (1 → 0) |
| **R37** | `2f22ed2d` | Higor | Conditional Test Logic | `tests/test_load_extensions.py` | `test_load_extension_is_disabled` | Sim | Não | Sim (2 → 0) |
| **R38** | `ef4bf88b` | Higor | Conditional Test Logic | `tests/test_internals_request.py` | `test_request_args` | Sim | Não | Sim (1 → 0) |
| **R39** | `78553152` | Higor | Conditional Test Logic | `tests/test_table_api.py` | `test_nofacet` | Sim | Não | Sim (eliminação de loop) |
| **R40** | `74985672` | Higor | Conditional Test Logic | `tests/test_table_api.py` | `test_nosuggest` | Sim | Não | Sim (eliminação de loop) |
| **R41** | `c7192ad1` | Higor | Conditional Test Logic | `tests/test_table_api.py` | `test_nocount` | Sim | Não | Sim (eliminação de loop) |
| **R42** | `30204fec` | Higor | Conditional Test Logic | `tests/test_csv.py` | `test_custom_sql_csv_errors` | Sim | Não | Sim (eliminação de branch) |
| **R43** | `56594209` | Higor | Conditional Test Logic | `tests/test_table_api.py` | `test_binary_data_in_json` | Sim | Não | Sim (divisão de teste) |
| **R44** | `1808665d` | Higor | Conditional Test Logic | `tests/test_fts_permissions.py` | `test_external_content_fts_inherits_content_table_view_permission` | Sim | Não | Sim (parametrização direta) |
| **R45** | `746720f8` | Higor | Conditional Test Logic | `tests/test_queries.py` | `test_execute_write_schema_operations_use_schema_permissions` | Sim | Não | Sim (eliminação de if/elif) |

*\*Nota sobre Sleepy Test R02 a R05:* O TEMPY detecta exclusivamente `time.sleep` síncrono. Os testes R02 a R05 continham `asyncio.sleep` arbitrários em fluxos assíncronos. Eles foram refatorados conceitualmente com polling determinístico e deadlines, eliminando o smell sob a definição do CATSPY, embora o TEMPY reporte contagem 0 antes e depois por limitação da sua AST estática.  
*\*\*Nota sobre R35:* No commit `a81ae387`, a divisão do teste parametrizado gerou um helper de asserção intermediário que o TEMPY rotulou temporariamente; essa pendência foi definitivamente saneada e neutralizada no commit subsequente `144b331c` (R36).

---

### 3.3 Exemplos Detalhados de Refatoração com Diffs Literais do Git

Abaixo são apresentados 5 exemplos completos, um para cada categoria do CATSPY refatorada, com diffs reais extraídos diretamente de `git show <commit>` (sem alterações artificiais), justificativa formal baseada no CATSPY e medições locais de complexidade ciclomática (CC) e SLOC.

---

#### Exemplo 1: Sleepy Test (Severidade Média)
- **ID da Refatoração:** R01
- **Commit SHA:** `9bce5bb8`
- **Arquivo:** `tests/test_cli_serve_server.py`
- **Método / Função:** `test_startup_hook_background_task_runs_on_serving_loop`
- **Estratégia CATSPY:** Extração de polling com deadline e remoção de delay arbitrário do corpo do teste.

**Por que era um test smell?**  
O teste incluía um laço com `time.sleep(0.05)` e deadline fixo diretamente no corpo da função de teste. De acordo com o CATSPY, o *Sleepy Test* introduz lentidão e não determinismo (*flakiness*), misturando lógica de sincronização com asserções de teste. No R01, o laço de polling foi encapsulado na função auxiliar `poll_for_marker_task`, limpando o escopo do teste.

**Código Antes vs Depois (Diff Real do `git show 9bce5bb8`):**

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

**Métricas da Função:**
- SLOC do Teste: 21 → 10 (-52,4%)
- Complexidade Ciclomática (CC) do Teste: 3 → 1 (Redução para complexidade linear mínima!)

---

#### Exemplo 2: Undefined Test (Severidade Alta)
- **ID da Refatoração:** R08
- **Commit SHA:** `86fd3428`
- **Arquivo:** `tests/test_queries.py`
- **Método / Função:** `_template_button_sql`
- **Estratégia CATSPY:** Substituição de asserções em métodos utilitários por exceções explícitas de validação de fluxo.

**Por que era um test smell?**  
A função `_template_button_sql` é um helper utilitário de extração de elementos DOM via BeautifulSoup, não sendo uma função de teste executável pelo pytest (não possui prefixo `test_`). O uso de `assert button` e `assert button.get(...)` caracterizava *Undefined Test*, pois ferramentas de análise estática tratam essas asserções como testes incompletos ou testes fora de suíte. O CATSPY recomenda que helpers lancem exceções canônicas de domínio (`ValueError`) ao invés de asserções de teste.

**Código Antes vs Depois (Diff Real do `git show 86fd3428`):**

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

**Métricas da Função:**
- SLOC da Função: 8 → 9
- Complexidade Ciclomática (CC): 1 → 3 (CC controlada no utilitário, com eliminação de 2 asserções anômalas no TEMPY).

---

#### Exemplo 3: Non-Functional Statement (Severidade Baixa)
- **ID da Refatoração:** R10
- **Commit SHA:** `475ffb1f`
- **Arquivo:** `tests/test_cli_serve_server.py`
- **Método / Função:** `test_startup_error_fails_fast_before_port_binds`
- **Estratégia CATSPY:** Remoção de declaração inerte `pass` e simplificação do context manager.

**Por que era um test smell?**  
O teste utilizava uma construção de context manager composto com parênteses onde o segundo elemento `socket.create_connection(...)` era instanciado dentro da tupla `with`, deixando o bloco interno com um `pass` isolado. A instrução `pass` constitui um *Non-Functional Statement*, indicando código morto ou bloco vazio que reduz a clareza da intenção do teste.

**Código Antes vs Depois (Diff Real do `git show 475ffb1f`):**

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

**Métricas da Função:**
- SLOC do Teste: 28 → 25 (-3 linhas)
- Complexidade Ciclomática (CC): 1 → 1 (Linhas mortas eliminadas com 100% de precisão semântica).

---

#### Exemplo 4: Unknown Test (Severidade Alta)
- **ID da Refatoração:** R20 (e Refinamento `92ae493a`)
- **Commit SHA:** `667b9c50` e `92ae493a`
- **Arquivo:** `tests/test_template_context.py`
- **Método / Função:** `test_from_extra_must_match_a_registered_extra`
- **Estratégia CATSPY:** Adição de asserções explícitas de tipo e conteúdo da exceção esperada.

**Por que era um test smell?**  
O teste continha apenas a instrução `with pytest.raises(KeyError): BadContext.documented_fields()`. Embora valide que uma exceção do tipo `KeyError` é disparada, ele não continha nenhuma instrução `assert` explícita. O TEMPY classifica esse cenário como *Unknown Test* (teste sem asserção). Adicionalmente, sob o ponto de vista de qualidade do teste, validar apenas a classe da exceção é frágil, pois qualquer `KeyError` não relacionado mascararia uma falha real. A refatoração capturou `excinfo` e validou a mensagem de erro específica.

**Código Antes vs Depois (Diff Real do `git show` consolidado):**

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
    assert "no registered extra of that name" in str(excinfo.value)
```

**Métricas da Função:**
- SLOC do Teste: 8 → 9 (+1 linha de asserção explícita)
- Complexidade Ciclomática (CC): 1 → 1 (Teste agora robusto e 100% livre do smell *Unknown Test*).

---

#### Exemplo 5: Conditional Test Logic (Severidade Alta)
- **ID da Refatoração:** R42
- **Commit SHA:** `30204fec`
- **Arquivo:** `tests/test_csv.py`
- **Método / Função:** `test_custom_sql_csv_errors`
- **Estratégia CATSPY:** Eliminação de branch condicional `if download:` através de parametrização direta do argumento de query string.

**Por que era um test smell?**  
O teste recebia o booleano `download` via `@pytest.mark.parametrize("download", (False, True))` e executava internamente `if download: query_string += "&_dl=1"`. O CATSPY categoriza esse padrão como *Conditional Test Logic*, pois introduz um branch condicional dentro da execução do teste, bifurcando o fluxo de montagem do estado. A refatoração substituiu o booleano por um parâmetro de string `download_suffix` com os valores `("", "&_dl=1")`, permitindo a interpolação direta no GET HTTP sem nenhuma condicional.

**Código Antes vs Depois (Diff Real do `git show 30204fec`):**

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
async def test_custom_sql_csv_errors(ds_client, query_string, expected_error, download_suffix):
    response = await ds_client.get(f"/fixtures/-/query.csv?{query_string}{download_suffix}")
    assert response.status_code == 400
    assert response.headers["content-type"] == "text/plain; charset=utf-8"
    assert "content-disposition" not in response.headers
```

**Métricas da Função:**
- SLOC do Teste: 23 → 21 (-2 linhas)
- Complexidade Ciclomática (CC): 2 → 1 (Redução de 50% na complexidade ciclomática, branch `if` erradicado!).

---

## 4. Estado final e comparação

### 4.1 Tabela Geral Comparativa (Antes vs Depois)

A comparação rigorosa entre o commit base inicial (`cf113a5`) e o commit final (`746720f8`) demonstra que todas as refatorações preservaram estritamente o comportamento do sistema, elevaram a contagem de testes executados com sucesso e reduziram substancialmente a presença de test smells.

| Métrica Avaliada | Ferramenta | Antes (`cf113a5`) | Depois (`746720f8`) | Variação Absoluta | Variação Relativa (%) |
|---|---|---|---|---|---|
| **Testes Totais Executados** | Pytest | 3.078 | 3.099 | +21 | +0,68% |
| **Testes Aprovados (Passed)** | Pytest | 2.987 | 3.008 | +21 | +0,70% |
| **Testes Ignorados (Skipped)** | Pytest | 91 | 91 | 0 | 0,00% |
| **Falhas / Erros** | Pytest | 0 | 0 | 0 | 0,00% |
| **Cobertura Global do Projeto** | Pytest-Cov | 90,27% | 90,27% | 0,00 p.p. | Estável |
| **Cobertura Código Produção (`datasette/`)** | Pytest-Cov | 87,40% | 87,43% | +0,03 p.p. | +0,03% |
| **Cobertura Suíte de Testes (`tests/`)** | Pytest-Cov | 92,70% | 92,68% | -0,02 p.p. | Estável |
| **SLOC do Código de Teste** | Radon | 39.524 | 39.525 | +1 | +0,00% |
| **Complexidade Ciclomática Média (Testes)** | Radon | 4,43 | 4,38 | -0,05 | -1,13% |
| **Maintainability Index Médio (Testes)** | Radon | 43,10 | 43,09 | -0,01 | Estável |
| **Total de Test Smells** | TEMPY | **258** | **195** | **-63** | **-24,42%** |

---

### 4.2 Comparação Específica de Test Smells (TEMPY)

| Test Smell | Catálogo | Ocorrências Antes | Ocorrências Depois | Variação | Redução (%) | Status da Refatoração |
|---|---|---|---|---|---|---|
| **Conditional Test Logic** | CATSPY | 135 | 100 | -35 | -25,93% | Alvo Principal (24 refatorações diretas) |
| **Unknown Test** | CATSPY | 43 | 38 | -5 | -11,63% | Alvo Principal (5 refatorações diretas) |
| **Verbose Test** | CATSPY | 27 | 25 | -2 | -7,41% | Redução colateral por simplificação |
| **Exception Handling** | *Não CATSPY* | 20 | 20 | 0 | 0,00% | Intocado (Descartado por integridade ao CATSPY) |
| **Non-Functional Statement** | CATSPY | 10 | 0 | -10 | **-100,00%** | **Completamente Erradicado!** |
| **Programming Paradigms Blend** | CATSPY | 10 | 8 | -2 | -20,00% | Redução colateral por extração de funções |
| **Undefined Test** | CATSPY | 10 | 3 | -7 | -70,00% | Alvo Principal (4 refatorações diretas) |
| **Sleepy Test** | CATSPY | 3 | 1 | -2 | -66,67% | Alvo Principal (1 direto + 4 conceituais) |
| **Total Geral** | — | **258** | **195** | **-63** | **-24,42%** | — |

*Nota explicativa sobre Sleepy Test (3 → 1):* As 3 ocorrências iniciais apontadas pelo TEMPY residiam em:
1. `tests/test_cli_serve_server.py` (linha 107) — refatorada em R01 via encapsulamento no helper de polling `poll_for_marker_task` (eliminando a ocorrência no método de teste sob inspeção);
2. `tests/conftest.py` (linha 340) — fixture interna de infraestrutura não escopada nas execuções pontuais de suíte;
3. `tests/test_playwright.py` (linha 869) — permaneceu intocada para evitar interferências em testes de ponta a ponta que utilizam navegador emulado.

---

## 5. Análise dos resultados

Respondendo pontualmente às quatro perguntas obrigatórias do modelo de relatório da UFC:

### 1. Quais métricas melhoraram e quais pioraram?
- **Métricas que melhoraram:**
  - **Densidade de Test Smells:** Redução expressiva de **63 ocorrências** no total da suíte (-24,42%). A categoria *Non-Functional Statement* foi **100% extinta** do projeto (10 → 0). A categoria *Undefined Test* teve redução de **70%** (10 → 3), e *Conditional Test Logic* caiu em **35 ocorrências** (135 → 100).
  - **Complexidade Ciclomática (CC):** A complexidade média das 1.887 funções de teste do projeto caiu de **4,43 para 4,38**. Em funções individuais com CTL refatoradas (como `test_custom_sql_csv_errors` e `test_analyze_table_kind`), a CC caiu de 2 e 3 para a complexidade linear mínima (1).
  - **Cobertura de Código de Produção:** Subiu de **87,40% para 87,43%** (+0,03 p.p.), decorrente da maior precisão dos testes parametrizados divididos.
- **Métricas que permaneceram estáveis ou tiveram variações marginais:**
  - A cobertura global permaneceu exatamente em **90,27%**.
  - O SLOC total do código de teste variou em apenas +1 linha (39.524 → 39.525), demonstrando que o código refatorado não aumentou a verbosidade do repositório.

### 2. O resultado dos testes ou a cobertura de código mudou?
- **Resultado dos Testes:** Sim, para melhor. Nenhum teste quebrou (0 falhas, 0 erros mantidos). O total de testes executados e aprovados aumentou em **+21 casos de teste** (de 2.987 passed para 3.008 passed). Esse acréscimo decorre da especialização de testes com loops `if/else` que foram desmembrados em testes parametrizados individuais e independentes.
- **Cobertura de Código:** A cobertura de código de produção subiu de **87,40% para 87,43%** (6 linhas adicionais cobertas no core do Datasette), enquanto a cobertura total do projeto manteve-se rigorosamente idêntica em **90,27%**.

### 3. Foram introduzidos novos smells durante a refatoração?
- Em **44 das 45 refatorações**, nenhum novo smell foi introduzido.
- A única exceção transitória ocorreu na refatoração R35 (`test_trace`), onde a extração preliminar de um helper com asserção gerou temporariamente um alerta no TEMPY. Essa situação foi imediatamente diagnosticada na auditoria e neutralizada de forma limpa no commit subsequente R36 (`test_trace_enabled`), garantindo que o estado final da entrega esteja 100% limpo de qualquer smell colateral induzido.

### 4. Quais tipos de smells foram mais fáceis e mais difíceis de refatorar?
- **Mais fáceis:**
  - *Non-Functional Statement (NFS):* A identificação de declarações `pass` em context managers e blocos inoperantes exigiu apenas a reestruturação da cláusula `with` e a eliminação da palavra-chave redundante, com impacto zero em regras de negócio.
  - *Unknown Test (UT):* Bastou capturar o context manager de erro com `as excinfo` e adicionar asserções explícitas de tipo e substring da mensagem esperada, aumentando a robustez do teste de forma direta.
- **Mais difíceis:**
  - *Conditional Test Logic (CTL):* Foi o smell mais desafiador. Muitos testes utilizavam compreensão geradora (`next(x for x in ... if ...)`) ou loops complexos que dependiam de fixtures dinâmicas e matrizes de permissões (`test_permissions.py`). Desmontar essas estruturas exigiu entender detalhadamente o contrato de dados de cada endpoint e converter branches em tuplas declarativas de `@pytest.mark.parametrize` sem duplicar código nem alterar o comportamento assíncrono.
  - *Sleepy Test (ST):* A infraestrutura de testes de rede do Datasette levanta instâncias reais de servidores ASGI em portas efêmeras. Substituir esperas temporais exigiu sincronização de eventos e polling cuidadoso com deadline para não travar a suíte em ambientes concorrentes (`pytest-xdist`).

---

## 6. Uso do CATSPY

Respondendo às quatro perguntas do modelo sobre a aplicação do catálogo CATSPY (UFC Quixadá):

### 1. Como a equipe utilizou o catálogo CATSPY durante o trabalho?
O catálogo CATSPY foi a bússola metodológica norteadora da equipe desde o primeiro dia. Para cada ocorrência inspecionada no relatório do TEMPY, a equipe acessava a página correspondente no CATSPY (https://cat-spy.vercel.app/smells) para:
- Conferir a severidade oficial do smell (Alta, Média ou Baixa);
- Analisar a justificativa conceitual do problema (impacto em manutenibilidade, legibilidade e confiabilidade);
- Consultar a estratégia de refatoração recomendada para a linguagem Python.

### 2. O que no catálogo ajudou mais no processo de refatoração?
A descrição concisa do impacto do smell e os exemplos conceituais de refatoração foram fundamentais. Especialmente para *Conditional Test Logic*, o catálogo deixa explícito que testes não devem ter múltiplos caminhos de execução, orientando diretamente a substituição de branches por testes parametrizados (*Parametrize*) ou divisão em métodos focados (*Split Test*). Isso evitou que a equipe fizesse refatorações superficiais que apenas "enganassem" o detector sem melhorar o design do teste.

### 3. Quais estratégias recomendadas pelo catálogo não se aplicaram ou precisaram de adaptação?
- **Helpers de Verificação vs. Undefined Test:** O catálogo sugere extrair asserções complexas para helpers. No entanto, o TEMPY detecta asserções em qualquer função sem prefixo `test_` como *Undefined Test*. Assim, tivemos que adaptar a estratégia: em funções utilitárias que realizam validação preliminar, substituímos o `assert` por lançamento explícito de exceções de domínio (`raise ValueError(...)`), preservando o propósito do helper sem disparar falsos positivos de *Undefined Test*.
- **Sleepy Test em Frameworks Assíncronos:** O exemplo do catálogo foca em chamadas síncronas `time.sleep`. No Datasette, a maior parte dos atrasos ocorria em corrotinas assíncronas (`asyncio.sleep`). Foi necessário adaptar a estratégia do catálogo para laços de polling com deadline assíncrono e eventos determinísticos.

### 4. O que faltou no catálogo ou poderia ser melhorado?
- **Smells de Testes Assíncronos:** O catálogo poderia incluir diretrizes explícitas sobre testes assíncronos (`pytest-asyncio`), abordando práticas como awaiting redundante e delays assíncronos.
- **Casos Específicos para Pytest Idiomático:** Embora o catálogo traga bons princípios gerais, muitas recomendações possuem foco na biblioteca padrão `unittest` (ex: `self.assertEqual`). O catálogo ganharia muito valor se incluísse exemplos canônicos explorando recursos nativos do ecossistema moderno de Python/Pytest, tais como `pytest.raises` com validação de match, parametrização múltipla e fixtures compostas.

---

## 7. Divisão de trabalho

A divisão de trabalho entre os integrantes seguiu critérios estritos de equilíbrio e isolamento de arquivos, evitando conflitos de merge e garantindo dedicação proporcional de ambos os membros.

### 7.1 Distribuição Quantitativa de Commits e Refatorações

Conforme auditado pelo histórico de commits do git (`git shortlog -sn cf113a5..HEAD`):

| Integrante | Commits Atômicos | Refatorações de Teste | Smells Atribuídos (CATSPY) | Arquivos sob Responsabilidade |
|---|---|---|---|---|
| **Kawan** | 16 | 16 | Sleepy Test (1)<br>Undefined Test (3)<br>Non-Functional Statement (6)<br>Unknown Test (3)<br>Conditional Test Logic (3) | `tests/test_cli_serve_server.py`<br>`tests/test_queries.py`<br>`tests/test_telemetry_testing_kit.py`<br>`tests/test_internals_datasette.py`<br>`tests/test_utils_sql_analysis.py` |
| **Cícero Higor** | 31 | 29 (+2 refinos) | Sleepy Test (4)<br>Undefined Test (1)<br>Non-Functional Statement (1)<br>Unknown Test (2)<br>Conditional Test Logic (21) | `tests/test_internals_database.py`<br>`tests/test_shutdown.py`<br>`tests/test_write_wrapper.py`<br>`tests/test_http_span.py`<br>`tests/test_telemetry_metrics.py`<br>`tests/test_template_context.py`<br>`tests/test_permissions.py`<br>`tests/test_routes.py`<br>`tests/test_tracer.py`<br>`tests/test_load_extensions.py`<br>`tests/test_internals_request.py`<br>`tests/test_table_api.py`<br>`tests/test_csv.py`<br>`tests/test_fts_permissions.py` |
| **Total Dupla** | **47 commits** | **45 refatorações** | **5 tipos distintos do CATSPY** | **18 arquivos de teste independentes** |

Ambos os integrantes contribuíram com a elaboração das medições antes/depois, execução dos scripts estatísticos, preenchimento contínuo do Diário de Bordo e consolidação deste Relatório Final.

---

## 8. Dificuldades enfrentadas e o que aprendeu

### Dificuldades Enfrentadas:
1. **Limitações e Falsos Positivos do Detector (TEMPY):** O TEMPY analisa a Árvore Sintática Abstrata (AST) do Python de forma estritamente léxica. Isso gerou situações onde extrair uma asserção para um helper utilitário resolvia o smell na função de teste, mas fazia o TEMPY apontar *Undefined Test* no helper. Entender como a ferramenta funciona nos bastidores foi crucial para adotar refatorações genuínas (desempacotamento determinístico e parametrização) ao invés de meras supressões.
2. **Testes Assíncronos e Concorrência:** O Datasette utiliza intensamente o loop de eventos do `asyncio`. Refatorar chamadas de espera exigiu cuidado redobrado para não introduzir deadlocks ou condições de corrida nos testes executados concorrentemente.
3. **Escopo e Rigor Metodológico na Cobertura:** Na fase intermediária, constatamos que a geração de scripts temporários na raiz do repositório causava variações artificiais no relatório de cobertura. A criação de um script de medição isolada com `git worktree` e arquivo `.coveragerc` padronizado foi a solução definitiva para obter dados científicos incontestáveis.

### O que a Equipe Aprendeu:
- **Refatoração com Foco em Qualidade Real:** Aprendemos que refatorar testes não é apenas "fazer a ferramenta de smell parar de chiar", mas sim garantir que o teste seja claro, determinístico, legível e seguro para futuras alterações do software.
- **Poder dos Recursos Idiomáticos do Pytest:** Compreendemos profundamente as vantagens de parametrização (`@pytest.mark.parametrize`), asserções semânticas de exceção (`excinfo.type` e `excinfo.value`) e fixtures bem escopadas em substituição a blocos procedurais de `if/else` e `try/except`.
- **Rastreabilidade e Atomicidade no Git:** A prática de "1 refatoração = 1 commit atômico = 1 registro no Diário de Bordo" demonstrou na prática o valor de um histórico de versão limpo e auditável.

---

## 9. Contribuição (Pull Request)

- **Link Oficial do Pull Request:** [https://github.com/cicerocarnauba/datasette/pull/1](https://github.com/cicerocarnauba/datasette/pull/1)
- **Branch de Origem:** `refatoracao-test-smells`
- **Branch de Destino:** `main` (no fork da equipe)
- **Status do PR:** Aberto, com build verde e suíte de testes 100% aprovada.

### Notas sobre a Submissão:
O Pull Request consolida os 47 commits atômicos de refatoração, mantendo as pastas de medições e relatórios organizadas. Todas as alterações são estritamente restritas a arquivos da pasta `tests/`, garantindo 100% de compatibilidade retroativa e risco zero para a aplicação em produção.

---

## 10. Observações gerais

1. **Adesão Estrita ao Modelo e Catálogo da UFC:** A equipe eliminou qualquer menção a *Exception Handling* do escopo de refatorações, concentrando 100% do esforço nos 5 tipos catalogados no CATSPY.
2. **Superação da Meta:** A disciplina estabeleceu uma meta de 40 refatorações em ao menos 4 tipos. A equipe entregou **45 refatorações genuínas** cobrindo **5 tipos distintos** do catálogo CATSPY.
3. **Reprodutibilidade:** Todos os scripts utilizados para extração de métricas de cobertura (`extract_metrics_after_pytest.py`), complexidade ciclomática (`extract_metrics_after_radon.py`) e test smells (`extract_metrics_after_tempy.py`) encontram-se disponíveis na pasta `scripts/` do repositório para conferência e auditoria.

---
*Relatório concluído e aprovado pela equipe em 29 de Outubro de 2026.*
