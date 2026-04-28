# Referencia: Pipeline RAG por Tipo de Arquivo

> Baseado em: `https://github.com/VIDORETTO/Agent-Skills/blob/main/rag-best-practices/SKILL.md`
> Data de referencia: 08/04/2026

---

## Matriz de Decisao Rapida

| Tipo | Tamanho | Query | Pipeline | Parser | Chunking | Embedding | Retrieval |
|------|---------|-------|----------|--------|----------|-----------|-----------|
| Texto/FAQ | < 100k tok | Qualquer | Full Context | direto | nenhum | - | - |
| Texto/FAQ | > 100k tok | Semantica | RAG Leve | markdown-it/txt | 128-256 tok | text-embedding-3-small | Hybrid |
| Markdown/Docs | Qualquer | Semantica | Hierarquico | MarkdownSplitter | Por heading | text-embedding-3-small | Hybrid+Rerank |
| Livro/Doc Longo | Grande | Simples | Parent-Child | Docling | 400 child/2000 parent | text-embedding-3-large | Hybrid+Rerank |
| Livro/Doc Longo | Grande | Multi-hop | RAPTOR | Docling | Clusters | BGE-M3 | Hybrid+Rerank |
| PDF Textual | Qualquer | Semantica | Semantico | PyMuPDF | 350-800 tok | text-embedding-3-small | Hybrid+Rerank |
| PDF Escaneado | Qualquer | Qualquer | Multimodal | Docling+OCR | Por pagina | ColPali/Vision | Hybrid+Rerank |
| PDF Visual | Qualquer | Qualquer | Page->Element | Docling | Pagina+Elemento | ColPali | Hybrid+Rerank |
| CSV | Pequeno | Semantica | Doc por linha | pandas | 1 linha = 1 doc | text-embedding-3-small | Hybrid |
| CSV | Grande | Analitica | SQL Agent | pandas | - | - | Text-to-SQL |
| XLSX | Simples | Semantica | Por aba | openpyxl | Por aba | text-embedding-3-small | Hybrid |
| XLSX | Complexo | Misto | Hibrido | openpyxl | Tabela+Contexto | text-embedding-3-small | SQL+Semantico |

---

## A. Texto Pequeno / FAQ

**Quando usar:** FAQ, pagina de ajuda, politica da empresa, notas de versao curtas

### Regra de Ouro
Se cabe no contexto do LLM (< 100k tokens ~= ~300 paginas), **mande tudo**. Nao complique.

```python
# Decisao automatica
def strategy_faq(doc_tokens: int, n_docs: int) -> str:
    if doc_tokens < 100_000:
        return "FULL_CONTEXT"  # sem RAG, usa prompt caching
    if n_docs > 100 and doc_tokens < 500:
        return "RAG_MINIMO"    # 1 chunk por doc, sem quebrar
    return "RAG_LEVE"          # chunking 128-256 tok
```

### Configuracao
```yaml
chunking:
  tamanho: 128-256 tokens
  overlap: 0-40 tokens
  metodo: paragrafo_ou_documento_inteiro

embeddings: text-embedding-3-small
vector_db: ChromaDB (local) ou pgvector (se ja tem Postgres)
retrieval: Hybrid BM25 + Dense
reranker: opcional (ms-marco-MiniLM se docs muito parecidos)
llm: GPT-4o-mini ou Gemini Flash
custo: ~$0.001-0.003/query
```

---

## B. Markdown / Docs Tecnicas / Wikis

**Quando usar:** Documentacao de produto, tutoriais, manuais internos, READMEs, wikis

### Regra de Ouro
**Markdown tem estrutura. Respeite ela.** Chunking por caractere em Markdown = erro grave.
A hierarquia de headers e o mapa semantico. Use sempre `MarkdownHeaderTextSplitter`.

### Implementacao
```python
from langchain.text_splitter import MarkdownHeaderTextSplitter

headers = [("#", "h1"), ("##", "h2"), ("###", "h3"), ("####", "h4")]
splitter = MarkdownHeaderTextSplitter(
    headers_to_split_on=headers,
    strip_headers=False  # IMPORTANTE: mantem header no chunk
)
chunks = splitter.split_text(markdown_content)

# Contextual enrichment: prefixar com path completo
for chunk in chunks:
    path = " > ".join(filter(None, [
        chunk.metadata.get("h1", ""),
        chunk.metadata.get("h2", ""),
        chunk.metadata.get("h3", ""),
    ]))
    chunk.page_content = f"[{path}]\n\n{chunk.page_content}"
```

### Preservar Intacto
- Blocos de codigo (```...```)
- Tabelas markdown completas
- Listas numeradas (nunca quebrar no meio)
- Blocos YAML/JSON/frontmatter

### Configuracao
```yaml
chunking:
  metodo: por_heading_hierarquico (H1 > H2 > H3)
  tamanho_alvo: 300-700 tokens
  overflow: split_semantico (se secao muito grande)
  overlap: 50-120 tokens
  preservar: [code_blocks, tables, numbered_lists]

metadata:
  - file_name, file_path
  - section_path
  - h1, h2, h3, h4
  - version, language, last_modified

retrieval:
  tipo: hybrid (BM25 + dense)
  top_k_candidatos: 20-50
  rerank: sim (cross-encoder/ms-marco-MiniLM-L-6-v2)
  top_k_final: 5-10

custo: ~$0.002-0.008/query
nota: MELHOR custo-beneficio de todos os tipos
```

---

## C. Livro / Documento Longo / Contrato / Apostila

**Quando usar:** Livros, relatorios anuais, regulamentos completos, apostilas

### Regra de Ouro
**"Recupere pequeno. Leia grande."**
Busque com chunks granulares (filhos). Entregue ao LLM o contexto expandido (pais).

### Parent-Child (padrao para a maioria dos casos)
```python
from langchain.retrievers import ParentDocumentRetriever
from langchain.text_splitter import RecursiveCharacterTextSplitter

parent_splitter = RecursiveCharacterTextSplitter(
    chunk_size=2000,
    chunk_overlap=150
)
child_splitter = RecursiveCharacterTextSplitter(
    chunk_size=400,
    chunk_overlap=50
)

retriever = ParentDocumentRetriever(
    vectorstore=vector_db,
    docstore=docstore,
    child_splitter=child_splitter,
    parent_splitter=parent_splitter,
)
```

### RAPTOR (para perguntas multi-hop / sintese de multiplas secoes)
```text
Chunks -> Clusterizacao (UMAP+GMM) -> Resumo de clusters (LLM) ->
Clusterizacao de resumos -> Resumo de resumos -> Arvore hierarquica

Query abstrata -> bate nos resumos do topo
Query especifica -> bate nos chunks originais
```

**Custo:** Alto na indexacao (muitas chamadas LLM)
**Quando usar:** Perguntas que sintetizam multiplas secoes distantes

### Arvore de Decisao
```text
Perguntas simples e diretas? -> Parent-Child + Hybrid + Rerank
Perguntas complexas/multi-hop? -> RAPTOR ou Hierarquico
Equilibrio custo/precisao? -> Late Chunking + Hybrid + Rerank
Precisao maxima? -> Contextual Retrieval + Hybrid + Rerank
```

### Configuracao
```yaml
indices:
  filho (busca):
    chunk_size: 300-600 tokens
    overlap: 60-120 tokens
  pai (contexto ao LLM):
    chunk_size: 1500-3000 tokens
    overlap: 150 tokens

metadata:
  - titulo, capitulo, secao, subsecao
  - numero_pagina
  - posicao_relativa
  - doc_title, doc_type, doc_date
  - referencias_cruzadas

retrieval:
  etapa_1: busca no indice de capitulo/secao
  etapa_2: busca no indice de chunk interno
  top_k: 20-50 filhos
  rerank: obrigatorio
  top_k_final: 5-8 pais

stack:
  parser: Docling (melhor para estrutura)
  embeddings: text-embedding-3-large ou BGE-M3
  vector_db: Qdrant (melhor suporte parent-child + filtros)
  reranker: Cohere Rerank v3 (producao) ou BGE-Reranker-v2-M3 (local)
  llm: Claude 3.5 Sonnet (melhor para contexto longo)

custo: ~$0.01-0.05/query
```

---

## D. PDF Limpo / Textual

**Quando usar:** PDF exportado digitalmente, artigo cientifico textual, contrato simples

### Regra de Ouro
**Parse bem primeiro. Depois indexa.** Indexar PDF bruto sem parsing = maior causa de baixa precisao.

### Deteccao Automatica de Tipo de PDF
```python
def detect_pdf_type(pdf_path: str) -> str:
    doc = fitz.open(pdf_path)
    total_text = "".join(page.get_text() for page in doc)
    text_ratio = len(total_text) / (doc.page_count * 500 + 1)
    has_tables = detect_tables(doc)

    if text_ratio > 0.7 and not has_tables:
        return "textual_limpo"
    elif text_ratio < 0.3:
        return "escaneado"
    elif has_tables:
        return "estruturado"
    else:
        return "misto"
```

### Configuracao
```yaml
parser_por_tipo:
  textual_limpo: PyMuPDF (fitz) - gratis, rapido
  escaneado: Docling com OCR
  com_tabelas: Docling ou pdfplumber
  cientifico_formulas: Nougat (Meta)
  complexo_pago: LlamaParse ou Azure Document Intelligence

chunking:
  tamanho: 350-800 tokens
  overlap: 60-150 tokens
  split_por: secao/titulo/pagina
  preservar: [tabelas_completas, listas, blocos_codigo]

metadata:
  - page_number
  - section_title
  - table_ids
  - doc_title, doc_date, doc_type

retrieval:
  tipo: hybrid
  top_k: 20-40
  rerank: sim
  top_k_final: 5-8

resposta:
  citar: pagina + secao
  abstain: quando evidencia insuficiente
```

---

## E. PDF Visual / Escaneado / Complexo

**Quando usar:** Notas fiscais, apresentacoes, papers com graficos, formularios, diagramas

### Quando Usar Multimodal
```text
USE MULTIMODAL:
- A resposta depende de posicao visual no documento
- OCR perde info critica (tabelas complexas, graficos)
- Diagramas, fluxogramas, mapas sao parte da resposta
- Documento e escaneado sem OCR de qualidade

NAO USE QUANDO:
- PDF e basicamente texto corrido
- Custo e restricao importante
- As tabelas sao simples e OCR captura bem
```

### Pipeline Page->Element
```python
# Etapa 1: Recuperar PAGINA relevante
# Etapa 2: Recuperar ELEMENTO dentro da pagina (tabela/figura/bloco)

class MultimodalRAG:
    def index_document(self, pdf_path):
        for page_num, page in enumerate(doc):
            pix = page.get_pixmap(matrix=fitz.Matrix(2, 2))
            img = pix.tobytes("png")

            page_embedding = vision_model.embed_image(img)

            for element in extract_elements(page):
                elem_embedding = vision_model.embed_element(element)
                element_index.add(elem_embedding, metadata={
                    "page": page_num,
                    "type": element["type"],
                    "caption": element.get("caption", ""),
                })
```

### Configuracao
```yaml
pipeline:
  renderizacao: 2x resolucao
  indice_pagina:
    embedding: ColPali ou vision model
    granularidade: pagina inteira
  indice_elemento:
    embedding: por tabela/figura/bloco
    granularidade: elemento
  indice_texto:
    embedding: OCR estruturado via Docling

ferramentas:
  colpali: retrieval visual state-of-art (gratuito)
  docling: OCR + layout extraction
  azure_di: Document Intelligence (pago, robusto)
  llamaparse: parsing avancado (pago)
```

---

## F. CSV

**Quando usar:** Lista de produtos, pedidos, clientes, estoque, tabela de precos

### Regra de Ouro
**CSV raramente deve ser tratado como texto puro.**
A decisao correta depende do TIPO DE PERGUNTA, nao do arquivo.

### Arvore de Decisao
```text
Query semantica/vaga?
  "me fale sobre o produto X", "qual item e bom para Y?"
  -> RAG estruturado (documento por linha)

Query lookup exato?
  "qual o produto com codigo ABC123?"
  -> RAG estruturado + filtro por metadata

Query analitica/agregacao?
  "soma do estoque", "faturamento por mes", "top 5 produtos"
  -> TEXT-TO-SQL ou PANDAS AGENT (nao RAG textual)

Query mista?
  -> Roteador automatico que detecta tipo de query
```

### RAG por Linha (para queries semanticas)
```python
import pandas as pd

def csv_to_documents(csv_path: str, text_columns: list, id_column: str) -> list:
    df = pd.read_csv(csv_path)
    documents = []

    for _, row in df.iterrows():
        text_parts = []
        for col in text_columns:
            if pd.notna(row[col]):
                text_parts.append(f"{col}: {row[col]}")

        text = "\n".join(text_parts)
        metadata = {col: row[col] for col in df.columns}
        metadata["source"] = csv_path
        metadata["row_id"] = row[id_column]

        documents.append({"text": text, "metadata": metadata})

    return documents
```

### Text-to-SQL (para queries analiticas)
```python
import sqlite3

def setup_sql_agent(csv_path: str, table_name: str):
    df = pd.read_csv(csv_path)
    conn = sqlite3.connect(":memory:")
    df.to_sql(table_name, conn, index=False)
    return conn
```

### Configuracao
```yaml
semantico:
  chunking: 1 linha = 1 documento
  overlap: nenhum
  metadata: todas as colunas como filtros
  retrieval: Hybrid + filtros por metadata

analitico:
  approach: Text-to-SQL ou Pandas Agent
  tool: SQLDatabaseChain (LangChain) ou pandasai

misto:
  roteador: detecta tipo de query antes de decidir
```

---

## G. XLSX / Excel

**Quando usar:** Planilhas financeiras, relatorios, catalogos, dados empresariais

### Arvore de Decisao
```text
Planilha simples (1 aba, dados homogeneos)?
  -> CSV pipeline (converter e tratar como CSV)

Multiplas abas com contexto relacionado?
  -> Uma colecao de documentos (1 doc por aba)

Aba com dados numericos/financeiros?
  -> Text-to-SQL (nao RAG semantico)

Aba com descricoes, comentarios, texto rico?
  -> RAG semantico por linha

Misto (algumas abas numericas, outras textuais)?
  -> Pipeline hibrido com roteador por aba
```

### Implementacao
```python
import openpyxl
from typing import list

def excel_to_documents(xlsx_path: str) -> list:
    wb = openpyxl.load_workbook(xlsx_path, data_only=True)
    documents = []

    for sheet_name in wb.sheetnames:
        ws = wb[sheet_name]
        sheet_type = classify_sheet(ws)

        if sheet_type == "numerico":
            df = pd.read_excel(xlsx_path, sheet_name=sheet_name)
            setup_sql_table(df, sheet_name)

        elif sheet_type == "textual":
            for row in ws.iter_rows(min_row=2, values_only=True):
                text = format_row_as_text(ws, row)
                documents.append({
                    "text": text,
                    "metadata": {
                        "sheet": sheet_name,
                        "source": xlsx_path
                    }
                })

    return documents

def classify_sheet(ws) -> str:
    numeric_count = sum(
        1 for row in ws.iter_rows(min_row=2, values_only=True)
        for cell in row if isinstance(cell, (int, float))
    )
    total_cells = ws.max_row * ws.max_column
    return "numerico" if numeric_count / total_cells > 0.6 else "textual"
```

---

## Stacks por Orcamento

### Open Source / Local
```yaml
parser: PyMuPDF + Docling
embeddings: nomic-embed-text ou BGE-M3 (Ollama)
vector_db: ChromaDB ou pgvector
reranker: ms-marco-MiniLM-L-6-v2 (local)
llm: Llama 3.1 70B ou Mistral (Ollama)
custo_operacional: $0 (so hardware)
```

### Startup / Custo Equilibrado
```yaml
parser: PyMuPDF + Docling
embeddings: text-embedding-3-small (OpenAI)
vector_db: Qdrant Cloud ou Supabase pgvector
reranker: ms-marco-MiniLM (local)
llm: GPT-4o-mini ou Claude Haiku
custo_operacional: ~$10-100/mes
```

### Producao / Alta Precisao
```yaml
parser: LlamaParse ou Azure Document Intelligence
embeddings: text-embedding-3-large ou Cohere Embed v3
vector_db: Pinecone ou Qdrant Cloud
reranker: Cohere Rerank v3
llm: Claude 3.5 Sonnet ou GPT-4o
custo_operacional: ~$100-1000+/mes
```

---

## Verificacao de Qualidade

### Metricas para Avaliar o RAG
```python
METRICAS = {
    "faithfulness": "Resposta e fiel aos trechos recuperados? (sem alucinacao)",
    "answer_relevancy": "Resposta responde a pergunta?",
    "context_precision": "Trechos recuperados sao relevantes?",
    "context_recall": "Toda info necessaria foi recuperada?",
}

# Frameworks: RAGAS, LangSmith, Arize Phoenix
```

### Sinais de Pipeline Ruim
```text
Muitas respostas "nao encontrei informacao" para queries validas -> retrieval fraco
Respostas com informacoes incorretas sobre os docs -> parsing ou chunking ruim
Respostas lentas (>5s) -> reranker pesado ou top_k muito alto
Respostas que misturam contexto de docs diferentes -> metadados ruins
```
