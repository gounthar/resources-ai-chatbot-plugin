# Jenkins Chatbot Installation Guide for Windows

This guide provides step-by-step instructions for installing and running the Jenkins Chatbot on Windows systems.

## Prerequisites

- Windows 10 or 11
- Python 3.11 or later
- Git (to clone the repository)
- Maven (for the Java components)
- Sufficient disk space (at least 5GB for models and dependencies)

## Installation Steps

### 1. Clone the Repository

```powershell
git clone <repository-url>
cd resources-ai-chatbot-plugin
```

### 2. Build the Maven Project

```powershell
mvn install
```

### 3. Set Up the Python Environment

Navigate to the Python subproject directory:

```powershell
cd chatbot-core
```

Create a Python virtual environment:

```powershell
python -m venv venv
```

Activate the virtual environment:

```powershell
.\venv\Scripts\activate
```

### 4. Install Dependencies

Install the Python dependencies using the CPU-only requirements file to avoid NVIDIA CUDA dependency issues:

```powershell
pip install -r requirements-cpu.txt
```

> **Note**: If you encounter any dependency issues, especially with NVIDIA packages, use the `requirements-cpu.txt` file which excludes GPU-specific dependencies.

### 5. Download the Required Model

1. Create the model directory if it doesn't exist:

```powershell
mkdir -p api\models\mistral
```

2. Download the Mistral 7B Instruct model from Hugging Face:
   - Go to [https://huggingface.co/TheBloke/Mistral-7B-Instruct-v0.2-GGUF](https://huggingface.co/TheBloke/Mistral-7B-Instruct-v0.2-GGUF)
   - Download the file named `mistral-7b-instruct-v0.2.Q4_K_M.gguf`
   - Place the downloaded file in `api\models\mistral\`

### 6. Set the PYTHONPATH

Set the PYTHONPATH environment variable to the current directory:

```powershell
$env:PYTHONPATH = (Get-Location).Path
```

### 7. Start the FastAPI Server

Start the FastAPI application with Uvicorn:

```powershell
uvicorn api.main:app --reload --host 0.0.0.0
```

> **Note**: Adding `--host 0.0.0.0` makes the server accessible from other devices on the network. If you only need local access, you can omit this parameter.

The API will be available at `http://127.0.0.1:8000` once it's running.

## API Endpoints

The chatbot API provides the following endpoints:

1. **Create a new chat session**
   - URL: `http://127.0.0.1:8000/api/chatbot/sessions`
   - Method: `POST`
   - Response: Returns a session ID for subsequent requests

2. **Send a message to the chatbot**
   - URL: `http://127.0.0.1:8000/api/chatbot/sessions/{session_id}/message`
   - Method: `POST`
   - Request Body: JSON with a "message" field
   - Example:
     ```json
     {
       "message": "How do I install Jenkins?"
     }
     ```

3. **Delete a chat session**
   - URL: `http://127.0.0.1:8000/api/chatbot/sessions/{session_id}`
   - Method: `DELETE`

## Testing the API

You can test the API using curl commands:

```powershell
# Create a new session
curl -X POST http://127.0.0.1:8000/api/chatbot/sessions

# Send a message (replace {session_id} with the actual session ID)
curl -X POST http://127.0.0.1:8000/api/chatbot/sessions/{session_id}/message -H "Content-Type: application/json" -d "{\"message\":\"How do I install Jenkins?\"}"

# Delete a session
curl -X DELETE http://127.0.0.1:8000/api/chatbot/sessions/{session_id}
```

Alternatively, you can access the FastAPI's automatic documentation at `http://127.0.0.1:8000/docs` to test the API through a web interface.

## Troubleshooting

### Connection Issues

If you encounter connection issues like:

```
curl: (7) Failed to connect to 127.0.0.1 port 8000: Connection refused
```

Try the following solutions:

1. **Check if the server is running**: Ensure the uvicorn server is still running and hasn't crashed.

2. **Firewall settings**: Check if Windows Firewall is blocking the connection.

3. **Use the correct host**: Try starting the server with explicit host binding:
   ```powershell
   uvicorn api.main:app --reload --host 0.0.0.0
   ```

4. **Check for port conflicts**: Ensure no other application is using port 8000. You can specify a different port:
   ```powershell
   uvicorn api.main:app --reload --port 8080
   ```

### Model Loading Issues

If the server fails to start due to model loading issues:

1. **Verify model path**: Ensure the model file is in the correct location as specified in the config file.

2. **Check model file integrity**: Re-download the model if necessary.

3. **Memory limitations**: The model requires significant RAM. Ensure your system has enough available memory.

## Frontend Setup (Optional)

If you want to use the frontend application:

1. Navigate to the frontend directory:
   ```powershell
   cd ../frontend
   ```

2. Install the frontend dependencies:
   ```powershell
   npm install
   ```

3. Start the development server:
   ```powershell
   npm run dev
   ```

The frontend will be available at the URL displayed in the terminal (typically `http://localhost:5173`).