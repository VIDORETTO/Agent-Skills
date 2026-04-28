---
name: rag-implementation
description: >
  Use esta skill SEMPRE que o usuario mencionar RAG, retrieval, embeddings, vector database,
  busca semantica, upload de arquivos com IA, indexacao de documentos, ou quiser adicionar
  capacidade de consulta inteligente a documentos em qualquer aplicacao.

  Esta skill suporta DOIS modos distintos:

  MODO 1 - RAG DIRETO: O usuario quer fazer RAG em um arquivo especifico que ele ja tem
  (ex: "quero fazer RAG nesse PDF", "como indexar esse CSV"). O agente analisa o projeto
  e o(s) arquivo(s) e cria um plano de implementacao personalizado.

  MODO 2 - SISTEMA DE UPLOAD DINAMICO: O usuario quer construir um sistema onde
  usuarios finais fazem upload de arquivos na aplicacao e a IA decide como processar cada
  um. O usuario final NAO entende de RAG, apenas faz o upload. A IA toma todas as
  decisoes tecnicas. O agente cria um plano de pipeline adaptativo completo.

  Se o usuario nao especificar qual modo quer, PERGUNTE antes de prosseguir.
  Esta skill deve ser acionada mesmo com pedidos vagos como "quero que minha app consulte
  documentos", "implementa RAG aqui", "quero busca inteligente nos meus arquivos".
metadata:
  short-description: Planeja e implementa RAG com pipelines adaptativos e referencias por tipo de arquivo
---

# RAG Implementer

Skill para analisar projetos e criar planos de implementacao RAG baseados nas melhores
praticas de 2025/2026 da documentacao de referencia.

> **Fonte de verdade:** `https://github.com/VIDORETTO/Agent-Skills/blob/main/rag-best-practices/SKILL.md`
> Consulte `references/file_types.md` para a matriz de decisao por tipo de arquivo.

---

## PASSO 0 - Identificar o Modo de Operacao

Antes de qualquer analise, determine qual modo o usuario quer:

```text
MODO 1 - RAG DIRETO
  Sinais: "quero fazer RAG nesse arquivo", "indexa esse PDF", arquivo especifico mencionado
  Acao: Va para o Passo 1A

MODO 2 - SISTEMA DE UPLOAD DINAMICO
  Sinais: "usuarios vao fazer upload", "quero um sistema de upload com IA",
          "a aplicacao deve processar arquivos que os usuarios enviam"
  Acao: Va para o Passo 1B

INCERTO
  Acao: Pergunte ao usuario:
  "Voce quer:
   (A) Fazer RAG em um arquivo ou conjunto de arquivos especificos que voce ja tem?
   (B) Construir um sistema onde seus usuarios fazem upload de qualquer tipo de arquivo
       e a IA decide automaticamente como processar cada um?"
```

---

## PASSO 1A - RAG Direto: Entender Projeto e Arquivo(s)

### 1A.1 Analise do Projeto

Examine o projeto do usuario:

```bash
# Detectar stack tecnica
ls -la
cat package.json 2>/dev/null || cat requirements.txt 2>/dev/null || cat Cargo.toml 2>/dev/null
# Ver estrutura geral
find . -type f -name "*.py" -o -name "*.ts" -o -name "*.js" | head -30
```

Identifique:
- **Stack:** Python/Node/outro
- **Framework:** FastAPI, Next.js, Django, Express
- **Banco de dados existente:** PostgreSQL, MongoDB, Redis
- **Infraestrutura:** Local, AWS, GCP, Azure, Vercel
- **Ja tem algum LLM integrado?** OpenAI, Anthropic, Ollama

### 1A.2 Analise dos Arquivos

Para cada arquivo alvo, detecte:
- **Tipo real** (nao confie so na extensao)
- **Tamanho** (cabe no contexto do LLM? `< 200k tokens` -> avaliar se RAG e necessario)
- **Estrutura interna** (headers, tabelas, codigo, texto corrido)
- **Tipo de queries esperadas** (semanticas, analiticas, lookup exato)

Consulte `references/file_types.md` para a matriz completa de decisao por tipo de arquivo.

### 1A.3 Verificacao de Necessidade de RAG

```python
# Antes de recomendar RAG, verifique:
def avaliar_necessidade_rag(tokens_doc, frequencia_query, tipo_query):
    if tokens_doc < 100_000:
        return "CONSIDERAR: Full Context pode ser mais simples e preciso que RAG"
    if tokens_doc < 200_000 and frequencia_query == "alta":
        return "AVALIAR: Prompt Caching pode vencer RAG em custo"
    if tipo_query in ["soma", "contagem", "top_n", "media"]:
        return "ALTERNATIVA: Text-to-SQL ou Pandas Agent pode ser melhor"
    return "RAG: necessario e adequado"
```

### 1A.4 Criar Plano de Implementacao

Com base na analise, crie um plano estruturado:

````markdown
## Plano RAG - [Nome do Projeto]

### Diagnostico
- Stack: [detectado]
- Arquivo(s): [tipo + tamanho]
- Queries esperadas: [semanticas/analiticas/lookup]
- Recomendacao principal: [RAG / Full Context / SQL Agent]

### Pipeline Recomendado
[Baseado no tipo do arquivo - ver references/file_types.md]

### Componentes
- Parser: [qual e por que]
- Chunking: [estrategia + tamanho + overlap]
- Embeddings: [modelo recomendado]
- Vector DB: [recomendado para o stack do projeto]
- Retrieval: [Hybrid BM25+Dense / so dense / SQL]
- Reranker: [se necessario]
- LLM: [recomendado]

### Implementacao
[Codigo adaptado ao stack do projeto]

### Custo estimado por query
[Baseado no pipeline escolhido]
````

---

## PASSO 1B - Sistema de Upload Dinamico: Pipeline Adaptativo

Neste modo, o **usuario final nao entende de RAG**. Ele apenas faz upload de arquivos.
A IA deve tomar todas as decisoes silenciosamente.

### 1B.1 Entender a Aplicacao

Analise o projeto:
- Stack e framework
- Onde arquivos serao armazenados (S3, local, GCS)
- Banco de dados disponivel
- Orcamento e latencia aceitaveis

### 1B.2 Pipeline de Deteccao Automatica

O sistema deve detectar o tipo real de cada arquivo e aplicar o pipeline correto:

```python
# Fluxo de decisao automatico (nunca pergunta ao usuario)
def detect_and_route(file_path: str, file_content: bytes) -> str:
    """
    Detecta tipo real do arquivo (nao confia so na extensao)
    e retorna a rota de processamento correta.
    """
    extension = Path(file_path).suffix.lower()
    mime = magic.from_buffer(file_content, mime=True)

    # Matriz de decisao
    if mime in ["application/pdf"]:
        return route_pdf(file_content)  # analisa se e textual/visual/escaneado

    elif extension in [".md", ".mdx"]:
        return "markdown_pipeline"

    elif extension in [".csv", ".tsv"]:
        return route_csv(file_content)  # semantico vs SQL

    elif extension in [".xlsx", ".xls"]:
        return route_excel(file_content)

    elif extension in [".txt"]:
        tokens = estimate_tokens(file_content)
        if tokens < 100_000:
            return "full_context"  # sem RAG
        return "text_rag_pipeline"

    elif mime.startswith("image/"):
        return "vision_pipeline"

    else:
        return "generic_text_pipeline"

def route_pdf(content: bytes) -> str:
    """PDF tem subtipos - detecta automaticamente"""
    text_ratio = extract_text_ratio(content)
    has_tables = detect_tables(content)
    is_scanned = text_ratio < 0.3

    if is_scanned:
        return "pdf_ocr_pipeline"         # Docling com OCR
    elif has_tables:
        return "pdf_structured_pipeline"  # Docling + table extraction
    else:
        return "pdf_text_pipeline"        # PyMuPDF rapido
```

### 1B.3 Pipelines por Tipo de Arquivo

Para cada pipeline, consulte `references/file_types.md`. Resumo:

| Arquivo | Pipeline | Parser | Chunking | Retrieval |
|---------|----------|--------|----------|-----------|
| PDF textual | Semantico | PyMuPDF | 350-800 tok | Hybrid + Rerank |
| PDF escaneado | Multimodal | Docling+OCR | Por pagina | Hybrid + Rerank |
| Markdown/Docs | Hierarquico | MarkdownSplitter | Por heading | Hybrid + Rerank |
| Livro longo | Parent-Child | Docling | 400 child/2000 parent | Hybrid + Rerank |
| CSV semantico | Doc por linha | pandas | 1 linha = 1 doc | Hybrid |
| CSV analitico | SQL Agent | pandas | - | Text-to-SQL |
| XLSX | Hibrido | openpyxl | Por aba/tabela | SQL + Semantico |

### 1B.4 Plano de Implementacao do Sistema

````markdown
## Plano - Sistema de Upload RAG Adaptativo

### Arquitetura Geral
[Diagrama do fluxo: Upload -> Deteccao -> Pipeline -> Index -> Query]

### Componente de Deteccao (sempre automatico)
[Codigo de deteccao adaptado ao stack]

### Pipelines Implementados
Para cada tipo de arquivo detectavel:
- [tipo]: parser + chunking + metadata + retrieval

### API de Upload
[Endpoint que recebe arquivo, detecta tipo, processa, indexa]

### API de Query
[Endpoint que recebe query, busca, rerank, responde]

### Infraestrutura
[Vector DB, armazenamento de arquivos, configuracoes]

### Decisoes Tomadas Automaticamente
- Chunking: adaptado por tipo
- Embeddings: modelo unico ou multi-modelo
- Retrieval: semantico vs SQL por tipo de pergunta
- Citacao: sempre indica fonte e localizacao
````

---

## PASSO 2 - Aplicar Melhores Praticas Obrigatorias

**Independente do modo, sempre aplique:**

### Hybrid Retrieval (padrao vencedor 2026)

```text
BM25 (keyword) + Dense (embeddings) -> RRF Fusion -> Reranker
Resultado: melhor precisao que qualquer metodo isolado
```

### Contextual Enrichment

```python
# Antes de embedar, enriqueza cada chunk com contexto:
chunk_enriquecido = f"""
Documento: {doc_title}
Secao: {section_path}
Conteudo: {chunk_content}
"""
# Isso reduz erros de retrieval ao preservar o contexto semantico do trecho
```

### Metadata Rica

```python
metadata = {
    "source_file": "nome_do_arquivo.pdf",
    "section": "Capitulo 3 > Secao 2.1",
    "page": 42,
    "doc_type": "relatorio",
    "created_at": "2024-01-15",
    "chunk_index": 7,
    "total_chunks": 45
}
```

### Abstention (nao alucinar)

```python
# O LLM deve sempre declarar quando nao tem evidencia:
system_prompt = """
Responda APENAS com base nos trechos fornecidos.
Se a informacao nao estiver nos trechos, diga:
"Nao encontrei essa informacao nos documentos disponiveis."
Sempre cite a fonte (arquivo + secao + pagina).
"""
```

---

## PASSO 3 - Implementar

Com o plano aprovado (ou diretamente, no caso do Modo 2), implemente:

1. **Instalar dependencias** adequadas ao stack
2. **Criar parser** especifico para o(s) tipo(s) de arquivo
3. **Criar pipeline de chunking** com metadata rica
4. **Configurar Vector DB** preferencialmente usando o que ja existe no projeto
5. **Implementar hybrid retrieval** (BM25 + Dense)
6. **Adicionar reranker** (ms-marco-MiniLM local ou Cohere Rerank)
7. **Criar endpoint de ingestao** (upload -> process -> index)
8. **Criar endpoint de query** (query -> hybrid search -> rerank -> LLM -> resposta com citacao)
9. **Testar** com exemplos reais do usuario

---

## Referencias

- `references/file_types.md` - Matriz completa de decisao por tipo de arquivo (PDF, Markdown, CSV, XLSX, livros, texto)
- **Fonte primaria:** `https://github.com/VIDORETTO/Agent-Skills/blob/main/rag-best-practices/SKILL.md`

Se no futuro forem adicionadas outras referencias locais, mantenha o `SKILL.md` apontando
apenas para arquivos que realmente existam dentro desta skill.

---

## Principios Fundamentais

```text
NAO FACA:
- Jogar documento bruto no vector DB sem parsing adequado
- Usar so dense retrieval (sem BM25)
- Embedar markdown de tabela bruta
- Chunking por caractere fixo em Markdown
- Recomendar RAG quando full context basta
- Perguntar ao usuario (Modo 2) sobre estrategias tecnicas

SEMPRE FACA:
- Parse primeiro, indexa depois
- Hybrid retrieval (BM25 + Dense)
- Reranker cross-encoder
- Metadata rica em todos os chunks
- Contextual enrichment antes de embedar
- Citacao da fonte na resposta
- Abstention quando nao ha evidencia
```
