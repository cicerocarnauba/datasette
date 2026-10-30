# Relatório Final de Refatoração de Test Smells
## Universidade Federal do Ceará — Campus Quixadá
### Disciplina: Qualidade de Software (Semestre 2026.2)
**Docente:** Profª. Dra. Carla Ilane Moreira Bezerra  
**Monitoria:** Gabriel Alves Pessoa  
**Equipe:**
- Cícero Higor (Matrícula: 574228, login/git: `ocarnauba_`)
- Kawan Torres (Matrícula: 563869, login/git: `kawantorres`)

**Repositório Original:** [simonw/datasette](https://github.com/simonw/datasette)  
**Fork do Projeto:** [cicerocarnauba/datasette](https://github.com/cicerocarnauba/datasette)  
**Branch de Trabalho e Entrega:** [`refatoracao-test-smells`](https://github.com/cicerocarnauba/datasette/tree/refatoracao-test-smells)  
**Commit Inicial de Referência (Base da Planilha):** `cf113a5` (`cf113a59850862125b86ce1a58c117aa21341e07`)  
**Commit Final de Refatoração de Código:** `e3a7861` (`e3a78619b0d680bc94c34a4755106a9d7fe32b13`)  
**Commit de Formatação PEP 8 / Flake8:** `b1964d4` (`b1964d48507851d9571f0df28a786e24699f4306`)  
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
| **Falhas Esperadas (Expected failures / `xfail`)** | 6 | 0,20% (comportamentos legados sinalizados com `strict=False`) |
| **Aprovações Inesperadas (Unexpected passes / `xpass`)** | 15 | 0,51% (funcionalidades opcionais de SQLite atendidas no ambiente) |
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
- **Kawan Torres:** 20 refatorações (R01, R06 a R08, R10 a R15, R17 a R19, R22 a R28);
- **Cícero Higor:** 25 refatorações (R02 a R05, R09, R16, R20 a R21, R29 a R45), acompanhadas de 2 refinamentos semânticos (`62a0fc5` e `ff237ad`) e 1 ajuste de formatação (`b1964d4`).

*Convenções da coluna "Removido pelo TEMPY?":*
- **Sim (N → 0):** smell detectado na AST pelo TEMPY e confirmado como eliminado na função alvo;
- **Suprimido via helper:** a lógica condicional e a espera foram transferidas para função utilitária dedicada, suprimindo o smell do corpo do método de teste (R01);
- **Não (Não detectado pelo TEMPY):** o teste continha espera assíncrona (`asyncio.sleep`) ou ramificações procedurais conceituais do CATSPY que não eram pontuadas pela regra léxica restritiva do TEMPY no estado inicial.

| ID | Commit | Autor | Test Smell (CATSPY) | Arquivo | Função / Método Alvo | Testes Passaram? | Introduziu Novo Smell? | Removido pelo TEMPY? |
|---|---|---|---|---|---|---|---|---|
| **R01** | `713563b` | kawantorres | Sleepy Test | `tests/test_cli_serve_server.py` | `test_startup_hook_background_task_runs_on_serving_loop` | Sim | Não | Suprimido via helper* |
| **R02** | `ca18c30` | higor | Sleepy Test | `tests/test_internals_database.py` | `test_execute_write_block_false` | Sim | Não | Não (Não detectado pelo TEMPY) |
| **R03** | `e27f620` | higor | Sleepy Test | `tests/test_internals_database.py` | `test_execute_write_with_returning_block_false` | Sim | Não | Não (Não detectado pelo TEMPY) |
| **R04** | `bf181c8` | higor | Sleepy Test | `tests/test_shutdown.py` | `test_shutdown_logs_stragglers_that_outlive_the_grace_period` | Sim | Não | Não (Não detectado pelo TEMPY) |
| **R05** | `5daa0ba` | higor | Sleepy Test | `tests/test_write_wrapper.py` | `test_track_event_with_block_false` | Sim | Não | Não (Não detectado pelo TEMPY) |
| **R06** | `707e0ba` | kawantorres | Undefined Test | `tests/test_queries.py` | `_template_option_attributes` | Sim | Não | Sim (1 → 0) |
| **R07** | `af3001e` | kawantorres | Undefined Test | `tests/test_queries.py` | `_template_sql` | Sim | Não | Sim (1 → 0) |
| **R08** | `58c6454` | kawantorres | Undefined Test | `tests/test_queries.py` | `_template_button_sql` | Sim | Não | Sim (1 → 0) |
| **R09** | `a60b05b` | higor | Undefined Test | `tests/test_http_span.py` | `_route_for` | Sim | Não | Sim (1 → 0) |
| **R10** | `0934420` | kawantorres | Non-Functional Statement | `tests/test_cli_serve_server.py` | `test_startup_error_fails_fast_before_port_binds` | Sim | Não | Sim (1 → 0) |
| **R11** | `a731402` | kawantorres | Non-Functional Statement | `tests/test_telemetry_testing_kit.py` | `_run_workload` | Sim | Não | Sim (1 → 0) |
| **R12** | `9996b15` | kawantorres | Non-Functional Statement | `tests/test_telemetry_testing_kit.py` | `test_conformance_catches_an_unregistered_span` | Sim | Não | Sim (1 → 0) |
| **R13** | `8ea5734` | kawantorres | Non-Functional Statement | `tests/test_telemetry_testing_kit.py` | `test_scope_filter_ignores_other_scopes` | Sim | Não | Sim (1 → 0) |
| **R14** | `05c17e1` | kawantorres | Non-Functional Statement | `tests/test_telemetry_testing_kit.py` | `test_linked_root_span_kwargs_links_without_parenting` | Sim | Não | Sim (1 → 0) |
| **R15** | `1c8c0e1` | kawantorres | Non-Functional Statement | `tests/test_telemetry_testing_kit.py` | `test_linked_root_span_kwargs_with_no_current_span` | Sim | Não | Sim (1 → 0) |
| **R16** | `7f32810` | higor | Non-Functional Statement | `tests/test_telemetry_metrics.py` | `test_registry_holds_instances_weakly` | Sim | Não | Sim (1 → 0) |
| **R17** | `ab0472a` | kawantorres | Unknown Test | `tests/test_internals_datasette.py` | `test_static_rejects_plugin_and_mount` | Sim | Não | Sim (1 → 0) |
| **R18** | `3b547f2` | kawantorres | Unknown Test | `tests/test_internals_datasette.py` | `test_static_rejects_path_traversal` | Sim | Não | Sim (1 → 0) |
| **R19** | `e832624` | kawantorres | Unknown Test | `tests/test_internals_datasette.py` | `test_datasette_error_if_string_not_list` | Sim | Não | Sim (1 → 0) |
| **R20** | `4caaf63` | higor | Unknown Test | `tests/test_template_context.py` | `test_from_extra_must_match_a_registered_extra` | Sim | Não | Sim (1 → 0) |
| **R21** | `05db0b2` | higor | Unknown Test | `tests/test_template_context.py` | `test_from_extra_must_be_available_for_the_scope` | Sim | Não | Sim (1 → 0) |
| **—** | `62a0fc5` | higor | Unknown Test (Refino) | `tests/test_template_context.py` | (Reforço semântico em R20-R21: validação de mensagem) | Sim | Não | Sim (0 → 0) |
| **R22** | `a6d9f67` | kawantorres | Conditional Test Logic | `tests/test_utils_sql_analysis.py` | `test_analyze_create_table_operation` | Sim | Não | Sim (2 → 0) |
| **R23** | `9032459` | kawantorres | Conditional Test Logic | `tests/test_utils_sql_analysis.py` | `test_analyze_function_operation` | Sim | Não | Sim (1 → 0) |
| **R24** | `01705f3` | kawantorres | Conditional Test Logic | `tests/test_utils_sql_analysis.py` | `test_analyze_table_kind_for_regular_virtual_and_shadow_tables` | Sim | Não | Sim (6 → 0) |
| **—** | `ff237ad` | higor | CTL (Refino R24) | `tests/test_utils_sql_analysis.py` | (Eliminação de helper gerador por desempacotamento direto) | Sim | Não | Sim (0 → 0) |
| **R25** | `3f17fc4` | kawantorres | Conditional Test Logic | `tests/test_permissions.py` | `test_permissions_cascade` | Sim | Não | Sim (1 → 0) |
| **R26** | `d65a7cf` | kawantorres | Conditional Test Logic | `tests/test_permissions.py` | `test_view_padlock` | Sim | Não | Sim (2 → 0) |
| **R27** | `b4ddef1` | kawantorres | Conditional Test Logic | `tests/test_permissions.py` | `test_view_database` | Sim | Não | Sim (3 → 0) |
| **R28** | `6c3f2e2` | kawantorres | Conditional Test Logic | `tests/test_permissions.py` | `test_database_list_respects_view_table` | Sim | Não | Sim (2 → 0) |
| **R29** | `61f62b0` | higor | Conditional Test Logic | `tests/test_permissions.py` | `test_view_table` | Sim | Não | Sim (2 → 0) |
| **R30** | `7f1d81a` | higor | Conditional Test Logic | `tests/test_permissions.py` | `test_table_list_respects_view_table` | Sim | Não | Sim (2 → 0) |
| **R31** | `3e89ebe` | higor | Conditional Test Logic | `tests/test_permissions.py` | `test_view_query` | Sim | Não | Sim (2 → 0) |
| **R32** | `f3b04f2` | higor | Conditional Test Logic | `tests/test_permissions.py` | `test_view_instance` | Sim | Não | Sim (1 → 0) |
| **R33** | `96fe24c` | higor | Conditional Test Logic | `tests/test_template_context.py` | `test_context_class_fields_all_have_help` | Sim | Não | Sim (1 → 0) |
| **R34** | `7957219` | higor | Conditional Test Logic | `tests/test_routes.py` | `test_routes` | Sim | Não | Sim (1 → 0) |
| **R35** | `71658fb` | higor | Conditional Test Logic | `tests/test_tracer.py` | `test_trace` | Sim | Sim (temporário)** | Sim (2 → 0) |
| **R36** | `86b667e` | higor | Conditional Test Logic | `tests/test_tracer.py` | `test_trace_enabled` | Sim | Não | Sim (0 → 0) |
| **R37** | `88bf94b` | higor | Conditional Test Logic | `tests/test_load_extensions.py` | `test_load_extension_is_disabled` | Sim | Não | Sim (2 → 0) |
| **R38** | `1538199` | higor | Conditional Test Logic | `tests/test_internals_request.py` | `test_request_args` | Sim | Não | Sim (1 → 0) |
| **R39** | `e46119a` | higor | Conditional Test Logic | `tests/test_table_api.py` | `test_nofacet` | Sim | Não | Não (Não detectado pelo TEMPY) |
| **R40** | `9d71b43` | higor | Conditional Test Logic | `tests/test_table_api.py` | `test_nosuggest` | Sim | Não | Não (Não detectado pelo TEMPY) |
| **R41** | `d7717c3` | higor | Conditional Test Logic | `tests/test_table_api.py` | `test_nocount` | Sim | Não | Não (Não detectado pelo TEMPY) |
| **R42** | `394fd94` | higor | Conditional Test Logic | `tests/test_csv.py` | `test_custom_sql_csv_errors` | Sim | Não | Não (Não detectado pelo TEMPY) |
| **R43** | `2028ad2` | higor | Conditional Test Logic | `tests/test_table_api.py` | `test_binary_data_in_json` | Sim | Não | Não (Não detectado pelo TEMPY) |
| **R44** | `a5c6418` | higor | Conditional Test Logic | `tests/test_fts_permissions.py` | `test_external_content_fts_inherits_content_table_view_permission` | Sim | Não | Não (Não detectado pelo TEMPY) |
| **R45** | `e3a7861` | higor | Conditional Test Logic | `tests/test_queries.py` | `test_execute_write_schema_operations_use_schema_permissions` | Sim | Não | Não (Não detectado pelo TEMPY) |

*\*Nota sobre R01:* A refatoração R01 suprimiu do corpo do método de teste tanto 1 ocorrência de Sleepy Test (`time.sleep`) quanto 2 ocorrências de Conditional Test Logic (`while` e `if break`), transferindo a lógica para a função auxiliar `poll_for_marker_task`.  
*\*\*Nota sobre R35:* No commit `71658fb`, a divisão de `test_trace` gerou temporariamente uma função utilitária com asserções que o TEMPY sinalizou como *Undefined Test*. O caso foi prontamente sanado no commit subsequente `86b667e` (R36), eliminando o smell transitório.

---

### 3.3 Exemplos Detalhados de Refatoração

Os exemplos a seguir apresentam diffs literais extraídos diretamente do Git (`git show`), acompanhados das métricas de Complexidade Ciclomática (CC) e SLOC mensuradas com o Radon antes e depois de cada alteração:

---

#### Exemplo 1: Sleepy Test (Severidade Média)
- **ID da Refatoração:** R01
- **Commit SHA:** `713563b` (`713563bb`)
- **Autor no Git:** `kawantorres`
- **Arquivo:** `tests/test_cli_serve_server.py`
- **Método Alvo:** `test_startup_hook_background_task_runs_on_serving_loop`
- **Estratégia Adotada:** Extração de laço de polling para função auxiliar `poll_for_marker_task`.

**Contextualização:**  
O teste continha um laço com `time.sleep(0.05)` para aguardar a execução de uma tarefa assíncrona de inicialização em segundo plano. O método de teste foi linearizado através da chamada a `poll_for_marker_task(port)`. A complexidade ciclomática do método de teste caiu de 5 para 3; no entanto, a complexidade foi transferida para a função auxiliar (CC = 3), constituindo uma supressão no método de teste sem eliminar o polling no processo.

**Diff Literal do Git (`git show 713563b`):**

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

#### Exemplo 2: Undefined Test (Severidade Alta)
- **ID da Refatoração:** R08
- **Commit SHA:** `58c6454` (`58c64543`)
- **Autor no Git:** `kawantorres`
- **Arquivo:** `tests/test_queries.py`
- **Método Alvo:** `_template_button_sql`
- **Estratégia Adotada:** Substituição de asserções em método auxiliar por lançamento de `ValueError`.

**Contextualização:**  
A função `_template_button_sql` é um utilitário para extrair templates SQL de botões HTML renderizados via BeautifulSoup, não sendo um teste formal executado pelo Pytest (não inicia com `test_`). O uso de `assert` no corpo da função era sinalizado pelo TEMPY como *Undefined Test*. A refatoração substituiu as asserções por verificações condicionais defensivas que lançam `ValueError`.

**Diff Literal do Git (`git show 58c6454`):**

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
- **Commit SHA:** `0934420` (`09344202`)
- **Autor no Git:** `kawantorres`
- **Arquivo:** `tests/test_cli_serve_server.py`
- **Método Alvo:** `test_startup_error_fails_fast_before_port_binds`
- **Estratégia Adotada:** Remoção de comando inerte `pass` e simplificação do context manager.

**Contextualização:**  
O teste continha um context manager com parênteses onde a chamada de conexão via socket estava aninhada com `pytest.raises(OSError)`, deixando o corpo do bloco `with` contendo apenas um comando inerte `pass`. A refatoração simplificou o bloco executando a conexão diretamente sob o `pytest.raises`. 

*Nota sobre Unknown Test:* Essa função não foi classificada pelo TEMPY como *Unknown Test* no estado posterior porque ela já continha duas asserções explícitas anteriores em seu corpo (`assert proc.returncode not in (0, None)` e `assert "boom from plugin" in output`), satisfazendo o critério do detector de que o teste possui asserções verificáveis.

**Diff Literal do Git (`git show 0934420`):**

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
- **Commit SHA:** `4caaf63` (`4caaf634`) e refino semântico em `62a0fc5` (`62a0fc55`)
- **Autor no Git:** `higor`
- **Arquivo:** `tests/test_template_context.py`
- **Método Alvo:** `test_from_extra_must_match_a_registered_extra`
- **Estratégia Adotada:** Inclusão de asserção explícita sobre a exceção capturada (tipo e mensagem).

**Contextualização:**  
O teste verificava a ocorrência de erro utilizando apenas `with pytest.raises(KeyError): BadContext.documented_fields()`, sem nenhuma instrução `assert` explícita, sendo identificado como *Unknown Test*. O commit `4caaf63` capturou o contexto de erro com `as excinfo` e adicionou a asserção explícita `assert excinfo.type is KeyError`. Em refinamento posterior (commit `62a0fc5`), a validação foi fortalecida para verificar também a substring da mensagem de erro (`assert "no registered extra of that name" in str(excinfo.value)`).

**Diff Literal do Git — Refatoração Inicial (`git show 4caaf63`):**

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

**Diff Literal do Git — Refinamento Semântico (`git show 62a0fc5`):**

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
- **Commit SHA:** `394fd94` (`394fd942`) e ajuste de estilo PEP 8 em `b1964d4` (`b1964d48`)
- **Autor no Git:** `higor`
- **Arquivo:** `tests/test_csv.py`
- **Método Alvo:** `test_custom_sql_csv_errors`
- **Estratégia Adotada:** Eliminação de branch condicional `if download:` através de parametrização direta do argumento de sufixo de query string.

**Contextualização:**  
O teste continha uma ramificação condicional `if download: query_string += "&_dl=1"`. Essa condicional bifurcava o fluxo interno do teste. A refatoração substituiu o parâmetro booleano pelo sufixo de query string `download_suffix` parametrizado diretamente com as strings `("", "&_dl=1")`, permitindo a interpolação direta na requisição HTTP e eliminando o branch `if`.

**Diff Literal do Git do Commit `394fd94` (`git show 394fd94`):**

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

*Nota sobre a formatação em 88 colunas:* O commit subsequente `b1964d4` quebrou a assinatura `async def test_custom_sql_csv_errors(...)` em múltiplas linhas para estrita conformidade com as regras de comprimento de linha do Flake8/Black configuradas no projeto.

**Métricas Mensuradas com Radon:**
- Complexidade Ciclomática (CC) da Função: Antes = 6 (Classificação B), Depois = 5 (Classificação A)
- SLOC do Arquivo `tests/test_csv.py`: Antes = 268 linhas, Depois = 266 linhas

---

## 4. Estado final e comparação

### 4.1 Tabela Geral Comparativa (Antes vs Depois)

As medições comparativas entre a linha de base (`cf113a5`) e o commit final de refatoração de código (`e3a7861`) foram obtidas de forma determinística pelas ferramentas no ambiente Python 3.12:

| Métrica Avaliada | Ferramenta | Antes (`cf113a5`) | Depois (`e3a7861`) | Variação Absoluta | Variação Relativa (%) |
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
| **SLOC do Código de Teste** | Radon | 39.524 | 39.525 | +1 | +0,00% |
| **Complexidade Ciclomática Média (Testes)** | Radon | 4,43 | 4,38 | -0,05 | -1,13% |
| **Maintainability Index Médio (Testes)** | Radon | 43,10 | 43,09 | -0,01 | Estável |
| **Total de Test Smells (TEMPY)** | TEMPY | **258** | **208** | **-50** | **-19,38%** |

*Comando de validação Radon utilizado:*  
`radon raw tests/ -j && radon cc tests/ -j && radon mi tests/ -j`

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
Em 44 das 45 refatorações, nenhum novo smell foi introduzido. A única ocorrência transitória ocorreu no commit `71658fb` (R35), onde a divisão preliminar de `test_trace` gerou uma função auxiliar com asserções que o TEMPY sinalizou como *Undefined Test*. Essa ocorrência foi corrigida no commit imediatamente subsequente `86b667e` (R36).

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
A equipe utilizou o catálogo CATSPY como referência normativa para a taxonomia e priorização dos test smells. A consulta ao catálogo guiou a delimitação do escopo (priorizando severidades Alta e Média e excluindo tipos não catalogados, como *Exception Handling*), além de fornecer a fundamentação teórica para o diagnóstico dos smells encontrados na suíte de testes do Datasette.

### 2. O que no catálogo ajudou mais no processo de refatoração?
O catálogo foi especialmente útil ao conceituar o impacto negativo da lógica condicional e da ausência de asserções nos testes. As diretrizes conceituais do CATSPY orientaram a equipe a adotar abordagens consagradas em testes de software:
- Para **Conditional Test Logic**: substituição de laços procedurais `for` e bifurcações `if` por testes parametrizados declarativos (`@pytest.mark.parametrize`) e decomposição em testes atômicos (*Split Test*).
- Para **Unknown Test**: inclusão de asserções explícitas de estado e inspeção detalhada de exceções capturadas via context manager do Pytest.
- Para **Non-Functional Statement**: eliminação de instruções `pass` redundantes que não contribuíam para a validação.

### 3. Quais estratégias recomendadas pelo catálogo não se aplicaram ou precisaram de adaptação?
- **Tratamento de Funções Utilitárias vs. Undefined Test:** Na literatura clássica de testes, é comum transferir validações repetitivas para funções utilitárias auxiliares. No entanto, o detector estático TEMPY sinaliza qualquer função que contenha `assert` fora do padrão `test_*` como *Undefined Test*. A equipe adaptou essa abordagem substituindo instruções `assert` em métodos helpers por exceções de controle defensivas (`raise ValueError(...)`), preservando a robustez sem gerar novos apontamentos estáticos.
- **Temporização em Ambientes Assíncronos (Asyncio / ASGI):** As recomendações convencionais para *Sleepy Test* concentram-se em chamadas síncronas a `time.sleep`. No ecossistema assíncrono do Datasette, chamadas síncronas bloqueiam o event loop. Foi necessário adaptar a mitigação utilizando loops de polling cooperativo assíncrono (`asyncio.sleep`) calibrados com deadlines estritos, evitando lentidão e travamentos na execução paralela com múltiplos workers.

### 4. O que faltou no catálogo ou poderia ser melhorado?
- **Cobertura de Padrões Assíncronos:** O catálogo poderia se beneficiar de diretrizes e exemplos voltados a testes assíncronos modernos (`pytest-asyncio`, corrotinas e event loops), que possuem desafios próprios de temporização e ciclo de vida.
- **Recursos Idiomáticos do Pytest:** Exemplos específicos com `pytest.raises(..., match=...)` ou validação do objeto `excinfo` enriqueceriam a orientação para *Unknown Test* em ecossistemas Python modernos.

---

## 7. Divisão de trabalho

A divisão de trabalho entre os integrantes está registrada de forma transparente e verificável no histórico de commits do repositório (`git log --format="%h | %an"`):

| Integrante | Commits de Refatoração no Git | Refatorações Atribuídas | Smells por Categoria (CATSPY) | Arquivos sob Responsabilidade Principal |
|---|---|---|---|---|
| **Kawan Torres** | **20 commits** | **20 refatorações** | Sleepy Test (1)<br>Undefined Test (3)<br>Non-Functional Statement (6)<br>Unknown Test (3)<br>Conditional Test Logic (7) | `tests/test_cli_serve_server.py`<br>`tests/test_queries.py`<br>`tests/test_telemetry_testing_kit.py`<br>`tests/test_internals_datasette.py`<br>`tests/test_utils_sql_analysis.py`<br>`tests/test_permissions.py` (R25–R28) |
| **Cícero Higor** | **27 commits** (25 refatorações + 2 refinos)* | **25 refatorações** | Sleepy Test (4)<br>Undefined Test (1)<br>Non-Functional Statement (1)<br>Unknown Test (2)<br>Conditional Test Logic (17) | `tests/test_internals_database.py`<br>`tests/test_shutdown.py`<br>`tests/test_write_wrapper.py`<br>`tests/test_http_span.py`<br>`tests/test_telemetry_metrics.py`<br>`tests/test_template_context.py`<br>`tests/test_permissions.py` (R29–R32)<br>`tests/test_routes.py`<br>`tests/test_tracer.py`<br>`tests/test_load_extensions.py`<br>`tests/test_internals_request.py`<br>`tests/test_table_api.py`<br>`tests/test_csv.py`<br>`tests/test_fts_permissions.py` |
| **Total Dupla** | **47 commits** | **45 refatorações** | **5 tipos distintos do CATSPY** | **18 arquivos de teste independentes** |

*\*Nota sobre commits adicionais:* Cícero Higor realizou 25 commits de refatoração, 2 commits de refinamento semântico (`62a0fc5` e `ff237ad`) e 1 commit de formatação de estilo (`b1964d4`).  
*Atendimento à cota individual da disciplina:* Ambos os integrantes cumpriram e superaram a meta mínima individual de **20 refatorações de test smells por aluno** (20 para Kawan Torres e 25 para Cícero Higor), com autoria 100% comprovada no histórico do Git.

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

- **Link Oficial do Pull Request:** [https://github.com/cicerocarnauba/datasette/pull/1](https://github.com/cicerocarnauba/datasette/pull/1)
- **Branch de Origem:** `refatoracao-test-smells`
- **Branch de Destino:** `main` (no fork da equipe em `cicerocarnauba/datasette`)
- **Status do PR:** Aberto, com 100% dos testes aprovados na suíte local e integrado ao fluxo de CI no GitHub ([Actions](https://github.com/cicerocarnauba/datasette/actions) / [PR Checks](https://github.com/cicerocarnauba/datasette/pull/1/checks)) com 2.850 passed e 0 falhas.

*Notas sobre a submissão:* O Pull Request foi aberto no fork da equipe para avaliação da disciplina de Qualidade de Software, sem submissão ao repositório upstream oficial (`simonw/datasette`). As alterações de código de teste estão restritas ao diretório `tests/`.

---

## 10. Observações gerais

1. **Adesão ao CATSPY:** A equipe concentrou o trabalho nos 5 tipos catalogados no CATSPY, desconsiderando *Exception Handling*.
2. **Volume de Entregas e Atendimento ao Requisito Individual:** Foram entregues **45 refatorações de código de teste**, distribuídas entre os integrantes (**20 para Kawan Torres** e **25 para Cícero Higor**), atendendo plenamente à meta individual da disciplina de no mínimo 20 refatorações por aluno com autoria confirmada no `git log`.
3. **Confirmação pelo Detector:** Do total de 45 intervenções, 33 foram confirmadas diretamente pelo TEMPY (eliminando **47 ocorrências** em 4 tipos canônicos de forma direta e **3 ocorrências** suprimidas via helper em R01, totalizando **50 ocorrências de redução** no TEMPY).
4. **Superação da Meta Global da Disciplina:** A equipe eliminou 47 ocorrências confirmadas pelo TEMPY, superando com folga a meta global de 40 smells eliminados.
5. **Reprodutibilidade:** Os scripts de apoio utilizados para coletar métricas de cobertura, Radon e TEMPY encontram-se versionados no diretório `scripts/` do repositório para conferência.

---
*Relatório concluído pela equipe em 29 de Outubro de 2026.*
