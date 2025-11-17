# Table of Contents

## Part I: Understanding the Machine

### Chapter 1: What Is a Transformer?
- The Architecture That Changed Everything
- From Attention to Generation
- Why Visualizing Matters

### Chapter 2: The Anatomy of GPT-2
- Embeddings: Where Words Become Numbers
- The Attention Mechanism: How Models "Think"
- MLP Layers: The Hidden Transformation
- Layer Normalization and Residual Connections
- From Logits to Tokens: The Final Prediction

### Chapter 3: Inference vs Training
- What We See (Inference)
- What We Don't See (Training)
- The Frozen Model Mystery
- Why Training Visualization Would Be Hard

## Part II: The Codebase Journey

### Chapter 4: Architecture of Understanding
- How the Transformer Explainer Works
- ONNX Runtime: Running Models in the Browser
- The Component Structure
- Data Flow Through the Visualization

### Chapter 5: The Case of the Missing Popovers
- A Debugging Tale
- Case Sensitivity in File Systems
- The Importance of Details
- Lessons in Software Archaeology

### Chapter 6: Making It Your Own
- Forking and Customization
- Adding New Examples
- Git Workflows for Collaborative Learning
- Deploying Your Own Instance

## Part III: Advanced Architectures

### Chapter 7: Decoder, Encoder, or Both?
- Decoder-Only Models (GPT)
- Encoder-Decoder Models (T5)
- Encoder-Only Models (BERT)
- Choosing the Right Architecture

### Chapter 8: Mixture of Experts
- The Scalability Problem
- How MoE Works
- Router Networks: The Smart Dispatcher
- Dense vs Sparse Computation
- Would Claude Use MoE?

### Chapter 9: From Dense to Sparse
- Why You Can't Just Convert
- Upcycling: Dense to MoE
- Training the Router
- Load Balancing and Expert Collapse

## Part IV: The Art of Inference

### Chapter 10: Query or Completion?
- The Newline Mystery
- Why GPT-2 Struggles with Questions
- Instruction Tuning vs Base Models
- The Evolution from GPT-2 to ChatGPT

### Chapter 11: The Magic Numbers
- Why 12 Layers?
- Why 12 Heads?
- Historical Precedent and Empirical Design
- When Engineering Meets Art

### Chapter 12: Browsers and Machines
- Running AI Without GPUs
- WebAssembly and ONNX
- The democratization of ML
- Performance vs Accessibility

## Part V: Reflections

### Interlude I: On Understanding
- Can We Truly "See" How Models Think?
- The Limits of Visualization
- Transparency vs Interpretability

### Interlude II: On Intelligence
- What Claude Doesn't Know About Claude
- The Paradox of Self-Knowledge
- Architecture Secrecy and AI Safety

### Interlude III: On Building
- The Zen of Debugging
- Quality in Software and Models
- The Iterative Path to Understanding

## Appendices

### Appendix A: Code Reference
- Component Hierarchy
- Key Functions and Their Purpose
- Data Structures

### Appendix B: Glossary
- Transformer Terminology
- Architecture Patterns
- Common Abbreviations

### Appendix C: Further Reading
- Research Papers
- Blog Posts and Tutorials
- Related Projects

### Appendix D: Exercises
- Hands-On Experiments
- Modification Challenges
- Research Questions

---

*"The place to improve the world is first in one's own heart and head and hands."*
— Robert M. Pirsig
