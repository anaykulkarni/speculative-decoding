# Speculative Decoding for fast inference in LLMs
We implement and optimize a speculative decoding algorithm for language models. Speculative decoding is a technique that accelerates text generation by using a smaller, faster "draft" model to propose tokens that a larger "target" model then verifies, potentially reducing the number of expensive forward passes needed during generation.

Traditional autoregressive text generation requires one forward pass through a language model for each generated token, which can be slow for large models. Speculative decoding accelerates this process by:

Using a smaller, faster model to draft a sequence of tokens
- Verifying these tokens with the larger model in a single forward pass
- Accepting correct tokens and regenerating from the first error

This approach can significantly reduce latency, especially when the draft model's predictions align well with the target model.

---

### **Experimental Strategies Evaluated**

- **Speculative Decoding with Greedy Draft Generation**
  - The draft model (`pythia-160m-deduped`) generates tokens using deterministic (greedy) decoding.
  - Achieved **high draft token acceptance rates** (85%–91%).
  - Resulted in significant **speedups** over baseline decoding, reaching up to **2.30×**.
  - Most effective for structured and repetitive text (e.g., song lyrics).

- **Speculative Decoding with Probabilistic Sampling**
  - The draft model samples tokens stochastically rather than greedily.
  - Introduced more diversity into generated text but drastically **lowered acceptance rates** (as low as 27%).
  - Caused speculative decoding to **underperform baseline decoding**, with **negative speedups** (e.g., 0.49×).
  - More suitable for creative tasks where diversity is prioritized over efficiency.

- **Speculative Decoding with Caching Disabled**
  - Investigated the performance of speculative decoding with the `use_cache` option turned off.
  - Found to still perform strongly with **speedups of 1.60× to 1.90×** and **latency reductions** of up to 47%.
  - Demonstrated that speculative decoding remains effective even without cache-based optimizations.

---

# Benchmarking results across different pairs

| Target Model                     | Draft Model                     | Benchmarking Prompt         | Acceptance Rate (%) | Speedup |
|----------------------------------|---------------------------------|----------------------------|----------------------|---------|
| EleutherAI/pythia-1.4b-deduped   | EleutherAI/pythia-70m-deduped   | The future of AI           | 70.29                | 1.90    |
| EleutherAI/pythia-1.4b-deduped   | EleutherAI/pythia-70m-deduped   | Robot learning emotions    | 60.38                | 1.66    |
| EleutherAI/pythia-1.4b-deduped   | EleutherAI/pythia-70m-deduped   | Happy Birthday lyrics      | 67.61                | 1.72    |
| EleutherAI/pythia-1.4b-deduped   | EleutherAI/pythia-410m-deduped  | The future of AI           | 85.34                | 0.87    |
| EleutherAI/pythia-1.4b-deduped   | EleutherAI/pythia-410m-deduped  | Robot learning emotions    | 86.84                | 0.84    |
| EleutherAI/pythia-1.4b-deduped   | EleutherAI/pythia-410m-deduped  | Happy Birthday lyrics      | 86.84                | 0.72    |
| EleutherAI/pythia-2.8b-deduped   | EleutherAI/pythia-160m-deduped  | The future of AI           | 85.34                | 1.34    |
| EleutherAI/pythia-2.8b-deduped   | EleutherAI/pythia-160m-deduped  | Robot learning emotions    | 41.07                | 0.99    |
| EleutherAI/pythia-2.8b-deduped   | EleutherAI/pythia-160m-deduped  | Happy Birthday lyrics      | 59.26                | 1.43    |
| meta-llama/Llama-3.2-3B         | meta-llama/Llama-3.2-1B        | The future of AI           | 76.56                | 1.00    |
| meta-llama/Llama-3.2-3B         | meta-llama/Llama-3.2-1B        | Robot learning emotions    | 43.69                | 0.67    |
| meta-llama/Llama-3.2-3B         | meta-llama/Llama-3.2-1B        | Happy Birthday lyrics      | 8.82                 | 0.20    |
