
# Natural Language Processing (NLP)
Natural Language Processing (NLP) enables machines to understand, generate, and analyze text. Language models learn patterns for tasks such as translation, summarization, classification, or text generation.

## 1. Classical NLP Algorithms

### 1.1 Bag of Words (BoW)

Represents text as word frequency vectors.

Example:
Text 1: "The cat sleeps"
Text 2: "The dog runs"

Vocabulary: ["The", "cat", "sleeps", "dog", "runs"]
BoW Text1: [1, 1, 1, 0, 0]
BoW Text2: [1, 0, 0, 1, 1]

### 1.2 TF-IDF

Weights words based on relative importance.

$$TF\text{-}IDF(w,d) = TF(w,d) \cdot \log \frac{N}{DF(w)}$$

### 1.3 Word Embeddings

Dense representations that capture semantic meaning.

Word2Vec: CBOW and Skip-Gram.

Conceptual Example:

$$v_{king} - v_{man} + v_{woman} \approx v_{queen}$$

## 2. Short Language Models (SLMs)

### 2.1 N-gram Models

Predicts the next word based on the previous (n-1) words:

$$P(w_1, ..., w_T) \approx \prod_{t=1}^{T} P(w_t | w_{t-(n-1)}, ..., w_{t-1})$$

### 2.2 LSTM (Long Short-Term Memory)

Captures long-term dependencies in sequences.

Cell update formula:

$$c_t = f_t \odot c_{t-1} + i_t \odot \tilde{c}_t$$

Conceptual Diagram:
x_t ---> [Input Gate] --->
[Forget Gate] ---> c_t ---> [Output Gate] ---> h_t

## 3. Large Language Models (LLMs)

Large Language Models (LLMs) are highly advanced neural networks with billions of parameters, trained on massive text datasets to understand, generate, and interact with human language at scale. They represent a significant evolution from traditional algorithms and SLMs, offering deep contextual understanding and the ability to perform complex natural language tasks with minimal or no task-specific fine-tuning.

### 3.1 Transformer

Before this formula was introduced in the 2017 "Attention Is All You Need" paper, models processed text sequentially, one word at a time. That made them incredibly slow to train and prone to forgetting context from the beginning of a long sentence. This formula changed everything by allowing the model to look at an entire sequence at once and calculate mathematically which words matter most to each other, regardless of distance.

Attention Formula:

$$Attention(Q,K,V) = softmax\left(\frac{QK^T}{\sqrt{d_k}}\right) V$$

Conceptual Diagram:
Input Embeddings --> [Multi-Head Attention] --> [Feed-Forward] --> Output
| ^
+-------- Positional Encoding ------------+

### 3.2 GPT (Generative Pretrained Transformer)

Generates text autoregressively.

Pre-training: predicts the next word.

Theoretical Example:
Input: "The capital of France is"
Prediction: "Paris"

### 3.3 BERT (Bidirectional Encoder Representations from Transformers)

Captures bidirectional context.

Pre-training: Masked Language Modeling and Next Sentence Prediction.

Example:
Input: "The cat is [MASK] on the bed"
Prediction: "sleeping"

## 4. Complementary Techniques

Sentence Embeddings: Vector representation of sentences (Sentence-BERT).

Fine-Tuning / Prompting: Adapting large models to specific tasks.

Transfer Learning: Reusing knowledge from pre-trained models.

Simplified conceptual diagram of NLP to LLM flow:
Raw Text --> Tokenization --> Embeddings --> Model (SLM or LLM) --> Task (Classification, Generation, Summarization)

## Conclusion

SLMs: N-grams, RNN, LSTM, useful for small corpora and specific tasks.

LLMs: Transformers, GPT, BERT, capture broad context and generate coherent language.

Word Embeddings: Foundation for almost all modern models.

Selection: Depends on corpus size, task complexity, and computational resources.