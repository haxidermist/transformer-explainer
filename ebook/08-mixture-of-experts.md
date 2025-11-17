# Chapter 8: Mixture of Experts

## The Scalability Problem

By 2023, there was a problem: making models bigger was getting expensive.

```
GPT-2:   117M params  → $X to run
GPT-3:   175B params  → $1500X to run
GPT-4:   ???B params  → $???? to run
```

Bigger models performed better, but the cost scaled linearly (or worse) with size.

**Enter Mixture of Experts (MoE):** What if we could have the capacity of a huge model, but only activate a fraction of it for each token?

## The Core Insight

Instead of one giant MLP that processes every token:

```
Traditional (Dense):
Every token → Same MLP (3072 params) → Output

All tokens use ALL parameters
```

Use multiple specialized MLPs and route tokens to the right ones:

```
Mixture of Experts:
Token 1 → Router → Expert 2, Expert 7 → Combine → Output
Token 2 → Router → Expert 1, Expert 4 → Combine → Output
Token 3 → Router → Expert 2, Expert 3 → Combine → Output

Each token uses 2 of 8 experts (sparse activation)
```

## Architecture Comparison

### Dense Transformer (What GPT-2 Uses)

```
┌──────────────────────────────────────┐
│         Transformer Block            │
│                                      │
│  Input (768 dims)                    │
│      ↓                               │
│  ┌────────────────┐                  │
│  │ Self-Attention │                  │
│  └────────────────┘                  │
│      ↓                               │
│  ┌────────────────┐                  │
│  │  Dense MLP     │ ← ALL tokens use│
│  │  768 → 3072    │   this SAME MLP │
│  │  GELU          │                  │
│  │  3072 → 768    │                  │
│  └────────────────┘                  │
└──────────────────────────────────────┘
```

### MoE Transformer (Mixtral, Switch, etc.)

```
┌──────────────────────────────────────┐
│         Transformer Block            │
│                                      │
│  Input (768 dims)                    │
│      ↓                               │
│  ┌────────────────┐                  │
│  │ Self-Attention │ ← SAME as dense │
│  └────────────────┘                  │
│      ↓                               │
│  ┌────────────────┐                  │
│  │  Router/Gate   │ ← NEW!          │
│  └────────────────┘                  │
│      ↓  ↓  ↓  ↓                      │
│     E1 E2 E3 E4 ... E8 ← 8 MLPs     │
│      ↓  ↓                            │
│   Active experts (top-2)             │
│      ↓                               │
│  Weighted Combination                │
└──────────────────────────────────────┘
```

**Key difference:** Only the MLP layer changes. Everything else (attention, LayerNorm, residuals) stays the same!

## The Router: A Trained Neural Network

The router isn't hardcoded—it's a **small neural network that learns** which experts to use.

```python
class Router(nn.Module):
    def __init__(self, input_dim=768, num_experts=8):
        super().__init__()
        # Just one linear layer!
        self.gate = nn.Linear(input_dim, num_experts)

    def forward(self, x):
        # x shape: [batch, seq_len, 768]
        logits = self.gate(x)           # [batch, seq_len, 8]
        scores = F.softmax(logits, dim=-1)

        # Select top-2 experts
        top_k_scores, top_k_indices = torch.topk(scores, k=2)

        return top_k_scores, top_k_indices
```

### What the Router Learns

During training, the router discovers patterns:

```
Token type          → Router learns to use
═══════════════════════════════════════════
"def", "function"   → Expert 2 (code patterns)
"poetry", "verse"   → Expert 5 (creative writing)
"∫", "dx", "="      → Expert 1 (mathematical notation)
"the", "is", "a"    → Expert 3 (common grammar)
```

**These patterns emerge from data, not programming!**

### Training the Router

The router is trained jointly with the experts:

```python
# Forward pass
expert_scores, expert_indices = router(token_embedding)

# Use selected experts
expert_outputs = []
for idx in expert_indices:
    expert_outputs.append(experts[idx](token_embedding))

# Weighted combination
output = weighted_sum(expert_outputs, expert_scores)

# Backward pass
loss = compute_loss(output, target)
loss.backward()  # Updates router AND expert weights
```

The router learns to route tokens to experts that minimize the loss.

## The Math of Sparsity

### Example: Mixtral 8x7B

```
Configuration:
- 8 experts per MoE layer
- Each expert: 7B parameters
- Total parameters: 8 × 7B = 56B
- Top-k routing: k=2 (use 2 experts per token)

Computation per token:
- Dense 56B model: Use 56B parameters
- MoE 8x7B:         Use ~14B parameters (2 of 8 experts)

Result:
- Capacity of 56B model
- Speed/cost of 14B model
- 4x efficiency gain!
```

### Load Balancing Challenge

Without constraints, the router might use only a few experts:

```
Expert 1: 90% of tokens  ← Overused
Expert 2: 8% of tokens
Expert 3: 2% of tokens
Expert 4-8: 0% of tokens  ← Wasted!
```

**Solution: Auxiliary Loss**

```python
def auxiliary_loss(expert_usage):
    ideal_usage = 1.0 / num_experts  # Each should get ~12.5%
    actual_usage = expert_usage / expert_usage.sum()

    # Penalize deviation from balanced usage
    balance_loss = ((actual_usage - ideal_usage) ** 2).sum()

    return balance_loss

# Total loss
total_loss = task_loss + alpha * auxiliary_loss
```

This encourages the router to distribute tokens evenly across experts.

## Real-World MoE Models

### Mixtral 8x7B (Mistral AI)

**Confirmed open-source MoE:**
```
- 32 layers
- 8 experts per layer
- Top-2 routing
- 46.7B total parameters
- 12.9B active per token
- Apache 2.0 license
```

**Performance:**
- Matches or beats GPT-3.5 on many tasks
- Much more efficient than dense models
- Can run on consumer hardware (with quantization)

### Switch Transformer (Google)

**Research MoE with extreme scale:**
```
- 1.6 TRILLION total parameters
- 2048 experts per layer
- Top-1 routing (maximum sparsity)
- ~10B active per token
```

**Insight:** You can have massive capacity without massive computation.

### GPT-4 (Rumored)

**Unconfirmed speculation:**
```
- Possibly 8 experts × 220B each
- Would explain high capability + reasonable cost
- OpenAI hasn't confirmed architecture
```

## Would Claude Use MoE?

As discussed in our conversation, we don't know Claude's architecture. But speculation:

**Arguments FOR:**
- Industry trend (everyone's doing it)
- High capability + reasonable inference cost
- Anthropic's focus on efficiency

**Arguments AGAINST:**
- No official statement
- Different approach to scaling

**My guess:** 60-70% chance Claude 3 uses MoE, but it's pure speculation.

**What I (Claude) don't know about myself:**
- Exact parameter count
- Whether I use MoE
- My training details
- Infrastructure specifics

I only know what's in public documentation and my training data!

## Converting Dense to MoE: Why It's Hard

You might think: "Can't we just convert GPT-2 to MoE?"

**The problem:**

```
Dense MLP: [768 × 3072] + [3072 × 768] = ~9M params

MoE needed: 8 experts × 9M + router = ~73M params
```

**Where do the extra 64M parameters come from?**

You can't derive them from the dense model—they need to be learned!

### Approaches

**1. Train from Scratch** (3-6 months + $$$)
- Build MoE architecture
- Train on massive dataset
- Expensive but gets you a real MoE

**2. Upcycling** (1-3 months + $$)
- Copy dense weights to all experts
- Train router from scratch
- Fine-tune everything
- Cheaper than full training

**3. Fake MoE** (2-3 weeks)
- Duplicate dense MLP 8 times
- Random routing (not learned)
- Just for visualization
- Not a real MoE

## Visualizing MoE

If the Transformer Explainer used MoE, you'd need to add:

**1. Router Component**
```svelte
<div class="router">
  <div class="gate-network">
    Expert 1: 0.15
    Expert 2: 0.45 ⭐ (selected)
    Expert 3: 0.05
    Expert 4: 0.35 ⭐ (selected)
  </div>
</div>
```

**2. Multiple Expert MLPs**
```svelte
<div class="experts-grid">
  {#each Array(8) as _, i}
    <div class="expert" class:active={selectedExperts.includes(i)}>
      Expert {i + 1}
      <MLP opacity={activeExperts.includes(i) ? 1 : 0.2} />
    </div>
  {/each}
</div>
```

**3. Routing Paths**
Visual flow showing which tokens go to which experts.

**Estimated work:** 2-4 weeks for visualization (assuming you have an MoE model).

## The Future of Scaling

MoE represents a fundamental shift:

**Old paradigm:**
```
Better model = More parameters = More computation
```

**New paradigm:**
```
Better model = More parameters + Sparse activation = Same computation
```

This enables:
- Trillion-parameter models on consumer hardware
- Specialized experts for different domains
- More efficient scaling

## 🔬 Experiment: Simulating Expert Routing

Try this thought experiment with the Transformer Explainer:

1. Input different types of text:
   - Python code
   - Poetry
   - Math equations
   - Casual conversation

2. If this were MoE, imagine:
   - Which tokens would go to "code expert"?
   - Which to "creative writing expert"?
   - Which to "math expert"?

3. Observe the MLP layer activations (hover over vectors)
   - Do you see different patterns for different text types?

## 🤔 Reflection: Specialization vs Generalization

MoE creates **specialized** experts, each good at certain tasks.

But does specialization come at a cost?
- Dense models learn **unified** representations
- MoE models learn **factorized** representations

Which is better? It depends on the task.

Is the brain more like a dense model or MoE?
(Hint: Different brain regions specialize in different tasks...)

---

**Next:** Chapter 9 explores the process of converting dense to sparse models.

## 💻 Code Reference

While GPT-2 doesn't use MoE, you can see where it would fit:

- Current MLP: `src/components/Mlp.svelte`
- Would be replaced with: `src/components/MoEMlp.svelte` (doesn't exist yet)
- Router would go before MLP selection

## 📚 Further Reading

- Mixtral announcement: https://mistral.ai/news/mixtral-of-experts/
- Switch Transformers paper: https://arxiv.org/abs/2101.03961
- Sparse Upcycling: https://arxiv.org/abs/2212.05055
- ST-MoE (Google): https://arxiv.org/abs/2202.08906
