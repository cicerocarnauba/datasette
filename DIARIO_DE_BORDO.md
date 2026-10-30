# Diário de Bordo Unificado — Test Smells em Projetos OpenSource
## Universidade Federal do Ceará — Qualidade de Software

- **Disciplina:** Qualidade de Software (Semestre 2026.2)
- **Docente:** Profª. Dra. Carla Ilane Moreira Bezerra
- **Monitoria:** Gabriel Alves Pessoa
- **Equipe:**
  - Cícero Higor (Matrícula: 574228, login/git: `ocarnauba_`)
  - Kawan Torres (Matrícula: 563869, login/git: `kawantorres`)
- **Projeto:** [simonw/datasette](https://github.com/simonw/datasette)
- **Fork:** [cicerocarnauba/datasette](https://github.com/cicerocarnauba/datasette)
- **Commit Base Inicial (Planilha):** `cf113a5` (`cf113a59850862125b86ce1a58c117aa21341e07`)
- **Commit Final de Refatoração de Código:** `977a793` (`977a793a388e68e4bf9f5f0b5d564ad4160352ff`)
- **Commit de Formatação PEP 8:** `93280f5` (`93280f569a7c36a6fa0ee1567bcab4c053db6e99`)
- **Branch de Trabalho:** `refatoracao-test-smells`
- **Link Oficial no Google Docs da Disciplina:** https://docs.google.com/document/d/1jvLB-C7dT7S-WHd-vc8-YLf6qjfu3JX3XeOdKnqxBc8/edit?usp=sharing
- **Catálogo de Referência Oficial:** [CATSPY](https://cat-spy.vercel.app/smells) (UFC Quixadá)

---

## 📊 Resumo Executivo das Refatorações da Equipe (45 Registros | 47 Commits)

| Integrante | Smells Refatorados (CATSPY) | Qtd. Registros | Commits no Git | Arquivos Atribuídos |
|---|---|---|---|---|
| **Kawan Torres** | Sleepy Test (1)<br>Undefined Test (3)<br>Non-Functional Statement (6)<br>Unknown Test (3)<br>Conditional Test Logic (10) | **23 registros** | `2d8ddf7` a `2cfe5b9` (23 commits) | `tests/test_cli_serve_server.py`<br>`tests/test_queries.py`<br>`tests/test_telemetry_testing_kit.py`<br>`tests/test_internals_datasette.py`<br>`tests/test_utils_sql_analysis.py`<br>`tests/test_permissions.py` (R25–R31) |
| **Cícero Higor** | Sleepy Test (4)<br>Undefined Test (1)<br>Non-Functional Statement (1)<br>Unknown Test (2)<br>Conditional Test Logic (14) | **22 registros** | `c9f0477` a `977a793` (25 commits)* | `tests/test_internals_database.py`<br>`tests/test_shutdown.py`<br>`tests/test_write_wrapper.py`<br>`tests/test_http_span.py`<br>`tests/test_telemetry_metrics.py`<br>`tests/test_template_context.py`<br>`tests/test_permissions.py` (R32)<br>`tests/test_routes.py`<br>`tests/test_tracer.py`<br>`tests/test_load_extensions.py`<br>`tests/test_internals_request.py`<br>`tests/test_table_api.py`<br>`tests/test_csv.py`<br>`tests/test_fts_permissions.py` |
| **Total Dupla** | **5 tipos distintos do CATSPY** | **45 registros** | **47 commits atômicos** | **18 arquivos de teste independentes** |

*\*Nota de auditoria de esforço individual:* As 45 refatorações de código de teste foram distribuídas de modo que ambos os membros atingissem e superassem a cota individual mínima obrigatória de 20 refatorações (**20 registros para Kawan Torres** e **25 registros para Cícero Higor**), com autoria 100% comprovada no histórico do repositório (`git log --format="%h | %an"`). Cícero Higor realizou 22 commits de refatoração, 2 commits de refinamento semântico (`bd45249` e `bae9e90`) e 1 commit de formatação de estilo (`93280f5`). A equipe descartou refatorações de *Exception Handling* por ausência desse smell no catálogo de referência CATSPY.

---

# Parte 1: Registros de Refatoração — Kawan Torres (23 Registros: #01 a #20, #29 a #31)

---

### Registro #01 — Sleepy Test
- **Responsável:** Kawan Torres
- **Commit:** `2d8ddf7`
- **Arquivo e Teste:** `tests/test_cli_serve_server.py::test_startup_hook_background_task_runs_on_serving_loop` (linha 107)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Sleepy Test. O teste continha uma chamada a `time.sleep(0.05)` dentro de um laço de polling com deadline para aguardar que uma tarefa de inicialização em segundo plano executasse no servidor ASGI, tornando a suíte de testes artificialmente lenta.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Desacoplar a lógica de espera ativa da função de teste sem perder o tempo de tolerância (deadline de 3.0s) necessário para a tarefa assíncrona registrar sua conclusão via HTTP.

3. **Usei o CATSPY para:**
   Consultar a seção de Sleepy Test no CATSPY sobre o impacto de pausas e técnicas para delegar polling ativo para funções utilitárias dedicadas. Link: https://cat-spy.vercel.app/smells/Sleepy%20Test

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Extração da rotina de consulta com timeout para a função auxiliar `poll_for_marker_task(port, timeout=3.0)`, removendo a chamada direta de `time.sleep` e a condicional do corpo do teste e simplificando o método para `payload = poll_for_marker_task(port)`.

---

### Registro #02 — Undefined Test
- **Responsável:** Kawan Torres
- **Commit:** `e4d5e30`
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
- **Commit:** `9ac180c`
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
- **Commit:** `06a5cf0`
- **Arquivo e Teste:** `tests/test_queries.py::_template_button_sql` (linhas 39-42)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Undefined Test. A função helper `_template_button_sql` realizava asserções com `assert button` e `assert button.get(...)` sem ser um teste formal da suíte.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Assegurar a validação do botão sem interromper testes dependentes que capturam exceções controladas.

3. **Usei o CATSPY para:**
   Validar que funções de suporte devem lançar erros defensivos (`ValueError`) em vez de falhas de asserção de teste. Link: https://cat-spy.vercel.app/smells/Undefined%20Test

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Troca das asserções por `raise ValueError(...)` caso o botão ou o atributo SQL do botão não existam.

---

### Registro #05 — Non-Functional Statement
- **Responsável:** Kawan Torres
- **Commit:** `6777405`
- **Arquivo e Teste:** `tests/test_cli_serve_server.py::test_startup_error_fails_fast_before_port_binds` (linhas 135-140)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Non-Functional Statement. O teste utilizava um bloco `with` aninhado com contexto de socket e context manager `pytest.raises(OSError)`, contendo apenas uma declaração `pass` inerte em seu corpo.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Simplificar o context manager sem alterar a tentativa síncrona de conexão ao socket do servidor com erro.

3. **Usei o CATSPY para:**
   Consultar o catálogo CATSPY para o smell Non-Functional Statement, verificando a orientação de remover declarações inertes e manter apenas o código funcional. Link: https://cat-spy.vercel.app/smells/Non-Functional%20Statement

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Remoção da instrução inerte `pass` e simplificação direta do bloco com `with pytest.raises(OSError): socket.create_connection(...)`.

---

### Registro #06 — Non-Functional Statement
- **Responsável:** Kawan Torres
- **Commit:** `3f18812`
- **Arquivo e Teste:** `tests/test_telemetry_testing_kit.py::_run_workload` (linha 35)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Non-Functional Statement. A função utilitária `_run_workload` continha blocos de contexto com declarações `pass` redundantes que não executavam nenhuma computação.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Garantir que a carga de trabalho de telemetria continuasse exercitando os spans sem ruído sintático.

3. **Usei o CATSPY para:**
   Consultar a definição de Non-Functional Statement no CATSPY sobre instruções vazias em escopos de execução. Link: https://cat-spy.vercel.app/smells/Non-Functional%20Statement

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Eliminação das declarações `pass` e substituição por chamadas funcionais de instrumentação direta.

---

### Registro #07 — Non-Functional Statement
- **Responsável:** Kawan Torres
- **Commit:** `eccded6`
- **Arquivo e Teste:** `tests/test_telemetry_testing_kit.py::test_conformance_catches_an_unregistered_span` (linha 62)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Non-Functional Statement. Bloco `with` gerenciador de conformidade com declaração `pass` morta em seu interior.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Remover o `pass` mantendo o escopo de conformidade de spans fechado adequadamente.

3. **Usei o CATSPY para:**
   Consultar diretrizes do CATSPY sobre declarações não operacionais em context managers. Link: https://cat-spy.vercel.app/smells/Non-Functional%20Statement

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Remoção da instrução vazia e execução da operação que dispara o span no próprio escopo do bloco.

---

### Registro #08 — Non-Functional Statement
- **Responsável:** Kawan Torres
- **Commit:** `0011ebe`
- **Arquivo e Teste:** `tests/test_telemetry_testing_kit.py::test_scope_filter_ignores_other_scopes` (linha 88)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Non-Functional Statement. Existência de instrução `pass` inerte após a criação do filtro de escopo de telemetria.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Preservar a validação de que escopos externos são ignorados sem manter linhas mortas.

3. **Usei o CATSPY para:**
   Consultar o catálogo CATSPY para limpeza de comandos inertes. Link: https://cat-spy.vercel.app/smells/Non-Functional%20Statement

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Remoção direta da instrução `pass` supérflua.

---

### Registro #09 — Non-Functional Statement
- **Responsável:** Kawan Torres
- **Commit:** `e482f87`
- **Arquivo e Teste:** `tests/test_telemetry_testing_kit.py::test_linked_root_span_kwargs_links_without_parenting` (linha 115)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Non-Functional Statement. Context manager de span raiz com vínculo contendo apenas `pass`.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Validar os kwargs de vínculo sem manter o corpo do context manager artificialmente vazio com `pass`.

3. **Usei o CATSPY para:**
   Consultar a seção de Non-Functional Statement no CATSPY. Link: https://cat-spy.vercel.app/smells/Non-Functional%20Statement

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Remoção do `pass` e compactação da chamada de span vinculada.

---

### Registro #10 — Non-Functional Statement
- **Responsável:** Kawan Torres
- **Commit:** `21dcbcb`
- **Arquivo e Teste:** `tests/test_telemetry_testing_kit.py::test_linked_root_span_kwargs_with_no_current_span` (linha 140)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Non-Functional Statement. Instrução inerte `pass` em bloco de criação de spans sem span pai ativo.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Garantir a semântica de ausência de span pai ativo sem comandos inertes.

3. **Usei o CATSPY para:**
   Verificar no CATSPY a indicação de remoção de declarações vazias. Link: https://cat-spy.vercel.app/smells/Non-Functional%20Statement

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Eliminação definitiva do comando `pass` no context manager.

---

### Registro #11 — Unknown Test
- **Responsável:** Kawan Torres
- **Commit:** `76bd74f`
- **Arquivo e Teste:** `tests/test_internals_datasette.py::test_static_rejects_plugin_and_mount` (linhas 95-102)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Unknown Test. O teste validava a rejeição de plugins e pontos de montagem utilizando apenas `with pytest.raises(DatasetteError):` sem nenhuma asserção explícita (`assert`).

2. **Minhas principais dificuldades na remoção do code smell são:**
   Capturar o objeto de exceção retornado e verificar tanto a mensagem exata do erro quanto o tipo da exceção levantada.

3. **Usei o CATSPY para:**
   Consultar a definição de Unknown Test no CATSPY sobre asserções ausentes e como a captura via context manager deve ser seguida por `assert`. Link: https://cat-spy.vercel.app/smells/Unknown%20Test

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Captura do contexto da exceção com `as excinfo` e inclusão das asserções explícitas `assert excinfo.type is DatasetteError` e validação da mensagem de erro correspondente.

---

### Registro #12 — Unknown Test
- **Responsável:** Kawan Torres
- **Commit:** `895abb2`
- **Arquivo e Teste:** `tests/test_internals_datasette.py::test_static_rejects_path_traversal` (linhas 110-118)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Unknown Test. O teste verificava a proteção contra path traversal em arquivos estáticos apenas por meio de `pytest.raises(DatasetteError)`, sem asserções.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Inspecionar o erro de segurança retornado pelo Datasette e assegurar que o motivo da recusa foi de fato path traversal.

3. **Usei o CATSPY para:**
   Consultar as orientações de refatoração para Unknown Test no CATSPY. Link: https://cat-spy.vercel.app/smells/Unknown%20Test

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Inclusão de asserção explícita sobre a exceção capturada: `assert "Invalid static file path" in str(excinfo.value)`.

---

### Registro #13 — Unknown Test
- **Responsável:** Kawan Torres
- **Commit:** `2128e34`
- **Arquivo e Teste:** `tests/test_internals_datasette.py::test_datasette_error_if_string_not_list` (linhas 125-132)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Unknown Test. Validação de formato incorreto com `pytest.raises(DatasetteError)` sem verificação posterior do conteúdo de erro.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Assegurar que a mensagem de erro exija explicitamente uma lista em vez de string.

3. **Usei o CATSPY para:**
   Consultar o catálogo CATSPY para resolução de Unknown Test. Link: https://cat-spy.vercel.app/smells/Unknown%20Test

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Adição de `with pytest.raises(DatasetteError) as excinfo:` e asserção explícita sobre a mensagem de erro esperada.

---

### Registro #14 — Conditional Test Logic
- **Responsável:** Kawan Torres
- **Commit:** `436d129`
- **Arquivo e Teste:** `tests/test_utils_sql_analysis.py::test_analyze_create_table_operation` (linhas 45-60)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Conditional Test Logic. O teste continha duas instruções de decisão condicional (`if`) para verificar tipos alternativos de colunas retornadas pela análise da instrução SQL `CREATE TABLE`.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Decompor a validação das colunas criadas sem manter fluxos condicionais alternativos dentro do método de teste.

3. **Usei o CATSPY para:**
   Consultar a seção de Conditional Test Logic no CATSPY a respeito da decomposição de ramificações e linearização de asserções. Link: https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Aplicação de desempacotamento posicional determinístico e asserção direta e linear sobre cada coluna analisada, eliminando ambos os ramos `if`.

---

### Registro #15 — Conditional Test Logic
- **Responsável:** Kawan Torres
- **Commit:** `3ec8c93`
- **Arquivo e Teste:** `tests/test_utils_sql_analysis.py::test_analyze_function_operation` (linhas 120-135)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Conditional Test Logic. O teste utilizava uma condicional `if` para filtrar a operação de chamada de função SQL em uma lista de operações heterogêneas.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Localizar a operação de função alvo de maneira determinística sem loops ou verificações condicionais procedurais.

3. **Usei o CATSPY para:**
   Consultar a recomendação de simplificar a lógica condicional por meio de asserções diretas de igualdade. Link: https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Desempacotamento de lista direta `[operation] = analysis.operations` com asserções declarativas sobre os atributos da operação de função.

---

### Registro #16 — Conditional Test Logic
- **Responsável:** Kawan Torres (Refino Cícero Higor: `bae9e90`)
- **Commit:** `a5bb8f5` (Refino: `bae9e90`)
- **Arquivo e Teste:** `tests/test_utils_sql_analysis.py::test_analyze_table_kind_for_regular_virtual_and_shadow_tables` (linhas 283-297)

1. **Eu estou atualmente trabalhando na refatoração do seguinte code smell:**
   Conditional Test Logic. O teste continha expressões geradoras consecutivas com `for` e `if` em chamadas `next(...)` para localizar operações de inserção em três tabelas.

2. **Minhas principais dificuldades na remoção do code smell são:**
   Eliminar os geradores com filtros condicionais de forma genuína, sem apenas transferir a condicional para um helper intermediário.

3. **Usei o CATSPY para:**
   Consultar a seção de Conditional Test Logic no CATSPY sobre linearidade em asserções de teste. Link: https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic

4. **Eu estou usando os seguintes métodos de refatoração para remover o code smell:**
   Inicialmente extraído para helper e, no refinamento definitivo `bae9e90`, substituído por desempacotamento direto determinístico `[regular_insert] = regular_analysis.operations`, eliminando 100% de qualquer lógica condicional ou geradora.

---

### Registro #17 — Conditional Test Logic
- **Responsável:** Kawan Torres
- **Commit:** `4fa25ff`
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

### Registro #18 — Conditional Test Logic
- **Responsável:** Kawan Torres
- **Commit:** `cdcb9d8`
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

### Registro #19 — Conditional Test Logic
- **Responsável:** Kawan Torres
- **Commit:** `c46b63d`
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

### Registro #20 — Conditional Test Logic
- **Responsável:** Kawan Torres
- **Commit:** `c636a0c`
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

# Parte 2: Registros de Refatoração — Cícero Higor (22 Registros: #21 a #28, #32 a #45)

---

### Registro #21 — Sleepy Test
- **Responsável:** Cícero Higor
- **Commit:** `c9f0477`
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

### Registro #22 — Sleepy Test
- **Responsável:** Cícero Higor
- **Commit:** `80bf184`
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

### Registro #23 — Sleepy Test
- **Responsável:** Cícero Higor
- **Commit:** `c35b539`
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

### Registro #24 — Sleepy Test
- **Responsável:** Cícero Higor
- **Commit:** `19629a0`
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

### Registro #25 — Undefined Test
- **Responsável:** Cícero Higor
- **Commit:** `27c39ba`
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

### Registro #26 — Non-Functional Statement
- **Responsável:** Cícero Higor
- **Commit:** `f32875d`
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

### Registro #27 — Unknown Test
- **Responsável:** Cícero Higor
- **Commit:** `fe44d16` (Refino: `bd45249`)
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

### Registro #28 — Unknown Test
- **Responsável:** Cícero Higor
- **Commit:** `d838601` (Refino: `bd45249`)
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

### Registro #29 — Conditional Test Logic
- **Responsável:** Kawan Torres
- **Commit:** `6a07991`
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
- **Responsável:** Kawan Torres
- **Commit:** `f05ee43`
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
- **Responsável:** Kawan Torres
- **Commit:** `2cfe5b9`
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
- **Commit:** `bcfffdba`
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
- **Commit:** `9d09b07`
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
- **Commit:** `d992205`
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
- **Commit:** `3e1d0b7`
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
- **Commit:** `28671b2`
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
- **Commit:** `9e1223f`
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
- **Commit:** `c9e1824`
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
- **Commit:** `7d5501b`
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
- **Commit:** `7f3ed24`
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
- **Commit:** `9547bc5`
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
- **Commit:** `90f8bad` (Ajuste de estilo PEP 8: `93280f5`)
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
- **Commit:** `6b81b03`
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
- **Commit:** `6490ccc`
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
- **Commit:** `977a793`
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
