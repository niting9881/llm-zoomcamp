# Study Notes: Large Language Model (LLM) Evaluation

## 1. Ground Truth Data for RAG Evaluation
Creating a reliable ground truth (gold standard) dataset is crucial for evaluating retrieval results.
- A ground truth dataset consists of thousands of queries, each linked to relevant documents from a knowledge base.
- For simplification, this course focuses on datasets where each query has one relevant document, though multiple may be relevant in practice.

### Methods for Data Generation
1. **Human Annotators/Domain Experts**: Highest quality, but time-consuming. Experts manually label relevant query-document pairs.
2. **Observing User Queries**: Analyze user interactions with a production system, then evaluate results (by humans or LLMs).
3. **LLMs for Synthetic Generation**: Used in this course. An LLM generates user questions from FAQ records; the original FAQ is the relevant document for these questions.

### Document ID Assignment
- Unique IDs are needed to link queries to documents.
- Simple numbering (e.g., doc.id = i) is problematic due to document updates.
- Section/heading IDs (from Google API for .docx) are ideal but not always available.
- **Content-based ID generation (MD5 hash)**: Used here. Hashes are created from question and text, making IDs content-dependent and stable.
- **Collision Handling**: Including more text (e.g., first 10 characters of the answer) helps resolve most collisions.

## 2. Offline LLM Evaluation: Cosine Similarity
Offline evaluation calculates metrics before deployment.
- **AQA Cosine Similarity**: Measures semantic closeness between LLM-generated and original answers.
  - Steps:
    1. Start with an original answer.
    2. Generate a question from it (LLM).
    3. Use RAG to get an LLM-generated answer.
    4. Compute cosine similarity between original and generated answers.
  - **Interpretation**: Ranges from -1 to 1. Higher is better.
  - **Model Comparison**:
    - GPT-4o: Good performance, but costly and slower.
    - GPT-3.5 Turbo: Cheaper, faster, similar scores (2% lower mean/median).
    - GPT-4o Mini: Even cheaper/faster, nearly identical performance to GPT-4o.

## 3. Offline RAG Evaluation: LLM Judges
LLMs can act as judges to evaluate RAG outputs.
- **Concept**: LLM is prompted to assess answer quality.
- **Prompt Cases**:
  1. With Original Answer: Judge compares generated answer to original (offline).
  2. Without Original Answer: Judge assesses answer relevance to question (online/offline).
- **Benefits**: Scalable, explainable, helps debugging.
- **Challenges**: Prompt engineering, bias, manual cleaning may be needed.

## 4. Retrieval System Evaluation: Metrics and Ground Truth
Evaluating retrieval is critical for RAG systems.
- **Reliable Evaluation**: Needs ground truth dataset.
- **Ranking Problem**: Systems return ranked lists; metrics assess ranking quality.

### Key Metrics
1. **Hit-rate (Recall)**:
   - Measures if at least one relevant document is in top N results.
   - Binary per query; percentage across queries.
2. **Mean Reciprocal Rank (MRR)**:
   - Measures how high relevant documents are ranked.
   - Score = 1/rank; average across queries.
   - Higher MRR = relevant docs ranked higher.

## 5. General LLM Evaluation and Monitoring
Evaluation and monitoring are essential for LLM applications.
- **Offline Evaluation**: Before deployment (cosine similarity, Hit-rate, MRR, LLM judge).
- **Online Evaluation**: After deployment (A/B tests, user feedback, monitoring).

### Types of LLM Evaluation Metrics
- **Reference-based**: Compare to ground truth.
  - N-gram: BLEU, ROUGE.
  - Text Similarity: Levenshtein Ratio.
  - Semantic: BERTScore, MoverScore, SMS, cosine similarity.
- **Reference-free (Context-based)**: Score without ground truth.
  - Quality-based: SUPERT, BLANC, ROUGE-C.
  - Entailment-based: SummaC, FactCC, DAE.
  - Factuality/QA/QG-based: SRLScore, QAFactEval, QuestEval.
- **LLM-based Evaluators**: Use LLM as judge (Reason-then-Score, MCQ, H2H, G-Eval).
  - Limitations: Bias, limited reasoning, numerical issues. Calibration strategies help.
- **RAG Pattern Metrics (RAGAS)**:
  - **Generation-related**:
    - Faithfulness: Factual consistency with context.
    - Answer Relevancy: Directness and appropriateness.
  - **Retrieval-related**:
    - Context Relevancy: Relevance of retrieved context.
    - Context Recall: Recall of ground truth in context.

---

## FAQ: Large Language Model Evaluation

**Q: What is the primary purpose of evaluating LLMs and RAG systems?**
A: To reliably determine the best methods/configurations for your system, based on data and requirements. Provides quantitative performance measures for informed decisions.

**Q: What is "ground truth" data and why is it important?**
A: Ground truth is a dataset where relevant documents/answers are definitively known for each query. It is crucial for benchmarking retrieval/generation accuracy.

**Q: How is ground truth data generated in this course?**
A: Synthetically, using an LLM to generate questions from FAQ records. The original FAQ is marked as relevant for these questions.

**Q: What are the main types of LLM evaluation?**
- Offline: Before deployment, for model/prompt selection (Hit-rate, MRR, Cosine, LLM judge).
- Online: After deployment, with real users (A/B tests, feedback).

**Q: Key metrics for RAG Search?**
- Hit-rate: % of queries with relevant doc in top N.
- MRR: Average reciprocal rank of first relevant doc.

**Q: How is RAG Evaluation performed using Cosine Similarity?**
A: Compare original and LLM-generated answers for semantic similarity.

**Q: What is "LLM as a Judge"?**
A: LLM evaluates RAG output quality, with or without original answer, providing scalable and explainable judgments.

**Q: Which LLM model is promising for RAG evaluation?**
A: GPT-4o Mini—cheaper, faster, nearly identical performance to GPT-4o.

**Q: Other important RAG metrics?**
- Faithfulness, Answer Relevancy, Context Relevancy, Context Recall (see RAGAS framework).
