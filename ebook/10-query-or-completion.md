# Chapter 10: Query or Completion?

## The Newline Mystery

You type into the Transformer Explainer:

```
"The cat sat on the mat."
```

The model predicts:
```
\n (newline)
\n (newline)
\n (newline)
...
```

What's happening? Why won't it continue the story?

## Understanding Base Models vs Instruction-Tuned Models

### GPT-2: A Base Model

GPT-2 was trained on raw internet text with one simple objective:

**"Predict the next token"**

It saw patterns like:
```
"The cat sat on the mat.\n\n"  ← Two newlines = paragraph break
"Next paragraph starts here"
```

When you write a complete sentence, GPT-2 thinks: **"This looks finished. Next should be a newline."**

It's not being stubborn—it's doing exactly what it was trained to do!

### Modern Chat Models

Models like GPT-3.5, GPT-4, Claude went through additional training:

**1. Instruction Tuning**
```
Input: "Continue this story: The cat sat on the mat."
Expected: [actual continuation, not newlines]

Input: "Q: What is 2+2? A:"
Expected: "4"
```

**2. RLHF (Reinforcement Learning from Human Feedback)**
- Humans rate outputs
- Model learns what humans prefer
- Gets better at following instructions

### The Key Difference

```
Base Model (GPT-2):
User: "The cat sat on the mat."
Model: "This pattern suggests end of paragraph. Predict: \n"

Instruction-Tuned (GPT-4):
User: "Continue this story: The cat sat on the mat."
Model: "User wants continuation. Predict: 'and looked around...'"
```

## Why GPT-2 Struggles with Questions

In our conversation, we added query examples to the code:

```javascript
// src/store/index.ts
export const inputTextExample = [
  'Data visualization empowers users to',  // Works well
  'Q: What is the capital of France? A:',  // Struggles
  'Translate to French: Hello world'       // Struggles
];
```

### What Happens

**Input:** "Q: What is the capital of France? A:"

**GPT-2's reasoning** (statistical pattern matching):
1. "I've seen Q&A patterns before"
2. "After 'A:' usually comes a short answer"
3. "But also often comes a newline (end of Q&A pair)"
4. "Let me predict based on probability..."

**Result:** Might predict "Paris" but also might predict "\n" or "bout" (as in "About")

It's **pattern matching**, not **understanding** the question.

### Instruction-Tuned Models

**GPT-3.5/4 reasoning:**
1. "This is a question format"
2. "I'm trained to answer questions"
3. "The capital of France is Paris"
4. "Predict: 'Paris'"

Much more reliable!

## Encouraging Continuation in GPT-2

### Strategy 1: Incomplete Prompts

```
❌ "The story ended."
   → Predicts: \n (looks complete)

✅ "The story ended, but"
   → Predicts: "then", "the", "suddenly" (needs continuation)
```

### Strategy 2: Mid-Sentence Prompts

```
❌ "Artificial Intelligence."
   → Predicts: \n (complete thought)

✅ "Artificial Intelligence is transforming the"
   → Predicts: "world", "way", "industry" (incomplete)
```

### Strategy 3: Continuation Cues

```
❌ "Data visualization is useful."
   → Predicts: \n (complete)

✅ "Data visualization is useful for"
   → Predicts: "exploring", "understanding", "analyzing"
```

### Strategy 4: Adjust Temperature

Higher temperature = more randomness = less likely to always pick newline:

```javascript
// In the visualization UI
temperature = 0.8  // Default
temperature = 1.5  // More creative, less likely to stick with \n
```

## The Evolution: GPT-2 → ChatGPT

### 2019: GPT-2 (Base Model)
```
User: "What is 2+2?"
GPT-2: [might continue with random text, newlines, or anything]
Purpose: Text completion, not instruction following
```

### 2020: GPT-3 (Base Model, Bigger)
```
User: "What is 2+2?"
GPT-3: [better but still unpredictable]
Purpose: Few-shot learning, but still completion-focused
```

### 2022: InstructGPT / GPT-3.5
```
User: "What is 2+2?"
GPT-3.5: "4"
Purpose: Follow instructions reliably
```

### 2023+: GPT-4, Claude, etc.
```
User: "What is 2+2?"
GPT-4: "2 + 2 = 4"
Purpose: Instruction following + reasoning + safety
```

### What Changed?

**Training process evolved:**

```
GPT-2:
└─ Pre-training (predict next token)

GPT-3.5:
├─ Pre-training (predict next token)
└─ Instruction tuning + RLHF

GPT-4:
├─ Pre-training (predict next token)
├─ Instruction tuning
├─ RLHF
└─ Additional safety training
```

## Seeing the Difference in the Visualization

### Experiment: Complete vs Incomplete

1. **Complete sentence:**
   ```
   Input: "The cat is sleeping."
   Observe: High probability for \n
   ```

2. **Incomplete sentence:**
   ```
   Input: "The cat is sleeping on"
   Observe: High probability for "the", "a", "top"
   ```

3. **Question format:**
   ```
   Input: "Q: What is AI? A:"
   Observe: GPT-2 struggles (might predict \n)
   ```

### What You Learn

The visualization shows you **exactly** what GPT-2 has learned:
- Patterns from internet text
- Statistical correlations
- NO understanding of "meaning" or "instructions"

## The Philosophical Question

When GPT-2 predicts "\n" after a complete sentence, is it:

**A) Correct?**
- The training data supports this
- It's doing what it was trained to do
- From a probabilistic standpoint, \n IS a valid next token

**B) Wrong?**
- We wanted it to continue
- It's not "understanding" our intent
- It's failing at the task we implied

**The answer:** Neither and both. It's doing exactly what it was trained to do—we just want it to do something different!

This is why instruction tuning was such a breakthrough.

## Modern Prompting Techniques

Even with instruction-tuned models, prompting matters:

### Few-Shot Prompting

```
Q: What is 2+2? A: 4
Q: What is 3+3? A: 6
Q: What is 5+5? A:
```

Show examples, then ask your question.

### Chain-of-Thought

```
Q: What is 15 × 24?
Let's think step by step:
1. 15 × 20 = 300
2. 15 × 4 = 60
3. 300 + 60 = 360
```

### System Prompts (ChatGPT/Claude)

```
System: You are a helpful assistant that always shows your work.
User: What is 15 × 24?
Assistant: [detailed calculation]
```

## 🔬 Experiment: Completion Bias

In the Transformer Explainer:

1. **Complete thought:**
   ```
   "Machine learning is fascinating."
   ```
   - Check top predictions
   - Is \n in top 3?

2. **Incomplete thought:**
   ```
   "Machine learning is fascinating because"
   ```
   - Check top predictions
   - Are content words in top 3?

3. **Temperature variation:**
   - Try temp=0.3 (conservative)
   - Try temp=1.5 (creative)
   - How does this affect newline probability?

## 🤔 Reflection: The Intent Gap

**The fundamental problem:**

```
What we type:     "The cat sat on the mat."
What we mean:     "Continue this story!"
What GPT-2 sees:  Statistical pattern → probably ends here
```

**The gap:** Our **intent** vs the model's **training objective**

Instruction tuning bridges this gap by teaching models to infer intent.

But even modern models sometimes fail at this. They're pattern matchers, not mind readers.

**Question:** If a model perfectly mimics understanding without actually understanding, does the distinction matter?

---

**Next:** Chapter 11 explores "The Magic Numbers" - why 12 layers, 12 heads, 768 dimensions?

## 💻 Code Reference

- Examples list: `src/store/index.ts` (line 30-38)
- Temperature control: `src/components/Temperature.svelte`
- Sampling strategies: `src/components/Sampling.svelte`
- Probability display: `src/components/LinearSoftmax.svelte`

## 📚 Further Reading

- InstructGPT paper: "Training language models to follow instructions with human feedback"
- Constitutional AI: Anthropic's approach to alignment
- Prompt engineering guide: https://platform.openai.com/docs/guides/prompt-engineering
