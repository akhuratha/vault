# Byte Pair Encoding

Related: [[tokenization-techniques]], [[LLM]]

The name comes from an older **compression** method that repeatedly merged the most frequent pair of bytes.

Byte Pair Encoding is a **subword tokenization** method. Instead of storing every full word in the vocabulary, it learns frequently occurring character or byte sequences as tokens.

Middle ground between:
- **word-level tokenization:** small sequence length, poor handling of unseen words, cannot capture repetitive structures in language.
- **character-level tokenization:** perfect coverage, but very long sequences, extremely inefficient

BPE tries to get the best of both:
- common words or word fragments become single tokens
- rare / unseen words can still be broken down and represented as multiple smaller tokens

BPE handles unseen words better than word-level tokenization. Even if a full word is not in the vocabulary, it can still be decomposed into learned subword pieces.

## Logic

1. Start from very small symbols, usually characters..
2. Count all adjacent symbol pairs in the training corpus.
3. Find the most frequent pair.
4. Merge that pair into a new symbol.
5. Repeat 2 - 4 until the vocabulary reaches the desired size

The learned merge rules define the tokenizer.

###  Pseudocode

```text
input:
  corpus
  base_vocabulary
  num_merges

tokens = split_every_word_into_base_symbols(corpus, base_vocabulary)
merges = []

repeat num_merges times:
  pair_counts = count_adjacent_pairs(tokens)
  if pair_counts is empty:
    break

  best_pair = most_frequent_pair(pair_counts)
  new_symbol = merge(best_pair)

  tokens = replace_all_occurrences(tokens, best_pair, new_symbol)
  merges.append(best_pair)

return merges, derived_vocabulary(tokens)
```

The order of merges matters. During inference, the tokenizer applies the same learned merge order.

### Simple example

Suppose the training corpus contains:

- `low`
- `lower`
- `lowest`
- `newer`

Start with character symbols:

- `l o w`
- `l o w e r`
- `l o w e s t`
- `n e w e r`

If the most common pair is `l o`, merge it into `lo`:

- `lo w`
- `lo w e r`
- `lo w e s t`
- `n e w e r`

If the next common pair is `lo w`, merge it into `low`:

- `low`
- `low e r`
- `low e s t`
- `n e w e r`

If `e r` is common, merge it into `er`:

- `low`
- `low er`
- `low e s t`
- `n e w er`

### Encoding new text

Once training is complete, encoding a new word follows the learned merge rules in exact order of their discovery in training.

Example:
The word `lower` starts as:

- `l o w e r`

After applying merges in order:

- `lo w e r`
- `low e r`
- `low er`

### Decoding

Decoding is the reverse process:

1. Convert token IDs back into token strings.
2. Concatenate the token strings in order.

For byte-level BPE:
- tokens may correspond to byte sequences rather than human-readable character chunks
- after concatenation, the byte stream is decoded into Unicode text

**This byte-level design is useful because it avoids an unknown-character problem. Any text can be represented as bytes.**

## Tradeoffs with vocabulary size

| Vocab size | Strengths                                                                          | Costs                                                                                                                                 |
| ---------- | ---------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| Small      | Smaller embedding/output vocab tables<br>Faster tokenization training and encoding | - Longer sequences; <br>- more compute per input; <br>- Misses rare words that are not captured by patterns if corpus is not diverse. |
| Large      | Shorter sequences; <br>better coverage of common words/phrases                     | More classification outputs for final next token prediction                                                                           |
