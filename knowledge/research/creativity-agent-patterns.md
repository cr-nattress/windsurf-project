# Biologically-Inspired Creative Agent

## 🌟 Overview
A neuroscience-grounded AI agent that emulates human creativity through divergent thinking, pattern recombination, and evaluative feedback loops. The architecture implements computational analogues of cognitive processes observed in the Default Mode Network (DMN), Executive Control Network, and dopaminergic reward systems.

## 🧠 Neuroscientific Foundations
| Biological System          | Computational Implementation        | Key Mechanism                     |
|----------------------------|-------------------------------------|-----------------------------------|
| Default Mode Network       | `CreativeExpansionTool`             | Semantic drift + distant associations |
| Executive Control Network  | `EvaluatorAgent`                    | Novelty-utility-coherence scoring |
| Dopaminergic Modulation    | `MetacognitionAgent`                | Exploration bias adjustment       |
| Hippocampal Recombination  | `MemoryAgent`                       | Hebbian association strengthening |
| Neuroplasticity Cycles     | `MetacognitionAgent` fatigue system | Engagement-based mode switching   |

## 🏗️ Architecture
```mermaid
graph TD
    A[CreativeAgent] --> B[Idea Generation]
    A --> C[Idea Evaluation]
    A --> D[Memory Management]
    A --> E[Metacognition]
    
    B --> F[CreativeExpansionTool]
    B --> G[HypothesisTool]
    
    C --> H[EvaluatorAgent]
    
    D --> I[MemoryAgent]
    
    E --> J[MetacognitionAgent]
    
    H --> K[FeedbackLoop]
    K --> E
    K --> D