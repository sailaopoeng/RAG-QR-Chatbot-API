# PDF QA API - FastAPI Version

This project has been converted from a Gradio interface to a FastAPI-based REST API for document question-answering using LangChain and OpenAI. The API loads PDF documents from a static folder at startup and allows users to query the content without uploading files.

## Features

- Pre-load PDF documents from a static folder at startup
- Ask questions about pre-loaded documents via REST API  
- Get AI-generated answers with source document references
- RESTful API endpoints for integration
- Automatic vector database initialization

## Installation

1. Install the required dependencies:
```bash
pip install -r requirements.txt
```

2. Set up your OpenAI API key in a `.env` file:
```
OPENAI_API_KEY=your_openai_api_key_here
```

3. Place your PDF files in the `pdfs` folder:
```bash
# The pdfs folder will be created automatically on first run
# Add your PDF files to: ./pdfs/
```

## Running the API

Start the FastAPI server:
```bash
python api.py
```

The API will be available at `http://127.0.0.1:7680`

**Note**: PDFs are loaded into the vector database during startup. If you add new PDFs, restart the application.

## API Endpoints

### Health Check
- **GET** `/health` - Check if the API is running and show loaded documents count
- **GET** `/` - Get API information and list of available documents

### Document Management
- **GET** `/documents` - List all available PDF documents in the pdfs folder

### Question Answering
- **POST** `/query` - Ask a question about the pre-loaded documents

**Request Format:**
```json
{
  "query": "What is the main topic of the documents?"
}
```

**Response Format:**
```json
{
  "answer": "AI-generated answer based on the PDF content",
  "success": true,
  "message": "Query processed successfully",
  "sources": ["document1.pdf", "document2.pdf"]
}
```

## Usage Examples

### Using curl
```bash
# Health check
curl -X GET http://127.0.0.1:7680/health

# List available documents
curl -X GET http://127.0.0.1:7680/documents

# Ask a question
curl -X POST http://127.0.0.1:7680/query \
  -H "Content-Type: application/json" \
  -d '{"query": "What is the main topic of the documents?"}'
```

### Using Python requests
```python
import requests
import json

# Check available documents
response = requests.get('http://127.0.0.1:7680/documents')
print("Available documents:", response.json())

# Ask a question
query_data = {
    "query": "What are the key findings in the documents?"
}

response = requests.post(
    'http://127.0.0.1:7680/query',
    headers={'Content-Type': 'application/json'},
    data=json.dumps(query_data)
)

result = response.json()
print("Answer:", result['answer'])
print("Sources:", result['sources'])
```

## Interactive API Documentation

Once the server is running, you can access:
- **Swagger UI**: http://127.0.0.1:7680/docs
- **ReDoc**: http://127.0.0.1:7680/redoc

## Changes from Upload Version

1. **No File Upload**: PDF files are loaded from the `pdfs` folder at startup
2. **Static Document Loading**: All documents are processed once during initialization
3. **Simplified API**: Single `/query` endpoint for questions
4. **Source Tracking**: Responses include which documents were used to generate answers
5. **Document Management**: New endpoint to list available documents
6. **Startup Initialization**: Vector database is created once at startup for better performance

## PDF Management

1. **Adding PDFs**: Place PDF files in the `pdfs` folder
2. **Updating PDFs**: Restart the application after adding/removing PDFs
3. **Supported Format**: Only PDF files (`.pdf` extension)
4. **File Names**: Use descriptive names as they appear in source references

## Dependencies

Key dependencies for the static file version:
- `fastapi`: Web framework
- `uvicorn`: ASGI server  
- `pydantic`: Data validation
- `glob`: File pattern matching (built-in Python)
- Removed: `python-multipart` (no longer needed for file uploads)

## Error Handling

The API returns appropriate HTTP status codes:
- `200`: Success
- `400`: Bad request (invalid file type, empty query)
- `500`: Internal server error

## Security Considerations

- No file upload endpoints (reduced attack surface)
- Input validation for queries only
- PDF files are read-only from the static folder
- Error messages don't expose sensitive system information
- Documents are processed at startup, not runtime
