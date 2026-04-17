#### Chunking strategies

*Chunking defines the unit of retrieval.*

It affects recall, precision, citation quality, prompt size, and index size, so it is one of the highest-leverage design choices in a RAG system.

**Primary chunking methods**

| Method                                         | How it works                                                                                             | Pros                                                           | Cons                                                                                                                             | Use when                                                                                               |
| ---------------------------------------------- | -------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| **Fixed-size chunking**                        | Split text into uniform length (of tokens or characters)                                                 | Simple, fast, and a good baseline.                             | Breaks semantic boundaries, so related information can be split across chunks and a single chunk can contain unrelated material. | Use as a baseline.                                                                                     |
| **Structure-aware chunking**                   | Split on natural boundaries such as headings, paragraphs, sections, speaker turns, or document elements. | Chunks are more coherent, easier to cite                       | Chunk lengths vary, and chunk quality depends on parsing logic.                                                                  | Use when documents are structured, such as FAQs or glossaries.                                         |
| **Similarity- or LLM-based semantic chunking** | Split where sentence or topic similarity drops, or use an LLM to segment text into coherent units.       | Produces good boundaries even when document structure is weak. | Slower, harder to tune, and nondeterministic.                                                                                    | Use when the source is weakly structured but semantic coherence matters for downstream LLM processing. |

**Modifiers**
- **Overlap:** include a small shared window between neighbouring chunks.
    - Pros: reduces boundary failures and helps when the answer spans adjacent chunks.
	- Cons: increases index size, duplicate retrieval, and prompt repetition.
    - Use when: important context is often cut across chunk boundaries.

Overlap is a modifier, not a primary strategy. It is usually more important for fixed-size chunking because those boundaries are artificial, but structure-aware chunking can still benefit from light overlap when answers span adjacent sections.

**Retrieval methods that improve chunking**
- **Parent-child retrieval:** retrieve a small child chunk, then expand to a larger parent section or document window before generation.
    - Pros: combines precise retrieval with richer context for synthesis.
    - Cons: needs more metadata, orchestration, and context packing logic.
    - Use when: answers are local enough for precise matching, but generation still needs surrounding context.

#### Practical guidance
- Start with the simplest strategy that matches the corpus structure. 
- Preserve structure whenever possible. 
- Attach metadata to every chunk: source document id, section, page, timestamps, and parent pointers etc.
- There is no universal best strategy or chunk size and overlap size
- Empirically evaluate chunking strategies. Refer [[evaluating-chunking]]. 
- Measure retrieval quality, answer quality, latency, and token cost together.

**Troubleshooting**
- If retrieval misses relevant passages, chunks may be too small or boundaries may be too aggressive, try more overlap, structure-aware boundaries, or parent-child retrieval.
- If answers are noisy, expensive, or citation-poor, chunks may be too large or overlap may be too high.
    - Here, "noisy" means the answer is related to the query but padded with irrelevant details, repeated context, or overly broad supporting text instead of giving a focused answer. In evals, treat an answer as noisy when it is less direct than the expected answer, buries the key fact in extra text, or cites a large chunk that contains a lot of unrelated material.
- If good evidence is split across adjacent chunks, stitch neighbouring sibling chunks together before sending context to the model.
