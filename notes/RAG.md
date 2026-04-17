### Why use RAG
Give an [[LLM]] access to knowledge that it was not trained on. 

### Alternatives to RAG

Design decisions to consider when evaluating RAG vs its alternatives:
- Query complexity
- Retrieval accuracy
- Latency bounds
- Churn of knowledge

Cost of RAG = ingestion-time cost + query-time costs
Whether the cost of RAG is worth it or not depends on how
- often the corpus changes, 
- how often it is queried,
- how much retrieval improves answer quality.

#### [[LLM-fine-tuning]]
#### **Long Context Prompt**
- Pros: Simplest setup, low orchestration overhead, and good latency when the document set is small.
- Cons: Only works when the material fits comfortably in context, and both cost and latency rise as prompts grow.

#### [[agentic-RAG]]

#### Methods that complement RAG
##### [[memory-in-llm-applications]]
#### Conclusion
- use normal RAG for large, changing, unstructured knowledge when a single retrieval pass is enough; 
	- use Agentic RAG when retrieval needs to be iterative or tool-driven; 
- use long context one shot prompt + docs for small document sets; 
- use memory for personalization; 

> In practice, the best systems are often hybrids rather than pure alternatives.

#### **When not to use RAG**
If a simple prompt, direct tool/API calls, or long-context read can solve the problem more reliably, start there. Add RAG only when corpus size, knowledge churn, or recall requirements make retrieval necessary.

[[economics-of-RAG]]
[[chunking]]