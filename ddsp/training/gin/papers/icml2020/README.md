# Self-supervised Pitch Detection by Inverse Audio Synthesis
_ICML SAS Workshop 2020, ([paper](https://openreview.net/forum?id=RlVTYWhsky7))_

Gin configs for reproducing the results of the paper.

## Generate synthetic dataset

After pip installing ddsp, create a synthetic dataset for the transcribing autoencoder using the installed script.
The command below creates a very small dataset as a test.

```bash
ddsp_generate_synthetic_dataset \
--output_tfrecord_path=/tmp/synthetic_data.tfrecord \
--gin_file=papers/icml2020/generate_synthetic_dataset.gin \
--num_shards=2 \
--num_examples=1000 \
--alsologtostderr
```

The synthetic dataset in the paper has 10,000,000 examples in 1000 shards, which may require runinng in a distributed setup such as [Dataflow](https://cloud.google.com/dataflow) on GCP.
If running on DataFlow, you'll need to set the `--pipeline_options` flag using
the execution parameters described at
https://cloud.google.com/dataflow/docs/guides/specifying-exec-params
E.g., `--pipeline_options=--runner=DataFlowRunner,--project=<my-gcp-project-id>,--temp_location=gs://<bucket/directory>,--region=us-central1,--job_name=<my-name-of-job>`.

## Pretrain transcribing autoencoder

### Train
Point the dataset file_pattern to the synthetic dataset created above.

```bash
ddsp_run \
--mode=train \
--save_dir=/tmp/$USER-tae-pretrain-0 \
--gin_file=papers/icml2020/pretrain_model.gin \
--gin_file=papers/icml2020/pretrain_dataset.gin \
--gin_param="SyntheticNotes.file_pattern='gs://ddsp-inv/datasets/notes_t125_h100_m65_v2.tfrecord*'" \
--gin_param="batch_size=32" \
--alsologtostderr
```

## Finetuning transcribing autoencoder

### Train
Point the dataset file_pattern to the synthetic dataset created above.

```bash
ddsp_run \
--mode=train \
--save_dir=/tmp/$USER-tae-finetune-0 \
--gin_file=papers/icml2020/finetune_model.gin \
--gin_file=papers/icml2020/finetune_dataset.gin \
--gin_param="SyntheticNotes.file_pattern='gs://ddsp-inv/datasets/notes_t125_h100_m65_v2.tfrecord*'" \
--gin_param="batch_size=8" \
--alsologtostderr
```


