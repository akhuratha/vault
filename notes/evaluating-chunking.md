#### Evaluating chunking
A good chunking strategy is the one that places answer-bearing text in the top-k retrieved context with the fewest unnecessary tokens.

1. Build a representative eval set.
	- Include real user questions, expected answers, and the source passages that should support those answers.
	- If you have multiple document types, make sure the eval set covers all / most of the document types in the corpus.
2. Compare one variable at a time.
	- Keep the embedder, retriever, reranker, top-k, and generation prompt fixed while changing only chunking strategy, chunk size, and overlap.
3. Measure retrieval quality first.
	- Track recall@k, precision@k, MRR, or nDCG.
		- **MRR (Mean Reciprocal Rank):** measures how early the first relevant result appears. It is useful when getting one good chunk near the top matters a lot.
		- **nDCG (Normalized Discounted Cumulative Gain):** measures the quality of the whole ranking, giving more credit when highly relevant chunks appear earlier and allowing graded relevance rather than a simple relevant/not relevant label.
	- Track chunk-specific failure signals. These are useful because they tell you how to improve chunking, not just whether retrieval was good or bad overall.
		- **Duplicate retrievals:** the top-k can contain several overlapping or near-identical chunks from the same source. This wastes context window space and reduces result diversity. If this happens often, reduce overlap, change chunk boundaries, or add retrieval-time deduplication.
		- **Boundary misses:** relevant evidence is split across adjacent chunks, so no single retrieved chunk contains enough context on its own.
		- Review and handle failure cases manually if required.
4. Measure answer quality next.
	- Check faithfulness, correctness, completeness, and citation quality.
		- **Faithfulness:** whether the answer is fully supported by the retrieved context. A faithful answer does not add claims, interpretations, or details that are not grounded in the retrieved evidence.
		- **Correctness:** whether the answer is actually correct for the query according to a gold answer, authoritative source, or human judgment.
			- If the LLM as a judge only sees the retrieved context, it is mostly measuring faithfulness, not correctness. To evaluate correctness separately, the judge needs an independent reference such as a labeled answer from the eval set.
		- **Completeness:** whether the answer includes all the important parts needed to fully address the query, rather than giving only a partial or fragmentary response.
		- **Citation quality:** whether the cited chunks actually support the answer, point to the most relevant evidence, and avoid forcing the reader to sift through unrelated text.
5. Measure cost and latency to balance tradeoffs with chunking accuracy metrics.
6. Pick the winner per corpus or document family.
	- You can use different chunking profiles for different sources instead of forcing one global default.
