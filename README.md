# RAG Implementation with Pinecone and Hybrid Search

## Overview
This project implements a Retrieval-Augmented Generation (RAG) system using Pinecone for vector storage and hybrid search capabilities. The system is specifically optimized for processing and querying financial statements, combining both semantic and keyword-based search approaches.

## System Architecture

### High-Level Flow
```mermaid
graph TD
    A[PDF Document] --> B[LlamaParse]
    B --> C[Markdown Conversion]
    C --> D[Text Chunking]
    D --> E[Document Embedder]
    E --> F[Pinecone Vector Store]
    
    G[User Query] --> H[Hybrid Searcher]
    H --> I[Vector Search]
    H --> J[BM25 Search]
    I --> K[Score Combination]
    J --> K
    K --> L[Context Selection]
    L --> M[LLM Response]
```

### Document Processing Pipeline
```mermaid
flowchart LR
    A[Raw PDF] --> B[LlamaParse]
    B --> C[Markdown Text]
    C --> D[Chunk Generator]
    D --> E[Metadata Extractor]
    E --> F[Embedding Generator]
    
    subgraph Storage
        F --> G[(Pinecone DB)]
    end
```

### Search Flow
```mermaid
sequenceDiagram
    participant U as User
    participant Q as Query Processor
    participant V as Vector Search
    participant B as BM25 Search
    participant S as Score Combiner
    participant D as Pinecone DB
    
    U->>Q: Submit Query
    Q->>V: Generate Query Embedding
    Q->>B: Process Query Text
    
    par Vector Search
        V->>D: Search Vectors
        D->>V: Return Similar Docs
    and BM25 Search
        B->>D: Keyword Search
        D->>B: Return Matching Docs
    end
    
    V->>S: Vector Scores
    B->>S: BM25 Scores
    S->>U: Combined Results
```

## Key Components

### Document Processing
- LlamaParse integration for PDF to markdown conversion
- Preservation of financial table formatting
- Maintained document hierarchy and structure

```python
document = LlamaParse(result_type="markdown").load_data("financial_statement.pdf")
```

### Document Chunking and Structure
After processing, documents are chunked into the following structure:

```json
{
  "metadata": {
    "last_processed": "2025-02-13 15:35:57",
    "processed_by": "financial-docs-processor",
    "version": "1.0",
    "document_type": "financial_statements"
  },
  "chunks": [
    {
      "id": "section_0_chunk_0",
      "type": "section",
      "content": "## INFOSYS LIMITED AND SUBSIDIARIES Condensed Consolidated Financial Statements under",
      "metadata": {
        "section_title": "## INFOSYS LIMITED AND SUBSIDIARIES",
        "section_type": "financial_statement",
        "sequence": 0
      }
    }
  ]
}
```

### Document Embedding
- Model: SentenceTransformer (all-mpnet-base-v2)
- Normalized embedding generation
- Comprehensive metadata storage
- Sequential document tracking

### Hybrid Search System

#### Vector Search
- Semantic similarity matching
- Embedding-based retrieval
- Cosine similarity metrics

#### BM25 Search
- Statistical keyword matching
- Precise numerical query handling
- Traditional information retrieval

#### Score Combination Algorithm
```python
hybrid_scores = (hybrid_weight * normalized_vector_scores + 
                (1 - hybrid_weight) * normalized_bm25_scores)
```

### Metadata Structure
```json
{
    "content": "document content",
    "section_title": "section name",
    "section_type": "financial_statement",
    "sequence": "document order"
}
```

## Configuration

### Search Parameters
- `k`: Number of retrieved results
- `hybrid_weight`: Vector vs BM25 balance (default: 0.7)
- `similarity_threshold`: Minimum inclusion score (0.3)

## Usage

### Basic Search Example
```python
searcher = HybridSearcher()
results = searcher.search(
    query="What was the revenue growth in Q3?",
    k=3,
    hybrid_weight=0.7
)
```

## Performance Optimizations
- Optimized batch processing
- Score normalization implementation
- BM25 computation caching
- Result deduplication

### Response Generation Pipeline

The system generates responses through the following process:

1. **Context Retrieval**
   - Hybrid search returns relevant document chunks
   - Top-k chunks selected as context

2. **LLM Integration**
   - Uses Gemini 2.0 Flash for response generation
   - Context chunks are provided to LLM
   - Responses are generated based on retrieved context


## Future Development 
1. Financial term query expansion
2. Context window optimization

## Implementation Benefits
- Combined semantic and keyword matching
- Preserved document sequencing
- Financial statement optimizations
- Normalized scoring system
- Efficient batch processing

## Requirements
- Python 3.8+
- Pinecone account
- SentenceTransformer
- LlamaParse
- Gemini API

## Implementation Challenges & Solutions

### PDF Data Extraction Challenges
During development, several PDF extraction approaches were evaluated:

1. **PyPDF2**
   - Simple text extraction without formatting
   - Fast but lost all table structure
   - Not suitable for financial documents with complex tables

2. **PyMuPDF4LLM**
   - Better formatting retention
   - Poor table extraction accuracy
   - Tables were unstructured compared to original

3. **Docling (IBM Deep Search)**
   - Improved structural information
   - Better table extraction
   - Performance issues (≈5 seconds per page)
   - Not scalable for large documents

4. **LlamaParse (Selected Solution)**
   - Superior table conversion accuracy
   - Significantly faster processing
   - Maintained formatting integrity
   - Best balance of speed and accuracy

## Setup Instructions
1. Install required dependencies
2. Configure Pinecone credentials
3. Initialize document processing pipeline
4. Set up search parameters

### Demo
View the [demo notebook in Google Colab](https://colab.research.google.com/drive/120K6Ks9KCPSsT1yUjnuogRwv0hxpy6Z3?usp=sharing)

### Basic Search Example
```python
searcher = HybridSearcher()
results = searcher.search(
    query="What was the revenue growth in Q3?",
    k=3,
    hybrid_weight=0.7
)
```
