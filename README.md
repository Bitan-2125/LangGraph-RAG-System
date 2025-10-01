# 🚀 LangGraph RAG Agent with Ollama

This project implements a Retrieval-Augmented Generation (RAG) agent using **LangGraph** and **Ollama**. The agent is designed to answer questions about  specific PDF document  by leveraging local language models, eliminating the need for paid API services.

The system uses a graph-based agentic architecture to decide when to retrieve information from the document and when to generate an answer, creating a robust and intelligent query pipeline.

---

## ✨ Features

* **Fully Local:** Runs entirely on your machine using Ollama. No API keys needed. 🤫
* **PDF Knowledge Base:** Can ingest any PDF document as its source of truth.
* **Agentic Workflow:** Uses LangGraph to create a stateful, cyclical agent that can call tools (like a retriever) multiple times if needed.
* **Persistent Vector Store:** Utilizes ChromaDB to store document embeddings locally, so you only need to process the PDF once.
* **Modular Design:** The code is structured with clear functions for each part of the RAG process (loading, splitting, retrieving, generating).

---

## 🧠 How It Works

The system follows a cyclical graph-based workflow managed by LangGraph. This allows the agent to think and act in steps, similar to a human thought process.

**1. Data Ingestion (One-time Setup):**
* The pdf is loaded using `PyPDFLoader`.
* The document is split into smaller, manageable chunks using `RecursiveCharacterTextSplitter`.
* The `nomic-embed-text` model via Ollama converts these text chunks into numerical vectors (embeddings).
* These embeddings are stored in a local **ChromaDB** vector store for efficient similarity searches.

**2. The Agentic RAG Workflow:**
* **User Input:** The user asks a question, which is wrapped in a `HumanMessage`.
* **LLM Node:** The question is passed to the LLM (`llama3.2`). The LLM, guided by its system prompt, determines that it needs more information to answer the question and decides to call the `retriever_tool`.
* **Conditional Edge (`should_continue`):** The graph checks if the LLM's last message contains a tool call. Since it does, the graph transitions to the `retriever_agent` node.
* **Retriever Agent Node (`take_action`):** This node executes the `retriever_tool`. The tool takes the LLM's query, performs a similarity search against the ChromaDB vector store, and returns the top 5 most relevant document chunks.
* **Return to LLM Node:** The retrieved document chunks are formatted into a `ToolMessage` and sent back into the graph, returning to the `llm` node.
* **Answer Generation:** The LLM now has the original question *and* the relevant context from the PDF. It uses this information to synthesize a comprehensive answer.
* **Conditional Edge & End:** The LLM's response is a final answer and does not contain a tool call. The `should_continue` function evaluates to `False`, and the graph transitions to the **END** state, returning the final answer to the user.

---

## ⚙️ Models Used

This project relies on two main models served by Ollama:

* **LLM (Language Model):** `llama3.2`
    * This is the "brain" of the operation. It understands the user's question, decides when to use tools, and generates the final answer based on the retrieved context.
* **Embedding Model:** `nomic-embed-text`
    * This model is specialized in converting text into high-quality numerical vectors (embeddings). These vectors are essential for the vector store to find relevant document chunks based on semantic similarity.

---

## 💻 Installation and Setup

### 1. Ollama Setup

Ollama is required to run the local language models.

#### Ollama System Requirements
* **Operating System:** macOS, Windows, or Linux.
* **RAM:** A minimum of **8 GB** of RAM is recommended for 7B models like Llama 3. 16 GB or more is highly recommended for better performance and larger models.


#### Ollama Installation Steps
1.  **Download Ollama:** Visit the [Ollama official website](https://ollama.com/) and download the installer for your operating system.
2.  **Install Ollama:** Run the downloaded installer. It will set up the Ollama server as a background service.
3.  **Pull the Required Models:** Open your terminal or command prompt and run the following commands one by one. This will download the models to your local machine.

    ```bash
    ollama pull llama3.2
    ```

    ```bash
    ollama pull nomic-embed-text
    ```

4.  **Verify Installation:** Run `ollama list` in your terminal to see the downloaded models. You should see `llama3.2` and `nomic-embed-text` in the list.

### 2. Project Setup

Follow these steps to set up the Python environment and run the project.

1.  **Clone the Repository (or save the file):**
    Save the provided `rag.py` file in a new project directory.

2.  **Create a Virtual Environment (Recommended):**
    It's best practice to create a virtual environment to manage project dependencies.
    ```bash
    # For Windows
    python -m venv venv
    venv\Scripts\activate

    # For macOS/Linux
    python3 -m venv venv
    source venv/bin/activate
    ```

3.  **Create the `requirements.txt` file:**
    In your project directory, create a file named `requirements.txt` and paste the following content into it:
    ```
    langgraph
    langchain
    langchain-core
    langchain-community
    langchain-ollama
    langchain-chroma
    pypdf
    ollama
    ```

4.  **Install Dependencies:**
    Install all the required Python packages using pip.
    ```bash
    pip install -r requirements.txt
    ```

---

## 🚀 How to Run the Agent

1.  **Place Your PDF:** Make sure the **pdf** file is in the same directory as your `rag.py` script. You can change the `pdf_path` variable in the script to use a different PDF.

2.  **Run the Script:**
    Execute the Python script from your terminal.
    ```bash
    python rag.py
    ```

3.  **First Run (Vector Store Creation):** The first time you run the script, it will process the PDF, create embeddings, and save them to the ChromaDB directory (specified by `persist_directory`). You will see messages like:
    ```
    PDF has been loaded and has X pages
    Created ChromaDB vector store!
    ```

4.  **Interact with the Agent:** Once the setup is complete, you will be prompted to ask a question.
    ```
    === RAG AGENT===

    What is your question:
    ```
    Type your question and press Enter. The agent will show its internal tool calls and then provide a final answer.

5.  **Exit the Program:** To stop the agent, type `exit` or `quit` at the prompt.
