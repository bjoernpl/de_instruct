# German Language LLM Instruction-Tuning

This README is an overview of ideas concerning finetuning LLMs to learn German instruction data.

## Brainstorming:
### Related work
- [`instruct-igel-001` ](https://huggingface.co/philschmid/instruct-igel-001) IGEL is built on top [BigScience BLOOM](https://bigscience.huggingface.co/blog/bloom), with naive-translated instruct dataset. [SnipAId](https://www.snipaid.tech) successfully use igel for news article summarization.
- [German BLOOM](https://huggingface.co/malteos/bloom-6b4-clp-german) finetuned on crawled german data.
- [Guanaco](https://huggingface.co/JosephusCheung/Guanaco) Llama 7b finetuned on [Guanaco Dataset](https://huggingface.co/datasets/JosephusCheung/GuanacoDataset) (chat/instructions in en, de, ja, zh)
- [EuroInstructProject](https://github.com/LEL-A/EuroInstructProject) german instruction dataset


### The model
There a few valid model choices. Most applicable is likely to be [Llama](https://github.com/facebookresearch/llama)
in the 60B parameter variant. 30B might also be sufficient.

Other options include [Pythia](https://github.com/EleutherAI/pythia), [StableLM](https://github.com/Stability-AI/StableLM), [Cerebras](https://www.cerebras.net), [GPT-NeoX-20B](https://github.com/EleutherAI/gpt-neox), ...

### Finetuning on mixed translation / high-quality data
To best enable knowledge transfer, an idea is to give the model some mix of data of approximately:
- 1/3 english -> german translated text (i.e. pairs of english and corresponding german text)
- 1/3 german -> english translated text
- 1/3 high-quality german texts (german wikipedia or gutenberg)

> How to obtain this data?

Use open-source translation model ([No Language Left Behind](https://github.com/facebookresearch/fairseq/tree/nllb/)) to translate english [wikipedia](https://huggingface.co/datasets/wikipedia) to german (and maybe vice-versa to ensure high-quality german text).

### Finetuning on an instruct dataset

After language training, finetune the model on a diverse instruct dataset. Some thoughts:
- meta-tags for conditional generation (funny, sad, pirate, angry, etc.)
- ...

### Multimodal support
Inspired by [MiniGPT4](https://github.com/Vision-CAIR/MiniGPT-4) and with architecture/training considerations from [Microsoft KOSMOS-1](https://arxiv.org/abs/2302.14045) adding image support is feasible. 
- Use pretrained (optionally frozen) LLM (as described above)
- Use pretrained (frozen) image patch encoder ([ViT](https://github.com/google-research/vision_transformer), [CLIP](https://github.com/mlfoundations/open_clip), [DINOv2](https://github.com/facebookresearch/dinov2), etc.)
- Send image features through 1 linear layer and then into model
- Train on interleaved data. Alternatively use mix of interleaved, image-text pair and text data according to KOSMOS-1.

