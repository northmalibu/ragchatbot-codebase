# RAG Chatbot Query Processing Flow

```mermaid
graph TD
    %% Frontend Layer
    A[User Input] --> B[sendMessage()]
    B --> C[Disable UI & Show Loading]
    C --> D[POST /api/query]
    
    %% API Layer
    D --> E[FastAPI Endpoint<br>/api/query]
    E --> F{Session Exists?}
    F -->|No| G[Create New Session]
    F -->|Yes| H[Get Session History]
    G --> I[RAGSystem.query()]
    H --> I
    
    %% RAG Processing
    I --> J[Format Query Prompt]
    J --> K[Get Conversation History]
    K --> L[AIGenerator.generate_response()]
    
    %% AI Generation with Tools
    L --> M{Tool Calling<br>Round 1}
    M -->|Tool Use| N[Execute Search Tools]
    M -->|No Tools| Z[Direct Response]
    
    %% Tool Execution
    N --> O{Tool Type?}
    O -->|search_course_content| P[CourseSearchTool]
    O -->|get_course_outline| Q[CourseOutlineTool]
    
    %% Search Processing
    P --> R[VectorStore.search()]
    Q --> S[Get Course Metadata]
    R --> T[ChromaDB Query<br>course_content]
    S --> U[ChromaDB Query<br>course_catalog]
    
    %% Vector Search
    T --> V[Sentence Transformers<br>Embedding Search]
    U --> W[Course Structure Lookup]
    V --> X[Ranked Results + Metadata]
    W --> Y[Lesson List + Links]
    
    %% Tool Results Processing
    X --> AA[Format Search Results<br>with Context]
    Y --> BB[Format Course Outline]
    AA --> CC[Track Sources & Links]
    BB --> DD[Return Tool Results]
    CC --> DD
    
    %% AI Response Generation
    DD --> EE{Round 2<br>Tool Calling?}
    EE -->|Yes| FF[Execute Additional Tools]
    EE -->|No| GG[Generate Final Response]
    FF --> GG
    Z --> GG
    
    %% Response Assembly
    GG --> HH[Extract Sources from Tools]
    HH --> II[Update Session History]
    II --> JJ[Return Response + Sources]
    
    %% API Response
    JJ --> KK[FastAPI Response<br>QueryResponse Model]
    KK --> LL[JSON Response]
    
    %% Frontend Display
    LL --> MM[Remove Loading Message]
    MM --> NN[Render Markdown Response]
    NN --> OO[Add Collapsible Sources]
    OO --> PP[Update Session ID]
    PP --> QQ[Re-enable UI]
    
    %% Styling
    classDef frontend fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef api fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef rag fill:#e8f5e8,stroke:#1b5e20,stroke-width:2px
    classDef tools fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef vector fill:#fce4ec,stroke:#880e4f,stroke-width:2px
    
    class A,B,C,D,MM,NN,OO,PP,QQ frontend
    class E,F,G,H,KK,LL api
    class I,J,K,L,M,EE,GG,HH,II,JJ rag
    class N,O,P,Q,AA,BB,CC,DD,FF tools
    class R,S,T,U,V,W,X,Y vector
```

## Key Components

### Frontend (`script.js`)
- **User Input**: Captures query from chat interface
- **HTTP Request**: POST to `/api/query` with session tracking
- **UI Updates**: Loading states, markdown rendering, source display

### Backend API (`app.py`)
- **Session Management**: Creates/retrieves conversation sessions
- **Request Routing**: Forwards queries to RAG system

### RAG System (`rag_system.py`)
- **Query Orchestration**: Coordinates AI generation with tools
- **Context Management**: Handles conversation history

### AI Generation (`ai_generator.py`)
- **Multi-Round Tool Calling**: Up to 2 rounds of tool execution
- **Response Generation**: Claude API integration with tool support

### Search Tools (`search_tools.py`)
- **CourseSearchTool**: Semantic content search with filtering
- **CourseOutlineTool**: Course structure and lesson lists
- **Source Tracking**: Captures sources and links for UI

### Vector Store (`vector_store.py`)
- **Dual Collections**: `course_catalog` (metadata) + `course_content` (chunks)
- **Semantic Search**: Sentence-transformers embeddings
- **Smart Filtering**: Course name resolution and lesson filtering

## Data Flow Highlights

1. **Session Continuity**: Maintains conversation context across queries
2. **Tool-Based Search**: AI decides which tools to use based on query type
3. **Multi-Round Processing**: Supports complex queries requiring multiple searches
4. **Source Attribution**: Tracks and displays source materials with links
5. **Real-Time UI**: Loading states and progressive content updates