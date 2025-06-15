# Virtual TA API Documentation

This documentation provides details about the Virtual TA API endpoints, request/response formats, and examples for frontend developers.

## Base URL

```
https://virtual-ta-drab.vercel.app
```

## Authentication

The API requires an API key to be set in the environment variables. Make sure the backend has the `API_KEY` environment variable configured.

## Endpoints

### 1. Query Knowledge Base

Query the knowledge base with a question and optionally an image.

**Endpoint:** `POST /query`

**Request Body:**
```json
{
    "question": "string",
    "image": "string (optional, base64 encoded image)"
}
```

**Response:**
```json
{
    "answer": "string",
    "links": [
        {
            "url": "string",
            "text": "string"
        }
    ]
}
```

**Example Request:**
```json
{
    "question": "What are the prerequisites for the course?",
    "image": "base64_encoded_image_string" // Optional
}
```

**Example Response:**
```json
{
    "answer": "Based on the provided context, the prerequisites for the course include...",
    "links": [
        {
            "url": "https://discourse.onlinedegree.iitm.ac.in/t/example-topic",
            "text": "The course requires basic knowledge of..."
        },
        {
            "url": "https://docs.onlinedegree.iitm.ac.in/prerequisites",
            "text": "Prerequisites section from documentation..."
        }
    ]
}
```

### 2. Health Check

Check the health status of the API and database.

**Endpoint:** `GET /health`

**Response:**
```json
{
    "status": "healthy",
    "database": "connected",
    "api_key_set": true,
    "discourse_chunks": 1000,
    "markdown_chunks": 500,
    "discourse_embeddings": 1000,
    "markdown_embeddings": 500
}
```

## Error Responses

The API may return the following error responses:

### 500 Internal Server Error
```json
{
    "error": "Error message describing the issue"
}
```

### 429 Too Many Requests
When rate limits are exceeded, the API will return a 429 status code.

## Notes for Frontend Developers

1. **Image Handling:**
   - When sending images, ensure they are properly base64 encoded
   - The image parameter is optional - you can send queries without images

2. **Response Format:**
   - The `answer` field contains the main response text
   - The `links` array contains source references with URLs and text snippets
   - Links are always included when available in the knowledge base

3. **Error Handling:**
   - Always implement proper error handling for 500 status codes
   - Consider implementing retry logic for rate limit errors (429)

4. **Health Check:**
   - Use the health check endpoint to verify API availability
   - Monitor the `api_key_set` status to ensure proper configuration

## Example Frontend Implementation

Here's a simple example of how to make a query using JavaScript:

```javascript
async function queryKnowledgeBase(question, image = null) {
    try {
        const response = await fetch('https://virtual-ta-drab.vercel.app/query', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify({
                question,
                image
            })
        });

        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }

        const data = await response.json();
        return data;
    } catch (error) {
        console.error('Error querying knowledge base:', error);
        throw error;
    }
}

// Example usage:
const question = "What are the course prerequisites?";
queryKnowledgeBase(question)
    .then(response => {
        console.log('Answer:', response.answer);
        console.log('Sources:', response.links);
    })
    .catch(error => {
        console.error('Error:', error);
    });
```

## CORS Support

The API has CORS enabled and accepts requests from any origin. The following headers are allowed:
- `Content-Type`
- `Authorization`
- All standard headers

## Rate Limiting

Please be mindful of rate limits when making requests to the API. If you receive a 429 status code, implement exponential backoff in your retry logic. 