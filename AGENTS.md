# AGENTS.md --- FinScope

> Documento-fonte de verdade para agentes de desenvolvimento.\
> Em caso de conflito entre este arquivo e `TASKS.md`, prevalecem os
> requisitos e regras deste arquivo.

## 1. Visão do produto

FinScope é uma aplicação web de análise de extratos bancários com
assistência de IA. O usuário importa arquivos `.ofx`, visualiza as
movimentações em um dashboard e conversa com um assistente que responde
perguntas fundamentadas nos dados de um extrato selecionado.

O projeto tem dois objetivos simultâneos:

1.  Entregar um MVP funcional, pequeno e coerente.
2.  Servir como projeto de aprofundamento em engenharia de aplicações
    com IA, especialmente embeddings, recuperação semântica, RAG, tool
    calling e avaliação de respostas.

A aplicação deve ser construída de forma incremental. Não adicionar
funcionalidades fora do escopo sem uma decisão explícita.

## 2. Escopo do MVP

O MVP contempla três áreas principais:

1.  **Importação e listagem de extratos:** upload de arquivos OFX,
    validação, parsing, normalização e persistência.
2.  **Dashboard:** visualização do resumo financeiro e das transações de
    um extrato selecionado.
3.  **Chat com IA:** conversa contextualizada sobre um extrato
    específico, utilizando consultas estruturadas e recuperação
    semântica quando apropriado.

### Fora do escopo inicial

-   Integração bancária/Open Finance.
-   Sincronização automática com bancos.
-   Aplicativo mobile.
-   Gestão de investimentos ou cartões.
-   Previsões financeiras complexas.
-   Agentes autônomos com capacidade de executar operações financeiras.
-   Compartilhamento de extratos entre usuários.
-   Importação de formatos diferentes de OFX, salvo decisão posterior.

## 3. Stack técnica definida

  -----------------------------------------------------------------------
  Camada                  Tecnologia              Responsabilidade
  ----------------------- ----------------------- -----------------------
  Framework               Next.js com TypeScript  Frontend e backend da
                                                  aplicação

  Backend HTTP            Next.js App Router /    Endpoints da aplicação
                          Route Handlers          

  UI                      Tailwind CSS +          Componentes e estilos
                          shadcn/ui               

  Linguagem visual        Glassmorphism           Identidade visual

  Embeddings              Transformers.js         Geração local de
                          (Hugging Face)          vetores

  LLM conversacional      Groq API                Chat, interpretação e
                                                  tool calling

  Banco de dados          PostgreSQL              Persistência relacional

  Busca vetorial          pgvector                Armazenamento e busca
                                                  de embeddings

  Validação               Zod                     Validação de entradas e
                                                  saídas

  ORM                     A definir antes da      Acesso ao PostgreSQL
                          implementação de        
                          persistência            
  -----------------------------------------------------------------------

### Regras de integração de IA

-   Groq é o provedor do modelo conversacional. Não pressupor que Groq
    gere embeddings.
-   Transformers.js gera embeddings localmente no ambiente servidor da
    aplicação.
-   PostgreSQL com pgvector armazena e recupera vetores.
-   O LLM recebe texto e dados recuperados, não vetores brutos.
-   A implementação deve encapsular os provedores em interfaces/serviços
    para permitir substituição futura.
-   Não incluir chaves de API no código-fonte. Usar variáveis de
    ambiente.
-   Não enviar arquivos OFX completos à Groq por padrão. Enviar somente
    o contexto mínimo necessário para responder.
-   Verificar a disponibilidade, licença, dimensão vetorial, desempenho
    e compatibilidade do modelo de embeddings escolhido antes de
    fixá-lo.
-   Executar Transformers.js no servidor Node.js, não no navegador, como
    decisão padrão do MVP.
-   Não presumir que a inferência local será instantânea. Medir tempo de
    carregamento e geração; evitar bloquear a resposta HTTP por longos
    períodos.

## 4.1 Modelo de embeddings

O modelo específico ainda precisa ser selecionado e validado. A seleção
deve considerar:

-   Qualidade em português.
-   Adequação a descrições curtas de transações e consultas de usuários.
-   Dimensão do vetor.
-   Licença e possibilidade de uso pretendida.
-   Compatibilidade com a versão adotada do Transformers.js.
-   Consumo de memória e tempo de inferência no ambiente de execução.

A dimensão configurada no tipo vetorial do pgvector deve corresponder
exatamente à dimensão retornada pelo modelo. Não misturar embeddings de
modelos ou dimensões diferentes no mesmo índice lógico.

## 4.2 Modelo conversacional

A integração com Groq deve:

-   Usar um modelo disponível na conta e compatível com tool calling,
    quando essa capacidade for necessária.
-   Configurar o identificador do modelo por variável de ambiente.
-   Tratar limites de requisição, erros, timeout e indisponibilidade do
    provedor.
-   Validar argumentos de ferramentas no servidor com Zod.
-   Nunca permitir que o modelo execute SQL arbitrário.
-   Tratar toda saída do modelo como conteúdo não confiável até ser
    validada.

## 5. Arquitetura e organização

Estrutura inicial recomendada:

``` text
finscope/
├── AGENTS.md
├── TASKS.md
├── README.md
├── prisma/                    # ou diretório equivalente, conforme ORM escolhido
├── public/
└── src/
    ├── app/
    │   ├── (app)/
    │   │   ├── dashboard/
    │   │   ├── statements/
    │   │   └── chat/
    │   ├── api/
    │   │   ├── statements/
    │   │   ├── transactions/
    │   │   └── chat/
    │   ├── layout.tsx
    │   └── globals.css
    ├── components/
    │   ├── layout/
    │   ├── dashboard/
    │   ├── statements/
    │   ├── transactions/
    │   ├── chat/
    │   └── ui/
    ├── features/
    │   ├── statements/
    │   ├── transactions/
    │   └── conversations/
    ├── services/
    │   ├── ofx/
    │   ├── embeddings/
    │   ├── llm/
    │   └── rag/
    ├── lib/
    │   ├── db/
    │   ├── env/
    │   └── utils/
    ├── schemas/
    └── types/
```

A estrutura pode ser ajustada quando necessário, mas as
responsabilidades devem permanecer separadas:

-   `app/api`: camada HTTP; valida entrada, chama serviços e traduz
    resultados em respostas HTTP.
-   `services/ofx`: parsing, normalização e validação de arquivos OFX.
-   `services/embeddings`: interface e implementação Transformers.js.
-   `services/llm`: cliente Groq e contratos de tool calling.
-   `services/rag`: recuperação de contexto e montagem do contexto para
    o LLM.
-   `features`: regras e casos de uso por domínio.
-   `lib/db`: conexão e utilitários de banco.
-   `schemas`: schemas Zod reutilizáveis.
-   `components`: componentes de apresentação.

Não concentrar parsing, regras financeiras, acesso ao banco e chamadas
de IA dentro de componentes React ou Route Handlers.

## 6. Modelo de domínio e dados

O modelo inicial deve contemplar as entidades abaixo. Os nomes físicos e
detalhes finais podem ser ajustados durante a modelagem, sem alterar as
regras de domínio.

### User

O MVP pode iniciar em modo single-user, mas os registros devem possuir
uma estratégia consistente de propriedade/escopo para não exigir uma
reescrita estrutural caso autenticação multiusuário seja adicionada.

Campos conceituais: - `id` - `email` (opcional enquanto não houver
autenticação) - `created_at` - `updated_at`

### Statement

Representa um arquivo/extrato importado.

Campos conceituais: - `id` - `user_id` ou identificador de
proprietário/escopo - `filename` - `bank_name` (nullable) -
`account_number` (nullable; evitar armazenar informação desnecessária) -
`start_date` - `end_date` - `initial_balance` (nullable) -
`final_balance` (nullable) - `currency` - `import_status` - `created_at`

### Transaction

Representa uma movimentação extraída do OFX.

Campos conceituais: - `id` - `statement_id` - `fit_id` (identificador
fornecido pelo OFX, quando presente) - `transaction_date` -
`posted_date` (nullable) - `description` - `memo` (nullable) -
`amount` - `transaction_type` ou sinal padronizado - `category_id`
(nullable) - `created_at`

### Category

Representa uma categoria atribuída à movimentação.

Campos conceituais: - `id` - `name` - `kind` (income, expense ou other,
conforme necessidade) - `source` (manual, rule ou ai) - `created_at`

### Conversation e Message

Representam a conversa vinculada a um extrato.

Conversation: - `id` - `statement_id` - `created_at` - `updated_at`

Message: - `id` - `conversation_id` - `role` (user, assistant ou tool,
conforme implementação) - `content` - `created_at`

### Document/Chunk para RAG

Representa conteúdo textual indexável, derivado de forma controlada dos
dados do extrato.

Campos conceituais: - `id` - `statement_id` - `content` - `metadata`
(JSON validado) - `embedding` - `embedding_model` - `created_at`

Não é obrigatório criar uma tabela separada chamada `documents` se a
solução de chunks tiver outro desenho. É obrigatório, porém, manter
rastreabilidade entre cada conteúdo indexado e o extrato de origem.

### Regras de persistência

-   Valores monetários devem usar tipo decimal/numeric no banco. Nunca
    usar ponto flutuante para cálculos financeiros.
-   Datas devem ser persistidas em tipo apropriado e tratadas
    consistentemente, considerando a data bancária do OFX.
-   Preservar a descrição original da transação; qualquer descrição
    enriquecida deve ser armazenada separadamente.
-   `FITID`, quando presente, deve ser utilizado como parte da
    estratégia de deduplicação, considerando o extrato/conta de origem.
-   Não presumir que `FITID` sempre existe ou que seja globalmente
    único.
-   Importações repetidas não devem criar duplicatas silenciosamente.
-   Uma falha no parsing ou na persistência não pode deixar um extrato
    marcado como importado com sucesso.
-   Manter o status da importação explícito, por exemplo: `pending`,
    `processing`, `completed`, `failed`.
-   A dimensão do campo vetorial deve corresponder ao modelo de
    embedding escolhido.

## 7. Requisitos funcionais

### RF-01 --- Importar extrato OFX

O sistema deve permitir selecionar e enviar um arquivo `.ofx`.

Critérios: - Validar extensão e conteúdo real do arquivo; não confiar
somente no nome/extensão. - Definir e validar limite de tamanho
configurável. - Rejeitar arquivos inválidos ou sem transações
interpretáveis. - Informar erros de importação de maneira
compreensível. - Não expor o conteúdo bruto do arquivo em logs.

### RF-02 --- Processar e normalizar extrato

O sistema deve interpretar os dados OFX e normalizar as movimentações.

Critérios: - Extrair metadados disponíveis da conta e do extrato. -
Extrair transações, datas, valores, identificadores e descrições. -
Preservar valores e descrições originais. - Tratar sinais de
débito/crédito de maneira consistente. - Detectar duplicidades. -
Persistir o resultado de forma consistente.

### RF-03 --- Listar extratos

O usuário deve visualizar os extratos importados, com período,
instituição quando disponível, quantidade de transações e status.

### RF-04 --- Visualizar dashboard por extrato

O usuário deve selecionar um extrato e visualizar: - Saldo inicial e
final, quando disponíveis. - Total de entradas. - Total de saídas. -
Quantidade de transações. - Evolução do saldo no período, quando os
dados permitirem. - Distribuição de despesas por categoria, quando
houver categorização. - Lista de transações.

Os valores exibidos devem derivar de consultas determinísticas ao banco,
não de estimativas do LLM.

### RF-05 --- Consultar e filtrar transações

Permitir filtrar movimentações por texto, período, tipo e categoria,
respeitando o extrato selecionado.

### RF-06 --- Conversar sobre um extrato

O usuário deve selecionar um extrato e enviar perguntas relacionadas a
ele.

Critérios: - Toda conversa deve estar vinculada a um `statement_id`. - O
servidor deve validar que o extrato existe e pertence ao escopo
autorizado. - O contexto enviado ao modelo deve ser limitado ao extrato
selecionado. - A aplicação deve persistir mensagens e respostas. - Erros
do provedor devem ser tratados sem perder a mensagem do usuário. - A
interface deve indicar carregamento e falhas.

### RF-07 --- Responder com consultas estruturadas

Para perguntas quantitativas, o sistema deve usar ferramentas com
consultas SQL parametrizadas, por exemplo: - Resumo do extrato. -
Maiores despesas. - Despesas por categoria. - Pesquisa de transações. -
Comparação de períodos, somente quando houver dados suficientes.

O modelo não pode gerar ou executar SQL arbitrário. Cada ferramenta deve
ter parâmetros validados com Zod, escopo obrigatório de extrato e
resultados limitados.

### RF-08 --- Recuperar contexto semântico

Para perguntas que dependam de significado textual, o sistema deve: -
Gerar embedding da consulta com o mesmo modelo usado na indexação. -
Executar busca vetorial filtrada pelo extrato selecionado. - Recuperar
conteúdo e metadados de origem. - Aplicar limite de resultados e
limiar/estratégia de relevância configuráveis. - Fornecer ao LLM o
conteúdo textual recuperado, não os vetores brutos.

### RF-09 --- Responder com transparência

As respostas devem: - Ser baseadas somente nos dados recuperados e
resultados das ferramentas. - Distinguir fatos encontrados de
inferências. - Informar quando não houver dados suficientes. - Não
inventar transações, valores, categorias ou períodos. - Quando
aplicável, apresentar período consultado e movimentações que fundamentam
a resposta.

## 8. Pipeline de ingestão e RAG

### 8.1 Ingestão OFX

Fluxo esperado:

1.  Receber arquivo.
2.  Validar tamanho, extensão e conteúdo.
3.  Criar registro de importação com status `processing`.
4.  Fazer parsing do OFX.
5.  Normalizar os campos.
6.  Validar consistência dos dados.
7.  Persistir extrato e transações em transação de banco.
8.  Preparar conteúdos textuais indexáveis.
9.  Gerar embeddings com Transformers.js.
10. Persistir chunks, metadados, modelo e vetores.
11. Marcar importação como `completed`.

Se a geração de embeddings falhar, o sistema deve preservar a distinção
entre importação financeira concluída e indexação pendente/falha. O
dashboard e as consultas SQL devem continuar utilizáveis. A política
exata de status da indexação deve ser documentada na implementação.

### 8.2 Conteúdo indexado

Não indexar indiscriminadamente cada coluna de cada transação. Construir
textos úteis à recuperação semântica, preservando referência às
transações/extrato de origem.

Exemplo de conteúdo:

``` text
Movimentação em 2026-10-12.
Descrição original: MERCADO CENTRAL.
Valor: -250,00 BRL.
Tipo: despesa.
Categoria: alimentação.
```

Dados numéricos e temporais devem continuar disponíveis em colunas
estruturadas para consultas SQL. A busca vetorial é complementar, não
substituta do banco relacional.

### 8.3 Consulta RAG

Fluxo esperado:

1.  Receber pergunta, conversa e `statement_id`.
2.  Validar autorização e escopo.
3.  Determinar se a pergunta exige cálculo estruturado, recuperação
    semântica ou ambos.
4.  Para busca semântica, gerar embedding da pergunta.
5.  Recuperar chunks filtrados pelo `statement_id`.
6.  Para cálculos, executar ferramentas SQL parametrizadas.
7.  Montar contexto com fontes/IDs e resultados.
8.  Enviar pergunta e contexto textual à Groq.
9.  Validar e persistir a resposta.
10. Retornar resposta e referências disponíveis.

## 9. Segurança, privacidade e confiabilidade

-   Tratar OFX como dado financeiro sensível.
-   Não registrar chaves, conteúdo integral de OFX ou dados financeiros
    completos em logs.
-   Manter segredos exclusivamente em variáveis de ambiente do servidor.
-   Validar todas as entradas com Zod.
-   Validar propriedade/escopo do extrato em toda rota e ferramenta.
-   Não confiar em `statement_id` enviado pelo cliente sem verificar
    autorização.
-   Não permitir acesso ao banco por SQL arbitrário gerado pelo LLM.
-   Limitar tamanho de arquivo, mensagens, contexto e quantidade de
    resultados.
-   Tratar conteúdo de OFX, descrições e mensagens como dados não
    confiáveis, nunca como instruções para o modelo.
-   Mitigar prompt injection: texto recuperado não pode substituir as
    instruções de sistema nem ampliar permissões.
-   Não executar ações financeiras; o assistente é somente consultivo.
-   Não enviar mais dados à Groq do que o necessário para responder.
-   Em implantação pública, implementar autenticação e isolamento por
    usuário antes de disponibilizar dados reais.

## 10. Interface e experiência visual

### Direção visual

O frontend deve adotar glassmorphism com tema escuro como padrão.

Paleta inicial de referência: - Background: `#0B1020` - Surface
translúcida: `rgba(255,255,255,0.06)` - Primary: `#8B5CF6` - Secondary:
`#38BDF8` - Success: `#34D399` - Text: `#F8FAFC` - Muted: `#94A3B8`

Características: - Superfícies translúcidas. - `backdrop-filter: blur()`
aplicado com moderação. - Bordas claras de baixa opacidade. - Gradientes
sutis no background. - Sombras discretas e hierarquia visual clara. -
Animações curtas e funcionais. - Contraste e legibilidade prioritários.

Não sacrificar acessibilidade, desempenho ou legibilidade para obter o
efeito visual. Fornecer fallback quando `backdrop-filter` não estiver
disponível. Evitar excesso de blur em tabelas e textos longos.

### Telas do MVP

1.  **Extratos:** upload, estado de processamento e listagem.
2.  **Dashboard:** resumo, gráficos e transações do extrato selecionado.
3.  **Chat:** seletor de extrato, histórico de mensagens, entrada de
    pergunta e respostas com referências.

A navegação deve deixar claro qual extrato está selecionado,
especialmente no chat.

## 11. Qualidade e testes

Toda funcionalidade deve incluir testes proporcionais ao risco.

### Testes unitários

-   Parser e normalizador OFX.
-   Conversão de sinais e valores.
-   Deduplicação.
-   Cálculos de resumo financeiro.
-   Construção dos chunks.
-   Validação de parâmetros das ferramentas.
-   Montagem de contexto RAG.

### Testes de integração

-   Importação completa de OFX.
-   Persistência de extrato e transações.
-   Busca vetorial filtrada por extrato.
-   Execução de ferramentas SQL.
-   Integração com provedores usando mocks quando apropriado.

### Testes de interface/aceitação

-   Upload válido e inválido.
-   Listagem e seleção de extrato.
-   Dashboard coerente com dados persistidos.
-   Chat vinculado ao extrato correto.
-   Estados de carregamento, vazio e erro.

### Avaliação de IA

Manter um conjunto pequeno de perguntas e respostas esperadas para
avaliar: - Relevância dos chunks recuperados. - Precisão dos cálculos. -
Fidelidade da resposta ao contexto. - Respostas quando não há dados
suficientes. - Isolamento entre extratos.

Não considerar uma resposta correta apenas porque é fluente.

## 12. Variáveis de ambiente

Manter um `.env.example` sem segredos reais. Os nomes definitivos podem
ser ajustados conforme as bibliotecas escolhidas.

Variáveis esperadas: - `DATABASE_URL` - `GROQ_API_KEY` - `GROQ_MODEL` -
`EMBEDDING_MODEL` - `MAX_UPLOAD_SIZE` - `APP_URL` (se necessário)

Validar as variáveis necessárias no início da aplicação ou no momento de
uso. Nunca expor variáveis secretas com prefixo `NEXT_PUBLIC_`.

## 13. Padrões de implementação

-   TypeScript estrito; evitar `any`.
-   Preferir funções pequenas com responsabilidades explícitas.
-   Separar camada HTTP, casos de uso, integrações externas e
    persistência.
-   Validar entradas na fronteira do sistema.
-   Usar consultas parametrizadas.
-   Usar transações de banco quando uma operação exigir consistência
    entre múltiplas gravações.
-   Tratar erros com mensagens seguras e úteis.
-   Não duplicar regras de negócio no frontend.
-   Não adicionar dependências sem necessidade demonstrável.
-   Não implementar funcionalidades não solicitadas.
-   Atualizar documentação e testes junto com mudanças de comportamento.

## 14. Critérios de conclusão de tarefas

Uma tarefa só pode ser marcada como `completed` quando: - O
comportamento solicitado foi implementado. - Os critérios de aceitação
foram atendidos. - Testes relevantes foram criados ou atualizados e
executados. - Não há erros de TypeScript, lint ou build relacionados à
alteração. - Documentação afetada foi atualizada. - Não foram
introduzidas alterações fora do escopo.

Se houver bloqueio externo (por exemplo, ausência de credencial Groq),
implementar e testar a lógica isoladamente com mocks quando possível,
registrar o bloqueio e não declarar a integração real como concluída.
