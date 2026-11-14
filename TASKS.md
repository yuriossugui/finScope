# TASKS.md --- FinScope

> Backlog de implementação derivado de `AGENTS.md`.\
> `AGENTS.md` é a fonte de verdade para escopo, arquitetura, regras de
> negócio e critérios de qualidade.

## Convenções

Status permitidos: - `pending` - `in progress` - `completed` - `blocked`

Regras para agentes: 1. Trabalhar na ordem das fases, respeitando
dependências. 2. Executar uma tarefa por vez, salvo tarefas
explicitamente independentes. 3. Não marcar uma tarefa como `completed`
sem cumprir seus critérios de aceitação e executar os testes relevantes.
4. Atualizar este arquivo ao iniciar, concluir ou bloquear uma tarefa.
5. Se uma decisão técnica ainda não estiver definida, registrar a
decisão antes de implementar a parte dependente. 6. Não ampliar o escopo
sem atualizar primeiro `AGENTS.md`.

------------------------------------------------------------------------

## Fase 0 --- Decisões e preparação

### FS-001 --- Inicializar projeto Next.js

-   **Status:** `pending`
-   **Objetivo:** Criar a aplicação Next.js com TypeScript e App Router.
-   **Tarefas:**
    -   [ ] Criar projeto com configuração TypeScript estrita.
    -   [ ] Configurar ESLint e scripts de desenvolvimento, build e
        lint.
    -   [ ] Instalar e configurar Tailwind CSS.
    -   [ ] Inicializar shadcn/ui.
    -   [ ] Criar estrutura inicial de diretórios.
-   **Aceitação:**
    -   [ ] Aplicação inicia localmente.
    -   [ ] `lint` e `build` executam sem erros.
    -   [ ] Estrutura inicial está documentada no README.

### FS-002 --- Definir ORM e estratégia de migrações

-   **Status:** `pending`
-   **Objetivo:** Escolher uma camada de acesso a dados compatível com
    PostgreSQL e pgvector.
-   **Tarefas:**
    -   [ ] Avaliar Prisma e Drizzle considerando suporte a PostgreSQL,
        migrações e extensão pgvector.
    -   [ ] Escolher uma opção e registrar justificativa no README.
    -   [ ] Definir como executar SQL específico para operações
        vetoriais, caso necessário.
-   **Aceitação:**
    -   [ ] ORM escolhido e configurado.
    -   [ ] Estratégia de migração documentada.
    -   [ ] Uma migração de teste é aplicada com sucesso.

### FS-003 --- Definir modelo de embeddings

-   **Status:** `pending`
-   **Objetivo:** Selecionar modelo compatível com Transformers.js para
    português.
-   **Tarefas:**
    -   [ ] Comparar modelos candidatos para busca semântica
        multilíngue.
    -   [ ] Verificar licença, dimensão vetorial e compatibilidade com
        Transformers.js.
    -   [ ] Medir geração de embedding com textos curtos em português.
    -   [ ] Registrar nome, versão/revisão e dimensão do modelo.
-   **Aceitação:**
    -   [ ] Modelo escolhido com justificativa.
    -   [ ] Dimensão vetorial documentada.
    -   [ ] Modelo pode ser carregado e gerar embedding no runtime
        Node.js.

### FS-004 --- Configurar ambiente e validação de variáveis

-   **Status:** `pending`
-   **Objetivo:** Padronizar configuração local e segredos.
-   **Tarefas:**
    -   [ ] Criar `.env.example`.
    -   [ ] Configurar validação de variáveis com Zod.
    -   [ ] Documentar configuração do PostgreSQL, Groq e modelo de
        embeddings.
    -   [ ] Garantir que segredos não sejam expostos ao cliente.
-   **Aceitação:**
    -   [ ] Aplicação informa claramente quais variáveis obrigatórias
        estão ausentes.
    -   [ ] Nenhuma chave real está versionada.

------------------------------------------------------------------------

## Fase 1 --- Banco de dados e domínio

### FS-005 --- Configurar PostgreSQL e pgvector

-   **Status:** `pending`
-   **Objetivo:** Preparar banco relacional e extensão vetorial.
-   **Tarefas:**
    -   [ ] Configurar conexão PostgreSQL.
    -   [ ] Habilitar extensão `vector`.
    -   [ ] Definir estratégia de migrações para a extensão.
    -   [ ] Criar verificação de conectividade.
-   **Aceitação:**
    -   [ ] Aplicação conecta ao PostgreSQL.
    -   [ ] Extensão pgvector está habilitada.
    -   [ ] Uma coluna vetorial de teste é criada com dimensão
        compatível com o modelo escolhido.

### FS-006 --- Modelar entidades principais

-   **Status:** `pending`
-   **Objetivo:** Criar estrutura de persistência do domínio.
-   **Tarefas:**
    -   [ ] Modelar User/escopo de proprietário para evolução
        multiusuário.
    -   [ ] Modelar Statement.
    -   [ ] Modelar Transaction.
    -   [ ] Modelar Category.
    -   [ ] Modelar Conversation e Message.
    -   [ ] Modelar Document/Chunk e metadados de embedding.
    -   [ ] Definir índices e restrições de integridade.
-   **Aceitação:**
    -   [ ] Relacionamentos e restrições estão documentados.
    -   [ ] Valores monetários usam tipo decimal/numeric.
    -   [ ] Migrações são aplicadas em banco limpo.
    -   [ ] Testes básicos de persistência passam.

### FS-007 --- Implementar camada de acesso a dados

-   **Status:** `pending`
-   **Objetivo:** Isolar consultas e persistência do restante da
    aplicação.
-   **Tarefas:**
    -   [ ] Criar conexão reutilizável com o banco.
    -   [ ] Criar repositórios ou funções de persistência para extratos
        e transações.
    -   [ ] Criar operações para conversas e mensagens.
    -   [ ] Criar operação de persistência e busca de chunks vetoriais.
-   **Aceitação:**
    -   [ ] Serviços não dependem de detalhes de componentes React.
    -   [ ] Consultas são parametrizadas.
    -   [ ] Testes de integração cobrem operações principais.

------------------------------------------------------------------------

## Fase 2 --- Importação OFX

### FS-008 --- Implementar parser OFX

-   **Status:** `pending`
-   **Objetivo:** Interpretar arquivos OFX e extrair dados bancários
    relevantes.
-   **Tarefas:**
    -   [ ] Selecionar biblioteca/parser e validar compatibilidade com
        os formatos OFX esperados.
    -   [ ] Criar interface interna para parsing.
    -   [ ] Extrair metadados da conta e do extrato quando disponíveis.
    -   [ ] Extrair transações, datas, valores, FITID, memo e descrição.
    -   [ ] Tratar campos opcionais e formatos inconsistentes.
    -   [ ] Criar fixtures OFX válidas e inválidas para testes.
-   **Aceitação:**
    -   [ ] Parser retorna estrutura tipada e normalizada.
    -   [ ] Arquivos inválidos produzem erros controlados.
    -   [ ] Testes cobrem campos ausentes, datas e sinais de valores.

### FS-009 --- Implementar validação e normalização OFX

-   **Status:** `pending`
-   **Objetivo:** Garantir consistência dos dados antes da persistência.
-   **Tarefas:**
    -   [ ] Validar extensão, conteúdo e tamanho máximo.
    -   [ ] Padronizar datas e moeda.
    -   [ ] Padronizar convenção de débito/crédito.
    -   [ ] Preservar descrição original.
    -   [ ] Definir deduplicação por FITID e escopo de conta/extrato.
    -   [ ] Definir comportamento quando FITID estiver ausente.
-   **Aceitação:**
    -   [ ] Valores e sinais seguem uma convenção única.
    -   [ ] Importação repetida não duplica movimentações indevidamente.
    -   [ ] Testes cobrem duplicidade e dados incompletos.

### FS-010 --- Criar endpoint de upload e importação

-   **Status:** `pending`
-   **Objetivo:** Disponibilizar importação de OFX pela API.
-   **Tarefas:**
    -   [ ] Criar Route Handler de upload.
    -   [ ] Validar arquivo recebido no servidor.
    -   [ ] Criar registro de importação e estados de processamento.
    -   [ ] Executar parsing e persistência com consistência.
    -   [ ] Retornar identificador e status da importação.
    -   [ ] Tratar falhas sem marcar importação como concluída.
-   **Aceitação:**
    -   [ ] Upload válido cria extrato e transações.
    -   [ ] Upload inválido retorna erro apropriado.
    -   [ ] Falhas não deixam dados parcialmente marcados como sucesso.
    -   [ ] Testes de integração cobrem sucesso e falha.

### FS-011 --- Criar tela de extratos

-   **Status:** `pending`
-   **Objetivo:** Permitir enviar e consultar extratos importados.
-   **Tarefas:**
    -   [ ] Criar componente de upload.
    -   [ ] Exibir progresso/estado de processamento.
    -   [ ] Listar extratos importados.
    -   [ ] Exibir período, instituição quando disponível, quantidade de
        transações e status.
    -   [ ] Implementar estados vazio, erro e carregamento.
-   **Aceitação:**
    -   [ ] Usuário consegue importar arquivo OFX pela interface.
    -   [ ] Lista reflete os dados persistidos.
    -   [ ] Mensagens de erro são compreensíveis.

------------------------------------------------------------------------

## Fase 3 --- Dashboard financeiro

### FS-012 --- Implementar consultas financeiras

-   **Status:** `pending`
-   **Objetivo:** Calcular indicadores a partir dos dados estruturados.
-   **Tarefas:**
    -   [ ] Calcular saldo inicial/final quando disponível.
    -   [ ] Calcular total de entradas e saídas.
    -   [ ] Calcular quantidade de transações.
    -   [ ] Preparar série temporal para evolução do saldo, quando
        possível.
    -   [ ] Preparar agregações por categoria.
    -   [ ] Garantir filtros pelo extrato selecionado.
-   **Aceitação:**
    -   [ ] Resultados são calculados por SQL/código determinístico.
    -   [ ] Não dependem de respostas do LLM.
    -   [ ] Testes cobrem valores positivos, negativos e ausência de
        dados.

### FS-013 --- Implementar endpoints do dashboard e transações

-   **Status:** `pending`
-   **Objetivo:** Expor indicadores e movimentações à interface.
-   **Tarefas:**
    -   [ ] Criar endpoint de resumo por extrato.
    -   [ ] Criar endpoint de transações paginadas.
    -   [ ] Criar filtros por texto, período, tipo e categoria.
    -   [ ] Validar parâmetros com Zod.
-   **Aceitação:**
    -   [ ] Todas as consultas exigem e respeitam o escopo do extrato.
    -   [ ] Paginação e filtros funcionam.
    -   [ ] Testes de integração cobrem filtros e limites.

### FS-014 --- Construir dashboard com glassmorphism

-   **Status:** `pending`
-   **Objetivo:** Apresentar visualmente os dados do extrato.
-   **Tarefas:**
    -   [ ] Criar layout principal e navegação.
    -   [ ] Criar cards de saldo, entradas e saídas.
    -   [ ] Criar gráficos de evolução e categorias.
    -   [ ] Criar tabela/lista de transações.
    -   [ ] Implementar seleção clara do extrato.
    -   [ ] Criar estados vazio, carregamento e erro.
-   **Aceitação:**
    -   [ ] Indicadores correspondem às consultas da API.
    -   [ ] Interface segue a linguagem glassmorphism definida em
        AGENTS.md.
    -   [ ] Contraste, legibilidade e responsividade são preservados.
    -   [ ] Não há dados financeiros fictícios apresentados como reais.

------------------------------------------------------------------------

## Fase 4 --- Embeddings e indexação

### FS-015 --- Criar serviço de embeddings com Transformers.js

-   **Status:** `pending`
-   **Objetivo:** Encapsular geração local de embeddings.
-   **Tarefas:**
    -   [ ] Criar interface `EmbeddingProvider`.
    -   [ ] Implementar provider com Transformers.js.
    -   [ ] Carregar modelo de forma reutilizável no servidor.
    -   [ ] Normalizar formato de entrada e saída.
    -   [ ] Tratar falhas de carregamento e inferência.
    -   [ ] Registrar modelo e dimensão usados.
-   **Aceitação:**
    -   [ ] Serviço gera vetor no servidor Node.js.
    -   [ ] Não carrega o modelo repetidamente a cada
        transação/requisição.
    -   [ ] Dimensão retornada é validada.
    -   [ ] Testes cobrem contrato, erros e dimensão.

### FS-016 --- Definir estratégia de documentos e chunks

-   **Status:** `pending`
-   **Objetivo:** Criar unidades textuais úteis para recuperação
    semântica.
-   **Tarefas:**
    -   [ ] Definir quais campos da transação entram no texto indexável.
    -   [ ] Preservar referência ao extrato e à transação de origem.
    -   [ ] Definir tamanho e agrupamento dos chunks.
    -   [ ] Separar conteúdo textual de valores estruturados usados em
        cálculos.
    -   [ ] Definir política de reindexação quando modelo ou conteúdo
        mudar.
-   **Aceitação:**
    -   [ ] Estratégia documentada com exemplos.
    -   [ ] Cada chunk possui origem rastreável.
    -   [ ] Não há dependência de busca vetorial para cálculos exatos.

### FS-017 --- Implementar pipeline de indexação

-   **Status:** `pending`
-   **Objetivo:** Gerar e persistir embeddings dos conteúdos preparados.
-   **Tarefas:**
    -   [ ] Criar serviço de preparação de documentos.
    -   [ ] Gerar embeddings em lote quando suportado.
    -   [ ] Persistir conteúdo, metadados, vetor e identificador do
        modelo.
    -   [ ] Implementar status de indexação separado do status
        financeiro da importação.
    -   [ ] Permitir reprocessamento de itens que falharam.
-   **Aceitação:**
    -   [ ] Chunks são persistidos com vetores da dimensão correta.
    -   [ ] Falha de embedding não invalida transações já importadas.
    -   [ ] Reprocessamento não cria chunks duplicados sem controle.

### FS-018 --- Implementar busca vetorial

-   **Status:** `pending`
-   **Objetivo:** Recuperar conteúdo semanticamente relevante dentro de
    um extrato.
-   **Tarefas:**
    -   [ ] Gerar embedding da pergunta com o mesmo provider/modelo.
    -   [ ] Implementar consulta pgvector com filtro obrigatório de
        extrato.
    -   [ ] Configurar quantidade máxima de resultados.
    -   [ ] Avaliar métrica de distância e estratégia de relevância.
    -   [ ] Retornar conteúdo e metadados de origem.
-   **Aceitação:**
    -   [ ] Busca retorna resultados relevantes para perguntas de teste.
    -   [ ] Nenhum resultado de outro extrato é retornado.
    -   [ ] Consultas são parametrizadas.
    -   [ ] Testes cobrem isolamento e ausência de resultados.

------------------------------------------------------------------------

## Fase 5 --- Chat, Groq e Tool Calling

### FS-019 --- Criar integração com Groq

-   **Status:** `pending`
-   **Objetivo:** Encapsular chamadas ao modelo conversacional.
-   **Tarefas:**
    -   [ ] Criar interface `LLMProvider`.
    -   [ ] Implementar cliente Groq no servidor.
    -   [ ] Configurar modelo por variável de ambiente.
    -   [ ] Definir prompt de sistema inicial.
    -   [ ] Tratar timeout, rate limit, indisponibilidade e erros.
    -   [ ] Definir limites de tokens/contexto.
-   **Aceitação:**
    -   [ ] Aplicação envia mensagens à Groq sem expor a chave ao
        cliente.
    -   [ ] Erros do provedor são convertidos em respostas controladas.
    -   [ ] Testes usam mocks para não depender da API em toda execução.

### FS-020 --- Implementar ferramentas financeiras

-   **Status:** `pending`
-   **Objetivo:** Permitir ao LLM solicitar consultas estruturadas e
    controladas.
-   **Ferramentas iniciais:**
    -   [ ] `get_statement_summary`
    -   [ ] `get_largest_expenses`
    -   [ ] `get_expenses_by_category`
    -   [ ] `search_transactions`
-   **Tarefas:**
    -   [ ] Definir schemas Zod de argumentos.
    -   [ ] Implementar funções de execução no servidor.
    -   [ ] Aplicar filtro obrigatório por extrato.
    -   [ ] Limitar resultados e validar intervalos.
    -   [ ] Retornar dados estruturados e rastreáveis.
-   **Aceitação:**
    -   [ ] Ferramentas não aceitam SQL arbitrário.
    -   [ ] Argumentos inválidos são rejeitados.
    -   [ ] Consultas respeitam o extrato selecionado.
    -   [ ] Testes unitários e de integração cobrem cada ferramenta.

### FS-021 --- Orquestrar RAG + SQL + Groq

-   **Status:** `pending`
-   **Objetivo:** Construir o fluxo que decide e combina recuperação
    semântica e consultas estruturadas.
-   **Tarefas:**
    -   [ ] Receber pergunta e extrato selecionado.
    -   [ ] Definir estratégia de seleção de ferramentas.
    -   [ ] Executar tool calling com validação.
    -   [ ] Executar busca vetorial quando houver necessidade semântica.
    -   [ ] Montar contexto textual com origem dos dados.
    -   [ ] Enviar contexto mínimo necessário à Groq.
    -   [ ] Tratar perguntas sem dados suficientes.
    -   [ ] Registrar metadados úteis para depuração sem registrar dados
        sensíveis desnecessários.
-   **Aceitação:**
    -   [ ] Perguntas quantitativas usam ferramentas SQL.
    -   [ ] Perguntas semânticas podem usar recuperação vetorial.
    -   [ ] Perguntas mistas combinam os dois mecanismos.
    -   [ ] Respostas não inventam dados ausentes.
    -   [ ] Testes cobrem os três tipos de pergunta.

### FS-022 --- Persistir conversas e mensagens

-   **Status:** `pending`
-   **Objetivo:** Manter histórico das conversas vinculadas a um
    extrato.
-   **Tarefas:**
    -   [ ] Criar operações para abrir conversa por extrato.
    -   [ ] Persistir mensagens do usuário e respostas.
    -   [ ] Definir tratamento para falha após persistência da mensagem
        do usuário.
    -   [ ] Recuperar histórico com paginação/limite.
-   **Aceitação:**
    -   [ ] Conversa sempre possui vínculo com extrato.
    -   [ ] Histórico é recuperado na ordem correta.
    -   [ ] Falhas do provedor não apagam a pergunta enviada.

### FS-023 --- Criar endpoint e interface de chat

-   **Status:** `pending`
-   **Objetivo:** Disponibilizar a experiência conversacional.
-   **Tarefas:**
    -   [ ] Criar Route Handler de chat.
    -   [ ] Validar mensagem, conversa e extrato.
    -   [ ] Criar tela de chat com glassmorphism.
    -   [ ] Implementar seletor de extrato.
    -   [ ] Exibir histórico, envio, carregamento e erro.
    -   [ ] Exibir referências ou transações usadas quando disponíveis.
-   **Aceitação:**
    -   [ ] Usuário conversa sobre o extrato selecionado.
    -   [ ] A interface deixa explícito o contexto da conversa.
    -   [ ] Estados de carregamento e erro são tratados.
    -   [ ] Respostas são exibidas sem perder estrutura e legibilidade.

------------------------------------------------------------------------

## Fase 6 --- Qualidade, segurança e validação

### FS-024 --- Implementar testes de ponta a ponta do fluxo principal

-   **Status:** `pending`
-   **Objetivo:** Validar o percurso completo do usuário.
-   **Tarefas:**
    -   [ ] Testar importação OFX.
    -   [ ] Testar dashboard com dados importados.
    -   [ ] Testar chat vinculado ao extrato.
    -   [ ] Testar estados de erro e ausência de dados.
-   **Aceitação:**
    -   [ ] Fluxo principal funciona em ambiente de teste.
    -   [ ] Testes não dependem de credenciais reais de IA quando mocks
        forem suficientes.

### FS-025 --- Criar conjunto de avaliação de respostas da IA

-   **Status:** `pending`
-   **Objetivo:** Avaliar a qualidade do RAG e das respostas.
-   **Tarefas:**
    -   [ ] Criar extratos de teste sintéticos.
    -   [ ] Criar perguntas quantitativas com resultados esperados.
    -   [ ] Criar perguntas semânticas com documentos relevantes
        esperados.
    -   [ ] Criar perguntas sem resposta nos dados.
    -   [ ] Avaliar recuperação, precisão numérica e fidelidade.
-   **Aceitação:**
    -   [ ] Casos de avaliação são reproduzíveis.
    -   [ ] Resultados e limitações são documentados.
    -   [ ] Respostas incorretas conhecidas geram tarefas de correção.

### FS-026 --- Revisar segurança e privacidade

-   **Status:** `pending`
-   **Objetivo:** Reduzir risco de exposição ou uso indevido de dados
    financeiros.
-   **Tarefas:**
    -   [ ] Revisar validação de upload.
    -   [ ] Revisar escopo de extrato em todas as rotas e ferramentas.
    -   [ ] Revisar logs e tratamento de erros.
    -   [ ] Revisar exposição de variáveis de ambiente.
    -   [ ] Revisar prompt injection em descrições e conteúdo
        recuperado.
    -   [ ] Documentar que autenticação e isolamento por usuário são
        pré-requisitos para publicação com dados reais.
-   **Aceitação:**
    -   [ ] Não há segredo exposto no frontend.
    -   [ ] Não há consulta sem filtro de escopo.
    -   [ ] Conteúdo recuperado não pode alterar instruções ou
        permissões do sistema.
    -   [ ] Limitações de segurança restantes estão documentadas.

### FS-027 --- Documentar execução e arquitetura

-   **Status:** `pending`
-   **Objetivo:** Permitir que outra pessoa configure, execute e
    compreenda o projeto.
-   **Tarefas:**
    -   [ ] Documentar pré-requisitos.
    -   [ ] Documentar configuração de PostgreSQL e pgvector.
    -   [ ] Documentar variáveis de ambiente.
    -   [ ] Documentar comandos de desenvolvimento, teste e build.
    -   [ ] Documentar arquitetura e fluxo de ingestão/RAG.
    -   [ ] Documentar limitações conhecidas.
-   **Aceitação:**
    -   [ ] Uma instalação limpa consegue seguir o README.
    -   [ ] Os comandos documentados foram verificados.
    -   [ ] A arquitetura descrita corresponde à implementação.

------------------------------------------------------------------------

## Definição de pronto do MVP

O MVP pode ser considerado concluído quando:

-   [ ] É possível importar um arquivo OFX válido.
-   [ ] As transações são normalizadas e persistidas sem duplicidade
    indevida.
-   [ ] O usuário consegue listar e selecionar extratos.
-   [ ] O dashboard apresenta indicadores calculados a partir do banco.
-   [ ] É possível abrir uma conversa vinculada a um extrato.
-   [ ] Groq responde perguntas utilizando contexto recuperado.
-   [ ] Transformers.js gera embeddings no servidor.
-   [ ] pgvector recupera conteúdo relevante e restrito ao extrato
    selecionado.
-   [ ] Consultas numéricas são executadas por ferramentas SQL
    controladas.
-   [ ] Respostas indicam ausência de dados quando aplicável.
-   [ ] Testes relevantes passam.
-   [ ] README e documentação técnica estão atualizados.
-   [ ] A interface segue a direção visual glassmorphism sem comprometer
    acessibilidade e legibilidade.

## Bloqueios e decisões pendentes

Registrar aqui decisões que impedem o avanço de tarefas:

-   [ ] Escolher ORM (Prisma ou Drizzle).
-   [ ] Selecionar e validar modelo específico de embeddings
    Transformers.js.
-   [ ] Escolher biblioteca de parsing OFX e validar formatos reais de
    entrada.
-   [ ] Definir limite máximo de upload.
-   [ ] Definir modelo Groq disponível na conta e compatível com tool
    calling.
-   [ ] Definir estratégia de execução assíncrona/reprocessamento para
    indexação, conforme desempenho medido.
