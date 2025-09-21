[TOC]

### **Definition**

A **confidence map** is a per-pixel prediction that estimates the reliability of the model’s geometric output (e.g., depth, point map, 3D coordinates). Each pixel is assigned a scalar value (confidence), typically normalized to [0,1][0,1][0,1], indicating how trustworthy the prediction is.

------

### **Why It Is Needed**

- **Uncertainty estimation**: Many pixels lie in challenging regions (occlusion, reflections, textureless areas). Confidence maps capture this uncertainty.
- **Robust downstream tasks**: In SLAM, fusion, or robotics, low-confidence pixels can be down-weighted or discarded, avoiding error propagation.
- **Better training stability**: Confidence maps allow adaptive loss weighting, reducing the influence of noisy or unreliable predictions.

------

### **Training Approaches**

1. **Supervised with ground-truth geometry**

   - Weighted reconstruction loss:
     $$
     L = \sum_pc_p\cdot||\hat{X}_p-X_p^{gt}||^2
     $$
     

   - Confidence $c_p$ is learned to correlate with reconstruction quality.

2. **Probabilistic modeling**

   - Model outputs both mean and variance.
   - Confidence is the inverse of predicted variance: lower variance → higher confidence.
   - Loss often formulated as negative log-likelihood (NLL).

   $$
   \hat{X}_p \sim \mathbf{N}(\mu_p,\sigma_p^2)\\
   L = \sum_p\frac{||X_p^{gt}-\mu_p||^2}{2\sigma_p^2}+\frac{1}{2}log\sigma_p^2
   $$

   

3. **Self-supervised (no ground truth)**

   - Confidence maps are learned through reprojection consistency across multiple views.
   - Used to “explain away” pixels that cannot be consistently reconstructed (dynamic objects, occlusions).

   ------

### **Regularization**

- Prevent trivial solutions (all-zero confidence).
- Common strategies:
  - Encourage average confidence within a range.
  - Entropy-based penalties to avoid extreme outputs.
  - Sparsity regularization so only uncertain regions get low confidence.

------

### **Key Takeaway**

The **confidence map** is not an auxiliary output but a critical component for **uncertainty-aware geometry reconstruction**. It empowers models to:

- Predict *what* the geometry is (point map).
- Estimate *how certain* they are about it (confidence).