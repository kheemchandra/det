# AngelOne RAG Chatbot

This project implements a Retrieval-Augmented Generation (RAG) chatbot system focused on information related to Angel One. It allows users to interact with a language model (Google Gemini) whose knowledge is augmented by documents uploaded by the user and content scraped from the Angel One support website.

The system consists of two main components:

1.  **Backend API (FastAPI):** Handles document processing, indexing, vector storage, RAG chain execution, chat history management, and web scraping.
2.  **Frontend Application (Streamlit):** Provides a user-friendly interface for chatting, uploading documents, managing uploaded content, and triggering web scraping.

## Features

- **RAG Chat Interface:** Interact with a Google Gemini model (Flash or Pro) using augmented context.
- **Document Upload:** Supports uploading `.pdf`, `.docx`, and `.html` files.
- **Vector Indexing:** Uploaded documents are split, embedded (using Google Generative AI Embeddings), and stored in a ChromaDB vector store.
- **Document Management:** List uploaded documents and delete them (removing from both the database record and the vector store).
- **Web Scraping:** Scrape FAQ content from the Angel One support website (`https://www.angelone.in/support/`) and index it into the knowledge base.
- **Chat History:** Maintains conversation history per session for contextual understanding.
- **Database Storage:** Uses SQLite to store metadata about uploaded documents and chat logs.
- **LangSmith Tracing (Optional):** Configuration included for tracing RAG chain execution with LangSmith.

## Technology Stack

- **Backend:**
  - FastAPI
  - Langchain (Core, Google Generative AI, Chroma, Community)
  - ChromaDB (Vector Store)
  - SQLite (Metadata & Chat Logs)
  - Google Generative AI API (Embeddings & Chat Models)
  - Pydantic (Data Validation)
  - BeautifulSoup4 (Web Scraping)
  - Uvicorn (ASGI Server)
- **Frontend:**
  - Streamlit
- **Language:** Python 3.x
- **Dependency Management:** pip
- **Environment Variables:** python-dotenv

## Project Structure

```plaintext
├── api/ # Backend FastAPI application
│ ├── main.py # FastAPI app definition and endpoints
│ ├── langchain_utils.py # Langchain RAG chain setup
│ ├── chroma_utils.py # ChromaDB interactions (indexing, retrieval, deletion)
│ ├── db_utils.py # SQLite database interactions
│ ├── scraper_utils.py # Web scraping logic for Angel One FAQs
│ ├── pydantic_models.py # Pydantic models for API requests/responses
│ ├── requirements.txt # Backend dependencies
│ ├── app.log # Log file (generated at runtime)
│ ├── chroma_db/ # ChromaDB persistent storage (generated at runtime)
│ └── rag_app.db # SQLite database file (generated at runtime)
│
├── app/ # Frontend Streamlit application
│ ├── streamlit_app.py # Main Streamlit app script
│ ├── sidebar.py # Defines the Streamlit sidebar UI
│ ├── chat_interface.py # Defines the Streamlit chat UI
│ ├── api_utils.py # Utility functions to call the backend API
│ └── requirements.txt # Frontend dependencies
│
├── .env.example # Example environment variables file
└── README.md # This file
```

## Setup and Installation

1.  **Clone the Repository:**

    ```bash
    git clone <your-repository-url>
    cd <your-repository-directory>
    ```

2.  **Set up Virtual Environments (Recommended):**
    It's best practice to create separate virtual environments for the backend and frontend.

    - **For the API:**

      ```bash
      cd api
      python -m venv venv_api
      source venv_api/bin/activate  # On Windows use `venv_api\Scripts\activate`
      pip install -r requirements.txt
      cd ..
      ```

    - **For the App:**
      `` bash
    cd app
    python -m venv venv_app
    source venv_app/bin/activate  # On Windows use `venv_app\Scripts\activate`
    pip install -r requirements.txt
    cd ..
     ``
      _(Note: If you prefer a single environment, create it in the root and install requirements from both `api/requirements.txt` and `app/requirements.txt`)_

3.  **Configure Environment Variables:**

    - Copy the example environment file:
      ```bash
      cp .env.example .env
      ```
    - Edit the `.env` file in the project root directory and add your API keys and configuration:

      ```dotenv
      # Required: Get from Google AI Studio or Google Cloud Console
      GOOGLE_API_KEY="YOUR_GOOGLE_API_KEY"

      # Optional: For LangSmith Tracing (https://smith.langchain.com/)
      LANGCHAIN_TRACING_V2=true
      LANGCHAIN_API_KEY="YOUR_LANGSMITH_API_KEY" # If using LangSmith
      LANGCHAIN_PROJECT="Your LangSmith Project Name" # If using LangSmith

      # URL for the backend API (used by the Streamlit app)
      # Default assumes backend runs on localhost port 8000
      BACKEND_API_URL="http://localhost:8000"
      ```

## Running the Application

You need to run the backend API and the frontend Streamlit app separately, typically in two different terminal windows.

1.  **Run the Backend API (FastAPI):**

    - Ensure you are in the `api` directory and the corresponding virtual environment (`venv_api`) is activated.
    - Start the Uvicorn server:
      ```bash
      cd api
      # Make sure venv_api is active
      uvicorn main:app --reload --host 0.0.0.0 --port 8000
      ```
    - The API will be available at `http://localhost:8000`. The `--reload` flag automatically restarts the server when code changes are detected.
    - The first time you run the API, it will create the `rag_app.db` SQLite database and the `chroma_db/` directory for the vector store within the `api` folder.

2.  **Run the Frontend Application (Streamlit):**
    - Open a _new_ terminal window.
    - Ensure you are in the `app` directory and the corresponding virtual environment (`venv_app`) is activated.
    - Start the Streamlit app:
      ```bash
      cd app
      # Make sure venv_app is active
      streamlit run streamlit_app.py
      ```
    - Streamlit will provide a URL (usually `http://localhost:8501`) to access the application in your web browser.

## Usage

1.  **Open the Streamlit App:** Navigate to the URL provided when you started the Streamlit app (e.g., `http://localhost:8501`).
2.  **Select Model:** Use the sidebar dropdown to choose the Google Gemini model (`gemini-2.0-flash` or `gemini-2.5-pro-exp-03-25`).
3.  **Upload Documents:**
    - Use the "Choose a file" uploader in the sidebar to select a `.pdf`, `.docx`, or `.html` file.
    - Click the "Upload" button. The file will be sent to the backend, processed, indexed into ChromaDB, and a record will be added to the SQLite database.
4.  **Scrape Angel One FAQs:**
    - In the sidebar under "Knowledge Base Management", click the "Scrape Angel One FAQs" button.
    - This will trigger the backend to scrape the Angel One support website, extract FAQ content, and index it into ChromaDB. Status updates will appear in the sidebar.
5.  **Manage Documents:**
    - The sidebar lists uploaded documents with their filenames and IDs.
    - Click "Refresh Document List" to update the list.
    - To delete a document, select its ID from the dropdown and click "Delete Selected Document". This removes it from the backend database and the ChromaDB vector store.
6.  **Chat:**
    - Type your questions related to the uploaded documents or scraped FAQs into the chat input box at the bottom of the main page.
    - Press Enter or click the send button.
    - The chatbot will retrieve relevant context from the vector store and generate a response using the selected Gemini model.
    - The conversation history is maintained for the current session.

## Notes

- Ensure you have valid API keys for Google Generative AI in your `.env` file.
- The quality of the responses depends heavily on the content of the uploaded documents/scraped data and the chosen language model.
- The web scraper is specifically designed for the structure of `https://www.angelone.in/support/` as observed during development and might need adjustments if the website structure changes significantly.
