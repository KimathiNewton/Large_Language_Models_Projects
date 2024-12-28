## Evaluating LLMs

In text we can find many different ways of saying the same thing, and whether one is more correct than another may depend more on the environment in which the text will be used than on the form of the text itself.
On the other hand, we not only have to evaluate the form of that text, but we also have to identify if the model is generating the text based on real data, or if, on the contrary, it has decided to use its imagination and complete the text in the way it thinks is best, making up the data. I suppose you already know that I’m referring to 
hallucinations. These are not bad in themselves; it will depend on the characteristics of our project. If the model is used as support for a fiction writer, it’s clear that in some moments, it may be required to provide completely fictional responses.

## 4.1 BLEU, ROUGE, and N-Grams

* BLEU, stands for Bilingual Evaluation Understudy, and it may be the first metric that was widely used to measure the quality of translations. It remains a valid and widely used metric.
* ROUGE is an evolution of BLEU adapted to measure the quality of summaries created by a language model. Like BLEU, it is based on the comparison of N-grams.

### N-Grams
An N-gram is a sequence of characters, symbols, or words, where N refers to the number of adjacent items. In our case, we will always work with words, so N will refer to the number of consecutive words in our sentences.
For example, let’s take the sentence: “The cat sleeps quietly.”
If we work with 1-gram (unigram), we will use each word separately, while if we use 2-gram (bigram), we take each pair of words: “The cat,” “cat sleeps,” “sleeps quietly.”

What N-gram-based metrics usually do is compare the n-grams of the generated sentence, with those of a reference text and indicate whether there is a lot, a little, or no difference between the generated text and the reference text.

## Measuring Translation Quality with BLEU
Bilingual Evaluation  Understudy(BLEU) is used to measure the quality of translations, by comparing the translation with a set of reference translations that we have indicated are correct. What BLEU measures is the similarity of the generated translation to the reference translations provided. It can help us decide what translation system to use, and to achieve this, we must first create a dataset consisting of the text to be translated and several translations that will be taken as the reference translations. 

The first step is to have the set of sentences to be translated and their reference translations:
```
sentences = [
 "In the previous chapters, you've mainly seen how to work with OpenAI 
models, and you've had a very practical introduction to Hugging Face's 
open-source models, the use of embeddings, vector databases, and 
agents.",
 "These have been very practical chapters in which I've tried to 
gradually introduce concepts that have allowed you, or at least I hope 
so, to scale up your knowledge and start creating projects using the 
current technology stack of large language models."]

#Spanish Translation References.
reference_translations = [
 ["En los capítulos anteriores has visto mayoritariamente como trabajar 
con los modelos de OpenAI, y has tenido una introducción muy práctica 
a los modelos Open Source de Hugging Face, al uso de embeddings, las 
bases de datos vectoriales, los agentes."],
 ["Han sido capítulos muy prácticos en los que he intentado ir 
introduciendo conceptos que te han permitido, o eso espero, ir 
escalando en tus conocimientos y empezar a crear proyectos usando el 
stack tecnológico actual de los grandes modelos de lenguaje."]
 ]

```
In this case, we only have one reference translation for each text to be translated. If we had more than one reference translation, the only difference would be the content of the reference_translations list.

The first list consists of two elements:the two paragraphs of text to be translated. However, the second list contains two more lists. The first sublist contains the reference translations of the first text from the list, and so on.
The format of the reference translations list could look like this:
```
reference_translations = [
 ["reference translation 1 for text 1", "reference translation 2 for text 1", ...],
 ["reference translation 1 for text 2", "reference translation 2 for 
text 2", ...],
 ...
]
```
the *** reference_translations *** list is a list of lists, where each sublist contains the reference translations for a corresponding text to be translated. The number of reference translations for each text can vary, and they are stored as strings in the sublists. 
