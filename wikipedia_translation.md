# Translate Wikipedia for Language-Finetuning

The entirety of Wikipedias articles is [available as a dataset on HF](https://huggingface.co/datasets/wikipedia), split into each different language.

The idea for creating a dataset to "teach" an already trained LLM a new language is to feed it mixed translation data between the two languages and additionally high-quality data in the target language.

## The Wikipedia Dataset
A data instace of the wikipedia dataset looks as follows:
```txt
{'id': '1',
 'url': 'https://simple.wikipedia.org/wiki/April',
 'title': 'April',
 'text': 'April is the fourth month...'
}
```

The dataset can be loaded with HuggingFace using:
```python
from datasets import load_dataset

load_dataset("wikipedia", "20220301.en")
# load_dataset("wikipedia", "20220301.de")
```

## Use NLLB for translation
[No Language Left Behind](https://research.facebook.com/publications/no-language-left-behind/) (available [on GitHub](https://github.com/facebookresearch/fairseq/tree/nllb/)) is a very strong translation model. It can be used to translate each article from source language (en) to target language (de).

An example using HuggingFace:
```python
from transformers import AutoModelForSeq2SeqLM, AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained("facebook/nllb-200-distilled-600M", use_auth_token=True)
model = AutoModelForSeq2SeqLM.from_pretrained("facebook/nllb-200-distilled-600M", use_auth_token=True)

article = "UN Chief says there is no military solution in Syria"
inputs = tokenizer(article, return_tensors="pt")

translated_tokens = model.generate(
    **inputs, forced_bos_token_id=tokenizer.lang_code_to_id["deu_Latn"], max_length=30
)
tokenizer.batch_decode(translated_tokens, skip_special_tokens=True)[0]
>>> "UN-Chef sagt, es gibt keine militärische Lösung in Syrien"
```

## Data processing
Some decisions regarding the processing of the data for the final dataset must be made.
- Consider context length
  - Given 2048 context length, most articles don't fit, let alone an article and its translation.
  - Articles need to be split somehow (ideally for a specific target context length)
  - Provide version with split articles for different lenghts?
- Language balancing necessary?
  - en wikipedia is larger than de wikipedia, i.e. en->de would be larger than de->en translation
  - remove some of en->de?
  - augment de->en with en->de (reversed)?