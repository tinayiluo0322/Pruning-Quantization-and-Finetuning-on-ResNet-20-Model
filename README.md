# Pruning, Quantization and Finetuning on ResNet-20 model

### Luopeiwen Yi

### 1. Introduction
Deep neural networks (DNNs) often require significant computation and memory resources, making them challenging to deploy on edge or embedded systems. To address this, model compression techniques such as pruning and fixed-point quantization are widely used. In this experiment, we explore different pruning strategies and quantization methods, both individually and in combination, to evaluate their effectiveness on the ResNet-20 model trained on CIFAR-10.

---

### 2. Experiment Design

#### 2.1. Baseline
We start with a pretrained floating-point ResNet-20 model and evaluate its baseline performance:
- **Test Accuracy:** 0.9151
- **Test Loss:** 0.3231

#### 2.2. Pruning Methods
We evaluate three pruning strategies, each aiming for 80% sparsity:
- **One-Shot Layer-Wise Pruning:** Prune 80% of the weights once, then fine-tune.
- **Iterative Layer-Wise Pruning:** Gradually prune over the first 10 epochs, increasing sparsity by 8% each epoch, followed by 10 epochs of fine-tuning.
- **Global Iterative Pruning:** Use a global threshold across all layers for pruning (same percent overall, variable per-layer sparsity), repeated in an iterative fashion.

#### 2.3. Quantization Methods
We apply fixed-point quantization to the residual blocks of the ResNet-20 model using:
- **Asymmetric Quantization**
- **Symmetric Quantization**
- **Both with and without fine-tuning**

Bit-widths (Nbits) tested: 6, 5, 4, 3, 2.

#### 2.4. Combined Pruning + Quantization
We examine the performance of applying fixed-point quantization (Nbits=5 to 2) on a model pruned to 80% sparsity using the best-performing pruning method, and evaluate both before and after finetuning.

---

### 3. Results and Observations

#### 3.1. Pruning + Fine-Tuning Accuracy
| Method                           | Test Accuracy | Test Loss |
|----------------------------------|----------------|------------|
| Floating-point Baseline          | 0.9151         | 0.3231     |
| One-Shot Pruning + Fine-tuning   | 0.8794         | 0.3664     |
| Iterative Pruning + Fine-tuning  | 0.8769         | 0.3750     |
| Global Iterative Pruning + FT    | **0.8841**     | **0.3483** |

**Observation:**
- All pruning methods incur accuracy drops from the FP baseline.
- Global iterative pruning performs best, balancing sparsity and accuracy.
- One-shot pruning is more aggressive and loses more performance.

#### 3.2. Quantization without Finetuning
| Nbits | Asymmetric Acc | Symmetric Acc |
|-------|----------------|----------------|
| 6     | 0.9144         | 0.9134         |
| 5     | 0.9113         | 0.9071         |
| 4     | 0.8973         | 0.8532         |
| 3     | 0.7660         | 0.7151         |
| 2     | 0.0899         | 0.1000         |

**Observation:**
- Both quantization types perform similarly at higher bit-widths.
- Symmetric quantization tends to degrade more quickly under low bit-width.
- Performance drops sharply at 3 and 2 bits, showing the need for finetuning.

#### 3.3. Asymmetric Quantization with Finetuning
| Nbits | Accuracy After Finetune |
|-------|--------------------------|
| 5     | 0.9156                   |
| 4     | 0.9140                   |
| 3     | 0.9058                   |
| 2     | 0.8597                   |

**Observation:**
- Finetuning significantly recovers performance, especially at lower precisions.
- At 5 and 4 bits, accuracy approaches or matches full-precision baseline.

#### 3.4. Quantized Pruned Model
| Nbits | Acc Before FT | Acc After FT |
|-------|---------------|---------------|
| 5     | 0.8778        | 0.9032        |
| 4     | 0.8603        | 0.8994        |
| 3     | 0.7186        | 0.8715        |
| 2     | 0.1000        | 0.3348        |

**Observation:**
- Finetuning is crucial when combining pruning and quantization.
- Accuracy recovers well up to 3-bit precision even after pruning.
- Performance degrades drastically at 2 bits.

---

### 4. Comparative Analysis

| Feature                      | One-Shot Pruning | Iterative Pruning | Global Iterative | Quantization Only | Prune + Quantize |
|-----------------------------|------------------|-------------------|------------------|-------------------|------------------|
| Performance vs. FP Baseline | ↓ -3.9%        | ↓ -4.2%         | **↓ -3.4%**     | ↓ varies by Nbits | ↓ more at low bits |
| Sparsity Flexibility        | Low              | Medium            | High             | N/A               | High             |
| Bit-width Impact            | N/A              | N/A               | N/A              | High              | High             |
| Best Tradeoff               | No               | No                | **Yes**          | Yes (with FT)     | Yes (with FT)    |

---

### 5. Conclusion
This report demonstrates the effectiveness of pruning and quantization in compressing neural networks with minimal loss in performance. Key takeaways include:
- Global iterative pruning outperforms other sparsity strategies under high compression.
- Asymmetric quantization performs slightly better than symmetric at very low bit-widths.
- Finetuning is essential for recovering performance, especially after aggressive pruning or low-bit quantization.
- A combined pruning + quantization pipeline, with proper fine-tuning, can significantly reduce model size while maintaining acceptable accuracy.

These findings support the feasibility of deploying compressed ResNet-20 models in resource-constrained environments without sacrificing much performance.
