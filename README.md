# Filipino-Text-Benchmarks
This repository contains open-source benchmark datasets and pretrained transformer models in the low-resource Filipino language.

---

# Update: 2024/08/26
This repository is **no longer maintained**. 

All datasets ([fake news](https://huggingface.co/datasets/jcblaise/fake_news_filipino), [hatespeech](https://huggingface.co/datasets/jcblaise/hatespeech_filipino), etc.) and models ([roberta-tagalog](https://huggingface.co/jcblaise/roberta-tagalog-base), [electra-tagalog](https://huggingface.co/jcblaise/electra-tagalog-base-cased-discriminator), etc.) have been moved to my HuggingFace profile ([@jcblaise](https://huggingface.co/jcblaise)). The previous download links for resources have also been deprecated in favor of hosting from HF. I have made sure that the files are properly being served from HF now. If there is an issue, please contact me directly via my email.

---

<img src="assets/banner.png" width="70%">

Resources and code released in this repository come from the following papers, with more to be added as they are released:
1. Investigating the True Performance of Transformers in Low-Resource Languages: A Case Study in Automatic Corpus Creation [(Cruz et al., 2020)](https://arxiv.org/abs/2010.11574) 
2. Establishing Baselines for Text Classification in Low-Resource Languages [(Cruz & Cheng, 2020)](https://arxiv.org/abs/2005.02068)
3. Evaluating Language Model Finetuning Techniques for Low-resource Languages [(Cruz & Cheng, 2019)](https://arxiv.org/abs/1907.00409)

*This repository is a continuous work in progress!*

### Table of Contents

* [Requirements](https://github.com/jcblaisecruz02/Filipino-Text-Benchmarks#requirements)
* [Reproducing Results](https://github.com/jcblaisecruz02/Filipino-Text-Benchmarks#reproducing-results)
    * [Sentence Classification Tasks](https://github.com/jcblaisecruz02/Filipino-Text-Benchmarks#sentence-classification-tasks)
    * [Sentence-Pair Classification Tasks](https://github.com/jcblaisecruz02/Filipino-Text-Benchmarks#sentence-pair-classification-tasks)
    * [Logging Results](https://github.com/jcblaisecruz02/Filipino-Text-Benchmarks#logging-results)
* [Hyperparameter Search](https://github.com/jcblaisecruz02/Filipino-Text-Benchmarks#hyperparameter-search)
* [Demos](https://github.com/jcblaisecruz02/Filipino-Text-Benchmarks#demos)
* [Released Datasets](https://github.com/jcblaisecruz02/Filipino-Text-Benchmarks#datasets)
* [Pretrained ELECTRA Models](https://github.com/jcblaisecruz02/Filipino-Text-Benchmarks#pretrained-electra-models)
* [Pretrained BERT Models](https://github.com/jcblaisecruz02/Filipino-Text-Benchmarks#pretrained-bert-models)
* [Other Pretrained Models](https://github.com/jcblaisecruz02/Filipino-Text-Benchmarks#other-pretrained-models)
* [Citations](https://github.com/jcblaisecruz02/Filipino-Text-Benchmarks#citations)
* [Related Repositories](https://github.com/jcblaisecruz02/Filipino-Text-Benchmarks#related-repositories)
* [Contributing and Acknowledgements](https://github.com/jcblaisecruz02/Filipino-Text-Benchmarks#contributions-and-acknowledgements)

# Requirements
* PyTorch v.1.x
* [HuggingFace Transformers v.4.0.0](https://huggingface.co/transformers/index.html)
* [Weight and Biases](https://www.wandb.com/) (Used for logging and hyperparameter search. Optional.)
* [pytorch-lamb](https://github.com/cybertronai/pytorch-lamb)(Optional) for using LAMB during finetuning.
* NVIDIA GPU (experiments are ran using P100 and V100 GPUs)

# Reproducing Results
First, download the data and put it in the cloned repository:

```bash
mkdir Filipino-Text-Benchmarks/data

# Hatespeech Dataset
wget https://s3.us-east-2.amazonaws.com/blaisecruz.com/datasets/hatenonhate/hatespeech_raw.zip
unzip hatespeech_raw.zip -d Filipino-Text-Benchmarks/data && rm hatespeech_raw.zip

# Dengue Dataset
wget https://s3.us-east-2.amazonaws.com/blaisecruz.com/datasets/dengue/dengue_raw.zip
unzip dengue_raw.zip -d Filipino-Text-Benchmarks/data && rm dengue_raw.zip

# NewsPH-NLI Dataset
wget https://s3.us-east-2.amazonaws.com/blaisecruz.com/datasets/newsph/newsph-nli.zip
unzip newsph-nli.zip -d Filipino-Text-Benchmarks/data && rm newsph-nli.zip
```

### Sentence Classification Tasks

To finetune for sentence classification tasks, use the ```train.py``` script provided in this repository. Here's an example finetuning a Tagalog ELECTRA model on the Hatespeech dataset:

```bash
export DATA_DIR='Filipino-Text-Benchmarks/data/hatespeech'

python Filipino-Text-Benchmarks/train.py \
    --pretrained jcblaise/electra-tagalog-small-cased-discriminator \
    --train_data ${DATA_DIR}/train.csv \
    --valid_data ${DATA_DIR}/valid.csv \
    --test_data ${DATA_DIR}/test.csv \
    --data_pct 1.0 \
    --checkpoint finetuned_model \
    --do_train true \
    --do_eval true \
    --msl 128 \
    --optimizer adam \
    --batch_size 32 \
    --add_token [LINK],[MENTION],[HASHTAG] \
    --weight_decay 1e-8 \
    --learning_rate 2e-4 \
    --adam_epsilon 1e-6 \
    --warmup_pct 0.1 \
    --epochs 3 \
    --seed 42
```

This should give you the following results: 
```
Valid Loss 0.5272
Valid Acc 0.7568
Test Loss 0.3366
Test Accuracy 0.8649
```

The script will also output checkpoints of the finetuned model at the end of every epoch. These checkpoints can directly be used in a HuggingFace Transformer `pipeline` or can be loaded via the Transformers package for testing.

To perform multiclass classification, specify the label column names with the ```--label_column``` option. Here's an example finetuning a Tagalog ELECTRA model on the Dengue dataset:

```bash
export DATA_DIR='Filipino-Text-Benchmarks/data/dengue'

python Filipino-Text-Benchmarks/train.py \
    --pretrained jcblaise/electra-tagalog-small-uncased-discriminator \
    --train_data ${DATA_DIR}/train.csv \
    --valid_data ${DATA_DIR}/valid.csv \
    --test_data ${DATA_DIR}/test.csv \
    --label_columns absent,dengue,health,mosquito,sick \
    --data_pct 1.0 \
    --checkpoint finetuned_model \
    --do_train true \
    --do_eval true \
    --msl 128 \
    --optimizer adam \
    --batch_size 32 \
    --add_token [LINK],[MENTION],[HASHTAG] \
    --weight_decay 1e-8 \
    --learning_rate 2e-4 \
    --adam_epsilon 1e-6 \
    --warmup_pct 0.1 \
    --epochs 3 \
    --seed 42
```

This should give you the following results: 
```
Valid Loss 0.1586
Valid Acc 0.9414
Test Loss 0.1662
Test Accuracy 0.9375
```

For more information, run ```train.py --help``` for details on each command line argument.

### Sentence-Pair Classification Tasks

To finetune for sentence-pair classification (entailment datasets), you can specify the text column names using the ```--text_column``` option. Here's an example finetuning an uncased Tagalog ELECTRA model on the NewsPH-NLI dataset:

```bash
export DATA_DIR='Filipino-Text-Benchmarks/data/newsph-nli'

python Filipino-Text-Benchmarks/train.py \
    --pretrained jcblaise/electra-tagalog-small-uncased-discriminator \
    --train_data ${DATA_DIR}/train.csv \
    --valid_data ${DATA_DIR}/valid.csv \
    --test_data ${DATA_DIR}/test.csv \
    --text_columns s1,s2 \
    --data_pct 1.0 \
    --checkpoint finetuned_model \
    --do_train true \
    --do_eval true \
    --msl 128 \
    --optimizer adam \
    --batch_size 32 \
    --weight_decay 1e-8 \
    --learning_rate 2e-4 \
    --adam_epsilon 1e-6 \
    --warmup_pct 0.1 \
    --epochs 3 \
    --seed 45
```

This should give you the following results:
```
Valid Loss 0.1846
Valid Acc 0.9299
Test Loss 0.1874
Test Accuracy 0.9292
```

### Logging Results

We use [Weights & Biases](https://www.wandb.com/) to log experiment results. To use Weights & Biases, you need to toggle three command line arguments:

```bash
export DATA_DIR='Filipino-Text-Benchmarks/data/hatespeech'

python Filipino-Text-Benchmarks/train.py \
    --pretrained jcblaise/electra-tagalog-small-uncased-discriminator \
    ...
    --use_wandb \
    --wandb_username USERNAME \
    --wandb_project_name PROJECT_NAME

```

Replace ```USERNAME``` with your Weights & Biases username, and the ```PROJECT_NAME``` with the name of the project you're logging in to.

# Hyperparameter Search
To reproduce hyperparameter search results, you need to use the sweep function of [Weights & Biases](https://www.wandb.com/). For an example sweep, a ```sample_sweep.yaml``` file is included. This configuration sweeps for good random seeds on the Hatespeech dataset. Edit the file to your specifications as needed. For more information, check the [documentation](https://docs.wandb.com/sweeps/configuration).

To start a sweep, make sure to login first via the terminal, then run:

```bash
wandb sweep -p PROJECT_NAME Filipino-Text-Benchmarks/sample_sweep.yaml
```

This creates a sweep agent that you can run. Perform the sweep by running:

```bash
cd Filipino-Text-Benchmarks && wandb agent USERNAME/PROJECT_NAME/SWEEP_ID
```

where ```SWEEP_ID``` is the id generated by the ```wandb sweep``` command, and ```USERNAME``` is your W&B username. Make sure that you have the necessary data files in the ```data/``` folder inside the repository when running this example sweep.

# Demos

## Finetuned Entailment Model Demo
We provide a finetuned version of the small-uncased ELECTRA model for the NewsPH-NLI sentence entailment task for demo purposes. Here's how to load it to make predictions:

```python
import torch
from transformers import AutoTokenizer, AutoModelForSequenceClassification

# Load the finetuned model
pretrained = 'jcblaise/electra-tagalog-small-uncased-discriminator-newsphnli'
tokenizer = AutoTokenizer.from_pretrained(pretrained)
model = AutoModelForSequenceClassification.from_pretrained(pretrained)

# Entailment Example
s1 = "Ayon sa mga respondents, nahihirapan daw ang pag-rescue sa mga biktima dahil sa flash flood."
s2 = "Kumuha ng tulong ang respondents sa Philippine Red Cross para sa mga lifeboat."

tokens = tokenizer([(s1, s2)], padding='max_length', truncation='longest_first', max_length=128, return_tensors='pt')
with torch.no_grad():
    out = model(**tokens)[0]
pred = out.argmax(1).item() # Outputs "0" which means "entailment"

# Contradiction example
s1 = "Ayon sa mga respondents, nahihirapan daw ang pag-rescue sa mga biktima dahil sa flash flood."
s2 = "Nagpulong ang mga guro sa National High School para sa papadating na Bridaga Eskwela"

tokens = tokenizer([(s1, s2)], padding='max_length', truncation='longest_first', max_length=128, return_tensors='pt')
with torch.no_grad():
    out = model(**tokens)[0]
pred = out.argmax(1).item() # Outputs "1" which means "contradiction"
```

## Using HuggingFace Pipelines
You can directly use the BERT and ELECTRA models in pipelines. Here's an example for mask filling:

```python
from transformers import pipeline

pipe = pipeline('fill-mask', model='jcblaise/electra-tagalog-base-cased-generator')
s = "Ito ang Pilipinas, ang aking [MASK] Hinirang"
pipe(s)

# [{'score': 0.9990490078926086,
#   'sequence': '[CLS] Ito ang Pilipinas, ang aking Lupang Hinirang [SEP]',
#   'token': 21327,
#   'token_str': 'Lupang'},
#  ...

```

Here's the same example but without using pipelines:

```python
from transformers import AutoTokenizer, AutoModelForMaskedLM

# Load the pretrained model
pretrained = 'jcblaise/electra-tagalog-base-cased-generator'
tokenizer = AutoTokenizer.from_pretrained(pretrained)
model = AutoModelForMaskedLM.from_pretrained(pretrained)

s = "Ito ang Pilipinas, ang aking [MASK] Hinirang"
inputs = tokenizer(s, return_tensors='pt')
with torch.no_grad():
    out = model(**inputs)

pred_ix = out['logits'][0, 7].argmax() 
pred = tokenizer.convert_ids_to_tokens([pred_ix])[0] # Outputs "Lupang"

```

# Datasets
* **NewsPH-NLI Dataset** [`download`](https://s3.us-east-2.amazonaws.com/blaisecruz.com/datasets/newsph/newsph-nli.zip)\
*Sentence Entailment Dataset in Filipino* \
First benchmark dataset for sentence entailment in the low-resource Filipino language. Constructed through exploting the structure of news articles. Contains 600,000 premise-hypothesis pairs, in 70-15-15 split for training, validation, and testing. Originally published in [(Cruz et al., 2020)](https://arxiv.org/abs/2010.11574).

* **WikiText-TL-39** [`download`](https://s3.us-east-2.amazonaws.com/blaisecruz.com/datasets/wikitext-tl-39/wikitext-tl-39.zip)\
*Large Scale Unlabeled Corpora in Filipino*\
Large scale, unlabeled text dataset with 39 Million tokens in the training set. Inspired by the original [WikiText Long Term Dependency dataset](https://blog.einstein.ai/the-wikitext-long-term-dependency-language-modeling-dataset/) (Merity et al., 2016). TL means "Tagalog." Originally published in [Cruz & Cheng (2019)](https://arxiv.org/abs/1907.00409).

* **Hate Speech Dataset** [`download`](https://s3.us-east-2.amazonaws.com/blaisecruz.com/datasets/hatenonhate/hatespeech_raw.zip)\
*Text Classification Dataset in Filipino* \
Contains 10k tweets (training set) that are labeled as hate speech or non-hate speech. Released with 4,232 validation and 4,232 testing samples. Collected during the 2016 Philippine Presidential Elections and originally used in Cabasag et al. (2019).

* **Dengue Dataset** [`download`](https://s3.us-east-2.amazonaws.com/blaisecruz.com/datasets/dengue/dengue_raw.zip)\
*Low-Resource Multiclass Text Classification Dataset in Filipino*\
Benchmark dataset for low-resource multiclass classification, with 4,015 training, 500 testing, and 500 validation examples, each labeled as part of five classes. Each sample can be a part of multiple classes. Collected as tweets and originally used in Livelo & Cheng (2018).

# Pretrained ELECTRA Models
We release new ELECTRA models in small and base configurations, with both the discriminator and generators available. All the models follow the same setups and were trained with the same hyperparameters as English ELECTRA models. Our models are available on HuggingFace Transformers and can be used on both PyTorch and Tensorflow. These models were released as part of [(Cruz et al., 2020)](https://arxiv.org/abs/2010.11574).

**Discriminator Models**

* ELECTRA Base Cased Discriminator - [`jcblaise/electra-tagalog-base-cased-discriminator`](https://huggingface.co/jcblaise/electra-tagalog-base-cased-discriminator) 
* ELECTRA Base Uncased Discriminator - [`jcblaise/electra-tagalog-base-uncased-discriminator`](https://huggingface.co/jcblaise/electra-tagalog-base-uncased-discriminator) 
* ELECTRA Small Cased Discriminator - [`jcblaise/electra-tagalog-small-cased-discriminator`](https://huggingface.co/jcblaise/electra-tagalog-small-cased-discriminator) 
* ELECTRA Small Uncased Discriminator - [`jcblaise/electra-tagalog-small-uncased-discriminator`](https://huggingface.co/jcblaise/electra-tagalog-small-uncased-discriminator)

**Generator Models**

* ELECTRA Base Cased Generator - [`jcblaise/electra-tagalog-base-cased-generator`](https://huggingface.co/jcblaise/electra-tagalog-base-cased-generator) 
* ELECTRA Base Uncased Generator - [`jcblaise/electra-tagalog-base-uncased-generator`](https://huggingface.co/jcblaise/electra-tagalog-base-uncased-generator) 
* ELECTRA Small Cased Generator - [`jcblaise/electra-tagalog-small-cased-generator`](https://huggingface.co/jcblaise/electra-tagalog-small-cased-generator)
* ELECTRA Small Uncased Generator - [`jcblaise/electra-tagalog-small-uncased-generator`](https://huggingface.co/jcblaise/electra-tagalog-small-uncased-generator)

The models can be loaded using the code below:

```Python
from transformers import TFAutoModel, AutoModel, AutoTokenizer

# TensorFlow
model = TFAutoModel.from_pretrained('jcblaise/electra-tagalog-small-cased-generator', from_pt=True)
tokenizer = AutoTokenizer.from_pretrained('jcblaise/electra-tagalog-small-cased-generator')

# PyTorch
model = AutoModel.from_pretrained('jcblaise/electra-tagalog-small-cased-generator')
tokenizer = AutoTokenizer.from_pretrained('jcblaise/electra-tagalog-small-cased-generator')
```

# Pretrained BERT Models
We release four Tagalog BERT Base models and one Tagalog DistilBERT Base model. All the models use the same configurations as the original English BERT models. Our models are available on HuggingFace Transformers and can be used on both PyTorch and Tensorflow. These models were released as part of [Cruz & Cheng (2019)](https://arxiv.org/abs/1907.00409).

* BERT Base Cased - [`jcblaise/bert-tagalog-base-cased`](https://huggingface.co/jcblaise/bert-tagalog-base-cased) 
* BERT Base Uncased - [`jcblaise/bert-tagalog-base-uncased`](https://huggingface.co/jcblaise/bert-tagalog-base-uncased) 
* BERT Base Cased WWM - [`jcblaise/bert-tagalog-base-cased-WWM`](https://huggingface.co/jcblaise/bert-tagalog-base-cased-WWM) 
* BERT Base Uncased WWM - [`jcblaise/bert-tagalog-base-uncased-WWM`](https://huggingface.co/jcblaise/bert-tagalog-base-uncased-WWM) 
* DistilBERT Base Cased - [`jcblaise/distilbert-tagalog-base-cased`](https://huggingface.co/jcblaise/distilbert-tagalog-base-cased) 

The models can be loaded using the code below:

```Python
from transformers import TFAutoModel, AutoModel, AutoTokenizer

# TensorFlow
model = TFAutoModel.from_pretrained('jcblaise/bert-tagalog-base-cased', from_pt=True)
tokenizer = AutoTokenizer.from_pretrained('jcblaise/bert-tagalog-base-cased')

# PyTorch
model = AutoModel.from_pretrained('jcblaise/bert-tagalog-base-cased')
tokenizer = AutoTokenizer.from_pretrained('jcblaise/bert-tagalog-base-cased')
```

# Other Pretrained Models
* **ULMFiT-Tagalog** [`download`](https://github.com/danjohnvelasco/Filipino-ULMFiT)\
Tagalog pretrained AWD-LSTM compatible with v2 of the FastAI library. Originally published in [Velasco (2020)](https://arxiv.org/abs/2010.06447).

* **ULMFiT-Tagalog (Old)** [`download`](https://s3.us-east-2.amazonaws.com/blaisecruz.com/ulmfit-tagalog/models/pretrained-wikitext-tl-39.zip)\
Tagalog pretrained AWD-LSTM compatible with the FastAI library. Originally published in [Cruz & Cheng (2019)](https://arxiv.org/abs/1907.00409).

# Citations
If you found our work useful, please make sure to cite!

```
@article{cruz2020investigating,
  title={Investigating the True Performance of Transformers in Low-Resource Languages: A Case Study in Automatic Corpus Creation}, 
  author={Jan Christian Blaise Cruz and Jose Kristian Resabal and James Lin and Dan John Velasco and Charibeth Cheng},
  journal={arXiv preprint arXiv:2010.11574},
  year={2020}
}

@article{cruz2020establishing,
  title={Establishing Baselines for Text Classification in Low-Resource Languages},
  author={Cruz, Jan Christian Blaise and Cheng, Charibeth},
  journal={arXiv preprint arXiv:2005.02068},
  year={2020}
}

@article{cruz2019evaluating,
  title={Evaluating Language Model Finetuning Techniques for Low-resource Languages},
  author={Cruz, Jan Christian Blaise and Cheng, Charibeth},
  journal={arXiv preprint arXiv:1907.00409},
  year={2019}
}
```

# Related Repositories
Repositories for adjacent projects and papers made by our team that use the models found here:
* [jcblaisecruz02/Tagalog-fake-news](https://github.com/jcblaisecruz02/Tagalog-fake-news) - Automatic low-resource fake news detection in Filipino.
* [danjohnvelasco/Filipino-ULMFiT](https://github.com/danjohnvelasco/Filipino-ULMFiT) - ULMFiT benchmarks on our released classification benchmark datasets.

# Contributions and Acknowledgements
Should you find any bugs or have any suggestions, feel free to drop by the Issues tab! We'll get back to you as soon as we can.

*This repository is managed by the De La Salle University Machine Learning Group*

<img src="assets/full_logo.png" width="30%">
