# Flower Image Classification with Transfer Learning (Keras)

A small but complete deep-learning project: a model that classifies photos of flowers
into five categories (daisy, dandelion, roses, sunflowers, tulips) using **transfer
learning** on top of a pre-trained **MobileNetV2**.

---

## 1. Idea

Training an image classifier from scratch requires huge amounts of data and compute.
The idea of this project is to avoid that by **reusing a network that has already
learned to see**.

MobileNetV2 was pre-trained on ImageNet (≈1.2 million images, 1000 categories). It has
therefore already learned generic visual features, edges, colors, textures, shapes.
Instead of relearning all of this, I keep that knowledge frozen and only train a small
new classification head on my five flower classes. The result is a model that reaches
good accuracy from just a few thousand images and a few minutes of training.

---

## 2. The Mathematics Behind It

### Frozen feature extractor

The pre-trained base acts as a fixed function that maps an image to a feature map:

$$
\phi_{\theta_\text{base}} : \mathbb{R}^{160 \times 160 \times 3} \rightarrow \mathbb{R}^{H \times W \times C}
$$

The parameters $\theta_\text{base}$ are **frozen** (`base_model.trainable = False`), so no
gradients update them during training.

### Global average pooling

The feature map is collapsed into a single vector by averaging each channel over its
spatial dimensions:

$$
h_c = \frac{1}{H \cdot W} \sum_{i=1}^{H} \sum_{j=1}^{W} F_{i,j,c}, \qquad h \in \mathbb{R}^{C}
$$

This turns "where in the image" into "how strongly present overall".

### The trainable classification head

The head is a single linear layer producing one **logit** per class:

$$
z = W h + b, \qquad W \in \mathbb{R}^{K \times C}, \; b \in \mathbb{R}^{K}, \; K = 5
$$

Logits are converted to probabilities with the **softmax** function:

$$
p_k = \frac{e^{z_k}}{\sum_{j=1}^{K} e^{z_j}}
$$

### Loss function

Training minimizes the **sparse categorical cross-entropy**. For one example with true
class $y$:

$$
\mathcal{L} = -\log p_y = -z_y + \log \sum_{j=1}^{K} e^{z_j}
$$

The code uses `from_logits=True`, which means the softmax is computed **inside** the loss
using the numerically stable log-sum-exp form, rather than applying softmax in the model
and then taking its log.

### Optimization

Only the head parameters $\{W, b\}$ (plus the head's internals) are updated, using the
**Adam** optimizer:

$$
\theta \leftarrow \theta - \eta \cdot \frac{\hat{m}}{\sqrt{\hat{v}} + \varepsilon}
$$

where $\hat{m}$ and $\hat{v}$ are bias-corrected estimates of the first and second moments
of the gradient, and $\eta = 10^{-3}$ is the learning rate. Because $\theta_\text{base}$ is
frozen, the number of trainable parameters is tiny compared to the full network — this is
why transfer learning needs little data and little time.

### Why transfer learning works

A deep network learns a **hierarchy of features**: low layers capture generic patterns
(edges, colors), higher layers capture more specific structures (shapes, object parts).
The generic features are not class-specific — an edge looks the same on a cat or a rose —
so they transfer to a new task. Only the final decision boundary needs to be relearned,
which is exactly what the new head does.

### Regularization

Two mechanisms fight overfitting:

- **Data augmentation** applies random transformations $T$ (flip, rotation, zoom) to each
  training image, so the model effectively minimizes the loss over a distribution of
  augmented images rather than the fixed training set. It never sees the exact same image
  twice, forcing it to learn stable features instead of memorizing photos.
- **Dropout** randomly zeroes 20% of the pooled features during training, preventing the
  head from relying on any single feature.

### Input preprocessing

MobileNetV2 expects pixel values in $[-1, 1]$, so raw pixels in $[0, 255]$ are rescaled:

$$
x' = \frac{x}{127.5} - 1
$$

### Reproducible, non-overlapping split

The data is split 80% / 20% into training and validation using a **fixed random seed**.
The same seed in both loading calls guarantees the two subsets are disjoint (no image in
both), so the validation accuracy is an **honest estimate of generalization** rather than
an inflated score caused by data leakage.

---

## 3. Implementation

The pipeline, end to end:

1. **Download & load** the `flower_photos` dataset and read it into a `tf.data` pipeline.
2. **Split** reproducibly into training (80%) and validation (20%) with a fixed seed.
3. **Build the model**: data augmentation → preprocessing → frozen MobileNetV2 → global
   average pooling → dropout → dense classification head.
4. **Compile** with the Adam optimizer and sparse categorical cross-entropy.
5. **Train** for 8 epochs.
6. **Evaluate** with training/loss curves and a confusion matrix.
7. **Save** the model and run a single-image prediction as a sanity check.

**How it was built.** The initial code construct was scaffolded with the help of an AI
assistant. I then went through it by **reverse-engineering every line** asking *why*
each step exists, what each parameter does, and what would break without it until I
could explain the whole pipeline in my own words. During that process I also found and
fixed several real bugs in the working version (a mistyped `weights` argument, a mistyped
preprocessing call, a wrong batch-size variable, and a nested-folder issue that collapsed
all five classes into one).

---

## 4. Results

After the first epoch the model already reaches around **80% validation accuracy**, which
is the expected behavior for transfer learning: the pre-trained features do most of the
work immediately, and only the small head has to adapt.

Evaluation artifacts produced by the notebook:

- `training_curves.png` — accuracy and loss over epochs (training vs. validation)
- `confusion_matrix.png` — which classes are confused with which

---

## 5. What I Learned

- **Working with Keras**: building an input pipeline with `image_dataset_from_directory`,
  assembling a model with the functional API, and compiling/training/evaluating it.
- **How transfer learning works**: freezing a pre-trained feature extractor and training
  only a new head, and *why* that is effective (feature reuse across tasks).
- **Practical correctness**: why the train/validation split must be reproducible and
  non-overlapping, why logits vs. softmax matters for the loss, and how augmentation and
  dropout act as regularization.
- **Debugging real errors**: reading tracebacks and tracing a wrong result (e.g. "1 class
  found" instead of 5) back to its root cause.

---

## 6. Use of AI

This project was developed with AI assistance. The AI provided the **mentoring and the
initial code scaffolding**; I evaluated, questioned, corrected, and **reverse-engineered**
the code line by line with that help, until I understood *why* each part works rather than
just that it runs. All code in the final notebook was reviewed and adapted by me.

---

*Stack: Python · TensorFlow / Keras · MobileNetV2 · Matplotlib · NumPy*
