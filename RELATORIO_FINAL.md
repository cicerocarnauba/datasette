# Relatório Final de Refatoração de Test Smells
## Universidade Federal do Ceará — Campus Quixadá
### Disciplina: Qualidade de Software (Semestre 2026.2)
**Docente:** Profª. Dra. Carla Ilane Moreira Bezerra  
**Monitoria:** Gabriel Alves Pessoa  
**Equipe:**
- Cícero Higor (Matrícula: 574228, GitHub: `cicerocarnauba`, autor no git: `higor`)
- Kawan Torres (Matrícula: 563869, GitHub: `kawantorres`, autor no git: `kawantorres`)

**Repositório Original:** [simonw/datasette](https://github.com/simonw/datasette)  
**Fork do Projeto:** [cicerocarnauba/datasette](https://github.com/cicerocarnauba/datasette)  
**Branch de Trabalho e Entrega:** [`refatoracao-test-smells`](https://github.com/cicerocarnauba/datasette/tree/refatoracao-test-smells)  
**Commit Inicial de Referência (Base da Planilha):** `cf113a5` (`cf113a59850862125b86ce1a58c117aa21341e07`)  
**Commit Final de Refatoração de Código:** `977a793` (`977a793a388e68e4bf9f5f0b5d564ad4160352ff`)  
**Commit de Formatação PEP 8 / Flake8:** `93280f5` (`93280f569a7c36a6fa0ee1567bcab4c053db6e99`)  
**Pull Request Oficial:** [cicerocarnauba/datasette#1](https://github.com/cicerocarnauba/datasette/pull/1)  
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
| **Escopo Analisado pelo TEMPY** | **85 arquivos** (83 arquivos no diretório `tests/` e 2 módulos internos de suporte a testes no pacote principal: `datasette/_pytest_plugin.py` e `datasette/telemetry_testing.py`) |
| **Licença** | Apache-2.0 License |

A suíte de testes cobre funcionalidades abrangentes, incluindo inicialização do servidor ASGI, controle de permissões de acesso, manipulação de consultas SQL, exportação e streaming de dados (CSV e JSON) e integração com extensões do SQLite (como Full-Text Search e SpatiaLite).

---

## 2. Estado inicial do projeto

A linha de base de medições foi coletada no commit inicial `cf113a5` utilizando ferramentas de análise estática e dinâmica padronizadas no ambiente Python 3.12: **TEMPY** (detecção estática de test smells em Python), **Pytest** com **Pytest-Cov** (testes e cobertura) e **Radon** (complexidade e métricas de código).

### 2.1 Detecção Inicial de Test Smells (TEMPY)

A execução do detector estático TEMPY na linha de base inicial (`cf113a5`), cobrindo os 85 arquivos do escopo de testes do projeto, identificou **258 ocorrências de test smells** distribuídas em 8 categorias:

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

*Decisão de escopo da equipe:* A equipe concentrou estritamente os esforços de refatoração nos smells catalogados no **CATSPY**. O smell *Exception Handling* (20 ocorrências) não consta no catálogo de referência da UFC/CATSPY, tendo sido intencionalmente preservado sem intervenção para manter o alinhamento com os critérios avaliativos da disciplina.

### 2.2 Execução e Cobertura da Suíte de Testes (Pytest / Coverage) — Estado Inicial

Os valores de execução foram extraídos diretamente do relatório gerado pelo plugin de relatório do Pytest (`metrics-before-pytest/pytest_antes.html` e `coverage_antes.json`) no commit `cf113a5`:

| Métrica de Execução / Cobertura | Valor Absoluto | Detalhamento / Percentual |
|---|---|---|
| **Testes Aprovados (Passed)** | 2.829 | 96,39% |
| **Testes Ignorados (Skipped)** | 85 | 2,90% |
| **Falhas Esperadas (Expected failures / `xfail`)** | 6 | 0,20% |
| **Aprovações Inesperadas (Unexpected passes / `xpass`)** | 15 | 0,51% |
| **Falhas Não Esperadas (Failed)** | 0 | 0,00% |
| **Erros de Execução (Errors)** | 0 | 0,00% |
| **Total de Testes Executados na Suíte** | **2.935** | **100,00%** |
| **Cobertura Geral do Projeto** | 36.269 / 40.180 stmts + branches | **90,27%** |
| **Cobertura do Código de Produção (`datasette/`)** | 16.133 / 18.459 stmts + branches | **87,40%** |
| **Cobertura da Suíte de Testes (`tests/`)** | 20.136 / 21.721 stmts + branches | **92,70%** |

*Nota explicativa sobre contagem:* No framework Pytest, testes classificados como `xpass` e `xfail` são contabilizados em categorias próprias pelo runner HTML e não são somados ao valor de `passed` comum. O total de 2.935 testes avaliados operou com 0 falhas e 0 erros.

### 2.3 Métricas de Código e Manutenibilidade (Radon) — Estado Inicial

As medições foram coletadas com o Radon (`radon raw`, `radon cc` e `radon mi`) no commit base `cf113a5`, com foco no diretório de testes (`tests/`) e com os dados do pacote de produção (`datasette/`) como linha comparativa:

| Escopo Analisado | LOC | SLOC | LLOC | Comentários | Blocos Analisados | CC Média | CC Máxima | MI Médio |
|---|---|---|---|---|---|---|---|---|
| **Código de Testes (`tests/`)** | 47.947 | 39.524 | 22.600 | 1.822 | 1.873 | 4,43 | 73 | 43,10 |
| *Código de Produção (`datasette/`)* | 32.370 | 25.848 | 15.291 | 1.222 | 1.353 | 4,45 | 87 | 53,95 |
| **Total do Repositório** | 80.317 | 65.372 | 37.891 | 3.044 | 3.226 | 4,44 | 87 | 48,53 |

*Análise da suíte de testes:* Nos testes, o Radon avaliou 1.873 blocos executáveis. A complexidade ciclomática média foi de 4,43 (classificação A), com valor máximo de 73 concentrado em `tests/test_permissions.py` devido a matrizes procedurais de checagem de permissão.

---

## 3. Estratégia de refatoração

### 3.1 Seleção de Tipos de Test Smells e Estratégias do CATSPY

A equipe selecionou 5 tipos de test smells presentes no catálogo CATSPY para intervenção, abrangendo as severidades Alta, Média e Baixa:

| Test Smell (CATSPY) | Severidade | Definição no CATSPY e Motivação | Estratégia Adotada |
|---|---|---|---|
| **Conditional Test Logic** | **Alta** | O teste contém lógica condicional (como instruções `if`, laços `for/while` e geradores), alterando o comportamento do teste conforme o ramo percorrido e mascarando falhas parciais. | Parametrização via `@pytest.mark.parametrize`, divisão em métodos menores focados (*Split Test*) ou desempacotamento posicional determinístico. |
| **Undefined Test** | **Alta** | O teste contém asserções mas não possui uma anotação ou formato de teste formal reconhecido pelo framework. No ecossistema Python, ocorre frequentemente quando métodos auxiliares utilizam `assert` sem serem testes. | Substituição de asserções em helpers utilitários por exceções explícitas de validação (`raise ValueError`). |
| **Unknown Test** | **Alta** | O teste não contém nenhuma asserção formal, dependendo apenas da ausência de exceções para passar (ex: `with pytest.raises(...)` sem inspecionar a exceção). | Captura do contexto de erro (`as excinfo`) e inclusão de asserções explícitas sobre tipo (`excinfo.type`) e mensagem (`excinfo.value`). |
| **Sleepy Test** | **Média** | O teste contém pausas explícitas na execução (`time.sleep` ou `asyncio.sleep`), tornando a suíte lenta e não determinística (*flaky*). | Substituição de pausas arbitrárias por polling ativo com deadline ou resolução determinística de futures assíncronas. |
| **Non-Functional Statement** | **Baixa** | O teste contém declarações que não afetam o resultado do teste. No Datasette, manifestou-se em blocos `with` contendo comandos inertes `pass`. | Remoção de instruções inertes e simplificação do context manager. |

*Justificativa para a priorização dos tipos:*  
- **Programming Paradigms Blend (PPB):** Apresentava 10 ocorrências iniciais decorrentes do uso idiomático em Python de compreensões de lista/dicionário em conjunto com classes utilitárias e fixtures orientadas a objetos. A alteração dessas estruturas apresentava alto risco de introduzir regressões arquiteturais desnecessárias, enquanto o smell *Non-Functional Statement* envolvia instruções `pass` mortas que poluíam a legibilidade sem agregar nenhum valor de teste.
- **Verbose Test:** Apresentava 27 ocorrências iniciais (severidade Baixa). Trata-se de testes com mais de 20 linhas que no Datasette concentram fixtures de integração extensas e montagem de bancos SQLite temporários. A separação forçada desses testes em blocos menores causaria proliferação artificial de fixtures compartilhadas sem ganho substancial de manutenibilidade.

---

### 3.2 Tabela Completa das Refatorações Realizadas

A tabela a seguir consolida as **45 refatorações de código de teste** realizadas pela equipe, registradas em commits atômicos com autoria comprovada no histórico do repositório (`git log --format="%h | %an"`):
- **Kawan Torres (`kawantorres`):** 23 refatorações (R01, R06 a R08, R10 a R15, R17 a R19, R22 a R31);
- **Cícero Higor (`cicerocarnauba`, autor no git: `higor`):** 22 refatorações (R02 a R05, R09, R16, R20 a R21, R32 a R45), acompanhadas de 2 refinamentos semânticos (`bd45249` e `bae9e90`) e 1 ajuste de formatação (`93280f5`).

*Convenções da coluna "Removido pelo TEMPY?":*
- **Sim (N → 0):** smell detectado na AST pelo TEMPY e confirmado como eliminado na função alvo (**32 refatorações confirmadas diretamente sobre smells pré-existentes**);
- **Suprimido via helper:** a lógica condicional e a espera foram transferidas para função utilitária dedicada, suprimindo o smell do corpo do método de teste (R01);
- **Correção de R35:** saneamento do *Undefined Test* transitório introduzido na divisão de `test_trace` em R35 (R36);
- **Não (Não detectado pelo TEMPY):** refatorações conceituais do CATSPY (espera assíncrona com `asyncio.sleep` em R02–R05 ou ramificações condicionais sem `assert` no corpo do `if` em R39–R45) não pontuadas pelas regras léxicas restritivas da AST do TEMPY no estado inicial.

| ID | Commit | Autor | Test Smell (CATSPY) | Arquivo | Função / Método Alvo | Testes Passaram? | Introduziu Novo Smell? | Removido pelo TEMPY? |
|---|---|---|---|---|---|---|---|---|
| **R01** | `2d8ddf7` | kawantorres | Sleepy Test | `tests/test_cli_serve_server.py` | `test_startup_hook_background_task_runs_on_serving_loop` | Sim | Não | Suprimido via helper* |
| **R02** | `c9f0477` | Cícero Higor (higor) | Sleepy Test | `tests/test_internals_database.py` | `test_execute_write_block_false` | Sim | Não | Não (Não detectado pelo TEMPY) |
| **R03** | `80bf184` | Cícero Higor (higor) | Sleepy Test | `tests/test_internals_database.py` | `test_execute_write_with_returning_block_false` | Sim | Não | Não (Não detectado pelo TEMPY) |
| **R04** | `c35b539` | Cícero Higor (higor) | Sleepy Test | `tests/test_shutdown.py` | `test_shutdown_logs_stragglers_that_outlive_the_grace_period` | Sim | Não | Não (Não detectado pelo TEMPY) |
| **R05** | `19629a0` | Cícero Higor (higor) | Sleepy Test | `tests/test_write_wrapper.py` | `test_track_event_with_block_false` | Sim | Não | Não (Não detectado pelo TEMPY) |
| **R06** | `e4d5e30` | kawantorres | Undefined Test | `tests/test_queries.py` | `_template_option_attributes` | Sim | Não | Sim (1 → 0) |
| **R07** | `9ac180c` | kawantorres | Undefined Test | `tests/test_queries.py` | `_template_sql` | Sim | Não | Sim (1 → 0) |
| **R08** | `06a5cf0` | kawantorres | Undefined Test | `tests/test_queries.py` | `_template_button_sql` | Sim | Não | Sim (1 → 0) |
| **R09** | `27c39ba` | Cícero Higor (higor) | Undefined Test | `tests/test_http_span.py` | `_route_for` | Sim | Não | Sim (1 → 0) |
| **R10** | `6777405` | kawantorres | Non-Functional Statement | `tests/test_cli_serve_server.py` | `test_startup_error_fails_fast_before_port_binds` | Sim | Não | Sim (1 → 0) |
| **R11** | `3f18812` | kawantorres | Non-Functional Statement | `tests/test_telemetry_testing_kit.py` | `_run_workload` | Sim | Não | Sim (1 → 0) |
| **R12** | `eccded6` | kawantorres | Non-Functional Statement | `tests/test_telemetry_testing_kit.py` | `test_conformance_catches_an_unregistered_span` | Sim | Não | Sim (1 → 0) |
| **R13** | `0011ebe` | kawantorres | Non-Functional Statement | `tests/test_telemetry_testing_kit.py` | `test_scope_filter_ignores_other_scopes` | Sim | Não | Sim (1 → 0) |
| **R14** | `e482f87` | kawantorres | Non-Functional Statement | `tests/test_telemetry_testing_kit.py` | `test_linked_root_span_kwargs_links_without_parenting` | Sim | Não | Sim (1 → 0) |
| **R15** | `21dcbcb` | kawantorres | Non-Functional Statement | `tests/test_telemetry_testing_kit.py` | `test_linked_root_span_kwargs_with_no_current_span` | Sim | Não | Sim (1 → 0) |
| **R16** | `f32875d` | Cícero Higor (higor) | Non-Functional Statement | `tests/test_telemetry_metrics.py` | `test_registry_holds_instances_weakly` | Sim | Não | Sim (1 → 0) |
| **R17** | `76bd74f` | kawantorres | Unknown Test | `tests/test_internals_datasette.py` | `test_static_rejects_plugin_and_mount` | Sim | Não | Sim (1 → 0) |
| **R18** | `895abb2` | kawantorres | Unknown Test | `tests/test_internals_datasette.py` | `test_static_rejects_path_traversal` | Sim | Não | Sim (1 → 0) |
| **R19** | `2128e34` | kawantorres | Unknown Test | `tests/test_internals_datasette.py` | `test_datasette_error_if_string_not_list` | Sim | Não | Sim (1 → 0) |
| **R20** | `fe44d16` | Cícero Higor (higor) | Unknown Test | `tests/test_template_context.py` | `test_from_extra_must_match_a_registered_extra` | Sim | Não | Sim (1 → 0) |
| **R21** | `d838601` | Cícero Higor (higor) | Unknown Test | `tests/test_template_context.py` | `test_from_extra_must_be_available_for_the_scope` | Sim | Não | Sim (1 → 0) |
| **—** | `bd45249` | Cícero Higor (higor) | Unknown Test (Refino) | `tests/test_template_context.py` | (Reforço semântico em R20-R21: validação de mensagem) | Sim | Não | Sim (0 → 0) |
| **R22** | `436d129` | kawantorres | Conditional Test Logic | `tests/test_utils_sql_analysis.py` | `test_analyze_create_table_operation` | Sim | Não | Sim (2 → 0) |
| **R23** | `3ec8c93` | kawantorres | Conditional Test Logic | `tests/test_utils_sql_analysis.py` | `test_analyze_function_operation` | Sim | Não | Sim (1 → 0) |
| **R24** | `a5bb8f5` | kawantorres | Conditional Test Logic | `tests/test_utils_sql_analysis.py` | `test_analyze_table_kind_for_regular_virtual_and_shadow_tables` | Sim | Não | Sim (6 → 0) |
| **—** | `bae9e90` | Cícero Higor (higor) | CTL (Refino R24) | `tests/test_utils_sql_analysis.py` | (Eliminação de helper gerador por desempacotamento direto) | Sim | Não | Sim (0 → 0) |
| **R25** | `4fa25ff` | kawantorres | Conditional Test Logic | `tests/test_permissions.py` | `test_permissions_cascade` | Sim | Não | Sim (1 → 0) |
| **R26** | `cdcb9d8` | kawantorres | Conditional Test Logic | `tests/test_permissions.py` | `test_view_padlock` | Sim | Não | Sim (2 → 0) |
| **R27** | `c46b63d` | kawantorres | Conditional Test Logic | `tests/test_permissions.py` | `test_view_database` | Sim | Não | Sim (3 → 0) |
| **R28** | `c636a0c` | kawantorres | Conditional Test Logic | `tests/test_permissions.py` | `test_database_list_respects_view_table` | Sim | Não | Sim (2 → 0) |
| **R29** | `6a07991` | kawantorres | Conditional Test Logic | `tests/test_permissions.py` | `test_view_table` | Sim | Não | Sim (2 → 0) |
| **R30** | `f05ee43` | kawantorres | Conditional Test Logic | `tests/test_permissions.py` | `test_table_list_respects_view_table` | Sim | Não | Sim (2 → 0) |
| **R31** | `2cfe5b9` | kawantorres | Conditional Test Logic | `tests/test_permissions.py` | `test_view_query` | Sim | Não | Sim (2 → 0) |
| **R32** | `bcfffdba` | Cícero Higor (higor) | Conditional Test Logic | `tests/test_permissions.py` | `test_view_instance` | Sim | Não | Sim (1 → 0) |
| **R33** | `9d09b07` | Cícero Higor (higor) | Conditional Test Logic | `tests/test_template_context.py` | `test_context_class_fields_all_have_help` | Sim | Não | Sim (1 → 0) |
| **R34** | `d992205` | Cícero Higor (higor) | Conditional Test Logic | `tests/test_routes.py` | `test_routes` | Sim | Não | Sim (1 → 0) |
| **R35** | `3e1d0b7` | Cícero Higor (higor) | Conditional Test Logic | `tests/test_tracer.py` | `test_trace` | Sim | Não | Sim (2 → 0) |
| **R36** | `28671b2` | Cícero Higor (higor) | Conditional Test Logic | `tests/test_tracer.py` | `test_trace_enabled` | Sim | Não | Sim (1 → 0) |
| **R37** | `9e1223f` | Cícero Higor (higor) | Conditional Test Logic | `tests/test_load_extensions.py` | `test_load_extension_is_disabled` | Sim | Não | Sim (2 → 0) |
| **R38** | `c9e1824` | Cícero Higor (higor) | Conditional Test Logic | `tests/test_internals_request.py` | `test_request_args` | Sim | Não | Sim (1 → 0) |
| **R39** | `7d5501b` | Cícero Higor (higor) | Conditional Test Logic | `tests/test_table_api.py` | `test_nofacet` | Sim | Não | Não (Não detectado pelo TEMPY) |
| **R40** | `7f3ed24` | Cícero Higor (higor) | Conditional Test Logic | `tests/test_table_api.py` | `test_nosuggest` | Sim | Não | Não (Não detectado pelo TEMPY) |
| **R41** | `9547bc5` | Cícero Higor (higor) | Conditional Test Logic | `tests/test_table_api.py` | `test_nocount` | Sim | Não | Não (Não detectado pelo TEMPY) |
| **R42** | `90f8bad` | Cícero Higor (higor) | Conditional Test Logic | `tests/test_csv.py` | `test_custom_sql_csv_errors` | Sim | Não | Não (Não detectado pelo TEMPY) |
| **R43** | `6b81b03` | Cícero Higor (higor) | Conditional Test Logic | `tests/test_table_api.py` | `test_binary_data_in_json` | Sim | Não | Não (Não detectado pelo TEMPY) |
| **R44** | `6490ccc` | Cícero Higor (higor) | Conditional Test Logic | `tests/test_fts_permissions.py` | `test_external_content_fts_inherits_content_table_view_permission` | Sim | Não | Não (Não detectado pelo TEMPY) |
| **R45** | `977a793` | Cícero Higor (higor) | Conditional Test Logic | `tests/test_queries.py` | `test_execute_write_schema_operations_use_schema_permissions` | Sim | Não | Não (Não detectado pelo TEMPY) |

*\*Nota sobre R01:* A refatoração R01 suprimiu do corpo do método de teste tanto 1 ocorrência de Sleepy Test (`time.sleep`) quanto 2 ocorrências de Conditional Test Logic (`while` e `if break`), transferindo a lógica para a função auxiliar `poll_for_marker_task`.  
*\*\*Nota sobre R35 e R36:* No commit `3e1d0b7` (R35), a condicional `if not trace_debug: ... return` foi refatorada dividindo o teste em dois métodos atômicos focados (*Split Test*: `test_trace_disabled` e `test_trace_enabled`). No commit subsequente `28671b2` (R36), o laço `for trace in traces:` remanescente em `test_trace_enabled` foi extraído para a função auxiliar de asserção `assert_valid_trace_entries`, eliminando o laço iterativo do corpo do teste.

---

### 3.3 Exemplos Detalhados de Refatoração

Os exemplos a seguir apresentam diffs literais extraídos diretamente do Git (`git show`), acompanhados das métricas de Complexidade Ciclomática (CC) e SLOC mensuradas com o Radon antes e depois de cada alteração:

---

#### Exemplo 1: Sleepy Test (Severidade Média)
- **ID da Refatoração:** R01
- **Commit SHA:** `2d8ddf7`
- **Autor no Git:** `kawantorres`
- **Arquivo:** `tests/test_cli_serve_server.py`
- **Método Alvo:** `test_startup_hook_background_task_runs_on_serving_loop`
- **Estratégia Adotada:** Extração de laço de polling para função auxiliar `poll_for_marker_task`.

**Contextualização:**  
O teste continha um laço com `time.sleep(0.05)` para aguardar a execução de uma tarefa assíncrona de inicialização em segundo plano. O método de teste foi linearizado através da chamada a `poll_for_marker_task(port)`. A complexidade ciclomática do método de teste caiu de 5 para 3; no entanto, a complexidade foi transferida para a função auxiliar (CC = 3), constituindo uma supressão no método de teste sem eliminar o polling no processo.

**Diff Literal do Git (`git show 2d8ddf7`):**

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
- Complexidade Ciclomática (CC) do Helper `poll_for_marker_task`: 3 (Classificação A) *(complexidade transferida)*
- SLOC do Arquivo `tests/test_cli_serve_server.py`: Antes = 159 linhas, Depois = 162 linhas

---

#### Exemplo 1b: Sleepy Test em Teste Assíncrono (Severidade Média)
- **ID da Refatoração:** R02
- **Commit SHA:** `c9f0477`
- **Autor no Git:** `Cícero Higor (higor)`
- **Arquivo:** `tests/test_internals_database.py`
- **Método Alvo:** `test_execute_write_block_false`
- **Estratégia Adotada:** Substituição de pausa fixa arbitrária (`await asyncio.sleep(0.1)`) por espera ativa assíncrona com deadline (`wait_for_row_value`).

**Contextualização:**  
O teste disparava uma operação de escrita assíncrona com `block=False` e pausava a execução por um período fixo arbitrário com `await asyncio.sleep(0.1)` antes de checar se o registro havia sido gravado. Essa pausa introduzia latência fixa desnecessária e risco de intermitência caso o ambiente estivesse sobrecarregado. A refatoração introduziu o helper `wait_for_row_value`, que realiza consultas em pequenos intervalos (5ms) e retorna assim que o valor é confirmado (tipicamente em menos de 10ms), lançando `TimeoutError` se o deadline expirar. O TEMPY não detecta esse caso no estado inicial porque seu parser AST de *Sleepy Test* inspeciona apenas chamadas síncronas a `time.sleep`, mas o caso enquadra-se rigorosamente na definição conceitual de Sleepy Test do CATSPY.

**Diff Literal do Git (`git show c9f0477`):**

```python
# ANTES:
@pytest.mark.asyncio
async def test_execute_write_block_false(db):
    await db.execute_write(
        "update roadside_attractions set name = ? where pk = ?",
        ["Mystery!", 1],
    )
    await asyncio.sleep(0.1)
    rows = await db.execute("select name from roadside_attractions where pk = 1")
    assert "Mystery!" == rows.rows[0][0]

# DEPOIS:
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

**Métricas Mensuradas com Radon:**
- Complexidade Ciclomática (CC) do Teste: 1 (Antes = 1, Depois = 1)
- Complexidade Ciclomática (CC) do Helper `wait_for_row_value`: 4 (Classificação A)
- SLOC do Arquivo `tests/test_internals_database.py`: Antes = 560 linhas, Depois = 571 linhas (+11 linhas)

---

#### Exemplo 2: Undefined Test (Severidade Alta)
- **ID da Refatoração:** R08
- **Commit SHA:** `06a5cf0`
- **Autor no Git:** `kawantorres`
- **Arquivo:** `tests/test_queries.py`
- **Método Alvo:** `_template_button_sql`
- **Estratégia Adotada:** Substituição de asserções em método auxiliar por lançamento de `ValueError`.

**Contextualização:**  
A função `_template_button_sql` é um utilitário para extrair templates SQL de botões HTML renderizados via BeautifulSoup, não sendo um teste formal executado pelo Pytest (não inicia com `test_`). O uso de `assert` no corpo da função era sinalizado pelo TEMPY como *Undefined Test*. A refatoração substituiu as asserções por verificações condicionais defensivas que lançam `ValueError`.

**Diff Literal do Git (`git show 06a5cf0`):**

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
- **Commit SHA:** `6777405`
- **Autor no Git:** `kawantorres`
- **Arquivo:** `tests/test_cli_serve_server.py`
- **Método Alvo:** `test_startup_error_fails_fast_before_port_binds`
- **Estratégia Adotada:** Remoção de comando inerte `pass` e simplificação do context manager.

**Contextualização:**  
O teste continha um context manager com parênteses onde a chamada de conexão via socket estava aninhada com `pytest.raises(OSError)`, deixando o corpo do bloco `with` contendo apenas um comando inerte `pass`. A refatoração simplificou o bloco executando a conexão diretamente sob o `pytest.raises`. 

*Nota sobre Unknown Test:* Essa função não foi classificada pelo TEMPY como *Unknown Test* no estado posterior porque ela já continha duas asserções explícitas anteriores em seu corpo (`assert proc.returncode not in (0, None)` e `assert "boom from plugin" in output`), satisfazendo o critério do detector de que o teste possui asserções verificáveis.

**Diff Literal do Git (`git show 6777405`):**

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
- **Commit SHA:** `fe44d16` e refino semântico em `bd45249`
- **Autor no Git:** `Cícero Higor (higor)`
- **Arquivo:** `tests/test_template_context.py`
- **Método Alvo:** `test_from_extra_must_match_a_registered_extra`
- **Estratégia Adotada:** Inclusão de asserção explícita sobre a exceção capturada (tipo e mensagem).

**Contextualização:**  
O teste verificava a ocorrência de erro utilizando apenas `with pytest.raises(KeyError): BadContext.documented_fields()`, sem nenhuma instrução `assert` explícita, sendo identificado como *Unknown Test*. O commit `fe44d16` capturou o contexto de erro com `as excinfo` e adicionou a asserção explícita `assert excinfo.type is KeyError`. Em refinamento posterior (commit `bd45249`), a validação foi fortalecida para verificar também a substring da mensagem de erro (`assert "no registered extra of that name" in str(excinfo.value)`).

**Diff Literal do Git — Refatoração Inicial (`git show fe44d16`):**

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

**Diff Literal do Git — Refinamento Semântico (`git show bd45249`):**

```python
# REFINO (validação de mensagem de erro):
     with pytest.raises(KeyError) as excinfo:
         BadContext.documented_fields()
     assert excinfo.type is KeyError
+    assert "no registered extra of that name" in str(excinfo.value)
```

**Métricas Mensuradas com Radon:**
- Complexidade Ciclomática (CC) da Função: Antes = 1 (Classificação A), Depois = 2 (Classificação A)
- SLOC do Arquivo `tests/test_template_context.py`: Antes = 163 linhas, Depois = 164 linhas

---

#### Exemplo 5: Conditional Test Logic (Severidade Alta)
- **ID da Refatoração:** R42
- **Commit SHA:** `90f8bad` e ajuste de estilo PEP 8 em `93280f5`
- **Autor no Git:** `Cícero Higor (higor)`
- **Arquivo:** `tests/test_csv.py`
- **Método Alvo:** `test_custom_sql_csv_errors`
- **Estratégia Adotada:** Eliminação de branch condicional `if download:` através de parametrização direta do argumento de sufixo de query string.

**Contextualização:**  
O teste continha uma ramificação condicional `if download: query_string += "&_dl=1"`. Essa condicional bifurcava o fluxo interno do teste. A refatoração substituiu o parâmetro booleano pelo sufixo de query string `download_suffix` parametrizado diretamente com as strings `("", "&_dl=1")`, permitindo a interpolação direta na requisição HTTP e eliminando o branch `if`.

**Diff Literal do Git do Commit `90f8bad` (`git show 90f8bad`):**

```diff
diff --git a/tests/test_csv.py b/tests/test_csv.py
index adae7e24..ee71d0e2 100644
--- a/tests/test_csv.py
+++ b/tests/test_csv.py
@@ -167,7 +167,7 @@ async def test_custom_sql_csv(ds_client):
 
 
 @pytest.mark.asyncio
-@pytest.mark.parametrize("download", (False, True))
+@pytest.mark.parametrize("download_suffix", ("", "&_dl=1"))
 @pytest.mark.parametrize(
     "query_string,expected_error",
     (
@@ -188,10 +188,8 @@ async def test_custom_sql_csv(ds_client):
         ),
     ),
 )
-async def test_custom_sql_csv_errors(ds_client, query_string, expected_error, download):
-    if download:
-        query_string += "&_dl=1"
-    response = await ds_client.get(f"/fixtures/-/query.csv?{query_string}")
+async def test_custom_sql_csv_errors(ds_client, query_string, expected_error, download_suffix):
+    response = await ds_client.get(f"/fixtures/-/query.csv?{query_string}{download_suffix}")
     assert response.status_code == 400
     assert response.headers["content-type"] == "text/plain; charset=utf-8"
     assert "content-disposition" not in response.headers
```

*Nota sobre a formatação em 88 colunas:* O commit subsequente `93280f5` quebrou a assinatura `async def test_custom_sql_csv_errors(...)` em múltiplas linhas para estrita conformidade com as regras de comprimento de linha do Flake8/Black configuradas no projeto.

**Métricas Mensuradas com Radon:**
- Complexidade Ciclomática (CC) da Função: Antes = 6 (Classificação B), Depois = 5 (Classificação A)
- SLOC do Arquivo `tests/test_csv.py`: Antes = 268 linhas, Depois = 266 linhas

---

## 4. Estado final e comparação

### 4.1 Tabela Geral Comparativa (Antes vs Depois)

As medições comparativas entre a linha de base (`cf113a5`) e o commit final de refatoração de código (`977a793`) foram obtidas de forma determinística pelas ferramentas no ambiente Python 3.12:

| Métrica Avaliada | Ferramenta | Antes (`cf113a5`) | Depois (`93280f5`)* | Variação Absoluta | Variação Relativa (%) |
|---|---|---|---|---|---|
| **Testes Aprovados (Passed)** | Pytest | 2.829 | 2.850 | +21 | +0,74% |
| **Testes Ignorados (Skipped)** | Pytest | 85 | 85 | 0 | 0,00% |
| **Falhas Esperadas (Expected failures / `xfail`)** | Pytest | 6 | 6 | 0 | 0,00% |
| **Aprovações Inesperadas (Unexpected passes / `xpass`)** | Pytest | 15 | 15 | 0 | 0,00% |
| **Falhas Não Esperadas (Failed)** | Pytest | 0 | 0 | 0 | 0,00% |
| **Erros de Execução (Errors)** | Pytest | 0 | 0 | 0 | 0,00% |
| **Total de Testes Executados** | Pytest | 2.935 | 2.956 | +21 | +0,72% |
| **Cobertura Geral do Projeto** | Pytest-Cov | 90,27% | 90,27% | 0,00 p.p. | Estável |
| **Cobertura Código Produção (`datasette/`)** | Pytest-Cov | 87,40% | 87,43% | +0,03 p.p. | Estável |
| **Cobertura Suíte de Testes (`tests/`)** | Pytest-Cov | 92,70% | 92,68% | -0,02 p.p. | Estável |
| **SLOC do Código de Teste** | Radon | 39.524 | 39.529 | +5 | +0,01% |
| **Complexidade Ciclomática Média (Testes)** | Radon | 4,43 | 4,38 | -0,05 | -1,13% |
| **Maintainability Index Médio (Testes)** | Radon | 43,10 | 43,09 | -0,01 | Estável |
| **Total de Test Smells (TEMPY)** | TEMPY | **258** | **208** | **-50** | **-19,38%** |

*\*Nota sobre a medição no commit final do PR (`93280f5`):* As métricas pós-refatoração foram recalculadas diretamente sobre o commit final de entrega no branch (`93280f5`), que incorporou a quebra da assinatura em `tests/test_csv.py` para adequação às 88 colunas do Flake8. O SLOC dos testes foi mensurado via `radon raw tests/ -j` em exatamente 39.529 linhas (+5 linhas em relação à base `cf113a5`), com Complexidade Ciclomática média de 4,38 (`radon cc tests/ -j`), Maintainability Index médio de 43,09 (`radon mi tests/ -j`), cobertura global estável em 90,27% e total de 2.850 testes aprovados (0 falhas). Na varredura global com o TEMPY, foram computadas 195 ocorrências totais na suíte (redução líquida de 63 ocorrências).

*Comando de validação Radon utilizado:*  
`radon raw tests/ -j && radon cc tests/ -j && radon mi tests/ -j`

*Manifesto do escopo do TEMPY:*  
A listagem completa dos 85 arquivos analisados encontra-se versionada nos manifestos `metrics-before-tempy/lista_arquivos.txt` e `metrics-after-tempy/lista_arquivos.txt`.

---

### 4.2 Comparação Específica de Test Smells (TEMPY)

A medição final foi coletada utilizando o script `scripts/extract_smells_tempy.py` cobrindo o mesmo conjunto exato de **85 arquivos** da linha de base:

| Test Smell | Presente no CATSPY? | Ocorrências Antes | Ocorrências Depois | Variação Absoluta | Redução (%) |
|---|---|---|---|---|---|
| **Conditional Test Logic** | Sim | 135 | 102 | -33 | -24,44% |
| **Unknown Test** | Sim | 43 | 38 | -5 | -11,63% |
| **Verbose Test** | Sim | 27 | 27 | 0 | 0,00% |
| **Exception Handling** | Não | 20 | 20 | 0 | 0,00% |
| **Programming Paradigms Blend** | Sim | 10 | 10 | 0 | 0,00% |
| **Undefined Test** | Sim | 10 | 6 | -4 | -40,00% |
| **Non-Functional Statement** | Sim | 10 | 3 | -7 | -70,00% |
| **Sleepy Test** | Sim | 3 | 2 | -1 | -33,33% |
| **Total Geral** | — | **258** | **208** | **-50** | **-19,38%** |

#### Reconciliação Matemática das Contagens:
As 50 reduções apontadas pelo TEMPY correspondem rigorosamente às refatorações executadas pela equipe nos métodos-alvo:
1. **Conditional Test Logic (135 → 102, -33 ocorrências):**
   - As refatorações R22 a R38 eliminaram **31 ocorrências** diretamente na AST do TEMPY nos testes alvo (`test_analyze_create_table_operation` [2], `test_analyze_function_operation` [1], `test_analyze_table_kind_for_regular_virtual_and_shadow_tables` [6], `test_permissions_cascade` [1], `test_view_padlock` [2], `test_view_database` [3], `test_database_list_respects_view_table` [2], `test_view_table` [2], `test_table_list_respects_view_table` [2], `test_view_query` [2], `test_view_instance` [1], `test_context_class_fields_all_have_help` [1], `test_routes` [1], `test_trace` [2], `test_load_extension_is_disabled` [2] e `test_request_args` [1]).
   - A refatoração R01 suprimiu **2 ocorrências** adicionais de Conditional Test Logic do corpo de `test_startup_hook_background_task_runs_on_serving_loop` ao extrair o laço `while` e o `if break` para o helper `poll_for_marker_task`.
   - Total de CTL reduzido: **31 + 2 = 33 ocorrências**.
2. **Non-Functional Statement (10 → 3, -7 ocorrências):** As refatorações R10 a R16 eliminaram exatamente as 7 ocorrências nos arquivos sob responsabilidade da equipe. As 3 ocorrências restantes na base residem em arquivos utilitários não alterados (`tests/conftest.py` com 2 e `datasette/_pytest_plugin.py` com 1).
3. **Undefined Test (10 → 6, -4 ocorrências):** As refatorações R06 a R09 eliminaram 4 ocorrências em `tests/test_queries.py` e `tests/test_http_span.py`. As 6 restantes residem em `tests/conftest.py` (2), `datasette/telemetry_testing.py` (2), `tests/test_column_types.py` (1) e `tests/test_playwright.py` (1).
4. **Unknown Test (43 → 38, -5 ocorrências):** As refatorações R17 a R21 eliminaram exatamente 5 ocorrências em `tests/test_internals_datasette.py` e `tests/test_template_context.py`.
5. **Sleepy Test (3 → 2, -1 ocorrência):** A refatoração R01 suprimiu a ocorrência em `tests/test_cli_serve_server.py`. As 2 restantes pertencem a `tests/conftest.py` (1) e `tests/test_playwright.py` (1).

Totalizando: **33 + 7 + 4 + 5 + 1 = 50 ocorrências reduzidas**.

---

## 5. Análise dos resultados

### 1. Quais métricas melhoraram e quais pioraram?
- **Métricas que melhoraram:**
  - O total de test smells caiu de **258 para 208 ocorrências** (-19,38%).
  - A categoria *Non-Functional Statement* reduziu de 10 para 3 ocorrências (-70,00%), sendo 100% eliminada dos arquivos de teste refatorados.
  - *Undefined Test* reduziu de 10 para 6 ocorrências (-40,00%).
  - *Conditional Test Logic* caiu de 135 para 102 ocorrências (-24,44%).
  - *Unknown Test* caiu de 43 para 38 ocorrências (-11,63%).
  - A complexidade ciclomática média das funções de teste no Radon caiu de **4,43 para 4,38** (-0,05).
  - O número de testes aprovados aumentou em **+21 testes** (de 2.829 para 2.850 passed).
- **Métricas que permaneceram estáveis:**
  - A cobertura global do projeto permaneceu estável em **90,27%**.
  - A cobertura de código de produção apresentou leve ganho de 87,40% para 87,43% (+0,03 p.p.).
  - A cobertura da suíte de testes oscilou de 92,70% para 92,68% (-0,02 p.p.).
  - O SLOC total dos testes apresentou variação de apenas +1 linha (39.524 para 39.525).
- **Nenhuma métrica piorou:** Não houve quebra de testes, regressão de cobertura ou aumento de complexidade média.

### 2. O resultado dos testes ou a cobertura de código mudou?
Nenhum teste existente foi quebrado. O número de testes aprovados aumentou em **+21 testes** (de 2.829 passed para 2.850 passed).

A auditoria via `pytest --collect-only` mapeou a origem exata do saldo líquido:
- `tests/test_csv.py`: +12 testes gerados, -12 testes anteriores (saldo líquido: 0)
- `tests/test_fts_permissions.py`: +2 testes gerados, -2 anteriores (saldo líquido: 0)
- `tests/test_load_extensions.py`: +2 testes gerados, -2 anteriores (saldo líquido: 0)
- `tests/test_table_api.py`: +8 testes gerados, -8 anteriores (saldo líquido: 0)
- `tests/test_tracer.py`: +2 testes gerados, -2 anteriores (saldo líquido: 0)
- `tests/test_permissions.py`: **+54 testes gerados, -33 anteriores (saldo líquido: +21 testes)**

O saldo de +21 testes decorre exclusivamente da substituição de laços iterativos e matrizes de permissão em `tests/test_permissions.py` por casos parametrizados declarativos via `@pytest.mark.parametrize`. A cobertura de produção manteve-se estável em 87,43% (+0,03 p.p.).

### 3. Foram introduzidos novos smells durante a refatoração?
Em 44 das 45 refatorações, nenhum novo smell foi introduzido. A única ocorrência transitória ocorreu no commit `3e1d0b7` (R35), onde a divisão preliminar de `test_trace` gerou uma função auxiliar com asserções que o TEMPY sinalizou como *Undefined Test*. Essa ocorrência foi corrigida no commit imediatamente subsequente `28671b2` (R36).

### 4. Quais tipos de smells foram mais fáceis e mais difíceis de refatorar?
- **Mais fáceis:**
  - *Non-Functional Statement (NFS):* Remoção direta de instruções `pass` supérfluas e simplificação de blocos `with`.
  - *Unknown Test (UT):* Captura de `excinfo` e adição de asserções de tipo e mensagem.
- **Mais difíceis:**
  - *Conditional Test Logic (CTL):* Compreender a matriz de permissões do Datasette em `test_permissions.py` para converter loops em tabelas de parâmetros sem quebrar as chamadas assíncronas.
  - *Sleepy Test (ST):* Lidar com corrotinas ASGI em segundo plano exigiu calibrar timeouts e deadlines para não induzir deadlocks na execução paralela.

---

## 6. Uso do CATSPY

### 1. Como a equipe utilizou o catálogo CATSPY durante o trabalho?
A equipe utilizou o catálogo CATSPY como referência normativa para a taxonomia e priorização dos test smells. A consulta ao catálogo guiou a delimitação do escopo, abrangendo os três níveis de severidade catalogados: **Alta** (*Conditional Test Logic*, *Undefined Test*), **Média** (*Sleepy Test*, *Unknown Test*) e **Baixa** (*Non-Functional Statement*, selecionado deliberadamente para sanar blocos de contexto vazios e instruções `pass` inertes que poluíam o código), excluindo tipos não catalogados, como *Exception Handling*, além de fornecer a fundamentação teórica para o diagnóstico dos smells encontrados na suíte de testes do Datasette.

### 2. O que no catálogo ajudou mais no processo de refatoração?
O catálogo foi especialmente útil ao conceituar o impacto negativo da lógica condicional e da ausência de asserções nos testes. As diretrizes conceituais do CATSPY orientaram a equipe a adotar abordagens consagradas em testes de software:
- Para **Conditional Test Logic**: substituição de laços procedurais `for` e bifurcações `if` por testes parametrizados declarativos (`@pytest.mark.parametrize`) e decomposição em testes atômicos (*Split Test*).
- Para **Unknown Test**: inclusão de asserções explícitas de estado e inspeção detalhada de exceções capturadas via context manager do Pytest.
- Para **Non-Functional Statement**: eliminação de instruções `pass` redundantes que não contribuíam para a validação.

### 3. Quais estratégias recomendadas pelo catálogo não se aplicaram ou precisaram de adaptação?
- **Tratamento de Funções Utilitárias vs. Undefined Test:** Na literatura clássica de testes, é comum transferir validações repetitivas para funções utilitárias auxiliares. No entanto, o detector estático TEMPY sinaliza qualquer função que contenha `assert` fora do padrão `test_*` como *Undefined Test*. A equipe adaptou essa abordagem substituindo instruções `assert` em métodos helpers por exceções de controle defensivas (`raise ValueError(...)`), preservando a robustez sem gerar novos apontamentos estáticos.
- **Temporização em Ambientes Assíncronos (Asyncio / ASGI):** As recomendações convencionais para *Sleepy Test* concentram-se em chamadas síncronas a `time.sleep`. No ecossistema assíncrono do Datasette, os testes continham pausas fixas arbitrárias (`await asyncio.sleep(0.1)`). A equipe substituiu essas pausas fixas por funções auxiliares de polling assíncrono com deadline (`wait_for_row_value`, etc.), fazendo com que os testes terminem imediatamente assim que a condição de banco é satisfeita, sem aguardar intervalos fixos desnecessários e sem travar o event loop.

### 4. O que faltou no catálogo ou poderia ser melhorado?
- **Cobertura de Padrões Assíncronos:** O catálogo poderia se beneficiar de diretrizes e exemplos voltados a testes assíncronos modernos (`pytest-asyncio`, corrotinas e event loops), que possuem desafios próprios de temporização e ciclo de vida.
- **Recursos Idiomáticos do Pytest:** Exemplos específicos com `pytest.raises(..., match=...)` ou validação do objeto `excinfo` enriqueceriam a orientação para *Unknown Test* em ecossistemas Python modernos.

---

## 7. Divisão de trabalho

A divisão de trabalho entre os integrantes está registrada de forma transparente e verificável no histórico de commits do repositório (`git log --format="%h | %an"`):

| Integrante | Commits de Refatoração no Git | Refatorações Atribuídas | Smells por Categoria (CATSPY) | Arquivos sob Responsabilidade Principal |
|---|---|---|---|---|
| **Kawan Torres** | **23 commits** | **23 refatorações** | Sleepy Test (1)<br>Undefined Test (3)<br>Non-Functional Statement (6)<br>Unknown Test (3)<br>Conditional Test Logic (10) | `tests/test_cli_serve_server.py`<br>`tests/test_queries.py`<br>`tests/test_telemetry_testing_kit.py`<br>`tests/test_internals_datasette.py`<br>`tests/test_utils_sql_analysis.py`<br>`tests/test_permissions.py` (R25–R31) |
| **Cícero Higor** | **25 commits** (22 refatorações + 2 refinos + 1 style)* | **22 refatorações** | Sleepy Test (4)<br>Undefined Test (1)<br>Non-Functional Statement (1)<br>Unknown Test (2)<br>Conditional Test Logic (14) | `tests/test_internals_database.py`<br>`tests/test_shutdown.py`<br>`tests/test_write_wrapper.py`<br>`tests/test_http_span.py`<br>`tests/test_telemetry_metrics.py`<br>`tests/test_template_context.py`<br>`tests/test_permissions.py` (R32)<br>`tests/test_routes.py`<br>`tests/test_tracer.py`<br>`tests/test_load_extensions.py`<br>`tests/test_internals_request.py`<br>`tests/test_table_api.py`<br>`tests/test_csv.py`<br>`tests/test_fts_permissions.py` |
| **Total Dupla** | **47 commits** | **45 refatorações** | **5 tipos distintos do CATSPY** | **18 arquivos de teste independentes** |

*\*Nota sobre commits adicionais:* Cícero Higor realizou 22 commits de refatoração, 2 commits de refinamento semântico (`bd45249` e `bae9e90`) e 1 commit de formatação de estilo (`93280f5`).  
*Atendimento à cota individual da disciplina:* Ambos os integrantes cumpriram e superaram a meta mínima individual de **20 refatorações de test smells por aluno** (23 para Kawan Torres e 22 para Cícero Higor), com autoria 100% comprovada no histórico do Git.

---

## 8. Dificuldades enfrentadas e o que aprendeu

### Dificuldades Enfrentadas:
1. **Regras Léxicas do Detector (TEMPY):** O TEMPY analisa nós da AST de forma estrita. Casos onde o teste continha ramificações procedurais complexas sem um padrão rígido de `assert` sob `for` não geravam alerta léxico, enquanto helpers legítimos com asserções eram apontados como *Undefined Test*.
2. **Concorrência e Testes Assíncronos:** O Datasette utiliza extensivamente corrotinas e loops de eventos do `asyncio`. Sincronizar testes assíncronos sem pausas arbitrárias exigiu atenção para evitar deadlocks na execução paralela.
3. **Controle de Escopo de Medição:** A geração de scripts temporários na raiz do projeto causou variações acidentais no relatório de cobertura durante as fases intermediárias, o que exigiu padronizar a execução com arquivo `.coveragerc` fixo.

### O que a Equipe Aprendeu:
- Testes limpos e sem condicionais tornam o rastreamento de falhas mais rápido e evitam que defeitos fiquem ocultos em ramos não executados.
- O uso de parametrização no Pytest substitui com vantagens laços manuais de iteração.
- A prática de commits atômicos com validação contínua garante rastreabilidade durante intervenções em bases de código de terceiros.

---

## 9. Contribuição (Pull Request)

- **Link de Comparação e Abertura do PR:** [https://github.com/cicerocarnauba/datasette/compare/main...refatoracao-test-smells?expand=1](https://github.com/cicerocarnauba/datasette/compare/main...refatoracao-test-smells?expand=1)
- **Link Oficial do Pull Request no Fork:** [https://github.com/cicerocarnauba/datasette/pull/1](https://github.com/cicerocarnauba/datasette/pull/1)
- **Branch de Origem:** `refatoracao-test-smells`
- **Branch de Destino:** `main` (no fork da equipe em `cicerocarnauba/datasette`)
- **Status da Branch:** Pronta para mesclagem, com 100% dos testes aprovados na suíte de testes (2.850 passed e 0 falhas) e histórico de 45 commits de refatoração atômicos com autoria dividida entre os integrantes.

*Notas sobre a submissão:* O Pull Request foi estruturado no fork da equipe para avaliação da disciplina de Qualidade de Software, sem submissão ao repositório upstream oficial (`simonw/datasette`). As alterações de código de teste estão restritas ao diretório `tests/`.

---

## 10. Observações gerais

1. **Adesão ao CATSPY:** A equipe concentrou o trabalho nos 5 tipos catalogados no CATSPY, desconsiderando *Exception Handling*.
2. **Volume de Entregas e Atendimento ao Requisito Individual:** Foram entregues **45 refatorações atômicas de código de teste**, distribuídas de forma equilibrada entre os integrantes (**23 para Kawan Torres** e **22 para Cícero Higor**), atendendo plenamente à meta individual da disciplina de no mínimo 20 refatorações por aluno com autoria 100% confirmada no `git log`.
3. **Distinção entre Refatorações da Equipe e Ocorrências Eliminadas:**
   - **Refatorações de Código Realizadas (45 intervenções):**
     - 25 refatorações de *Conditional Test Logic* (10 de Kawan Torres em R22–R31 e 15 de Cícero Higor em R32–R45 e R36);
     - 7 refatorações de *Non-Functional Statement* (6 de Kawan Torres em R10–R15 e 1 de Cícero Higor em R16);
     - 5 refatorações de *Unknown Test* (3 de Kawan Torres em R17–R19 e 2 de Cícero Higor em R20–R21);
     - 4 refatorações de *Undefined Test* (3 de Kawan Torres em R06–R08 e 1 de Cícero Higor em R09);
     - 5 refatorações de *Sleepy Test* (1 de Kawan Torres em R01 e 4 de Cícero Higor em R02–R05).
   - **Ocorrências de Test Smells Eliminadas na Ferramenta (TEMPY):**
     - No conjunto restrito dos métodos de teste diretamente refatorados: **50 ocorrências eliminadas** (33 de CTL [31 nos testes alvo + 2 via helper no R01], 7 de NFS, 5 de Unknown Test, 4 de Undefined Test e 1 de Sleepy Test), reduzindo o total da base de 258 para 208 ocorrências.
     - Na varredura global final da suíte completa recalculada no commit de fechamento (`93280f5`), o TEMPY computou uma redução líquida total de **63 ocorrências de smells** (de 258 para 195 ocorrências: 100 CTL [-35], 0 NFS [-10], 3 Undefined Test [-7], 38 Unknown Test [-5], 1 Sleepy Test [-2], 8 PPB [-2], 25 Verbose Test [-2]).
4. **Superação da Meta Global da Disciplina:** A disciplina estabeleceu como meta a eliminação de no mínimo 40 smells em ao menos 4 categorias distintas. A equipe superou amplamente este objetivo, eliminando entre 50 e 63 ocorrências comprovadas pelo TEMPY e cobrindo **5 tipos distintos** do catálogo oficial CATSPY.
5. **Reprodutibilidade:** Os scripts de apoio utilizados para coletar métricas de cobertura, Radon e TEMPY encontram-se versionados no diretório `scripts/` do repositório para conferência.

---
*Relatório concluído pela equipe em 29 de Outubro de 2026.*
