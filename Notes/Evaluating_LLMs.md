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
The *** reference_translations *** list is a list of lists, where each sublist contains the reference translations for a corresponding text to be translated. The number of reference translations for each text can vary, and they are stored as strings in the sublists. 

Two different methods will be used for the translations, and then they will be analyzed which one is better according to BLEU.  The first method is an open source model from Hugging Face called nllb-200-distilled-600.
Here is a link to its page on Hugging Face in case you want to take a look [*nllb-200-distilled-600M*](https://huggingface.co/facebook/nllb-200-distilled-600M)

This is a 600 million parameter model. Nowadays it can be considered a small model, as the most recent modes are measured in billions of parameters.It is specifically designed to translate short phrases or paragraphs of no more than 512 characters.

The second option for translating the text is the GoogleTranslator API. Therefore, this is a battle between Facebook and Google, and BLEU will tell us who comes out on top.
```
import transformers
from transformers import AutoTokenizer, AutoModelForSeq2SeqLM, pipeline
model_id = "facebook/nllb-200-distilled-600M"
tokenizer = AutoTokenizer.from_pretrained(model_id)
model = AutoModelForSeq2SeqLM.from_pretrained(model_id)
translator = pipeline('translation', model=model, tokenizer=tokenizer, 
src_lang="eng_Latn", tgt_lang="spa_Latn")

```

Pipeline takes care of the transformation from text to embeddings and vice versa.  In other words, when you call the pipeline, you only need to pass it the text, and its response will also be in text form. 
When calling a pipeline, you must specify which task or action you want to perform with the model. In this case, I indicated that I wanted to perform the “translation” task. However, I could have been more specific and indicated that I wanted to perform the “translation_en_to_es” task, in which case I wouldn’t have needed to pass the src_lang and tgt_lang parameters. But it’s not that simple; I don’t know how the transformers library is implemented, and using one pipeline or another doesn’t always yield the same results.

The pipeline can now be used to obtain translations.
```
translations_nllb = []
for text in sentences:
print ("to translate: " + text)
 translation = ""
 translation = translator(text)
 #Add the summary to the summaries list
 translations_nllb += translation[0].values()
```
This code, it iterates through the list containing the texts to be translated and translates them using the priviously created pipeline. The result is stored in a list where all generated translations are kept. Before evaluating with BLEU, let's take a look at the generated translations:
```
['En los capítulos anteriores, han visto principalmente cómo trabajar con 
modelos OpenAI, y han tenido una introducción muy práctica a los modelos 
de código abierto de Hugging Face, el uso de embebidos, bases de datos 
vectoriales y agentes.',

'Estos han sido capítulos muy prácticos en los que he intentado introducir gradualmente conceptos que han permitido, o al menos espero que lo hagan, ampliar sus conocimientos y comenzar a crear proyectos utilizando la tecnología actual de los modelos de lenguaje grande.']
```

To have some translations for comparison, I'm going to create others using the Google Translator API.
```
!pip install -q googletrans==3.1.0a0
from googletrans import Translator
translator_google = Translator()
translations_google = []
for text in sentences:
print ("to translate: " + text)
 translation = ""
 translation = translator_google.translate(text, dest="es")
 #Add the summary to summaries list
 translations_google.append(translation.text)
```
Let’s see the result obtained by the Google API.

```
['En los capítulos anteriores, vio principalmente cómo trabajar con modelos OpenAI y tuvo una introducción muy práctica a los modelos de código abierto de Hugging Face, el uso de incrustaciones, bases de datos vectoriales y agentes.',

'Estos han sido capítulos muy prácticos en los que he intentado introducir gradualmente conceptos que te han permitido, o al menos eso espero, ampliar tus conocimientos y empezar a crear proyectos utilizando la tecnología actual de grandes modelos de lenguaje.']
```
I would prefer these translations to the ones generated by NLLP, especially the second paragraph, which is much clearer and more natural. However, it's best to ask BLEU for its opinion to indicate which translation receives the best score

```
!pip install -q evaluate==0.4.1
import evaluate
bleu = evaluate.load('bleu')
```
I’ve decided to use the BLEU implementation from Hugging Face’s Evaluate library. 
It’s not the only one; for example, the famous NLTK (Natural Language Toolkit) library 
also implements this metric. However, as you’ll see, obtaining the metric result using 
the Evaluate library is extremely simple, much more so than if we used the NLTK library. 
In any case, and since you’re probably curious, I’ll leave you the link to the official 
[NLTK documentation where you can see how to use BLEU with NLTK:]( www.nltk.org/_modules/nltk/translate/bleu_score.html)

Once the evaluator is loaded, you only need to compare the two generated translations with the reference translation.
```
results_nllb = bleu.compute(predictions=translations_nllb, 
references=reference_translations)
results_google = bleu.compute(predictions=translations_google, 
references=reference_translations)
print(results_nllb)
print(results_google)
```
With a simple call to the evaluator’s compute function, you can obtain all the metric data. You just need to pass the generated translations and the reference translations to it.
Let’s see what it returned:
```
results_nllb: {'bleu': 0.3686324165619373, 'precisions': 
[0.7159090909090909, 0.47674418604651164, 0.30952380952380953, 
0.18292682926829268], 'brevity_penalty': 0.988700685876667, 'length_ratio': 
0.9887640449438202, 'translation_length': 88, 'reference_length': 89}
Chapter 4 Evaluating Model
141
results_google: {'bleu': 0.44975901966417653, 'precisions': 
[0.7710843373493976, 0.5679012345679012, 0.4177215189873418, 
0.2987012987012987], 'brevity_penalty': 0.9302618655343314, 'length_ratio': 
0.9325842696629213, 'translation_length': 83, 'reference_length': 89}
```
At first glance, focusing only on the first metric (bleu), the conclusion that can be drawn is that the translation performed by the Google API is better. I’m happy that BLEU agrees with my assessment

BLEU isn't returning just one number. The first metric is labeled as 'bleu' and it serves as a general indicator of translation quality. However, it is accompanied by many other values that should also be interpreted. Let briefly explain each of the values returned by the call to bleu compute means:
* BLEU: A value between 0 and 1 is returned. The closer this value 
    is to 1, the better the translation. The two values we obtained with 
    our translations are below 0.5. This might lead us to think that they 
    are poor-quality translations, but nothing could be further from 
    the truth. A value between 0.3 and 0.4, though not perfect, cannot 
    be considered bad; it retains the meaning and is understandable. 
    If the BLEU value is between 0.4 and 0.5, it’s considered a very 
    good translation. Values above 0.6 are rarely seen, even in human 
    translators. Therefore, we could say that the translation performed by NLLB is correct, while the Google Translation could be considered 
    very good.

*  Precisions: Precision evaluates the number of identical n-grams 
    found between the translation being evaluated and the reference 
    translation. As you can see, it returns four precision values, 
    corresponding to 1-gram, 2-gram, 3-gram, and 4-gram. Naturally, 
    the precision value decreases as the n-gram size increases. In the 
    translations we’re evaluating, NLLB achieves a 0.72 precision for 
    the first data point, while Google achieves 0.77. This indicates that 
    both translations likely use a similar set of words to the reference 
    Chapter 4 Evaluating Model
    142 translation. The most significant difference lies in how the precision decreases as the n-gram size increases. For the last precision 
    value, NLLB obtains a 0.18, compared to Google’s 0.30. This is a 
    considerable difference, suggesting that the sentence structure 
    Google produces is much closer to the reference translation than the 
    one NLLB achieves.
* Brevity_penalty: The brevity penalty is primarily an internal 
    multiplier that affects the overall BLEU score, penalizing translations 
    that contain fewer words than the reference text. Values of 1 or 
    higher indicate that the translated text contains more words than the 
    reference text. NLLB has a brevity penalty of 0.98, while Google has 
    a penalty of 0.93. This tells us two things: first, that NLLB has created 
    a translation with a length very similar to the reference, and second, 
    that if Google had used a couple more words, its score would have 
    been even better, further surpassing NLLB.
* Length_ratio: Indicates the relationship between the length of the 
    generated text and the reference texts. The closer it is to 1, the more 
    similar the lengths of the texts are. It is calculated using translation_
    length and reference_length.
