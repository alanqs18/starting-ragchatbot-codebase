# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Running the Application
```bash
# Quick start using provided script
chmod +x run.sh && ./run.sh

# Manual start
cd backend && uv run uvicorn app:app --reload --port 8000
```

### Dependencies and Setup
```bash
# Install dependencies
uv sync

# The application requires Python 3.13+ and uv package manager
# Set ANTHROPIC_API_KEY in .env file
```

### Application Access
- Web Interface: http://localhost:8000
- API Documentation: http://localhost:8000/docs

## Architecture Overview

This is a Retrieval-Augmented Generation (RAG) system for course materials with the following key architectural components:

### Core Components

**RAG System (`backend/rag_system.py`)**: Central orchestrator that coordinates document processing, vector storage, AI generation, and session management. Handles the complete query flow from user input to AI response.

**Document Processing Pipeline**:
- `DocumentProcessor` (`backend/document_processor.py`): Parses structured course documents with specific format requirements
- Expected document format: Line 1 (Course Title), Line 2 (Course Link), Line 3 (Course Instructor), followed by lessons marked as "Lesson X: Title"
- Creates intelligent text chunks with configurable size and overlap for better semantic search

**Vector Storage (`backend/vector_store.py`)**: ChromaDB-based vector store with:
- Separate collections for course metadata and content chunks
- Sentence transformer embeddings (`all-MiniLM-L6-v2`)
- Smart semantic search with course name matching and lesson filtering

**AI Generation (`backend/ai_generator.py`)**: Anthropic Claude integration with:
- Specialized system prompt for educational content
- Tool-based search capability using `CourseSearchTool`
- Strict response protocol for course materials

**Tool System (`backend/search_tools.py`)**: Abstract tool framework with:
- `CourseSearchTool` for semantic course content search
- Source tracking for response attribution
- Integration with Claude's tool use API

**Session Management (`backend/session_manager.py`)**: Maintains conversation context across queries with configurable history limits.

### Data Flow

1. **Document Ingestion**: Course documents are parsed, chunked, and stored in ChromaDB with metadata
2. **Query Processing**: User queries flow through FastAPI → RAG System → AI Generator with search tools
3. **Response Generation**: AI uses tools to search vector store, then synthesizes responses based on retrieved content
4. **Session Tracking**: Conversation history is maintained for contextual follow-up queries

### Key Configuration

**Settings (`backend/config.py`)**:
- Chunk size: 800 characters with 100 character overlap
- Embedding model: `all-MiniLM-L6-v2`
- AI model: `claude-sonnet-4-20250514`
- Max search results: 5, conversation history: 2 messages

**Document Format Requirements**:
- Structured text format with specific header lines
- Lesson markers using "Lesson X: Title" pattern
- Support for course and lesson links

### Frontend Integration

Simple web interface (`frontend/`) with:
- Real-time chat interface
- Course statistics display
- Session management
- Suggested question prompts

The system automatically loads course materials from the `docs/` directory on startup.
- Always use UV to run the server, do not use pip directly
- Make sure to use UV to manage all dependencies.
- Use uv to run python files