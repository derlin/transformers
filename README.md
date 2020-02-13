Fork of pytorch-pretrained-BERT (now transformers) at commit `78462aad6113d50063d8251e27dbaadb7f44fbf0`.

Note: this is targeted at **BERT classification tasks**, i.e. fine-tuning a BERT model for language identification. 


## Additions / Improvements

Main changes:

1. ability to freeze BERT layers (hence training only the classifier on top);
2. ability to load data from a CSV file + specify label weights (see below);
3. ability to run the classifier using the bash command `bert_finetune_lid` after install (new entrypoint).


## Usage

**train.csv**

The CSV file with the training data should have two mandatory columns (others are ignored):
* `text`: the input examples,
* `label`: the true label (i.e. the "language").

**environ.txt**

A file defining the labels and label weights (optional):
```bash
export BERT_LABELS="afr,bar,deu,est,frr,grb,gsw,knn,ksh,lim,ltz,nds,pfl,som,swa-swh,zea-vls-nld"
# the below line is needed only if you have unbalanced classes. If not, comment it !
export BERT_LABELS_WEIGHTS="0.1812,0.2648,0.1812,0.1812,0.8824,0.1812,0.1812,1.0,0.8208,0.1812,0.1812,0.1812,0.8334,0.1812,0.2222,0.1812"
```

Alternatively, you can just run those commands in the terminal before calling `bert_finetune_lid`.
This file is only necessary if you plan to use the script below.

**running the classifier**

What is important:

* ensure you "source" the `environ.txt` file, or set the environment variable `BERT_LABELS`;
* ensure you use the `--task_name` is set to `pandas`;
* ensure the `--data-dir` points to the directory where `train.csv` is located;
* the rest is up to you.

Here is an example script:

```bash
#!/usr/bin/env bash

# Data parameters
BASE_DIR="$PWD"
DATA_DIR_BERT="$BASE_DIR/bert" # where are data.csv and environ.txt located
OUTPUT_DIR="$BASE_DIR/out" # where to export the model after fine-tuning
mkdir -p $OUTPUT_DIR # create the output dir if not exist

# Model parameters
BASE_MODEL='bert-base-german-cased'
MAX_SEQ_LENGTH=64

cp $DATA_DIR_BERT/*.* $OUTPUT_DIR # copy for reference
. $OUTPUT_DIR/environ.txt         # source environ.txt

echo "Using LABELS  $BERT_LABELS"
echo "Using WEIGHTS $BERT_LABELS_WEIGHTS"

# Train parameters
BATCH_SIZE=32
LEARNING_RATE=2e-5
NUM_TRAIN_EPOCHS=1.0

export PYTHONUNBUFFERED=1

# launch
command="bert_finetune_lid \
  --task_name "pandas" \
  --do_train \
  --data_dir $OUTPUT_DIR \
  --bert_model $BASE_MODEL \
  --max_seq_length $MAX_SEQ_LENGTH \
  --train_batch_size $BATCH_SIZE \
  --eval_batch_size $BATCH_SIZE \
  --learning_rate $LEARNING_RATE \
  --num_train_epochs $NUM_TRAIN_EPOCHS \
  --output_dir $OUTPUT_DIR/bert"
  
  echo -e "USING COMMAND:\n$command\n" | tee "$OUTPUT_DIR/train_output.log"
  $command |& tee -a "$OUTPUT_DIR/train_output.log"
```