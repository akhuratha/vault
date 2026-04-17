### Chunking strategies

*Chunking defines the unit of retrieval.* 

It strongly affects recall, precision, citation quality, prompt size, and index size, which makes it one of the highest-leverage design choices in a RAG system.

- **Fixed-size chunking:** split text into uniform token or character windows.
	- Pros: simple, fast, and a strong baseline.
	- Cons: can break semantic boundaries, separate a question from its answer, or mix unrelated content in the same chunk.
	- Use when: the corpus is fairly uniform or you want the simplest reliable baseline.
- **Semantic / structure-aware chunking:** split on natural boundaries such as headings, paragraphs, sections, speaker turns, or document elements. Some tooling uses "semantic chunking" more narrowly for embedding-similarity-based splits; here the main idea is to respect content boundaries instead of fixed windows.
	- **Note:** in practice, "semantic chunking" can mean several related approaches: splitting on document structure, splitting where sentence/topic similarity drops, or using an LLM to segment text into coherent units. They all try to keep meaning together rather than cut at arbitrary token windows.
	- Pros: chunks are more coherent and usually easier for the model to use.
	- Cons: chunk lengths vary, and quality depends on good parsing.
	- Use when: document structure is meaningful and retrieval quality matters more than ingestion simplicity.
- **Overlap:** include a small shared window between neighboring chunks.
	- Pros: reduces boundary failures and helps when the answer spans adjacent chunks.
	- Cons: increases index size, duplicate retrieval, and prompt repetition.
	- Use when: important context is often cut across chunk boundaries.
- **Parent-child or hierarchical chunking:** retrieve a small child chunk, then expand to a larger parent section or document window before generation.
	- Pros: combines precise retrieval with richer context for synthesis.
	- Cons: needs more metadata, orchestration, and context packing logic.
	- Use when: answers are local enough for precise retrieval, but generation still needs surrounding context.

#### Choosing by document type
Choose chunk size based on answer granularity, not document length.

- Use **semantic / structure-aware chunking** when the source has meaningful natural boundaries such as FAQ pairs, glossary entries, headings, API sections, or function/class boundaries. It usually improves coherence and precision because chunks align with the document's structure.
- Use **fixed-size chunking** when the source is weakly structured, structure extraction is unreliable, or you want a simple baseline across mixed document types. Pick the window size based on how much context the task usually needs, not on document length.
- Use **parent-child retrieval** when the best retrieval unit is smaller than the best reading unit: retrieve small chunks for precise matching, then return the larger section or document window for full interpretation.

Overlap is a modifier, not a primary strategy. It is usually more important for fixed-size chunking because those boundaries are artificial, but semantic / structure-aware chunking can still benefit from light overlap when answers span adjacent sections.

#### Practical guidance
- Start with the simplest strategy that matches the corpus structure. 
- FAQs, API docs, contracts, support tickets, and codebases often need different chunking strategies.
- Preserve structure whenever possible. Headings, tables, lists, page numbers, and code blocks are often retrieval signals, not formatting noise.
- Attach metadata to every chunk: source document id, section, page, timestamps, ACLs, and parent pointers.
- Evaluate chunking empirically. There is no universal best chunk size; measure retrieval quality, answer quality, latency, and token cost together.

 - **How to overcome boundary issues:** 
	 - increase overlap, 
	 - align chunks to semantic boundaries, 
	 - use parent-child retrieval,
	 - stitch adjacent sibling chunks together before sending context to the model.

- if retrieval is missing relevant passages, 
	- chunks may be too small or 
	- boundaries too aggressive; 
- if answers are noisy, expensive, or citation-poor, chunks may be too large or overlap may be too high.
	- Here, "noisy" means the answer is related to the query but padded with irrelevant details, repeated context, or overly broad supporting text instead of giving a focused answer.
	- In evals, treat an answer as noisy when it is less direct than the expected answer, buries the key fact in extra text, or cites a large chunk that contains a lot of unrelated material.

#### How to evaluate chunking strategies?
A good chunking strategy is the one that places answer-bearing text in the top-k retrieved context with the fewest unnecessary tokens.

1. Build a representative eval set.
	- Include real user questions, expected answers, and the source passages that should support those answers.
	- Make sure the set covers the main document types in the corpus, since one strategy may help FAQs but hurt contracts or code.
2. Compare one variable at a time.
	- Keep the embedder, retriever, reranker, top-k, and generation prompt fixed while changing only chunking strategy, chunk size, and overlap.
3. Measure retrieval quality first.
	- Track recall@k, precision@k, MRR, or nDCG.
		- **MRR (Mean Reciprocal Rank):** measures how early the first relevant result appears. It is useful when getting one good chunk near the top matters a lot.
		- **nDCG (Normalized Discounted Cumulative Gain):** measures the quality of the whole ranking, giving more credit when highly relevant chunks appear earlier and allowing graded relevance rather than a simple relevant/not relevant label.
	- Track chunk-specific failure signals. These are useful because they tell you how to improve chunking, not just whether retrieval was good or bad overall.
		- **Duplicate retrievals:** even with a correct implementation, the top-k can still contain overlapping or near-identical sibling chunks from the same source because overlap, repeated headings, boilerplate text, or a missing deduplication step can make several chunks look relevant. This wastes context window space and reduces result diversity.
		- Boundary misses
		- Review and handle failure cases manually if required.
4. Measure answer quality next.
	- Check faithfulness, correctness, completeness, and citation quality.
		- **Faithfulness:** whether the answer is fully supported by the retrieved context. A faithful answer does not add claims, interpretations, or details that are not grounded in the retrieved evidence.
		- **Correctness:** whether the answer is actually correct for the query according to a gold answer, authoritative source, or human judgment. 
			- If the LLM as a judge only sees the retrieved context, it is mostly measuring faithfulness, not correctness. To evaluate correctness separately, the judge needs an independent reference such as a labeled answer, trusted source, or reviewer.
		- **Completeness:** whether the answer includes all the important parts needed to fully address the query, rather than giving only a partial or fragmentary response.
		- **Citation quality:** whether the cited chunks actually support the answer, point to the most relevant evidence, and avoid forcing the reader to sift through unrelated text.
	- Use human review for a smaller high-value subset, especially when answers require interpretation rather than exact extraction.
5. Measure cost and latency to balance with chunking accuracy metrics. 
	- A chunking setup that improves recall but doubles prompt size may still be the wrong choice for a high-QPS system.
6. Pick the winner per corpus or document family.
	- You can use different chunking profiles for different sources instead of forcing one global default.



