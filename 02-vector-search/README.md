# LLM Vector Search with Qdrant: Study Notes and FAQ

This document provides a comprehensive overview of vector search concepts, data organization, embedding models, and practical implementation using Qdrant, particularly in the context of Large Language Model (LLM) applications like Retrieval Augmented Generation (RAG).

---

## 1. Overview of LLM Vector Search with Qdrant

### What is Vector Search?

Vector search introduces new capabilities beyond traditional keyword-based search. Instead of matching keywords, it works on the semantics level (meaning). This allows it to:

- Match queries with different data modalities like images, videos, or sounds.
- Match two different ideas expressed with different words but sharing the same meaning (e.g., "bat" and "flying mouse").

In vector search, texts, images, or other data are transformed into vectorized representations (embeddings), and their similarity is measured numerically using similarity metrics like cosine similarity.

### Why Qdrant?

Qdrant is an open-source vector search engine written in Rust, designed to make vector search scalable, fast, and production-ready for solutions involving millions or billions of vectors. Dedicated vector search solutions like Qdrant are needed for:

- Scalability of vector search.
- Staying in sync with the latest trends and research in vector search.
- Utilizing full vector search capabilities beyond simple semantic similarity.

### Setting up Qdrant

Qdrant is flexible and can be run in various ways, including on your own infrastructure, Kubernetes, or in a managed cloud. For local setup, you can use a Docker container.

- After pulling the Qdrant Docker image, you can run the container, mounting local storage to ensure data persistence.
- Running Qdrant in Docker provides immediate access to its Web UI, which is beneficial for visually studying data and semantic similarity.

### Required Libraries

To work with Qdrant in Python, you typically install two main libraries in your virtual environment:

- **qdrant-client**: The official Python client for connecting to Qdrant. Official clients are also available for other languages.
- **fastembed**: Qdrant's own library specifically for vectorizing data. It simplifies the process of turning data into vectors and uploading them to Qdrant. FastEmbed uses ONNX runtime, making it lightweight and CPU-friendly, often faster than PyTorch Sentence Transformers, and supports local inference.

---

## 2. Data Organization for Vector Search

### Importance of Data Study

Organizing your data is a crucial step for any vector search solution. Studying the nature of your data helps define:

- How to organize data within the vector search solution.
- What data goes into embeddings (for semantic searching).
- What data goes into metadata (for more strict filtering conditions).

### Embeddings vs. Metadata

- **Embeddings**: Fields that make sense to be used for semantic search. This data is vectorized to be searched against by meaning. In a Q&A system, questions and answers are good candidates for embeddings because they might have different keywords but similar meaning.
- **Metadata (Payload)**: Data that makes more sense for strict filtering. This information is stored alongside the embeddings but not directly used for semantic similarity. For instance, in a course Q&A system, "course" or "section" names are suitable for metadata, allowing users to ask specific questions about a particular section or course.

### Data Preparation

For effective embedding, data should be well-prepared, cleaned, and chunked (divided into small, digestible pieces). This makes the data more suitable for embedding models to vectorize. When indexing, it's beneficial to combine information like questions and answers into a single text for vectorization, as a user's query might relate to either part.

---

## 3. Understanding Embedding Models

### Choosing the Right Model

Selecting the appropriate embedding model is a very crucial step for a successful vector search solution. You need to study your data's:

- Modality (e.g., text, image).
- Specifics and domain (e.g., short English texts, generic Q&A vs. specific technical domains).

The choice of model significantly impacts resources and the final search quality. It also involves choosing an embedding provider, which comes with cost and resource considerations.

### FastEmbed: Qdrant's Library

FastEmbed is recommended for its deep integration with Qdrant, simplifying the process of handling vectors and format conversions. It's lightweight, CPU-friendly, and uses ONNX runtime, making it faster than some alternatives. FastEmbed also supports local inference, meaning it doesn't incur external costs beyond your machine's resources. It supports various embedding types:

- Dense text embeddings: The most classical ones used in vector search (e.g., for semantic similarity today).
- Sparse embeddings: Important for hybrid search.
- Multi-vectors and image embeddings.

### Vector Dimensions

The dimension of the outputted vectors is a critical factor for choice and configuration.

- More dimensions generally mean more resources are needed, as vectors are "heavy data objects".
- However, more dimensions typically lead to better expression of the object's meaning being encoded.

For toy examples, small or moderate-sized embeddings like 512 dimensions are often suitable. For English textual embeddings, models like the Gina embedding small model are a good choice.

### Semantic Similarity Metric: Cosine Similarity

Most embedding models use cosine similarity as the metric to measure semantic distance between embeddings.

- Closer embeddings (smaller angle between them in vector space) will have a cosine similarity value close to 1. This indicates higher semantic similarity.
- Opposite vectors (larger or fully opposite angles) will have a cosine similarity value close to -1, indicating high dissimilarity in meaning.

---

## 4. Qdrant Collections and Data Points

When setting up a Qdrant solution, you operate with two main entities:

### Key Entities: Points and Collections

- **Point**: Represents a data point in Qdrant. In the context of course Q&A, a point would be an answer along with its metadata.
  - Each point has an ID (e.g., incremental integers or UUIDs).
  - It contains an embedding vector (or several vectors), generated by models like Gina, representing the semantic content (e.g., of an answer).
  - It includes optional metadata, referred to as payload (e.g., course and section information).
- **Collection**: A container for data points. It's typically designed to solve a single conceptual business problem (e.g., a collection for course FAQs).

### Creating and Populating Collections

To create a Qdrant collection, you must provide:

- A collection name (e.g., zoom_camp_rag or Zoom_camp_FAQ).
- The size (dimensionality) of the embedding vectors (e.g., 512 dimensions).
- The distance metric used to compare vectors (e.g., cosine similarity).

Once created, you can embed and insert your data points into the collection. The upsert operation handles both fetching/downloading the selected embedding model and then embedding each piece of text into vectors before uploading them to the Qdrant collection. For larger datasets, batch upserting operations (upload_collection, upload_points) offered by the Python client can improve efficiency and allow for parallelization.

### Visualizing Vectors

Qdrant's dashboard allows for visualizing data points. You can see uploaded payloads and vectors. More importantly, it projects the high-dimensional vectors to 2D, enabling you to visually study how points from different categories (e.g., courses) are semantically close or different based on their text content. This visual representation helps understand patterns in unstructured data and how vector search finds closest neighbors.

---

## 5. Performing Semantic Search

### How Semantic Search Works

Once data is indexed, semantic search works by:

1. Taking an incoming query (question).
2. Embedding the query using the same model that embedded the stored data.
3. Using the vector index to compare the query vector to all stored answers.
4. Finding the closest (most semantically similar) answer based on the configured distance metric (e.g., cosine similarity).
5. Returning the most similar answers, ranked by their similarity scores.

### Approximate Nearest Neighbor (ANN) Search

The "closest neighbor" search performed by vector indexes is typically approximate, not exact, especially at scale. This is due to the inherent nature of vector search with large vectors and the need for search speed. The search speed and quality depend on the selection of the vector index. Qdrant has a custom vector index implementation designed for this.

### Adding Filters for Finer Results

You can run semantic search with filters to get more fine-grained results. This uses the metadata (payload) that was uploaded with the embeddings (e.g., course or section).

- Qdrant's vector index implementation is specifically designed to make semantic similarity search with filters accurate, as filtering conditions can typically affect vector indexes.
- To work effectively with filtering, a specific index needs to be switched on in Qdrant.
- This allows you to ask specific questions about a particular course or section, restricting the search to a subset of your data. For example, searching for "late homeworks" specifically within the "data engineering" course yields a different answer than for "machine learning".

---

## 6. Advanced Search: Hybrid Search

### Introduction to Hybrid Search

Hybrid search is a broader term describing systems that use more than one search method to find the most relevant documents. It's particularly useful because user queries can vary widely:

- Some users prefer keyword-like search queries (e.g., "Google-like" with a few keywords).
- Others, especially in chatbot scenarios, will type questions using natural language.

Hybrid search aims to serve both types of users and prevent losing customers due to an inability to express their intent in a desired way.

### Keyword-Based Search with Sparse Vectors (BM25)

Surprisingly, keyword-based search can be implemented as vector search but using sparse vectors.

- Sparse vectors: Most dimensions are zeros; only non-zero dimensions are stored, saving resources.
- They focus on the exact presence of particular words or phrases, not semantic meaning.
- Useful when users provide identifiers or exact terms (e.g., a specific library name like "pandas"). Semantic search might struggle with these exact matches.
- Sparse vectors have a flexible dictionary and can virtually support new words, unlike dense models with fixed tokenizers.

BM25 (Best Matching 25) is an industry standard method to implement keyword-based search with sparse vectors.

- It calculates document relevancy based on a query.
- TF (Term Frequency): Rewards documents with multiple occurrences of query terms, but diminishes impact for excessive duplicates.
- IDF (Inverse Document Frequency): A global component that boosts the importance of rare words. Qdrant has built-in functionality to handle IDF calculation within collection configuration.
- BM25 is a purely statistical model, not a neural one, so it doesn't require heavy inference computations and is faster than dense embeddings for vectorization.
- BM25/sparse vector scores are unbounded and not compatible with cosine distance; they can only be compared for results from a single query, not across different queries.
- BM25 may struggle with longer, natural language queries.

### Combining Dense and Sparse Search

Qdrant allows you to configure a single collection with multiple named vectors (e.g., one for dense embeddings and one for BM25 sparse vectors). This is useful for testing different models or creating a search engine that is fast in some cases and uses more sophisticated methods for others.

You can build multi-stage retrieval pipelines using Qdrant's universal query API and prefetching mechanism.

- A common pipeline is to use a fast retriever (e.g., a small dense embedding model) to retrieve an initial set of documents, then rerank them using a better but slower model (e.g., a larger neural network or a cross-encoder).
- Reranking is about reordering retrieved points based on additional rules (e.g., business rules) or using a superior model.
- Fusion is a set of methods based on combining rankings from individual search methods.
  - Reciprocal Rank Fusion (RRF) is a commonly used algorithm for fusion. It calculates an intermediate score based on individual rankings (not raw scores) from different methods (e.g., dense and sparse). RRF identifies documents that are consistently ranked well by multiple methods, even if they aren't the top result from any single method. RRF is built into Qdrant.

### Beyond Dense and Sparse

The field of search is continually evolving. Beyond dense and sparse models, multi-vector representations and late interaction models are gaining traction. These approaches represent documents as multiple lower-dimensional embeddings, offering superior quality but are more resource-heavy.

---

## 7. Integrating Vector Search into RAG Workflow

### Seamless Integration

Integrating vector search into a Retrieval Augmented Generation (RAG) workflow is straightforward with Qdrant. A typical RAG function consists of three main steps: Search -> Build Prompt -> LLM. The "search" step is replaceable, allowing you to plug in any search function.

### Example Workflow

1. **Initialize Qdrant Client**: Connect to your running Qdrant instance.
2. **Define Collection**: Create a collection in Qdrant with appropriate vector configuration (size, distance metric) and indexes for metadata if filtering is desired.
3. **Embed and Index Data**:
   - Iterate through your documents.
   - For each document, combine relevant text (e.g., question + answer) into a single string.
   - Use fastembed with the chosen model (e.g., Gina AI embedding) to vectorize this combined text.
   - Store the vector along with its payload (original document, metadata like course/section) in Qdrant.
4. **Wrap Search Logic**: Create a vector_search function that takes a query (question) and performs a Qdrant query_points operation.
   - Inside this function, the query is embedded, and the search is performed against the collection.
   - Filters can be applied to narrow down results based on metadata.
   - The function processes the raw Qdrant response to extract the desired payload (the retrieved text).
5. **Integrate into RAG**: Replace the existing search component in your RAG function with your newly created vector_search function. This modular approach makes it easy to switch between different search methods.

---

## FAQ (Frequently Asked Questions)

**Q: What is a Qdrant "Point"?**  
A: In Qdrant, a "point" is essentially a data point. It consists of an ID, one or more embedding vectors, and optional metadata called a "payload". For example, in a Q&A system, an answer document would be stored as a point.

**Q: What is a Qdrant "Collection"?**  
A: A "collection" in Qdrant is a container that holds all your data points. It's typically designed to solve a single conceptual business problem, such as storing all vectorized answers for a specific course. When creating a collection, you define the size of the vectors and the distance metric for comparison.

**Q: Why use fastembed with Qdrant?**  
A: fastembed is Qdrant's own library that simplifies the process of vectorizing data for vector search. It's well-integrated with Qdrant, eliminating the headache of converting vectors to desired formats. It's also lightweight, CPU-friendly due to ONNX runtime, and supports local inference, reducing external costs.

**Q: How are embeddings compared in vector search?**  
A: Most embedding models use cosine similarity as the metric to compare embeddings. When two embedding vectors are semantically similar, the cosine of the angle between them will be close to 1. If they are very dissimilar, the cosine will be close to -1.

**Q: Is vector search exact at scale?**  
A: No, the "closest neighbor" search in vector indexes is typically approximate (Approximate Nearest Neighbor or ANN) at scale. Achieving exact search at a large scale is computationally intensive due to the size of vectors. The performance (speed and quality) depends on the specific vector index implementation.

**Q: What are sparse vectors and when are they useful?**  
A: Sparse vectors are a type of vector representation where the majority of dimensions are zero. They are particularly useful for implementing keyword-based search. Unlike dense embeddings that capture meaning, sparse vectors focus on the exact presence of words or phrases. They are effective for queries involving identifiers, specific product names, or when exact matches are preferred.

**Q: How does BM25 work?**  
A: BM25 (Best Matching 25) is an industry-standard statistical algorithm for keyword-based search using sparse vectors. It determines document relevancy based on two main components:
- Term Frequency (TF): Rewards documents that contain query terms multiple times but diminishes the reward for excessive repetitions.
- Inverse Document Frequency (IDF): A global component that boosts the importance of rare words that appear in fewer documents. Qdrant can handle IDF calculations automatically.

**Q: Can I combine dense and sparse search?**  
A: Yes, this is known as hybrid search. Qdrant allows you to store both dense and sparse vectors within the same collection. You can then create multi-stage retrieval pipelines where, for example, a fast dense model retrieves initial candidates, and a sparse model (like BM25) or another reranking method then reorders them.

**Q: What is reranking and fusion in hybrid search?**  
A: Reranking is the process of reordering the points retrieved in an initial search phase to incorporate additional rules or use a better, potentially slower, model. Fusion is a specific type of reranking that combines the rankings from individual search methods (e.g., dense and sparse). Reciprocal Rank Fusion (RRF) is a common fusion algorithm that calculates a combined score based on the individual rankings, not the raw scores, to find documents consistently ranked well by multiple methods.

---

## Resources

- Overview: https://qdrant.tech/articles/vector-search-manuals/
- Qdrant Internals: https://qdrant.tech/articles/qdrant-internals/
- RAG & GenAI: https://qdrant.tech/articles/rag-and-genai/
- Practical Examples: https://qdrant.tech/articles/practicle-examples/