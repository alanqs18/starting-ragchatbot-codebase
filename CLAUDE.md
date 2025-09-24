# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Running the Application
- **Quick start**: `chmod +x run.sh && ./run.sh`
- **Manual start**: `cd backend && uv run uvicorn app:app --reload --port 8000`
- **Access points**:
  - Web interface: `http://localhost:8000`
  - API docs: `http://localhost:8000/docs`

### Environment Setup
- **Install dependencies**: `uv sync`
- **Package management**: Uses `uv` instead of pip - always use uv to manage all dependencies
- **Python execution**: Always use uv to run any Python files (`uv run python script.py`)
- **Python version**: Requires Python 3.13+
- **Environment variables**: Configure in `.env` file (ANTHROPIC_API_KEY, ANTHROPIC_BASE_URL, MODEL)

## Architecture Overview

This is a RAG (Retrieval-Augmented Generation) system for course materials Q&A with a dual-layer vector storage architecture:

### Core Components

**RAG System (`rag_system.py`)**: Main orchestrator that coordinates all components:
- Document processing and ingestion pipeline
- Query processing with tool-based search
- Integration between retrieval and generation

**Dual Vector Storage (`vector_store.py`)**: ChromaDB with two collections:
- `course_catalog`: Course metadata (title, instructor, links, structure)
- `course_content`: Text chunks with course/lesson context
- Semantic search with course name resolution and lesson filtering

**Document Processing (`document_processor.py`)**: Structured document parsing:
- Expected format: Course Title, Course Link, Course Instructor, Lesson X: content
- Intelligent chunking: 800 characters + 100 overlap at sentence level
- Context enhancement: Adds course/lesson metadata to chunks

**AI Integration (`ai_generator.py`)**: Tool-capable AI generation:
- Uses Anthropic Claude API (configured for Zhipu AI)
- Implements tool calling for search operations
- Optimized system prompt for course-specific responses

**Tool System (`search_tools.py`)**: Abstract tool framework:
- `CourseSearchTool`: Semantic search with course/lesson filtering
- Tool manager for AI tool execution
- Source tracking for response attribution

### Document Processing Pipeline

1. **Ingestion**: Course documents follow structured format with metadata headers
2. **Parsing**: Extracts course info and lesson structure using regex patterns
3. **Chunking**: Sentence-based splitting with overlap for context preservation
4. **Context Enhancement**: Adds course/lesson identifiers to each chunk
5. **Vectorization**: Uses all-MiniLM-L6-v2 embeddings
6. **Dual Storage**: Separates metadata from content for efficient retrieval

### Query Processing Flow

1. **Query Reception**: User question received via API
2. **Tool Invocation**: AI decides whether to use search tool
3. **Semantic Search**: Vector search with optional course/lesson filters
4. **Result Integration**: Search results provided to AI for response generation
5. **Context Management**: Conversation history maintained across sessions

### Key Configuration

- **Chunk settings**: 800 characters with 100 overlap for semantic continuity
- **Embedding model**: all-MiniLM-L6-v2 for efficiency
- **Search results**: Maximum 5 results per query
- **Conversation memory**: 2 message history length
- **Storage**: Persistent ChromaDB in `./chroma_db`

### Frontend Integration

- Single-page application served statically from FastAPI
- Real-time chat interface with session management
- Suggested questions and course statistics display
- Markdown rendering for AI responses

## Development Notes

- The system automatically loads documents from `../docs` on startup
- Course processing is idempotent - existing courses are skipped
- All components use dependency injection via the RAGSystem orchestrator
- Error handling is centralized with graceful degradation
- The system uses Zhipu AI instead of Anthropic in the current configuration
- always use uv to run the server do not use pip directly
- Make sure to use uv to manage all dependencies
- Always use uv to run any Python files.