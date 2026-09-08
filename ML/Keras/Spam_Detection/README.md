# Spam Detection with Logistic Regression in Keras

A small but complete machine-learning project: a model that classifies text messages
as **spam** or **ham** (legitimate) using **logistic regression**, implemented in
**Keras** on top of a TF-IDF text representation.

---

## Idea

Most spam filters have to answer one simple question for every message: *is this spam,
yes or no?* That makes it a **binary classification** problem, and the classic starting
point for binary classification is **logistic regression**.

The idea of this project is to build that classifier end to end and understand every
part of the pipeline rather than treat it as a black box. Raw text cannot be fed into a
model directly, so the messages are first turned into numbers using **TF-IDF**, which
scores how important each word is. Those numeric features then go into a logistic
regression model, which in Keras is nothing more than a single neuron with a sigmoid
activation. Building it in Keras (instead of the one-line scikit-learn version) makes
the connection between "logistic regression" and "a minimal neural network" explicit.

---

## Goal

- Build a working spam classifier from raw text to prediction.
- Understand *why* each step exists: text vectorization, the sigmoid, the loss function,
  and the evaluation metrics.
- Evaluate the model honestly on an **imbalanced** dataset, where accuracy alone is
  misleading, using precision, recall, and the F1-score.
- Produce a clean, reproducible, documented project suitable for a portfolio.

---

### Roadmap & Resources Utilized

**Roadmap**

1. **Data** – download the SMS Spam Collection dataset and load it with pandas.
2. **Exploration** – inspect the class balance (far more ham than spam) and message
   lengths.
3. **Preprocessing** – clean the text, encode labels (`ham → 0`, `spam → 1`), and split
   into train/test sets with a fixed seed and stratification.
4. **Vectorization** – transform text into TF-IDF features, fitting only on the training
   set to avoid data leakage.
5. **Model** – build a logistic regression model in Keras (a single `Dense` layer with
   sigmoid activation).
6. **Training** – train with binary cross-entropy and track training vs. validation loss.
7. **Evaluation** – report precision, recall, F1, and a confusion matrix.
8. **Documentation** – notebook, figures, and this README.

**Resources**

- **Dataset:** SMS Spam Collection Dataset (UCI Machine Learning Repository / Kaggle).
- **Libraries:** Python, TensorFlow / Keras, scikit-learn (`TfidfVectorizer`,
  `train_test_split`, `classification_report`), pandas, NumPy, Matplotlib.
- **Mentoring & scaffolding:** Claude (see *Use of AI* below).

---

## The Mathematics Behind It

### From text to numbers (TF-IDF)

Each message is turned into a vector where every entry corresponds to a word in the
vocabulary. The value combines how often a word appears in the message (**term
frequency**) with how rare that word is across the whole dataset (**inverse document
frequency**):

$$
\text{tfidf}(t, d) = \text{tf}(t, d) \cdot \log\!\left(\frac{N}{\text{df}(t)}\right)
$$

Here $N$ is the total number of messages and $\text{df}(t)$ is the number of messages
containing word $t$. Common words like "the" get a low weight; distinctive words like
"free" or "winner" get a high one, which is exactly the signal a spam filter needs.

### Logistic regression

The model computes a weighted sum of the features and passes it through the **sigmoid**
function to produce a probability between 0 and 1:

$$
z = \mathbf{w} \cdot \mathbf{x} + b, \qquad \hat{y} = \sigma(z) = \frac{1}{1 + e^{-z}}
$$

A prediction above 0.5 is classified as spam, below as ham.

### The loss function

Training minimizes the **binary cross-entropy** between the predicted probability
$\hat{y}$ and the true label $y$:

$$
\mathcal{L} = -\frac{1}{N} \sum_{i=1}^{N}
\Big[\, y_i \log(\hat{y}_i) + (1 - y_i)\log(1 - \hat{y}_i) \,\Big]
$$

This penalizes confident wrong predictions heavily, which pushes the weights $\mathbf{w}$
toward values that separate spam from ham.

---

## Implementation

In Keras the entire model is a single dense layer with one output neuron:

```python
model = keras.Sequential([
    keras.layers.Dense(1, activation='sigmoid', input_shape=(n_features,))
])

model.compile(optimizer='adam',
              loss='binary_crossentropy',
              metrics=['accuracy'])
```

The concept and initial scaffolding were developed with AI support; I then
reverse-engineered the code line by line, making sure I could explain *why* each choice
is made rather than just that it runs. Why sigmoid and not softmax for a binary problem?
Why fit the TF-IDF vectorizer only on the training data? Why is accuracy the wrong metric
here? Working through those questions is where the actual learning happened.

---

## What I Learned

- **The full ML pipeline:** turning raw text into features with TF-IDF, and why
  vectorization must be fit on training data only to prevent leakage.
- **Logistic regression as a neural network:** that the "classic" algorithm is just a
  single sigmoid neuron, which made the bridge from my earlier modules to Keras concrete.
- **Honest evaluation on imbalanced data:** why a 98% accuracy can still be a bad model,
  and how precision, recall, and F1 tell the real story via the confusion matrix.
- **The spam trade-off:** that a false positive (a real message flagged as spam) is often
  worse than a false negative, and how the decision threshold controls that balance.

---

## Use of AI

This project was developed with AI assistance. The AI provided the **mentoring and the
initial code scaffolding**; I evaluated, questioned, corrected, and **reverse-engineered**
the code line by line with that help, until I understood *why* each part works rather than
just that it runs. All code in the final notebook was reviewed and adapted by me.

---

*Stack: Python · TensorFlow / Keras · scikit-learn (TF-IDF) · pandas · NumPy · Matplotlib*