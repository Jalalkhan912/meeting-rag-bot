# Meeting Transcript RAG Agent – n8n Workflow Documentation

## Image:

![Contact Sync Workflow](./n8nWorkFlow.png)


## Overview
This workflow creates a Slack bot that answers questions about meeting transcripts using RAG (Retrieval-Augmented Generation). It combines Pinecone vector search with Airtable queries to provide accurate, context-aware responses.

## Workflow Steps

### 0. Slack Trigger
- Monitors the `#agent-dev` Slack channel for `@mentions`  
- Captures user messages and any attached files  

### 1. Create Conversation ID
- Generates unique conversation IDs for thread-based memory  
- Format: `{channel}-{thread_ts}` (or `{channel}-{ts}` for non-threaded messages)  
- Cleans message text by removing user mentions  

### 2. File Detection
- Checks if the user attached a file (`snippet`, `txt`, `pdf`, etc.)  
- Routes to file processing if present, or directly to agent if not  

### 3. File Processing (if attached)
- Downloads file from Slack using authenticated HTTP request  
- Extracts text content from the file  
- Passes extracted content along with user query to the agent  

### 4. AI Agent Processing

#### 4a. Parent Agent
- Orchestrates the entire response generation  
- Uses **Google Gemini 2.5 Pro** model (temperature: `0.2`)  
- Maintains conversation memory (10-message window)  
- Routes queries to appropriate sub-tools  

#### 4b. RAG Agent (Vector Search)
- Searches Pinecone vector store for relevant meeting transcripts  
- Retrieves top **100** results from `"Testing"` namespace  
- Uses **Google Gemini embeddings** for semantic search  

#### 4c. Reranking
- **Cohere** reranker refines results to top **30** most relevant documents  
- Improves accuracy by prioritizing best matches  

#### 4d. Airtable Search
- Queries Airtable for structured meeting metadata  
- Searches by title, date ranges, categories, etc.  
- Available fields:
  - `url`
  - `title`
  - `date`
  - `transcript`
  - `summary`
  - `main_category`
  - `condensed_summaries`

### 5. Response
- Sends AI-generated answer back to Slack thread  
- Maintains conversation context across messages  

## Key Features
- **Multi-Source Intelligence:** Combines vector search (Pinecone) with structured data (Airtable)  
- **File Support:** Processes attached documents (`txt`, `pdf`, etc.)  
- **Context Preservation:** Thread-based memory keeps conversations coherent  
- **Advanced Retrieval:** 100-doc retrieval + reranking to 30 ensures high-quality results  
- **Date-Aware Queries:** Supports complex Airtable date filtering (ranges, before/after)  

## Use Cases
- “What responsibilities were assigned to Kamal on [date]?”  
- “Summarize all Engineering Check-In meetings in September.”  
- “What’s in this attached document?” (with file upload)  
- “What was discussed about [topic] in recent meetings?”  

## Technical Stack
- **LLM:** Google Gemini 2.5 Pro  
- **Vector DB:** Pinecone (index: `kamal2`)  
- **Embeddings:** Google Gemini  
- **Reranker:** Cohere  
- **Structured Data:** Airtable  
- **Integration:** Slack  
