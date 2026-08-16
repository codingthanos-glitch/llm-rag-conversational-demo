# Conversational PDF RAG with Session-Based Chat History

A simple **Conversational RAG (Retrieval-Augmented Generation)** application built with Python, LangChain, FAISS, Ollama, and SQLite.

The application allows you to ask questions about a PDF document while maintaining conversation history using a **session ID**. Previous questions and answers are stored persistently in SQLite, allowing the conversation to continue even after restarting the application.

## Features

* Extract text from PDF files using `pdfplumber`
* Convert PDF pages into LangChain `Document` objects
* Split documents into smaller chunks using `RecursiveCharacterTextSplitter`
* Generate embeddings using Hugging Face
* Store and search embeddings using FAISS
* Retrieve relevant PDF chunks using **MMR (Maximal Marginal Relevance)**
* Use a local LLM through Ollama
* Maintain conversational context using LangChain messages
* Store chat history persistently using SQLite
* Support multiple conversations using different session IDs
* Continue a previous conversation after restarting the application
* Handle follow-up questions such as *"Where did he study?"* using chat history

---

## Architecture

```text
                    PDF
                     │
                     ▼
              pdfplumber
                     │
                     ▼
          LangChain Documents
                     │
                     ▼
        RecursiveCharacterTextSplitter
                     │
                     ▼
              Text Chunks
                     │
                     ▼
         HuggingFace Embeddings
                     │
                     ▼
                   FAISS
                     │
                     │
              User Question
                     │
                     ▼
                Retriever
                     │
                     ▼
          Relevant PDF Chunks
                     │
                     ▼
             Document Context
                     │
                     │
                     ├───────────────┐
                     │               │
                     ▼               ▼
              Chat History      Current Question
                 SQLite
                     │               │
                     └───────┬───────┘
                             ▼
                      ChatPromptTemplate
                             │
                             ▼
                         Ollama LLM
                             │
                             ▼
                          Answer
                             │
                             ▼
                    Save to SQLite
```

---

## Technologies Used

* Python
* LangChain
* FAISS
* Hugging Face Embeddings
* Ollama
* Qwen
* SQLite
* pdfplumber

---

## Ollama Setup

Install Ollama and make sure the Ollama service is running.

Pull the model used in the application:

```bash
ollama pull qwen3:8b
```

The model can be changed in the code:

```python
OLLAMA_MODEL = "qwen3:8b"
```

---

## PDF Configuration

Update `PDF_PATH` in `main.py`:

```python
PDF_PATH = r"C:\Users\abc\Downloads\abc.pdf"
```

Set it to the location of the PDF you want to use.

---

## Embedding Model

The application uses:

```text
sentence-transformers/all-MiniLM-L6-v2
```

The embedding model is configured as:

```python
EMBEDDING_MODEL = "sentence-transformers/all-MiniLM-L6-v2"
```

The code currently uses:

```python
model_kwargs={
    "local_files_only": True
}
```

This means the embedding model must already be available locally.

---

## How It Works

### 1. Load the PDF

The PDF is opened using `pdfplumber`, and text is extracted from each page.

Each page is converted into a LangChain `Document` object:

```text
Document
├── page_content
└── metadata
    └── page number
```

---

### 2. Split the Document

The extracted documents are split into smaller chunks.

```python
CHUNK_SIZE = 800
CHUNK_OVERLAP = 150
```

Chunk overlap helps preserve context between consecutive chunks.

---

### 3. Create Embeddings and FAISS Vector Store

Each chunk is converted into a numerical vector using the Hugging Face embedding model.

```text
PDF Text
   ↓
Chunks
   ↓
Embedding Model
   ↓
Vectors
   ↓
FAISS Vector Store
```

---

### 4. Retrieve Relevant PDF Content

When the user asks a question:

```text
Who is the CEO?
```

The question is used to search FAISS.

The application retrieves the top relevant chunks using MMR:

```python
TOP_K = 4
```

The retrieved chunks are combined into the document context.

---

### 5. Load Conversation History

Previous conversation messages are stored in SQLite.

For example:

```text
Human: Who is the CEO?
AI: John Smith is the CEO.

Human: Where did he study?
AI: He studied at XYZ University.
```

Each conversation is identified using a session ID.

---

### 6. Create the Prompt

The LLM receives three main pieces of information:

```text
Document Context
        +
Chat History
        +
Current Question
```

The prompt uses:

```python
ChatPromptTemplate
```

and:

```python
MessagesPlaceholder(variable_name="chat_history")
```

to dynamically insert previous conversation messages.

---

### 7. Generate the Answer

The prompt and LLM are connected using LangChain Expression Language (LCEL):

```python
chain = prompt | llm
```

When the chain is invoked:

```python
response = chain.invoke({
    "context": context,
    "chat_history": chat_history,
    "question": question
})
```

The flow is:

```text
Input Dictionary
       ↓
Prompt
       ↓
Formatted Chat Messages
       ↓
LLM
       ↓
AIMessage
       ↓
response.content
```

---

### 8. Save the Conversation

After generating the answer, both the user's question and the AI's answer are saved in SQLite.

```text
User Question
      ↓
SQLite

AI Answer
      ↓
SQLite
```

This allows the conversation to continue later using the same session ID.

---

## Session IDs

When the application starts, enter a session ID:

```text
Enter Session ID: user1
```

All questions and answers for that session are stored separately.

Example database structure:

| id | session_id | role  | message                              |
| -- | ---------- | ----- | ------------------------------------ |
| 1  | user1      | human | Who is the CEO?                      |
| 2  | user1      | ai    | John Smith is the CEO.               |
| 3  | user2      | human | What products does the company sell? |
| 4  | user2      | ai    | The company sells...                 |

Using the same session ID later allows you to continue the previous conversation.

---

## Running the Application

Run:

```bash
python main.py
```

Example:

```text
============================================================
CONVERSATIONAL PDF RAG
============================================================

Enter Session ID: user1

Conversation started for session: user1
Type 'exit' or 'quit' to stop the conversation.

You: Who is the CEO?

AI: John Smith is the CEO.

You: Where did he study?

AI: He studied at XYZ University.

You: exit

Conversation ended.
```

---

## Key Concepts Demonstrated

This project demonstrates:

* Retrieval-Augmented Generation (RAG)
* PDF text extraction
* LangChain `Document` objects
* Text chunking
* Embeddings
* Vector search with FAISS
* MMR retrieval
* Prompt templates
* LangChain Expression Language (LCEL)
* `HumanMessage` and `AIMessage`
* Conversational memory
* Session-based conversations
* SQLite persistence
* Local LLM inference with Ollama

---

## Future Improvements

Possible next steps for the project include:

* Persisting the FAISS vector store instead of rebuilding it on every run
* Displaying source page numbers with answers
* Limiting or summarizing long chat histories
* Adding multiple PDF support
* Adding metadata filtering
* Using LangGraph for conversation state and memory management
* Building a Streamlit or web-based interface
