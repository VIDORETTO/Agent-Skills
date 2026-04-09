# RAG em 2025/2026: Guia Definitivo e Completo

> **Data de referência:** 08/04/2026
> **Prioridade:** Precisão > Eficácia > Custo > Velocidade
> **Princípio:** Não existe um RAG único — existe o RAG certo para cada situação

---

## Índice

1. [Fundamentos e Filosofia](#fundamentos)
2. [A Arquitetura Vencedora](#arquitetura)
3. [Mapa de Decisão Rápido](#mapa)
4. [RAG por Tipo de Conteúdo](#por-conteudo)
   - A. Texto Pequeno / FAQ
   - B. Markdown / Docs Técnicas
   - C. Livro / Documento Longo
   - D. PDF Limpo
   - E. PDF Visual Complexo
   - F. CSV
   - G. XLS / XLSX
   - H. Catálogo de Produtos / SKUs
5. [RAG por Tipo de Arquivo](#por-arquivo)
6. [Técnicas Avançadas de Precisão](#tecnicas)
7. [Componentes: Embeddings, Vector DBs, Rerankers](#componentes)
8. [Stacks por Perfil e Orçamento](#stacks)
9. [Avaliação e Monitoramento](#avaliacao)
10. [Resumo Executivo Final](#resumo)

---

## 1. Fundamentos e Filosofia {#fundamentos}

### O Erro Mais Comum

```
❌ MENTALIDADE ERRADA:
"Vou pegar meus documentos, jogar num vector DB
 e conectar ao ChatGPT"

✅ MENTALIDADE CERTA:
"Qual é o tipo de dado? Qual é o tipo de pergunta?
 Qual é o custo aceitável? Só então escolho o pipeline."
```

### A Hierarquia de Prioridades Real

A maioria das pessoas tenta resolver tudo no **prompt**. O que a pesquisa de 2025/2026 mostra é que a ordem de impacto real é:

```
┌─────────────────────────────────────────────────────────────────┐
│              ORDEM DE IMPACTO NA PRECISÃO                       │
│                                                                 │
│  1º  Qualidade do parsing/ingestão    ████████████████████ 95% │
│  2º  Modelagem certa por tipo         ████████████████████ 90% │
│  3º  Hybrid Retrieval                 ██████████████████   85% │
│  4º  Reranking                        █████████████████    80% │
│  5º  Chunking / Contextual Retrieval  █████████████        65% │
│  6º  Prompt Engineering               ████████             40% │
│                                                                 │
│  Muita gente investe no 6º e ignora o 1º.                      │
└─────────────────────────────────────────────────────────────────┘
```

### Quando NÃO Usar RAG

A Anthropic afirma explicitamente: se sua base cabe em **~200 mil tokens** (~500 páginas) e é consultada com frequência, **mandar tudo no prompt com cache pode ser mais simples, mais barato e mais preciso** do que montar um pipeline de retrieval.

```python
def should_use_rag(doc_tokens: int, query_frequency: str, budget: str) -> str:
    
    if doc_tokens < 100_000 and budget == "baixo":
        return "FULL CONTEXT - sem RAG, use prompt caching"
    
    if doc_tokens < 200_000 and query_frequency == "alta":
        return "AVALIAR - prompt caching pode vencer RAG em custo/precisão"
    
    if doc_tokens > 200_000:
        return "RAG OBRIGATÓRIO"
    
    if "planilha" in tipo and query_type == "analítica":
        return "SQL/PANDAS - não RAG textual"
    
    return "RAG com pipeline adequado ao tipo"
```

---

## 2. A Arquitetura Vencedora {#arquitetura}

### O Padrão que Domina em 2026

O benchmark **T2-RAGBench** (arXiv 2026) e a documentação de OpenAI, Azure, Weaviate e Anthropic convergem para o mesmo padrão:

```
parser bom
  + busca híbrida (BM25/keyword + dense/vector)
    + reranker
      + expansão de contexto
        + resposta com citação/trecho-fonte
```

Esse é o **default vencedor para a maioria dos casos**. Híbrido + cross-encoder rerank ficou claramente acima de BM25 sozinho, dense sozinho e híbrido sem rerank no benchmark.

### Pipeline Completo Visualizado

```
┌─────────────────────────────────────────────────────────────────────┐
│                      PIPELINE RAG COMPLETO 2026                     │
│                                                                     │
│  ┌─────────────┐                                                    │
│  │  DOCUMENTO  │                                                    │
│  └──────┬──────┘                                                    │
│         │                                                           │
│         ▼                                                           │
│  ┌─────────────┐    ┌──────────────┐    ┌───────────────────────┐  │
│  │   PARSER    │───▶│  ESTRUTURAR  │───▶│  ENRIQUECER METADATA  │  │
│  │  (Docling,  │    │  (headings,  │    │  (path, seção, página,│  │
│  │  PyMuPDF,   │    │  tabelas,    │    │   tipo, fonte, data)  │  │
│  │  pandas...) │    │  listas)     │    └───────────┬───────────┘  │
│  └─────────────┘    └──────────────┘                │              │
│                                                      │              │
│                                                      ▼              │
│                                          ┌───────────────────────┐ │
│                                          │       CHUNKING        │ │
│                                          │  (por tipo de doc:    │ │
│                                          │   heading, semântico, │ │
│                                          │   parent-child,       │ │
│                                          │   late chunking...)   │ │
│                                          └───────────┬───────────┘ │
│                                                      │              │
│                                                      ▼              │
│                                     ┌────────────────────────────┐ │
│                                     │   CONTEXTUAL ENRICHMENT    │ │
│                                     │  (adiciona contexto ao     │ │
│                                     │   chunk antes de embedar)  │ │
│                                     └──────────────┬─────────────┘ │
│                                                     │               │
│                               ┌─────────────────────┘               │
│                               │                                     │
│                    ┌──────────┴──────────┐                         │
│                    │                     │                         │
│                    ▼                     ▼                         │
│           ┌──────────────┐    ┌──────────────────┐                │
│           │ DENSE INDEX  │    │  SPARSE INDEX    │                │
│           │ (embeddings/ │    │  (BM25/keyword)  │                │
│           │  vector DB)  │    │                  │                │
│           └──────────────┘    └──────────────────┘                │
│                    │                     │                         │
│                    └─────────┬───────────┘                         │
│                              │                                     │
│  ┌────────────┐              │                                     │
│  │   QUERY    │──────────────┘                                     │
│  └──────┬─────┘         │                                         │
│         │               ▼                                         │
│         │      ┌──────────────────┐                               │
│         │      │  HYBRID SEARCH   │                               │
│         │      │  (RRF fusion)    │                               │
│         │      │  top-20 a 50     │                               │
│         │      └────────┬─────────┘                               │
│         │               │                                         │
│         │               ▼                                         │
│         │      ┌──────────────────┐                               │
│         │      │    RERANKER      │                               │
│         │      │  (cross-encoder) │                               │
│         │      │  top-5 a 10      │                               │
│         │      └────────┬─────────┘                               │
│         │               │                                         │
│         │               ▼                                         │
│         │      ┌──────────────────┐                               │
│         │      │ CONTEXT EXPANSION│                               │
│         │      │ (chunks vizinhos │                               │
│         │      │  ou parent doc)  │                               │
│         │      └────────┬─────────┘                               │
│         │               │                                         │
│         └───────────────┘                                         │
│                          │                                         │
│                          ▼                                         │
│                 ┌──────────────────┐                               │
│                 │       LLM        │                               │
│                 │  (com contexto + │                               │
│                 │   citação fonte) │                               │
│                 └──────────────────┘                               │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3. Mapa de Decisão Rápido {#mapa}

```
                         QUAL PIPELINE USAR?
                                │
              ┌─────────────────┼─────────────────┐
              │                 │                 │
         TEXTO/DOC          TABULAR            MISTO
              │                 │           (combinar)
              │                 │
         Tamanho?          Tipo de Query?
              │                 │
       ┌──────┴──────┐    ┌─────┴──────┐
    Pequeno       Grande  Semântica  Analítica
    (<200k tok)     │       │            │
       │            │    Vector DB    SQL/Pandas
       │         Estrutura?           Tool Use
  Full Context      │
  (sem RAG)    ┌────┴────────────┐
               │                │
          Estruturado        Não struct.
          (capítulos,        (texto corrido)
          seções)                │
               │            Late Chunking
          Parent-Child      + Hybrid
          + RAPTOR           + Rerank
          (se multi-hop)

SEMPRE QUE USAR RAG:
✅ Hybrid Retrieval (BM25 + Dense)
✅ Reranker
✅ Metadata rica
✅ Contextual enrichment nos chunks
✅ Citação da fonte na resposta
✅ Abstention quando não há evidência
```

---

## 4. RAG por Tipo de Conteúdo {#por-conteudo}

---

### 🗒️ A. Texto Pequeno / FAQ / Página de Ajuda

**Exemplos:** FAQ curto, política de empresa, notas de versão, página de ajuda simples

#### Filosofia

```
┌──────────────────────────────────────────────────────────────────┐
│ REGRA DE OURO: Não complique o que é simples.                    │
│                                                                  │
│ O maior risco aqui NÃO é "falta de contexto".                   │
│ É criar um pipeline de retrieval desnecessário para algo         │
│ que cabia direto no contexto do LLM.                             │
└──────────────────────────────────────────────────────────────────┘
```

#### Estratégia de Decisão

```python
def strategy_for_small_text(doc_tokens: int, n_documents: int) -> str:
    
    # Caso 1: Cabe no contexto → manda tudo
    if doc_tokens < 100_000:
        return """
        FULL CONTEXT RAG
        - Sem chunking
        - Sem vector DB
        - Prompt caching para reduzir custo em queries repetidas
        - Gemini 1.5 Flash (1M ctx), Claude 3.5 (200k), GPT-4o (128k)
        """
    
    # Caso 2: Muitos documentos pequenos
    if n_documents > 100 and doc_tokens < 500:
        return """
        RAG MÍNIMO
        - 1 chunk por documento (não quebre)
        - Hybrid BM25 + dense
        - Rerank opcional (vale quando há docs muito parecidos)
        """
    
    # Caso 3: Texto médio que não cabe no contexto
    return """
    RAG PADRÃO LEVE
    - Chunking: 128-256 tokens
    - Overlap: 0-40 tokens
    - Hybrid + rerank
    """
```

#### Configuração Técnica

```yaml
chunking:
  tamanho: 128–256 tokens
  overlap: 0–40 tokens
  metodo: por_paragrafo_ou_documento_inteiro

metadata:
  - source_file
  - section
  - last_updated
  - language

retrieval:
  tipo: hybrid (BM25 + dense)
  top_k_candidatos: 10–20
  rerank: opcional

embeddings:
  modelo: text-embedding-3-small ou nomic-embed-text
  justificativa: overkill usar large aqui

llm: GPT-4o-mini ou Gemini Flash (custo-benefício)
```

#### Stack Recomendado

```
Parser:     PyMuPDF (PDF) / markdown-it (MD) / direto (TXT)
Embeddings: nomic-embed-text (grátis local) ou text-embedding-3-small
Vector DB:  ChromaDB (local) ou pgvector (se já tem Postgres)
Retrieval:  Hybrid BM25 + Dense
Reranker:   opcional - ms-marco-MiniLM (local, leve)
LLM:        GPT-4o-mini ou Gemini 1.5 Flash
```

**Custo estimado:** ~$0.001–0.003 por query

---

### 📘 B. Markdown / Docs Técnicas / Manuais / Base de Conhecimento

**Exemplos:** Documentação de produto, tutoriais, manuais internos, wikis, READMEs

#### Filosofia

```
┌──────────────────────────────────────────────────────────────────┐
│ Markdown tem ESTRUTURA. Respeite ela.                            │
│                                                                  │
│ Chunking por caractere em Markdown = crime técnico.              │
│ A hierarquia de headers É o mapa semântico do documento.         │
│                                                                  │
│ AWS destaca: headings/subheadings ajudam o RAG a entender        │
│ estrutura. LlamaIndex tem MarkdownNodeParser baseado em headers. │
└──────────────────────────────────────────────────────────────────┘
```

#### Estratégia de Chunking

```
DOCUMENTO MARKDOWN:
# Guia de Instalação                     ← H1: chunk pai
## Requisitos do Sistema                 ← H2: chunk filho
### Windows                              ← H3: sub-chunk
...conteúdo...
### Linux                                ← H3: sub-chunk
...conteúdo...
## Passo a Passo                         ← H2: chunk filho
...conteúdo...

RESULTADO:
Chunk 1: "Guia de Instalação > Requisitos do Sistema > Windows"
  metadata: {h1: "Guia de Instalação", h2: "Requisitos", h3: "Windows"}

Chunk 2: "Guia de Instalação > Requisitos do Sistema > Linux"
  metadata: {h1: "Guia de Instalação", h2: "Requisitos", h3: "Linux"}
```

#### Implementação Prática

```python
from langchain.text_splitter import MarkdownHeaderTextSplitter

headers_to_split_on = [
    ("#",    "h1"),
    ("##",   "h2"),
    ("###",  "h3"),
    ("####", "h4"),
]

splitter = MarkdownHeaderTextSplitter(
    headers_to_split_on=headers_to_split_on,
    strip_headers=False  # ← IMPORTANTE: mantém o header no chunk
)

chunks = splitter.split_text(markdown_content)

# Cada chunk tem metadados automáticos de estrutura:
# chunk.metadata = {"h1": "Cap 1", "h2": "Seção 2", "h3": "Tópico A"}

# Prefixo de contexto (Contextual Retrieval aplicado):
for chunk in chunks:
    path = " > ".join([
        chunk.metadata.get("h1", ""),
        chunk.metadata.get("h2", ""),
        chunk.metadata.get("h3", ""),
    ]).strip(" > ")
    
    chunk.page_content = f"[{path}]\n\n{chunk.page_content}"
```

#### Regras de Ouro para Markdown

```python
REGRAS_MARKDOWN = {
    
    "preservar_intacto": [
        "blocos de código (```...```)",
        "tabelas markdown completas",
        "listas numeradas (não quebrar no meio)",
        "admonitions/callouts",
        "blocos YAML/JSON/frontmatter"
    ],
    
    "nunca_fazer": [
        "quebrar código no meio de uma função",
        "separar uma lista numerada em dois chunks",
        "perder o path de headers no chunk",
        "ignorar a hierarquia de seções"
    ],
    
    "sempre_fazer": [
        "prefixar chunk com caminho completo dos headers",
        "manter nome do arquivo + versão como metadata",
        "indexar chunks vizinhos para expansão de contexto"
    ]
}
```

#### Configuração Técnica

```yaml
chunking:
  metodo: por_heading_hierarquico (H1 > H2 > H3)
  tamanho_alvo: 300–700 tokens
  overflow_strategy: split_semantico (se seção muito grande)
  overlap: 50–120 tokens
  preservar: [code_blocks, tables, numbered_lists]

metadata:
  - file_name
  - file_path
  - section_path  # "Manual > Estoque > Ajuste de custo > Passo 3"
  - h1, h2, h3, h4
  - version
  - language
  - last_modified

retrieval:
  tipo: hybrid (BM25 + dense)
  top_k_candidatos: 20–50
  
rerank:
  ativo: sim
  top_k_final: 5–10
  modelo: cross-encoder/ms-marco-MiniLM-L-6-v2

expansao_contexto:
  tipo: chunk_vizinho ou parent_section
  janela: 1-2 chunks antes e depois
```

**Custo estimado:** ~$0.002–0.008 por query
**Observação:** Este é o cenário de **melhor custo-benefício** de todos. Markdown bem estruturado + pipeline correto = resultados excelentes com custo baixo.

---

### 📚 C. Livro / Documento Longo / Relatório / Contrato / Apostila

**Exemplos:** Livro inteiro, relatório anual, regulamento completo, documentação extensa, apostila de curso

#### Filosofia

```
┌──────────────────────────────────────────────────────────────────┐
│ PRINCÍPIO FUNDAMENTAL:                                           │
│ "Recupere pequeno. Leia grande."                                 │
│                                                                  │
│ Busque com chunks granulares (precisão no retrieval)             │
│ Entregue ao LLM o contexto expandido (qualidade na resposta)     │
└──────────────────────────────────────────────────────────────────┘
```

#### Arquitetura Parent-Child

```
LIVRO
├── Capítulo 1 (PARENT - 1500-3000 tokens)
│   ├── Parágrafo 1.1 (CHILD - 300-500 tokens) ← busca aqui
│   ├── Parágrafo 1.2 (CHILD - 300-500 tokens) ← busca aqui
│   └── Parágrafo 1.3 (CHILD - 300-500 tokens) ← busca aqui
├── Capítulo 2 (PARENT - 1500-3000 tokens)
│   ├── Parágrafo 2.1 (CHILD - 300-500 tokens) ← busca aqui
│   └── ...

QUERY → busca nos CHILDs (granular, preciso)
      → encontra os mais relevantes
      → recupera os PARENTs correspondentes (contexto completo)
      → envia PARENTs ao LLM (sem perder contexto)
```

#### Implementação Parent-Child

```python
from langchain.retrievers import ParentDocumentRetriever
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.storage import InMemoryStore

# Splitter para chunks PAI (contexto)
parent_splitter = RecursiveCharacterTextSplitter(
    chunk_size=2000,    # tokens
    chunk_overlap=150
)

# Splitter para chunks FILHO (busca)
child_splitter = RecursiveCharacterTextSplitter(
    chunk_size=400,     # tokens - granular para busca precisa
    chunk_overlap=50
)

# Store para documentos pai
docstore = InMemoryStore()  # ou Redis para produção

retriever = ParentDocumentRetriever(
    vectorstore=vector_db,
    docstore=docstore,
    child_splitter=child_splitter,
    parent_splitter=parent_splitter,
)

# Indexa (cria ambos os níveis automaticamente)
retriever.add_documents(documents)

# Na query: busca em filhos, retorna pais
relevant_docs = retriever.get_relevant_documents(query)
```

#### RAPTOR: Para Livros com Temas Complexos e Multi-hop

```
QUANDO USAR RAPTOR:
✅ Perguntas que precisam sintetizar múltiplas seções distantes
✅ Livros com temas complexos e interconectados
✅ "Quais as principais diferenças entre o capítulo 3 e 7?"
✅ Relatórios onde a conclusão depende de dados espalhados

COMO FUNCIONA:
Texto → Chunks
         ↓
    Clusterização (UMAP + GMM)
         ↓
    Resumo de cada cluster (LLM)
         ↓
    Clusterização dos resumos
         ↓
    Resumo dos resumos
         ↓
    Árvore hierárquica de abstração

RESULTADO:
- Query abstrata/geral → bate nos resumos do topo
- Query específica → bate nos chunks originais da base
- Query intermediária → bate no nível certo da árvore

CUSTO: Alto na indexação (muitas chamadas LLM)
PRECISÃO: ★★★★★ para documentos complexos
```

#### Qual Técnica Escolher?

```
DOCUMENTO LONGO - DECISÃO:

Perguntas simples e diretas?
└── Parent-Child + Hybrid + Rerank
    (mais barato, muito eficaz)

Perguntas complexas/multi-hop?
└── RAPTOR ou Hierárquico
    (mais caro na indexação, máxima precisão)

Quer equilíbrio custo/precisão?
└── Late Chunking + Hybrid + Rerank + Parent Expansion
    (melhor meio-termo segundo pesquisa 2025)

Precisão máxima acima de tudo?
└── Contextual Retrieval + Hybrid + Rerank
    (mais caro no ingest, -67% de erros de retrieval)
```

#### Para Legislação / Documentos Jurídicos

```python
METADATA_JURIDICO = {
    # Identificação
    "artigo": "Art. 5º",
    "inciso": "II",
    "paragrafo": "§ 1º",
    "capitulo": "Dos Direitos Fundamentais",
    "titulo": "Título II",
    
    # Referência
    "lei": "Constituição Federal",
    "numero_lei": "CF/1988",
    "ano": 1988,
    "data_vigencia": "1988-10-05",
    
    # Relacionamentos
    "referencias_cruzadas": ["Art. 6º", "Art. 7º", "Art. 196"],
    "revogado_por": None,
    "alterado_por": ["EC 19/1998", "EC 45/2004"],
    
    # Semântico
    "palavras_chave": ["direitos", "fundamentais", "cidadão"],
    "tema": "direitos_fundamentais"
}

# Estratégia: filtro por metadata + busca semântica
# Isso evita buscar em artigos completamente irrelevantes
results = vector_db.search(
    query=query,
    filter={"lei": "Constituição Federal", "tema": "direitos_fundamentais"},
    top_k=20
)
```

#### Configuração Técnica

```yaml
indices:
  folha_child:
    chunk_size: 300–600 tokens
    overlap: 60–120 tokens
    uso: retrieval/busca
    
  pai_parent:
    chunk_size: 1500–3000 tokens
    overlap: 150 tokens
    uso: contexto enviado ao LLM

metadata:
  - titulo, capitulo, secao, subsecao
  - numero_pagina
  - posicao_relativa  # % no documento
  - doc_title, doc_type, doc_date
  - referencias_cruzadas

retrieval:
  etapa_1: busca no índice de capítulo/seção
  etapa_2: busca no índice de chunk interno
  top_k_candidatos: 20–50 filhos
  
rerank:
  ativo: sim (obrigatório para docs longos)
  top_k_final: 5–8 pais

expansao:
  tipo: devolver chunk + vizinhos + seção pai
  dedup_pais: sim
```

**Stack Recomendado:**

```
Parser:      Docling (melhor para PDFs estruturados)
             LlamaParse (pago, excelente para livros complexos)
Embeddings:  text-embedding-3-large (complexidade justifica custo)
             BGE-M3 (local, multilingual, state-of-art)
Vector DB:   Qdrant (melhor suporte a parent-child + filtros)
Retrieval:   Hybrid + HyDE para queries vagas
Reranker:    Cohere Rerank v3 (produção)
             BGE-Reranker-v2-M3 (local, excelente)
LLM:         Claude 3.5 Sonnet (melhor para contexto longo)
```

**Custo estimado:** ~$0.01–0.05 por query

---

### 📄 D. PDF Limpo / Textual

**Exemplos:** PDF exportado digitalmente, relatório sem gráficos, artigo científico textual, contrato simples

#### Filosofia

```
┌──────────────────────────────────────────────────────────────────┐
│ PRIMEIRO: Parse bem. DEPOIS: Indexa bem.                         │
│                                                                  │
│ Indexar o PDF bruto sem parsing adequado = maior causa de        │
│ baixa precisão em RAG de documentos.                             │
│                                                                  │
│ Multimodal em PDF textual simples = custo extra sem retorno.     │
└──────────────────────────────────────────────────────────────────┘
```

#### Detecção e Parsing Inteligente

```python
import fitz  # PyMuPDF
from docling.document_converter import DocumentConverter

def smart_pdf_parser(pdf_path: str) -> dict:
    """Detecta tipo de PDF e aplica melhor parser"""
    
    doc = fitz.open(pdf_path)
    
    # Análise do PDF
    total_text = "".join(page.get_text() for page in doc)
    total_images = sum(len(page.get_images()) for page in doc)
    text_ratio = len(total_text) / (doc.page_count * 500 + 1)
    
    # Tem tabelas? (heurística)
    has_tables = detect_tables(doc)
    
    # Decisão por tipo
    if text_ratio > 0.7 and not has_tables:
        # PDF nativo textual limpo → PyMuPDF (mais rápido)
        return {
            "tipo": "textual_limpo",
            "parser": "pymupdf",
            "resultado": parse_with_pymupdf(pdf_path)
        }
    
    elif has_tables or text_ratio < 0.3:
        # PDF com tabelas ou escaneado → Docling
        return {
            "tipo": "estruturado_ou_escaneado",
            "parser": "docling",
            "resultado": parse_with_docling(pdf_path)
        }
    
    else:
        # Misto → Docling como padrão seguro
        return {
            "tipo": "misto",
            "parser": "docling",
            "resultado": parse_with_docling(pdf_path)
        }

def parse_with_docling(pdf_path: str) -> dict:
    converter = DocumentConverter()
    result = converter.convert(pdf_path)
    
    return {
        "text": result.document.export_to_markdown(),
        "tables": result.document.tables,
        "images": result.document.pictures,
        "structure": result.document.body,
        "metadata": extract_metadata(result)
    }
```

#### Tratamento de Tabelas em PDF

```python
def process_pdf_table(table, surrounding_context: str) -> str:
    """
    Converte tabela em texto rico para embedding.
    
    ❌ NÃO faça: indexar markdown de tabela bruta
    | Col1 | Col2 | → embedding ruim, busca imprecisa
    
    ✅ FAÇA: converter para prosa contextualizada
    """
    
    description = f"""
    Contexto: {surrounding_context[:200]}
    
    Tabela: {table.caption if table.caption else "Dados tabulares"}
    Esta tabela apresenta {inferir_topico(table)}.
    Possui {table.row_count} linhas e {table.col_count} colunas.
    Colunas: {', '.join(table.headers)}.
    
    Resumo dos dados: {summarize_table(table)}
    
    [REF_TABELA]: {table.to_json()}
    """
    
    return description
```

#### Configuração Técnica

```yaml
parsers_por_tipo:
  pdf_textual_limpo: PyMuPDF (fitz) - rápido, gratuito
  pdf_escaneado: Docling - OCR + layout detection
  pdf_com_tabelas: Docling ou pdfplumber (especialista em tabelas)
  pdf_cientifico_formulas: Nougat (Meta) - especializado
  pdf_complexo_pago: LlamaParse ou Azure Document Intelligence

chunking:
  tamanho: 350–800 tokens
  overlap: 60–150 tokens
  split_por: seção/título/página
  preservar: [tabelas_completas, listas, blocos_codigo]

metadata:
  - page_number
  - section_title
  - table_ids  # referência a tabelas na página
  - doc_title
  - doc_date
  - doc_type

retrieval:
  tipo: hybrid
  top_k: 20–40

rerank:
  ativo: sim
  top_k_final: 5–8

resposta:
  citar: pagina + secao
  abstain: quando evidência insuficiente
```

---

### 🖼️ E. PDF Visualmente Rico / Escaneado / Complexo

**Exemplos:** Notas fiscais, apresentações, papers com gráficos/tabelas, catálogos escaneados, formulários, diagramas

#### Filosofia

```
┌──────────────────────────────────────────────────────────────────┐
│ QUANDO O VISUAL FAZ PARTE DA INFORMAÇÃO, texto puro falha.       │
│                                                                  │
│ O paper ColPali (2024) é um marco: embute imagens de páginas     │
│ diretamente e supera pipelines textuais tradicionais para        │
│ retrieval de documentos visuais.                                 │
│                                                                  │
│ REGRA: Use multimodal SÓ quando layout/gráfico/tabela            │
│        importam para a resposta. Não use por padrão.             │
└──────────────────────────────────────────────────────────────────┘
```

#### Quando Usar Multimodal

```
USE MULTIMODAL QUANDO:
✅ A resposta depende de posição visual no documento
✅ OCR perde informação crítica (tabelas complexas, gráficos)
✅ Diagramas, fluxogramas, mapas são parte da resposta
✅ Layout da página importa (formulários, NFs, contratos estruturados)
✅ Documento é escaneado sem OCR de qualidade

NÃO USE QUANDO:
❌ PDF é basicamente texto corrido
❌ Custo é restrição importante
❌ Velocidade é crítica
❌ As tabelas são simples e OCR captura bem
```

#### Arquitetura Page→Element Retrieval

```python
"""
PIPELINE MULTIMODAL 2026:
Baseado na survey de Multimodal RAG (arXiv 2025) e ColPali

Recuperação em 2 etapas:
  Etapa 1: Recuperar PÁGINA relevante
  Etapa 2: Recuperar ELEMENTO dentro da página
"""

class MultimodalRAG:
    
    def __init__(self):
        self.page_index = VectorDB()      # embeddings de páginas inteiras
        self.element_index = VectorDB()   # embeddings de elementos
        self.vision_model = load_vision_model()
    
    def index_document(self, pdf_path: str):
        doc = fitz.open(pdf_path)
        
        for page_num, page in enumerate(doc):
            # Renderiza página como imagem
            pix = page.get_pixmap(matrix=fitz.Matrix(2, 2))
            img = pix.tobytes("png")
            
            # Embedding visual da página completa
            page_embedding = self.vision_model.embed_image(img)
            
            # Extrai elementos: texto, tabelas, figuras
            elements = extract_page_elements(page)
            
            # Indexa página
            self.page_index.add({
                "embedding": page_embedding,
                "metadata": {
                    "page": page_num,
                    "has_tables": bool(elements["tables"]),
                    "has_figures": bool(elements["figures"]),
                    "text_preview": elements["text"][:200]
                }
            })
            
            # Indexa elementos individualmente
            for elem in elements["tables"] + elements["figures"]:
                elem_embedding = self.vision_model.embed_element(elem)
                self.element_index.add({
                    "embedding": elem_embedding,
                    "metadata": {
                        "page": page_num,
                        "type": elem["type"],
                        "caption": elem.get("caption", ""),
                        "position": elem["bbox"]
                    }
                })
    
    def query(self, question: str, top_pages: int = 5):
        # Etapa 1: Recupera páginas mais relevantes
        relevant_pages = self.page_index.search(
            query=question,
            top_k=top_pages
        )
        
        # Etapa 2: Dentro dessas páginas, busca elementos
        relevant_elements = self.element_index.search(
            query=question,
            filter={"page": [p["metadata"]["page"] for p in relevant_pages]},
            top_k=10
        )
        
        # Rerank combinado
        final_context = self.rerank_combined(
            question, relevant_pages, relevant_elements
        )
        
        return final_context
```

#### Configuração Técnica

```yaml
abordagem:
  pdf_simples: text-first (seções anteriores)
  pdf_visual_complexo: multimodal/page→element

pipeline_multimodal:
  renderizacao: 2x resolução (qualidade OCR)
  
  indice_1_pagina:
    embedding: ColPali ou vision model
    granularidade: página inteira
    
  indice_2_elemento:
    embedding: por tabela/figura/bloco
    granularidade: elemento dentro da página
    
  indice_texto:
    embedding: OCR estruturado
    granularidade: parágrafo/seção
    
  retrieval:
    etapa_1: recuperar páginas
    etapa_2: recuperar elementos nas páginas
    
  rerank: sim, combinando scores visuais e textuais

ferramentas:
  colpali: retrieval visual de páginas (state-of-art)
  docling: OCR + layout extraction estruturado
  azure_di: Document Intelligence (pago, robusto)
  llamaparse: parsing avançado (pago)
```

---

### 📋 F. CSV

**Exemplos:** Lista de produtos, pedidos, clientes, estoque, tabela de preços

#### Filosofia

```
┌──────────────────────────────────────────────────────────────────┐
│ REGRA DE OURO: CSV raramente deve ser tratado como texto puro.   │
│                                                                  │
│ Anyscale é direta: embedding de linhas FALHA em:                 │
│ - cálculo e agregação                                            │
│ - matching numérico exato                                        │
│ - lógica relacional                                              │
│                                                                  │
│ A decisão correta depende do TIPO DE PERGUNTA, não do arquivo.   │
└──────────────────────────────────────────────────────────────────┘
```

#### Árvore de Decisão para CSV

```
CSV recebido
    │
    ├── Pergunta é semântica/vaga?
    │   ("me fale do produto X", "qual item é bom para Y?")
    │   └── RAG estruturado (documento por linha)
    │
    ├── Pergunta é lookup exato?
    │   ("qual o produto com código ABC123?")
    │   └── RAG estruturado + filtro por metadata
    │
    ├── Pergunta é analítica/agregação?
    │   ("soma do estoque", "faturamento por mês", "top 5")
    │   └── TEXT-TO-SQL / PANDAS AGENT
    │
    └── CSV pequeno o suficiente? (< 50k tokens)
        └── FULL CONTEXT (manda tudo no prompt)
```

#### F1. CSV para Lookup/Semântico - RAG Estruturado

```python
import pandas as pd

def csv_to_rag_documents(csv_path: str) -> list[dict]:
    """
    Converte CSV em documentos ricos para RAG.
    
    ❌ ERRADO: indexar linha CSV bruta
    "ID,Nome,Preço\n1,Tênis Nike,299.90"
    
    ✅ CORRETO: texto canônico + metadata estruturada
    """
    
    df = pd.read_csv(csv_path)
    documents = []
    
    for _, row in df.iterrows():
        # Texto canônico (para embedding semântico)
        text_parts = []
        for col, val in row.items():
            if pd.notna(val) and str(val).strip():
                text_parts.append(f"{col}: {val}")
        
        canonical_text = " | ".join(text_parts)
        
        # Metadata estruturada (para filtros)
        metadata = {}
        for col, val in row.items():
            if pd.notna(val):
                metadata[col.lower().replace(" ", "_")] = str(val)
        
        # Campos numéricos como float para range filters
        for col in ["preco", "price", "estoque", "stock", "quantidade"]:
            if col in metadata:
                try:
                    metadata[f"{col}_num"] = float(metadata[col])
                except:
                    pass
        
        documents.append({
            "text": canonical_text,
            "metadata": metadata
        })
    
    return documents
```

#### F2. CSV para Analytics - Text-to-SQL

```python
from langchain_community.agent_toolkits import create_sql_agent
import duckdb

def csv_analytics_agent(csv_path: str, query: str):
    """
    Para queries analíticas em CSV: SQL é superior ao RAG.
    
    Exemplos de queries que DEVEM ir para SQL:
    - "qual foi o faturamento total de março?"
    - "top 10 produtos com maior margem"
    - "estoque total por fornecedor"
    - "quantos pedidos por status?"
    """
    
    # DuckDB: lê CSV diretamente sem banco de dados
    conn = duckdb.connect()
    conn.execute(f"CREATE TABLE dados AS SELECT * FROM '{csv_path}'")
    
    # O LLM gera SQL, não responde diretamente
    sql_query = llm(f"""
    Gere APENAS uma query SQL DuckDB válida para responder:
    "{query}"
    
    Tabela: dados
    Colunas: {get_column_info(conn)}
    
    Retorne APENAS o SQL, sem explicação.
    """)
    
    result = conn.execute(sql_query).fetchdf()
    
    # LLM interpreta o resultado
    response = llm(f"""
    Query: {query}
    Resultado SQL: {result.to_markdown()}
    
    Responda a pergunta do usuário baseado no resultado.
    """)
    
    return response
```

#### Configuração Técnica

```yaml
csv_lookup_semantico:
  chunk_strategy: uma_linha_por_documento
  texto: concatenacao_canonica_dos_campos
  metadata:
    - todos_os_campos_filtráveis
    - campos_numericos_como_float
    - campos_categoricos_normalizados
  retrieval: hybrid (BM25 forte + dense)
  rerank: opcional para < 1000 itens, recomendado para mais
  filtros: por metadata antes do embedding search

csv_analitico:
  abordagem: text_to_sql
  engine: DuckDB (leve, rápido, lê CSV direto)
  llm_role: gerador_de_sql + interpretador_resultado
  embedding: zero ou mínimo
  cuidado: sandbox de segurança obrigatório

csv_pequeno:
  threshold: < 50k tokens
  abordagem: full_context (manda tudo no prompt)
  formato: df.to_markdown()
```

---

### 📊 G. XLS / XLSX / Planilhas com Múltiplas Abas

**Exemplos:** Planilha de estoque, tabela de custos, catálogo com várias abas, financeiro, relatório

#### Filosofia

```
┌──────────────────────────────────────────────────────────────────┐
│ Excel é o tipo de arquivo mais ambíguo.                          │
│ Antes de escolher a estratégia, entenda O QUE É aquela planilha. │
│                                                                  │
│ Uma planilha pode ser:                                           │
│ - Uma lista (→ trate como CSV)                                   │
│ - Um relatório (→ trate como documento)                          │
│ - Um banco de dados relacional (→ trate com SQL)                 │
│ - Uma mistura de tudo (→ trate cada aba separadamente)           │
└──────────────────────────────────────────────────────────────────┘
```

#### Parser Inteligente de Excel

```python
import pandas as pd
import openpyxl
from typing import Literal

def parse_excel_smart(excel_path: str) -> list[dict]:
    documents = []
    xl = pd.ExcelFile(excel_path)
    
    for sheet_name in xl.sheet_names:
        df = xl.parse(sheet_name)
        
        # Detecta o tipo da aba
        sheet_type = classify_sheet(df, sheet_name)
        
        if sheet_type == "lista_cadastro":
            # → Trata como CSV: um documento por linha
            docs = sheet_as_row_documents(df, sheet_name)
            documents.extend(docs)
        
        elif sheet_type == "relatorio":
            # → Trata como documento: preserva estrutura
            docs = sheet_as_report(df, sheet_name)
            documents.extend(docs)
        
        elif sheet_type == "matriz_pivot":
            # → Converte para série de afirmações
            docs = matrix_to_statements(df, sheet_name)
            documents.extend(docs)
        
        elif sheet_type == "analitico":
            # → Não indexa: usa SQL/DuckDB
            register_for_sql(df, sheet_name)
        
    return documents

def classify_sheet(df: pd.DataFrame, name: str) -> Literal[
    "lista_cadastro", "relatorio", "matriz_pivot", "analitico"
]:
    """Heurística para classificar tipo de aba"""
    
    # Muitas linhas + poucos tipos únicos nas colunas = lista
    if len(df) > 50 and df.dtypes.nunique() <= 4:
        return "lista_cadastro"
    
    # Muitas fórmulas + totais + poucas linhas = relatório
    if len(df) < 50 and "total" in str(df.columns).lower():
        return "relatorio"
    
    # Headers em ambos os eixos = pivot/matriz
    if df.index.dtype != int:
        return "matriz_pivot"
    
    # Números puros + datas = analítico
    numeric_ratio = df.select_dtypes(include='number').shape[1] / df.shape[1]
    if numeric_ratio > 0.7:
        return "analitico"
    
    return "lista_cadastro"  # default
```

#### Casos Específicos

```python
# G1. Aba como Lista/Cadastro
def sheet_as_row_documents(df, sheet_name):
    """Igual ao CSV: linha → documento canônico"""
    docs = []
    for _, row in df.iterrows():
        text = f"[{sheet_name}] " + " | ".join([
            f"{col}: {val}" 
            for col, val in row.items() 
            if pd.notna(val)
        ])
        docs.append({
            "text": text,
            "metadata": {
                "sheet": sheet_name,
                "source_type": "xlsx_row",
                **{str(k): str(v) for k, v in row.items() if pd.notna(v)}
            }
        })
    return docs

# G2. Aba como Relatório
def sheet_as_report(df, sheet_name):
    """Preserva estrutura hierárquica do relatório"""
    
    # Identifica seções (linhas de cabeçalho/subtotal)
    sections = detect_report_sections(df)
    
    docs = []
    for section in sections:
        docs.append({
            "text": f"""
            Planilha: {sheet_name}
            Seção: {section['header']}
            
            {section['content']}
            
            {f"Total/Subtotal: {section['subtotal']}" if section.get('subtotal') else ""}
            """,
            "metadata": {
                "sheet": sheet_name,
                "section": section['header'],
                "row_range": f"{section['start_row']}-{section['end_row']}",
                "has_numbers": section['has_numeric_data']
            }
        })
    return docs
```

---

### 🛒 H. Catálogo de Produtos / SKUs / Peças / Compatibilidades

**Exemplos:** E-commerce, estoque de peças, cardápio, lista de preços, catálogo industrial

#### Filosofia

```
┌──────────────────────────────────────────────────────────────────┐
│ Esse é o caso mais especial e mais mal-implementado.             │
│                                                                  │
│ Códigos, SKUs, referências cruzadas, sinônimos, compatibilidades │
│ SÃO O CORAÇÃO do retrieval de produtos.                          │
│                                                                  │
│ Azure: códigos e jargão funcionam MELHOR com keyword search.     │
│ Anyscale: IDs e product codes = caso crítico para híbrido.       │
│ Benchmark: BM25 > dense em termos precisos; híbrido+rerank > tudo│
└──────────────────────────────────────────────────────────────────┘
```

#### Estrutura de Documento Ideal por Produto

```python
# ❌ ERRADO: Embeder CSV bruto
"ID,Nome,Preço,Categoria\n1,Tênis Nike Air Max,299.90,Calçados"

# ❌ ERRADO: Texto genérico sem estrutura
"Este produto é um tênis da Nike que custa 299 reais."

# ✅ CORRETO: Documento rico, canônico, com metadata completa
PRODUTO_IDEAL = {
    
    # Texto para embedding semântico
    "text_to_embed": """
        Tênis Nike Air Max 270 masculino.
        Referência: NK-AM270-001 | Nike AM270 | Air Max 270.
        Categoria: calçados esportivos / tênis de corrida.
        Cor: preto com detalhes brancos.
        Material: mesh respirável com solado Air Max de espuma.
        Ideal para corrida leve, treino e uso casual urbano.
        Compatível com: uso masculino, tamanhos 38-44.
        Equivalências: similar ao Nike React, alternativa ao Adidas Ultraboost.
    """,
    
    # Metadata para FILTROS (não para embedding)
    "metadata": {
        # Identificação
        "sku": "NK-AM270-001",
        "ean": "7891234567890",
        "codigo_interno": "CAL-00142",
        
        # Aliases e variações de nome (CRÍTICO para busca)
        "nome_principal": "Tênis Nike Air Max 270",
        "aliases": ["Air Max 270", "AM270", "Nike 270", "airmax270"],
        "sinonimos_busca": ["tênis nike", "air max", "calçado esportivo"],
        
        # Comercial
        "preco": 299.90,
        "preco_num": 299.90,        # Para range filter
        "desconto_pct": 0,
        "preco_promocional": 299.90,
        
        # Classificação
        "categoria": "calcados",
        "subcategoria": "esportivo",
        "marca": "nike",
        "linha": "air_max",
        "genero": "masculino",
        
        # Atributos
        "tamanhos_disponiveis": [38, 39, 40, 41, 42, 43, 44],
        "cores": ["preto", "branco"],
        "material_principal": "mesh",
        
        # Logística
        "estoque": 45,
        "em_estoque": True,
        "peso_kg": 0.85,
        
        # Qualidade
        "avaliacao_media": 4.7,
        "num_avaliacoes": 234,
        
        # Compatibilidade (para peças/autopeças)
        # "aplicacao": ["Toyota Corolla 2018-2022", "Toyota Camry 2019-2023"]
        # "codigo_original": "04466-02370"
        # "equivalentes": ["TRW GDB1234", "Bosch 0986494123"]
    }
}
```

#### Pipeline de Query para Produtos

```python
class ProductRAG:
    
    def query(self, user_query: str) -> list[dict]:
        
        # Etapa 1: Classificar tipo de query
        query_type = self.classify_query(user_query)
        
        # Etapa 2: Extrair entidades e filtros
        entities = self.extract_entities(user_query)
        # → {"marca": "nike", "preco_max": 300, "genero": "masculino"}
        
        # Etapa 3: Pipeline por tipo
        if query_type == "analitica":
            # "quantos produtos Nike temos?"
            return self.sql_pipeline(user_query)
        
        elif query_type == "lookup_exato":
            # "produto código NK-AM270-001"
            return self.exact_lookup(entities.get("sku") or entities.get("codigo"))
        
        elif query_type == "filtrada_pura":
            # "tênis até R$200"
            return self.filtered_search(
                query=user_query,
                filters=entities,
                boost_keyword=True  # Peso alto para BM25
            )
        
        elif query_type == "semantica_pura":
            # "algo confortável para presente"
            return self.semantic_search(user_query)
        
        else:  # híbrida (maioria dos casos)
            # "tênis nike confortável até R$300"
            query_limpa = self.remove_filter_terms(user_query, entities)
            return self.hybrid_search(
                query=query_limpa,
                filters=entities,
                keyword_weight=0.6,   # Peso maior para keyword em produtos
                dense_weight=0.4
            )
    
    def hybrid_search(self, query, filters, keyword_weight, dense_weight):
        
        # Pré-filtra por metadata (reduz o espaço de busca)
        pre_filtered = self.vector_db.search(
            query=query,
            filter=filters,  # Filtra ANTES
            top_k=50,        # Amplo para rerank depois
            alpha=dense_weight  # Proporção dense vs sparse
        )
        
        # Rerank no conjunto pré-filtrado
        reranked = self.reranker.rank(
            query=query,
            documents=pre_filtered,
            top_k=10
        )
        
        return reranked
```

#### Para Autopeças / Compatibilidades (Caso Crítico)

```python
AUTOPEÇA_DOCUMENTO = {
    "text_to_embed": """
        Pastilha de freio dianteira, marca TRW, modelo GDB1234.
        Aplicação: Toyota Corolla 2018 a 2022, motor 1.8 e 2.0.
        Toyota Camry 2019 a 2023.
        Material: cerâmica de alta performance.
        Dimensões: 147mm x 58mm x 17mm.
        Posição: dianteiro, eixo dianteiro.
        OEM equivalente: 04466-02370.
        Equivalentes de mercado: Bosch 0986494123, Fremax FD1234.
    """,
    "metadata": {
        "sku": "TRW-GDB1234",
        "marca": "trw",
        "tipo": "pastilha_freio",
        "posicao": "dianteiro",
        "material": "ceramica",
        
        # Compatibilidade estruturada (para filtros)
        "marca_veiculo": ["toyota"],
        "modelos": ["corolla", "camry"],
        "anos": list(range(2018, 2024)),  # [2018, 2019, ..., 2023]
        "motores": ["1.8", "2.0"],
        
        # Códigos (CRÍTICO para BM25)
        "codigo_oem": "04466-02370",
        "equivalentes": ["0986494123", "FD1234"],
        "todos_codigos": ["TRW-GDB1234", "04466-02370", "0986494123", "FD1234"]
    }
}

# Busca por compatibilidade
def busca_por_veiculo(marca, modelo, ano, tipo_peca):
    return vector_db.search(
        query=f"{tipo_peca} para {marca} {modelo} {ano}",
        filter={
            "marca_veiculo": marca.lower(),
            "modelos": modelo.lower(),
            "anos": {"$gte": ano, "$lte": ano},  # filtro de range
            "tipo": tipo_peca.lower()
        },
        top_k=20
    )
```

#### Configuração Técnica

```yaml
estrutura:
  granularidade: um_documento_por_sku_variacao
  texto: template_canonico_por_categoria
  metadata: todos_os_campos_filtráveis

campos_criticos:
  - sku, ean, codigo_interno
  - aliases e sinonimos de busca
  - aplicacao e compatibilidade (para peças)
  - equivalentes e codigos_cruzados

chunking: nenhum (produto é atômico)

embeddings:
  modelo: text-embedding-3-small (suficiente + barato para produtos)
  dimensoes: 512 (reduzido, mais rápido)
  cache: sim (produto raramente muda)

retrieval:
  tipo: hybrid com peso lexical ALTO
  keyword_weight: 0.6
  dense_weight: 0.4
  pre_filter: metadata antes do embedding search
  top_k_candidatos: 50

rerank:
  ativo: sempre
  top_k_final: 10-20
  modelo: cross-encoder/ms-marco-MiniLM-L-6-v2

cache:
  query_cache: sim (queries repetidas são comuns)
  ttl: 3600 segundos
  invalidacao: na atualização de produtos
```

---

## 5. RAG por Tipo de Arquivo {#por-arquivo}

### Tabela de Parsers Recomendados

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    MELHOR PARSER POR ARQUIVO (2026)                     │
├──────────────────┬───────────────────────────┬──────────┬───────────────┤
│ Arquivo          │ Parser Recomendado        │ Custo    │ Qualidade     │
├──────────────────┼───────────────────────────┼──────────┼───────────────┤
│ PDF textual      │ PyMuPDF (fitz)            │ Grátis   │ ★★★★☆        │
│ PDF escaneado    │ Docling                   │ Grátis   │ ★★★★★        │
│ PDF com tabelas  │ Docling ou pdfplumber     │ Grátis   │ ★★★★☆        │
│ PDF científico   │ Nougat (Meta)             │ Grátis   │ ★★★★★        │
│ PDF complexo     │ LlamaParse ou Azure DI    │ Pago     │ ★★★★★        │
│ Markdown         │ MarkdownHeaderTextSplitter│ Grátis   │ ★★★★★        │
│ EPUB/eBook       │ ebooklib + BeautifulSoup  │ Grátis   │ ★★★★☆        │
│ XLS/XLSX         │ pandas + openpyxl         │ Grátis   │ ★★★★☆        │
│ CSV              │ pandas                    │ Grátis   │ ★★★★★        │
│ TXT              │ nativo Python             │ Grátis   │ ★★★★★        │
│ DOCX/DOC         │ python-docx               │ Grátis   │ ★★★★☆        │
│ HTML             │ BeautifulSoup + trafilatura│ Grátis  │ ★★★★☆        │
│ PowerPoint       │ python-pptx               │ Grátis   │ ★★★☆☆        │
│ JSON/JSONL       │ nativo Python             │ Grátis   │ ★★★★★        │
└──────────────────┴───────────────────────────┴──────────┴───────────────┘
```

### Código de Parser Universal

```python
import os
from pathlib import Path

class UniversalParser:
    """
    Parser inteligente que detecta o tipo de arquivo
    e aplica o melhor parser automaticamente.
    """
    
    PARSERS = {
        ".pdf":   "smart_pdf_parser",
        ".md":    "markdown_parser",
        ".mdx":   "markdown_parser",
        ".epub":  "epub_parser",
        ".xlsx":  "excel_parser",
        ".xls":   "excel_parser",
        ".csv":   "csv_parser",
        ".txt":   "text_parser",
        ".docx":  "docx_parser",
        ".html":  "html_parser",
        ".htm":   "html_parser",
        ".pptx":  "pptx_parser",
        ".json":  "json_parser",
        ".jsonl": "jsonl_parser",
    }
    
    def parse(self, file_path: str) -> list[dict]:
        ext = Path(file_path).suffix.lower()
        parser_name = self.PARSERS.get(ext)
        
        if not parser_name:
            raise ValueError(f"Formato não suportado: {ext}")
        
        parser_fn = getattr(self, parser_name)
        return parser_fn(file_path)
    
    def markdown_parser(self, path: str) -> list[dict]:
        with open(path, 'r', encoding='utf-8') as f:
            content = f.read()
        
        from langchain.text_splitter import MarkdownHeaderTextSplitter
        
        splitter = MarkdownHeaderTextSplitter(
            headers_to_split_on=[
                ("#", "h1"), ("##", "h2"), 
                ("###", "h3"), ("####", "h4")
            ],
            strip_headers=False
        )
        
        chunks = splitter.split_text(content)
        
        return [{
            "text": chunk.page_content,
            "metadata": {
                "source": path,
                "type": "markdown",
                **chunk.metadata
            }
        } for chunk in chunks]
    
    def epub_parser(self, path: str) -> list[dict]:
        import ebooklib
        from ebooklib import epub
        from bs4 import BeautifulSoup
        
        book = epub.read_epub(path)
        chapters = []
        
        for item in book.get_items():
            if item.get_type() == ebooklib.ITEM_DOCUMENT:
                soup = BeautifulSoup(item.get_content(), 'html.parser')
                
                # Remove elementos desnecessários
                for tag in soup(['style', 'script', 'nav', 'aside']):
                    tag.decompose()
                
                text = soup.get_text(separator='\n', strip=True)
                
                if len(text.strip()) > 100:  # Ignora páginas vazias
                    chapters.append({
                        "text": text,
                        "metadata": {
                            "source": path,
                            "type": "epub",
                            "chapter": item.get_name(),
                            "book_title": book.title,
                            "author": ", ".join([
                                str(a) for a in book.get_metadata('DC', 'creator')
                            ])
                        }
                    })
        
        return chapters
    
    def html_parser(self, path: str) -> list[dict]:
        import trafilatura
        
        with open(path, 'r', encoding='utf-8') as f:
            html = f.read()
        
        # trafilatura extrai texto limpo (sem nav, ads, footers)
        text = trafilatura.extract(html, include_tables=True)
        
        return [{
            "text": text,
            "metadata": {
                "source": path,
                "type": "html"
            }
        }]
```

---

## 6. Técnicas Avançadas de Precisão {#tecnicas}

### 6.1 Hybrid Retrieval + RRF (O Default Obrigatório)

```python
"""
HYBRID = BM25 (keyword) + Dense (embedding)
RRF = Reciprocal Rank Fusion (combina os rankings)

Por que funciona melhor:
- BM25: ótimo para códigos, nomes, siglas, termos exatos
- Dense: ótimo para semântica, sinônimos, paráfrases
- Juntos: cobrem os dois mundos
"""

def hybrid_search_rrf(
    query: str,
    vector_db,
    bm25_index,
    top_k: int = 10,
    rrf_k: int = 60
) -> list:
    
    # Busca 1: Dense (embedding)
    dense_results = vector_db.similarity_search(query, k=50)
    
    # Busca 2: Sparse (BM25)
    bm25_results = bm25_index.search(query, k=50)
    
    # Fusão com RRF
    scores = {}
    
    for rank, doc in enumerate(dense_results):
        doc_id = doc.metadata["id"]
        scores[doc_id] = scores.get(doc_id, 0) + 1 / (rrf_k + rank + 1)
    
    for rank, doc in enumerate(bm25_results):
        doc_id = doc.metadata["id"]
        scores[doc_id] = scores.get(doc_id, 0) + 1 / (rrf_k + rank + 1)
    
    # Ordena por score RRF
    sorted_ids = sorted(scores.items(), key=lambda x: x[1], reverse=True)
    
    return get_docs_by_ids([id for id, _ in sorted_ids[:top_k]])
```

### 6.2 Reranking (O Maior Salto de Qualidade)

```python
"""
ANALOGIA PERFEITA:
- Embedding Search: pescador com rede grande (pega muito, nem tudo relevante)
- Reranker: especialista que inspeciona cada peixe e escolhe os melhores

O benchmark T2-RAGBench 2026 mostra salto SIGNIFICATIVO com rerank
sobre híbrido sem rerank. É o upgrade com melhor custo-benefício.
"""

from sentence_transformers import CrossEncoder

class Reranker:
    
    def __init__(self, model_name: str = "cross-encoder/ms-marco-MiniLM-L-6-v2"):
        self.model = CrossEncoder(model_name)
    
    def rerank(
        self, 
        query: str, 
        documents: list[str], 
        top_k: int = 5,
        score_threshold: float = 0.1
    ) -> list[dict]:
        
        # Cross-encoder analisa query + doc JUNTOS (mais preciso que bi-encoder)
        pairs = [(query, doc) for doc in documents]
        scores = self.model.predict(pairs)
        
        # Ordena por relevância
        ranked = sorted(
            zip(documents, scores),
            key=lambda x: x[1],
            reverse=True
        )
        
        # Filtra por threshold e retorna top_k
        results = [
            {"text": doc, "score": float(score)}
            for doc, score in ranked[:top_k]
            if score >= score_threshold
        ]
        
        return results

RERANKERS_2026 = {
    "cohere_rerank_v3": {
        "tipo": "API",
        "custo": "$0.002/1k tokens",
        "precisao": "★★★★★",
        "velocidade": "rápido",
        "melhor_para": "produção, multilingual"
    },
    "ms_marco_minilm_l6": {
        "modelo": "cross-encoder/ms-marco-MiniLM-L-6-v2",
        "tipo": "local",
        "custo": "grátis",
        "precisao": "★★★★☆",
        "velocidade": "muito rápido",
        "melhor_para": "produção local, inglês"
    },
    "bge_reranker_v2_m3": {
        "modelo": "BAAI/bge-reranker-v2-m3",
        "tipo": "local",
        "custo": "grátis",
        "precisao": "★★★★★",
        "velocidade": "médio",
        "melhor_para": "produção local, multilingual, português"
    },
    "jina_reranker_v2": {
        "tipo": "API ou local",
        "custo": "grátis (local)",
        "precisao": "★★★★☆",
        "melhor_para": "alternativa gratuita de qualidade"
    }
}
```

### 6.3 Contextual Retrieval (Anthropic, 2024)

```python
"""
PUBLICADO PELA ANTHROPIC - Reduz falhas de retrieval em:
- 49% sozinho
- 67% combinado com reranking

PROBLEMA que resolve:
"O resultado aumentou 23% no trimestre"
→ Sem contexto: qual empresa? qual métrica? qual trimestre?

SOLUÇÃO: Adiciona contexto situacional a CADA chunk ANTES de embedar
"""

def contextual_retrieval_indexing(
    full_document: str,
    chunks: list[str],
    llm,
    cheap_model: str = "gpt-4o-mini"  # Usa modelo barato para isso
) -> list[str]:
    
    # Contexto do documento (resumo dos primeiros 3000 tokens)
    doc_context = full_document[:3000]
    
    contextualized_chunks = []
    
    for chunk in chunks:
        # Gera contexto específico para este chunk
        context = llm(
            model=cheap_model,
            prompt=f"""
            <document>
            {doc_context}
            </document>
            
            <chunk>
            {chunk}
            </chunk>
            
            Escreva 2-3 frases CONCISAS que contextualizam este chunk
            dentro do documento completo. Inclua:
            - De qual seção/parte do documento ele é
            - Qual o tema geral do documento
            - O que especificamente este chunk informa
            
            Seja direto. Não use frases como "Este chunk...".
            """
        )
        
        # Prepende o contexto ao chunk
        contextualized = f"{context}\n\n{chunk}"
        contextualized_chunks.append(contextualized)
    
    return contextualized_chunks

# ANTES: "O resultado aumentou 23% no trimestre"
# DEPOIS: "Relatório financeiro Q3 2024 da Empresa XYZ, seção de 
#           resultados trimestrais. Apresenta crescimento da receita
#           líquida comparado ao trimestre anterior.
#           O resultado aumentou 23% no trimestre"
```

### 6.4 Late Chunking (Melhor Meio-Termo)

```python
"""
PUBLICADO EM: arXiv:2409.04701

INSIGHT: Embeda o texto LONGO primeiro, depois quebra em chunks.
Isso preserva o contexto global no embedding de cada chunk.

VANTAGEM vs Contextual Retrieval:
- Mais eficiente (sem chamadas LLM por chunk)
- Mais barato na indexação
- Quase tão bom em qualidade

DESVANTAGEM:
- Perde um pouco em relevância/completude vs Contextual Retrieval
- Requer modelo de embedding com contexto longo (8k+ tokens)

QUANDO USAR:
- Equilíbrio custo/qualidade para documentos longos
- Quando budget de indexação é limitado
"""

def late_chunking(
    document: str,
    embedding_model,  # Modelo com contexto longo (ex: nomic-embed-text)
    chunk_size: int = 400,
    overlap: int = 50
) -> list[dict]:
    
    # Passo 1: Embeda o documento INTEIRO (contexto global)
    # O modelo vê todo o documento antes de criar embeddings
    full_embedding = embedding_model.embed_long_document(document)
    
    # Passo 2: Divide em chunks (DEPOIS do embedding)
    chunks = split_into_chunks(document, chunk_size, overlap)
    
    # Passo 3: Extrai embedding de cada chunk da representação completa
    # (ao invés de embedar cada chunk isoladamente)
    chunk_embeddings = embedding_model.extract_chunk_embeddings(
        full_embedding=full_embedding,
        chunks=chunks
    )
    
    return [{
        "text": chunk,
        "embedding": embedding,
        "metadata": {
            "chunk_index": i,
            "total_chunks": len(chunks),
            "context_aware": True
        }
    } for i, (chunk, embedding) in enumerate(zip(chunks, chunk_embeddings))]
```

### 6.5 HyDE - Hypothetical Document Embeddings

```python
"""
QUANDO USAR: Queries vagas, muito curtas, ou abstratas

CUIDADO: Benchmark T2-RAGBench 2026 mostrou que HyDE ficou ABAIXO
do dense padrão em cenário misto texto+tabela.

Use como OPÇÃO, não como default universal.
Funciona melhor com: textos longos e narrativos, queries abstratas.
"""

def hyde_retrieval(query: str, vector_db, llm_cheap):
    
    # Gera documento hipotético que responderia à query
    hypothetical_doc = llm_cheap(f"""
    Escreva um parágrafo técnico e específico que seria a resposta 
    ideal para a seguinte pergunta. Use terminologia do domínio.
    Seja concreto e informativo.
    
    Pergunta: "{query}"
    
    Parágrafo de resposta:
    """)
    
    # Busca com o embedding do documento hipotético
    # (mais próximo dos documentos reais do que a query original)
    results = vector_db.search(
        query=hypothetical_doc,
        top_k=10
    )
    
    return results
```

### 6.6 Multi-Query Decomposition

```python
"""
PROBLEMA: "Quais as diferenças entre Python e Java em performance,
           facilidade e ecossistema?"
→ É 3 queries em 1. Busca única = contexto incompleto.

SOLUÇÃO: Decompõe em sub-queries, busca separadamente, combina.
"""

def multi_query_rag(query: str, vector_db, llm):
    
    # Decompõe a query complexa
    sub_queries_text = llm(f"""
    Decomponha esta pergunta em 3-5 sub-perguntas específicas e independentes.
    Cada sub-pergunta deve ser autocontida e clara.
    Retorne APENAS as sub-perguntas, uma por linha, sem numeração.
    
    Pergunta: {query}
    """)
    
    sub_queries = [q.strip() for q in sub_queries_text.split('\n') if q.strip()]
    
    # Busca independente para cada sub-query
    all_results = []
    seen_ids = set()
    
    for sub_q in sub_queries:
        results = vector_db.search(sub_q, top_k=5)
        for doc in results:
            if doc.id not in seen_ids:
                all_results.append(doc)
                seen_ids.add(doc.id)
    
    # Rerank do conjunto completo contra a query original
    reranked = reranker.rerank(
        query=query,
        documents=[doc.text for doc in all_results],
        top_k=8
    )
    
    return reranked
```

### 6.7 Self-RAG (Verificação de Relevância)

```python
"""
Self-RAG: O modelo decide SE precisa buscar E verifica se o resultado
é relevante antes de usar. Evita alucinações e respostas baseadas
em contexto errado.

CUSTO: Mais chamadas LLM, mais lento
PRECISÃO: Máxima (quando bem calibrado)
"""

def self_rag_pipeline(query: str, vector_db, llm):
    
    # PASSO 1: Precisa buscar?
    needs_retrieval = llm(f"""
    Você consegue responder "{query}" com certeza usando apenas
    seu conhecimento geral sem risco de erro factual?
    Responda: SIM ou NÃO
    """).strip().upper()
    
    if needs_retrieval == "SIM":
        return llm(query)  # Responde direto, sem RAG
    
    # PASSO 2: Busca candidatos
    candidates = vector_db.search(query, top_k=8)
    
    # PASSO 3: Filtra por relevância individual
    relevant_docs = []
    for doc in candidates:
        is_relevant = llm(f"""
        Este trecho é relevante para responder: "{query}"?
        
        Trecho: {doc.text[:500]}
        
        Responda: SIM ou NÃO
        """).strip().upper()
        
        if is_relevant == "SIM":
            relevant_docs.append(doc)
    
    if not relevant_docs:
        return "Não encontrei informações suficientes nos documentos disponíveis para responder com segurança."
    
    # PASSO 4: Gera resposta
    context = "\n\n".join([doc.text for doc in relevant_docs])
    response = llm(f"""
    Responda baseado APENAS nos trechos abaixo. 
    Se a informação não estiver nos trechos, diga que não encontrou.
    
    Trechos:
    {context}
    
    Pergunta: {query}
    """)
    
    # PASSO 5: Verifica se resposta é suportada (grounding)
    is_grounded = llm(f"""
    A resposta abaixo é completamente suportada pelos trechos fornecidos?
    Não há informações inventadas?
    
    Resposta: {response[:300]}
    Trechos: {context[:500]}
    
    Responda: SIM ou NÃO
    """).strip().upper()
    
    if is_grounded == "NÃO":
        return "Não consigo confirmar essa informação nos documentos disponíveis."
    
    return response
```

### 6.8 GraphRAG (Microsoft, para Conhecimento Complexo)

```
QUANDO USAR GraphRAG:
✅ Base de conhecimento com muitas entidades relacionadas
✅ Perguntas sobre relacionamentos: "quem trabalhou com quem?"
✅ Análise de redes (pessoas, empresas, eventos)
✅ Quando perguntas globais sobre o corpus são frequentes

QUANDO NÃO USAR:
❌ Documentos simples ou isolados
❌ Custo é restrição importante
❌ Velocidade é crítica
❌ Pequena quantidade de documentos

ARQUITETURA:
```

```
TEXTO
  ↓
Extração de Entidades + Relações (LLM)
  ↓
GRAFO: [Empresa A] ──adquiriu──▶ [Empresa B]
       [Empresa B] ──fundada por──▶ [Pessoa X]
       [Pessoa X] ──trabalhou em──▶ [Empresa C]
  ↓
Detecção de Comunidades (clusters de entidades)
  ↓
Resumo de cada Comunidade (LLM)
  ↓
Índice Hierárquico: grafo + resumos + chunks originais
  ↓
QUERY: busca no grafo + resumos + chunks
```

```yaml
graphrag_config:
  llm_extracao: gpt-4o-mini    # Barato para extração
  llm_resumo: gpt-4o-mini
  embeddings: text-embedding-3-small
  
  chunks:
    size: 1200
    overlap: 100
    
  entity_extraction:
    max_gleanings: 1
    
  community_reports:
    max_length: 2000
    
  search_modes:
    local: busca em entidades e relações específicas
    global: busca em resumos de comunidade (visão macro)
```

---

## 7. Componentes: Embeddings, Vector DBs, Rerankers {#componentes}

### 7.1 Embeddings 2026

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MELHORES EMBEDDINGS 2026                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  GRATUITOS / LOCAIS                                                 │
│                                                                     │
│  BGE-M3 (BAAI)                                    ★★★★★           │
│  ├── Multilingual (100+ idiomas, ótimo português)                   │
│  ├── Multi-granularity: dense + sparse + colbert                    │
│  ├── State-of-art em benchmarks MTEB                                │
│  ├── Contexto: 8192 tokens                                          │
│  └── Contra: modelo grande (560M params), mais lento                │
│                                                                     │
│  nomic-embed-text-v1.5                            ★★★★☆           │
│  ├── Open source, grátis, roda local                                │
│  ├── Contexto longo: 8192 tokens (ótimo para Late Chunking)         │
│  ├── Matryoshka (reduz dimensões sem perda proporcional)            │
│  └── Bom para português                                             │
│                                                                     │
│  PAGOS (melhor custo-benefício)                                     │
│                                                                     │
│  text-embedding-3-small (OpenAI)                  ★★★★☆           │
│  ├── $0.02/1M tokens (muito barato)                                 │
│  ├── Matryoshka dimensions                                          │
│  ├── Rápido                                                         │
│  └── Recomendado para produção                                      │
│                                                                     │
│  text-embedding-3-large (OpenAI)                  ★★★★★           │
│  ├── Mais preciso (~15-20% vs small)                                │
│  ├── 5x mais caro que small                                         │
│  └── Use para: livros longos, documentos complexos                  │
│                                                                     │
│  voyage-3 (Anthropic/Voyage)                      ★★★★★           │
│  ├── State-of-art em retrieval de contexto longo                    │
│  └── Bom para documentos técnicos                                   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 7.2 Vector Databases 2026

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    VECTOR DATABASES 2026                                │
├──────────────┬──────────┬──────────┬──────────┬────────────────────────┤
│ DB           │ Custo    │ Escala   │ Filtros  │ Melhor para            │
├──────────────┼──────────┼──────────┼──────────┼────────────────────────┤
│ ChromaDB     │ Grátis   │ Pequena  │ Básico   │ Dev, prototipagem      │
│ LanceDB      │ Grátis   │ Grande   │ Avançado │ Local + S3, serverless │
│ Qdrant       │ Grátis*  │ Grande   │ Avançado │ Produção geral ✅      │
│ Weaviate     │ Grátis*  │ Grande   │ Avançado │ GraphQL, módulos       │
│ Milvus       │ Grátis   │ Enorme   │ Avançado │ Self-hosted enterprise │
│ pgvector     │ Grátis   │ Média    │ SQL full │ Já usa PostgreSQL ✅   │
│ Pinecone     │ Pago     │ Enorme   │ Avançado │ Serverless enterprise  │
│ Faiss        │ Grátis   │ Enorme   │ Nenhum   │ Pesquisa pura, offline │
└──────────────┴──────────┴──────────┴──────────┴────────────────────────┘
*Free tier disponível

RECOMENDAÇÕES PRÁTICAS:
- Prototipagem local:     ChromaDB ou LanceDB
- Produção self-hosted:   Qdrant (melhor suporte parent-child + filtros)
- Já tem PostgreSQL:      pgvector (zero infra extra)
- Budget cloud:           Qdrant Cloud free tier (até 1GB)
- Enterprise grande:      Pinecone ou Milvus
```

### 7.3 Sparse Search (BM25) para Híbrido

```python
"""
Para implementar a parte BM25 do hybrid search:
"""

# Opção 1: rank_bm25 (puro Python, simples)
from rank_bm25 import BM25Okapi

corpus = [doc.text.split() for doc in documents]
bm25 = BM25Okapi(corpus)
scores = bm25.get_scores(query.split())

# Opção 2: Elasticsearch/OpenSearch (produção robusta)
# Opção 3: Qdrant tem BM25 nativo (mais simples)
# Opção 4: Weaviate BM25 nativo
# Opção 5: pgvector + pgroonga (BM25 em Postgres)
```

---

## 8. Stacks por Perfil e Orçamento {#stacks}

### 🆓 Stack Gratuito / Open Source (Self-Hosted)

```yaml
# IDEAL PARA: Desenvolvimento, empresas com dados sensíveis, VPS
# CUSTO: ~$0 (só computação)

ingestao:
  PDF_textual: PyMuPDF (fitz)
  PDF_escaneado: Docling (IBM, open source)
  Excel_CSV: pandas + openpyxl
  EPUB: ebooklib + BeautifulSoup
  Markdown: MarkdownHeaderTextSplitter (LangChain)
  HTML: trafilatura + BeautifulSoup

embeddings:
  texto_geral: nomic-embed-text-v1.5
  multilingual: BGE-M3 (BAAI)
  execucao: Ollama (fácil de rodar localmente)

vector_db:
  desenvolvimento: ChromaDB
  producao: Qdrant (self-hosted Docker)
  postgres: pgvector

sparse_search: rank_bm25 ou Qdrant BM25 nativo

reranker: BAAI/bge-reranker-v2-m3 (local, excelente português)

llm:
  local: Ollama (llama3.2, qwen2.5, mistral)
  api_barata: Groq (llama3 via API, muito rápido e barato)

framework: LlamaIndex ou LangChain

cache: Redis (queries frequentes)

avaliacao: RAGAS (open source)
```

### 💰 Stack Custo-Benefício (Produção)

```yaml
# IDEAL PARA: Startups, produtos em produção com budget controlado
# CUSTO: ~$50-200/mês dependendo do volume

ingestao:
  PDF: Docling (grátis, robusto)
  complexo: LlamaParse ($0.003/página, vale para PDFs difíceis)
  tabelar: pandas

embeddings:
  principal: text-embedding-3-small (OpenAI, $0.02/1M tokens)
  fallback_local: nomic-embed-text

vector_db: Qdrant Cloud (free até 1GB, depois ~$25/mês)

hybrid_search: BM25 nativo do Qdrant + dense

reranker:
  default: ms-marco-MiniLM-L-6-v2 (local, grátis)
  alta_precisao: Cohere Rerank v3 ($0.002/1k tokens)

llm:
  rapido_barato: GPT-4o-mini ou Gemini 1.5 Flash
  alta_qualidade: GPT-4o ou Claude 3.5 Sonnet
  estrategia: mini para draft, 4o para revisão crítica

framework: LlamaIndex

cache: Redis Cloud (free tier suficiente para início)

observabilidade: LangSmith (free tier) ou Arize Phoenix

avaliacao: RAGAS
```

### 🚀 Stack Enterprise (Máxima Precisão)

```yaml
# IDEAL PARA: Empresas com requisitos críticos de precisão
# CUSTO: $500-5000+/mês dependendo do volume

ingestao:
  padrao: LlamaParse + Docling (fallback)
  financeiro: Azure Document Intelligence
  juridico: AWS Textract

embeddings:
  texto: text-embedding-3-large ou voyage-3
  multimodal: ColPali ou GPT-4o Vision

vector_db:
  principal: Qdrant Cloud (Enterprise) ou Pinecone
  analitico: pgvector no PostgreSQL

hybrid_search:
  dense: vector DB nativo
  sparse: Elasticsearch ou BM25 nativo

reranker:
  principal: Cohere Rerank v3
  validacao: BGE-Reranker-v2-M3 (cross-check)

retrieval_avancado:
  tecnica: Contextual Retrieval + Hybrid + Rerank
  opcional: GraphRAG para bases com relações complexas

llm:
  principal: Claude 3.5 Sonnet (melhor contexto longo)
  alternativa: GPT-4o
  analise: GPT-4o com code interpreter

framework: LlamaIndex Enterprise

cache: Redis Enterprise

observabilidade: LangSmith Pro ou Arize Phoenix

avaliacao:
  automatica: RAGAS
  humana: pipeline de anotação para casos críticos
  continua: drift detection em produção
```

---

## 9. Avaliação e Monitoramento {#avaliacao}

### Por Que Avaliar é Obrigatório

```
RAG SEM AVALIAÇÃO = PILOTO AUTOMÁTICO SEM INSTRUMENTOS

"Parece bom" não é métrica.
"O usuário não reclamou" não é métrica.

A survey de avaliação de RAG 2025 (arXiv) reforça:
RAG deve ser avaliado em RETRIEVAL e GENERATION separadamente.
```

### Métricas Essenciais

```python
METRICAS_RAG_2026 = {
    
    # RETRIEVAL
    "recall_at_k": {
        "o_que": "O documento certo está nos top-K recuperados?",
        "meta": "> 0.80 para K=10",
        "impacto": "Se isso está baixo, nada mais importa"
    },
    
    "mrr": {
        "o_que": "Mean Reciprocal Rank - quão alto está o doc certo?",
        "meta": "> 0.70",
        "impacto": "Mede se o melhor resultado está no topo"
    },
    
    "ndcg": {
        "o_que": "Normalized Discounted Cumulative Gain",
        "meta": "> 0.75",
        "impacto": "Considera a ordem dos resultados"
    },
    
    # GENERATION
    "faithfulness": {
        "o_que": "Resposta é fiel aos documentos recuperados?",
        "meta": "> 0.85",
        "impacto": "Detecta alucinações"
    },
    
    "answer_relevancy": {
        "o_que": "Resposta é relevante para a pergunta?",
        "meta": "> 0.80",
        "impacto": "Detecta respostas corretas mas fora do assunto"
    },
    
    "context_precision": {
        "o_que": "Proporção de contexto recuperado que é relevante?",
        "meta": "> 0.75",
        "impacto": "Ruído no contexto = resposta pior"
    },
    
    "context_recall": {
        "o_que": "Toda informação necessária foi recuperada?",
        "meta": "> 0.70",
        "impacto": "Informação faltante = resposta incompleta"
    },
    
    # SISTEMA
    "latencia_p95": {
        "o_que": "Tempo de resposta no percentil 95",
        "meta": "< 3s para chat, < 10s para análise"
    },
    
    "custo_por_query": {
        "o_que": "Custo médio por consulta",
        "meta": "Depende do negócio"
    },
    
    "taxa_abstencao": {
        "o_que": "Quando não há evidência, o sistema admite?",
        "meta": "> 0.90 (deve abster quando não sabe)"
    }
}
```

### Set de Testes Obrigatório

```python
TEST_CATEGORIES = {
    
    "lookup_exato": {
        "exemplos": [
            "Qual o preço do produto SKU-123?",
            "O que diz o artigo 5º da CF?"
        ],
        "o_que_testa": "Precisão em busca exata, BM25 principalmente"
    },
    
    "semantico": {
        "exemplos": [
            "Algo confortável para presente de criança",
            "Como funciona o processo de devolução?"
        ],
        "o_que_testa": "Qualidade do embedding e busca semântica"
    },
    
    "multi_hop": {
        "exemplos": [
            "Como a política de X se relaciona com a seção Y?",
            "Quais produtos são compatíveis com o modelo Z?"
        ],
        "o_que_testa": "Capacidade de conectar informações distantes"
    },
    
    "numerico": {
        "exemplos": [
            "Qual produto tem melhor avaliação até R$200?",
            "Qual o total de estoque da categoria A?"
        ],
        "o_que_testa": "Precisão numérica, filtros, SQL"
    },
    
    "ambiguo": {
        "exemplos": [
            "tênis" (sem mais contexto),
            "Como cancelo?" (sem especificar o quê)
        ],
        "o_que_testa": "Clarificação, fallback gracioso"
    },
    
    "impossivel": {
        "exemplos": [
            "Qual o preço do produto que não existe?",
            "Quando a empresa foi fundada?" (se não está nos docs)
        ],
        "o_que_testa": "Taxa de abstention correta"
    }
}
```

### Implementação com RAGAS

```python
from ragas import evaluate
from ragas.metrics import (
    faithfulness,
    answer_relevancy,
    context_precision,
    context_recall,
    answer_correctness
)
from datasets import Dataset

# Prepara dataset de avaliação
eval_data = {
    "question": [...],          # Perguntas
    "answer": [...],            # Respostas do seu RAG
    "contexts": [...],          # Contextos recuperados (lista de listas)
    "ground_truth": [...]       # Respostas corretas (para supervised metrics)
}

dataset = Dataset.from_dict(eval_data)

# Executa avaliação
results = evaluate(
    dataset=dataset,
    metrics=[
        faithfulness,
        answer_relevancy,
        context_precision,
        context_recall,
        answer_correctness
    ]
)

print(results)
# Output:
# faithfulness: 0.87
# answer_relevancy: 0.83
# context_precision: 0.79
# context_recall: 0.74
# answer_correctness: 0.81
```

---

## 10. Resumo Executivo Final {#resumo}

### A Ordem de Prioridade Real

```
Se sua meta é PRECISÃO, invista nesta ordem:

1º  Qualidade do parsing (Docling, PyMuPDF, pandas)
    "Lixo entra, lixo sai. Sem negociação."

2º  Modelagem certa por tipo de dado
    "Produto ≠ Livro ≠ Planilha. Cada um tem seu pipeline."

3º  Hybrid Retrieval (BM25 + Dense + RRF)
    "Quase sempre o default. Cobre semântico E exato."

4º  Reranking (cross-encoder)
    "Maior salto de qualidade por menor custo."

5º  Chunking contextualizado
    "Contextual Retrieval para máxima precisão,
     Late Chunking para equilíbrio."

6º  Prompt engineering
    "Vem por último, não por primeiro."
```

### Tabela de Decisão Final

```
┌──────────────────────────────────────────────────────────────────────┐
│                    DECISÃO RÁPIDA POR SITUAÇÃO                       │
├─────────────────────────┬────────────────────────────────────────────┤
│ SITUAÇÃO                │ MELHOR ESTRATÉGIA                          │
├─────────────────────────┼────────────────────────────────────────────┤
│ Texto pequeno           │ Full Context (sem RAG)                     │
│ FAQ / página de ajuda   │ RAG mínimo + full context se couber        │
├─────────────────────────┼────────────────────────────────────────────┤
│ Markdown / docs técnicas│ Split por heading + hybrid + rerank        │
│ Wiki / manual interno   │ Metadata de path + expansão de contexto   │
├─────────────────────────┼────────────────────────────────────────────┤
│ Livro / relatório longo │ Parent-Child + hybrid + rerank             │
│ Perguntas complexas     │ RAPTOR ou hierárquico                      │
│ Máxima precisão         │ Contextual Retrieval + hybrid + rerank     │
│ Equilíbrio custo/prec.  │ Late Chunking + hybrid + rerank            │
├─────────────────────────┼────────────────────────────────────────────┤
│ PDF textual limpo        │ PyMuPDF + chunking semântico + hybrid     │
│ PDF escaneado/complexo  │ Docling + hybrid + rerank                  │
│ PDF com tabelas/gráficos│ Multimodal page→element retrieval          │
├─────────────────────────┼────────────────────────────────────────────┤
│ CSV lookup semântico    │ Linha → documento canônico + hybrid        │
│ CSV analítico           │ Text-to-SQL / DuckDB / Pandas Agent        │
├─────────────────────────┼────────────────────────────────────────────┤
│ Excel lista/cadastro    │ Mesmo que CSV lookup                       │
│ Excel relatório         │ Blocos semânticos + parent-child           │
│ Excel analítico         │ SQL/DuckDB (não RAG textual)               │
├─────────────────────────┼────────────────────────────────────────────┤
│ Catálogo de produtos    │ SKU como entidade + hybrid (BM25 forte)    │
│ Autopeças / SKUs        │ Aliases + compatib. + rerank sempre        │
└─────────────────────────┴────────────────────────────────────────────┘
```

### O Default Que Funciona para 80% dos Casos

```
Se você não sabe por onde começar, use isto:

┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. Docling para parsear                                        │
│  2. Chunk por estrutura (heading/parágrafo), 300-700 tokens    │
│  3. Metadata rica (fonte, seção, data, tipo)                   │
│  4. Contextual enrichment em cada chunk                         │
│  5. Hybrid search (BM25 + dense, RRF)                          │
│  6. Reranker (BGE-Reranker-v2-M3 local ou Cohere)             │
│  7. Expansão para parent doc ou chunks vizinhos                 │
│  8. LLM com instrução de citar fonte e abster se não souber    │
│  9. RAGAS para avaliar e iterar                                 │
│                                                                 │
│  Isso resolve a maioria dos casos com precisão alta.           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### O Que Evitar (Anti-Patterns Comuns)

```
❌ Chunk fixo de 512 tokens sem overlap em qualquer documento
❌ Dense-only (sem BM25) em produção
❌ Sem reranker "para economizar"
❌ Embedar linha de CSV bruta para queries analíticas
❌ Multimodal em tudo sem necessidade real
❌ HyDE como default universal (pode piorar em texto+tabela)
❌ CRAG como default universal (ganhos moderados, custo extra)
❌ Resolver tudo no prompt sem cuidar do retrieval
❌ RAG sem avaliação quantitativa
❌ Ignorar abstention ("dizer que não sabe")
```

---

> **Referências principais:** arXiv T2-RAGBench (2026), Anthropic Contextual Retrieval, Microsoft GraphRAG, arXiv RAPTOR, arXiv Late Chunking, ColPali, AWS RAG Best Practices, Azure Hybrid Search, Weaviate Hybrid Search, Anyscale Structured Data RAG, Pinecone Chunking Strategies, OpenAI file_search, LangChain ParentDocumentRetriever, Docling, arXiv RAG Evaluation Survey (2025), arXiv Multimodal RAG Survey (2025).
