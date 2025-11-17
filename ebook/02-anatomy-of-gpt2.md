# Chapter 2: The Anatomy of GPT-2

## Inside the Black Box

GPT-2 consists of repeating blocks, each containing the same basic components. Understanding one block means understanding the whole model.

In the codebase, you can see the structure clearly:

```svelte
<!-- src/routes/+page.svelte:207-226 -->
<Embedding className="step" />
<div class="blocks relative">
  <div class="block-steps main">
    <QKV className="step" />           <!-- Query, Key, Value -->
    <Attention className="step" />      <!-- Multi-head attention -->
    <Mlp className="step" />           <!-- Feed-forward network -->
  </div>
</div>
<SubsequentBlocks className="step" />
<LinearSoftmax className="step" />
```

This simple structure contains profound computation.

## Embeddings: Where Words Become Numbers

### Token Embeddings

The model starts by converting text into numbers:

```
"Data visualization"
    ↓ (tokenization)
["Data", " visualization"]
    ↓ (token IDs)
[6601, 32704]
    ↓ (embedding lookup)
[[0.23, -0.45, 0.67, ...],   ← 768 dimensions
 [-0.12, 0.89, -0.34, ...]]
```

Each token becomes a 768-dimensional vector. Why 768? It's a balance:
- **Too small** (e.g., 128): Not enough capacity to represent meaning
- **Too large** (e.g., 4096): Computationally expensive, diminishing returns
- **768**: Sweet spot for GPT-2's scale (117M parameters)

### Positional Embeddings

But there's a problem: these embeddings don't encode **position**.

```
"Dog bites man" vs "Man bites dog"
```

Same words, different meanings! The model needs to know **where** each word appears.

**Solution: Positional Embeddings**

```python
# Simplified from src/utils/model/model.py
tok_emb = token_embedding_table[token_ids]  # Word meanings
pos_emb = position_embedding_table[positions]  # Positions
input_emb = tok_emb + pos_emb  # Combined representation
```

Now "Dog" at position 0 has a different representation than "Dog" at position 2.

## The Attention Mechanism: How Models "Think"

Attention is where the magic happens.

### The Core Idea

When processing each word, the model asks: **"Which other words should I pay attention to?"**

Example: "The trophy doesn't fit in the suitcase because it is too large."

When processing **"it"**:
- High attention to "trophy" (likely referring to trophy)
- Lower attention to "suitcase"
- Attention weights determine the interpretation

### Query, Key, Value (QKV)

The attention mechanism uses three transformations:

```
For each token:
Q (Query) = "What am I looking for?"
K (Key)   = "What do I contain?"
V (Value) = "What information do I have?"
```

**Code from the visualization:**

```svelte
<!-- src/components/Attention.svelte:72-126 -->
<div class="qkv">
  <div class="column key">
    <div class="head1 title">Key</div>
    {#each $tokens as token, index}
      <div class="vector x1-12 bg-red-300"></div>
    {/each}
  </div>

  <div class="column query">
    <div class="head1 title">Query</div>
    {#each $tokens as token, index}
      <div class="vector x1-12 bg-blue-300"></div>
    {/each}
  </div>

  <div class="column value">
    <div class="head1 title">Value</div>
    {#each $tokens as token, index}
      <div class="vector x1-12 bg-green-300"></div>
    {/each}
  </div>
</div>
```

### The Attention Calculation

```python
# Compute attention scores
scores = Query @ Key.T / sqrt(head_dimension)

# Apply causal mask (can't see future tokens)
masked_scores = apply_mask(scores)

# Convert to probabilities
attention_weights = softmax(masked_scores)

# Weighted combination of values
output = attention_weights @ Value
```

**In the visualization:** You see this as a matrix where each cell shows how much token i attends to token j.

### Multi-Head Attention

GPT-2 doesn't use just one attention mechanism—it uses **12 heads** simultaneously.

```
Head 1: Might focus on grammatical relationships
Head 2: Might focus on semantic similarity
Head 3: Might focus on co-reference
...
Head 12: Might focus on positional patterns
```

**Why 12 heads?**
```javascript
// src/store/index.ts:52
gpt2: { layer_num: 12, attention_head_num: 12, dimension: 768 }
```

- Each head: 768 ÷ 12 = **64 dimensions**
- 64 is a power of 2 (efficient for GPUs)
- 12 heads allow different patterns to be learned
- Not too few (limited perspective) or too many (redundancy)

### Causal Masking

A critical detail: GPT-2 is **autoregressive** (predicts left-to-right).

```
Token 1: Can see [1]
Token 2: Can see [1, 2]
Token 3: Can see [1, 2, 3]
Token 4: Can see [1, 2, 3, 4]
```

**It can't see the future!** This is enforced by the causal mask.

```python
# Mask future tokens (set to -infinity)
mask = torch.triu(torch.ones(seq_len, seq_len), diagonal=1) * -1e10
scores = scores + mask
```

When you see the attention matrix in the visualization, you'll only see the lower triangle filled in—that's the causal mask in action.

## MLP Layers: The Hidden Transformation

After attention, each token goes through an **MLP (Multi-Layer Perceptron)**.

```
Input (768 dims)
    ↓
Linear Layer 1: 768 → 3072  (expand 4x)
    ↓
GELU Activation
    ↓
Linear Layer 2: 3072 → 768  (compress back)
    ↓
Output (768 dims)
```

### Why Expand 4x?

```javascript
// Hidden dimension is 4x the model dimension
hidden_dim = 768 * 4 = 3072
```

This expansion creates a "bottleneck":
- **Expand**: Create a rich, high-dimensional representation
- **Activate**: Introduce non-linearity (GELU)
- **Compress**: Distill back to 768 dimensions

Think of it as:
1. Spread information out (expand)
2. Mix it around (activation)
3. Compress the result (project back)

### GELU Activation

Not ReLU (Rectified Linear Unit), but **GELU** (Gaussian Error Linear Unit):

```python
GELU(x) = x * Φ(x)
# where Φ is the cumulative distribution function
# of the standard normal distribution
```

It's smoother than ReLU and works better for language models.

### In the Visualization

```svelte
<!-- src/components/Mlp.svelte:78-80 -->
<div class="layer mlpUp mlpDown second-layer">
  <div class="column mlp-mid-column">
    <!-- 3072-dimensional intermediate representation -->
  </div>
</div>
```

You can **hover** over vectors to see the actual values flowing through the network.

## Layer Normalization and Residual Connections

Two critical stabilization techniques:

### Layer Normalization

Before each major operation (attention, MLP), normalize the inputs:

```python
# Normalize to mean=0, variance=1
normalized = (x - mean(x)) / std(x)
# Learn scale and shift
output = normalized * gamma + beta
```

**Why?** Prevents activations from exploding or vanishing during training.

### Residual Connections

The **skip connections** that make deep networks trainable:

```python
# Instead of:
x = Attention(x)
x = MLP(x)

# We do:
x = x + Attention(LayerNorm(x))  # Add input back!
x = x + MLP(LayerNorm(x))        # Add input back!
```

**Visualized in the code:**

```svelte
<!-- src/components/OperationGroup.svelte:74-76 -->
<OperationGroup type="residual-start" id={'mlp-residual'} />
<!-- ... computation happens ... -->
<OperationGroup type="residual-end" id={'embedding-residual'} />
```

This allows gradients to flow backward through the network during training.

## From Logits to Tokens: The Final Prediction

After 12 transformer blocks, we have rich representations. Now we need a prediction:

```
Final hidden state (768 dims)
    ↓
Linear projection: 768 → 50257 (vocabulary size)
    ↓
Logits (raw scores for each possible next token)
    ↓
Softmax (convert to probabilities)
    ↓
Sampling (pick a token)
```

### The Vocabulary

GPT-2 has **50,257 possible tokens**. The final layer outputs a score for each:

```python
# src/components/LinearSoftmax.svelte shows this
logits = model.final_layer(hidden_state)  # [768] → [50257]
probabilities = softmax(logits / temperature)
```

### Temperature and Sampling

**Temperature** controls randomness:
- Temperature = 0.1: Nearly deterministic (always picks top token)
- Temperature = 0.8: Balanced (default in the app)
- Temperature = 2.0: Very random

**Sampling strategies** (visible in the UI):
- **Top-k**: Only consider top k tokens
- **Top-p (nucleus)**: Consider tokens until cumulative probability ≥ p

```svelte
<!-- src/components/Sampling.svelte -->
<select bind:value={$sampling.type}>
  <option value="top-k">Top-k</option>
  <option value="top-p">Top-p</option>
</select>
```

## The Complete Flow

Putting it all together:

```
"Data visualization empowers users to" (input)
    ↓
Tokenize: [6601, 32704, 795, 30132, 2985, 284]
    ↓
Embed: 6 tokens × 768 dims
    ↓
┌─────────────────────┐
│   Block 1 of 12     │
│                     │
│   LayerNorm         │
│   Multi-head Attn   │  ← 12 heads × 64 dims each
│   Add & LayerNorm   │
│   MLP (768→3072→768)│
│   Add & LayerNorm   │
└─────────────────────┘
    ↓
[Repeat 11 more times]
    ↓
Final LayerNorm
    ↓
Project to vocabulary: [50257 logits]
    ↓
Apply temperature & sampling
    ↓
Predicted token: "explore" (or "understand", etc.)
```

## 🔬 Experiment 2: Attention Patterns

In the visualization:

1. Input: "The cat sat on the mat"
2. Click on "Multi-head Self Attention"
3. Observe the attention matrix
4. Try different heads (1-12) using the arrows

**Questions:**
- Which tokens does "mat" attend to most strongly?
- Do different heads show different patterns?
- What happens when you change the input?

## 🤔 Reflection: The Illusion of Understanding

You can now trace every computation from input to output. You've seen:
- Embeddings
- Attention weights
- MLP transformations
- Final probabilities

**But do we understand why "the cat sat on the ___" predicts "mat"?**

The model has learned statistical patterns from billions of words. It's not "reasoning" about cats or furniture—it's recognizing patterns.

Is this understanding? Or sophisticated pattern matching?

---

**Next:** Chapter 3 explores the difference between inference (what we see) and training (what we don't see).

## 💻 Code Locations

- Embeddings: `src/components/Embedding.svelte`
- Attention: `src/components/Attention.svelte`
- QKV: `src/components/QKV.svelte`
- MLP: `src/components/Mlp.svelte`
- Softmax: `src/components/LinearSoftmax.svelte`
- Model definition: `src/utils/model/model.py`

## 📚 Further Reading

- "The Illustrated GPT-2": http://jalammar.github.io/illustrated-gpt2/
- "Attention Is All You Need" paper
- Layer Normalization: https://arxiv.org/abs/1607.06450
