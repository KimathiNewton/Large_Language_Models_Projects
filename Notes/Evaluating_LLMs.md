## Evaluating LLMs

In text we can find many different ways of saying the same thing, and whether one is more correct than another may depend more on the environment in which the text will be used than on the form of the text itself.
On the other hand, we not only have to evaluate the form of that text, but we also have to identify if the model is generating the text based on real data, or if, on the contrary, it has decided to use its imagination and complete the text in the way it thinks is best, making up the data. I suppose you already know that I’m referring to 
hallucinations. These are not bad in themselves; it will depend on the characteristics of our project. If the model is used as support for a fiction writer, it’s clear that in some moments, it may be required to provide completely fictional responses.

## 4.1 BLEU, ROUGE, and N-Grams

* BLEU, stands for Bilingual Evaluation Understudy, and it may be the first metric that was widely used to measure the quality of translations. It remains a valid and widely used metric.
* ROUGE is an evolution of BLEU adapted to measure the quality of summaries created by a language model. Like BLEU, it is based on the comparison of N-grams.

### N-Grams
An N-gram is a sequence of characters, symbols, or words, where N refers to the number of adjacent items. In our case, we will always work with words, so N will refer to the number of consecutive words in our sentences.