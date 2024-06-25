---
sidebar_label: Stable Diffusion -CKPT
sidebar_position: 4
---

# Stable Diffusion Checkpoint Inference

[![stars - badge-generator](https://img.shields.io/github/stars/bacalhau-project/bacalhau?style=social)](https://github.com/bacalhau-project/bacalhau)

## Introduction

[Stable Diffusion](https://github.com/CompVis/stable-diffusion) is a state of the art text-to-image model that generates images from text and was developed as an open-source alternative to [DALL·E 2](https://openai.com/dall-e-2/). It is based on a [Diffusion Probabilistic Model](https://arxiv.org/abs/2102.09672) and uses a [Transformer](https://arxiv.org/abs/1706.03762) to generate images from text.

This example demonstrates how to use stable diffusion using a finetuned model and run inference on it. The first section describes the development of the code and the container - it is optional as users don't need to build their own containers to use their own custom model. The second section demonstrates how to convert your model weights to ckpt. The third section demonstrates how to run the job using Bacalhau.

## TD;LR

Running fine-tuned stable diffusion model converted to ckpt on Bacalhau.

## Prerequisite

To get started, you need to install:

* Bacalhau client, see more information [here](https://docs.bacalhau.org/getting-started/installation)
* NVIDIA GPU
* CUDA drivers
* NVIDIA docker

## Running Locally

The following image is an example generated by the fine-tuned model, it was finetuned on Bacalhau to learn how to finetune your own stable diffusion model refer https://docs.bacalhau.org/examples/model-training/Stable-Diffusion-Dreambooth

## Containerize your Script using Docker

:::tip You can skip this step and go straight to running a Bacalhau job :::

To build your own docker container, create a `Dockerfile`, which contains instructions to containerize the code for inference.

```Dockerfile
FROM  pytorch/pytorch:1.13.0-cuda11.6-cudnn8-runtime

WORKDIR /

RUN apt update &&  apt install -y git

RUN git clone https://github.com/runwayml/stable-diffusion.git

WORKDIR /stable-diffusion

RUN conda env create -f environment.yaml

SHELL ["conda", "run", "-n", "ldm", "/bin/bash", "-c"]

RUN pip install opencv-python

RUN apt update

RUN apt-get install ffmpeg libsm6 libxext6 libxrender-dev  -y
```

This container is using the `pytorch/pytorch:1.13.0-cuda11.6-cudnn8-runtime` image and the working directory is set. Next the Dockerfile installs the same dependencies from earlier in this notebook. Then we add our custom code and pull the dependent repositories.

:::info See more information on how to containerize your script/app [here](https://docs.docker.com/get-started/02\_our\_app/) :::

### Build the container

We will run `docker build` command to build the container;

```
docker build -t hub-user/repo-name:tag .
```

Before running the command replace;

* **hub-user** with your docker hub username, If you don’t have a docker hub account [follow these instructions to create the Docker account](https://docs.docker.com/docker-id/), and use the username of the account you created
* **repo-name** with the name of the container, you can name it anything you want
* **tag** this is not required but you can use the latest tag

In our case

```bash
docker build -t jsacex/stable-diffusion-ckpt
```

### Push the container

Next, upload the image to the registry. This can be done by using the Docker hub username, repo name or tag.

```
docker push <hub-user>/<repo-name>:<tag>
```

In our case

```bash
docker push jsacex/stable-diffusion-ckpt
```

After the repo image has been pushed to Docker Hub, you can now use the container for running on Bacalhau but before that you need to check whether your model is a ckpt file or not, if your model is a ckpt file you can skip to the running on Bacalhau if not the next section describes how to convert your model into the ckpt format.

## Converting model weights to CKPT

To download the convert script:

```
wget -q https://github.com/TheLastBen/diffusers/raw/main/scripts/convert_diffusers_to_original_stable_diffusion.py
```

To convert the model weights into CKPT format, the `--half` flag cuts the size of the output model from 4GB to 2GB:

```
python convert_diffusers_to_original_stable_diffusion.py --model_path <path-to-the-model-weights>  --checkpoint_path <path-to-save-the-checkpoint>/model.ckpt --half
```

## Running a Bacalhau Job

To do inference on your own checkpoint on Bacalhau you need to first upload it to your public storage, which can be mounted anywhere on your machine. In this case, we will be using [NFT.Storage](https://nft.storage/) (Recommended Option). To upload your dataset using [NFTup](https://nft.storage/docs/how-to/nftup/) just drag and drop your directory it will upload it to IPFS

After the checkpoint file has been uploaded copy its CID.

To submit a job, run the following Bacalhau command:

```bash
%%bash --out job_id
bacalhau docker run \
--gpu 1 \
--timeout 3600 \
--wait-timeout-secs 3600 \
--wait \
--id-only \
-i ipfs://QmUCJuFZ2v7KvjBGHRP2K1TMPFce3reTkKVGF2BJY5bXdZ:/DavidAronchick.ckpt \
jsacex/stable-diffusion-ckpt \
-- conda run --no-capture-output -n ldm python scripts/txt2img.py --prompt "a photo of aronchick drinking coffee" --plms --ckpt ../DavidAronchick.ckpt --skip_grid --n_samples 1 --skip_grid --outdir ../outputs
```

### Structure of the command

Let's look closely at the command above:

* `--gpu` : here we request 1 GPU
* `-i ipfs://QmUCJuFZ2v7KvjBGHRP2K1TMPFce3reTkKVGF2BJY5bXdZ:/DavidAronchick.ckpt`: Path-to-mount-the-checkpoint
* `-- conda run --no-capture-output -n ldm`: since we are using conda we need to specify the name of the environment which we are going to use in this case its `ldm`
* `scripts/txt2img.py`: running the python script
* `--prompt "a photo of aronchick drinking coffee"`: the prompt you need to specify the session name in the prompt eg the session name here is aronchick
* `--plms`: the sampler you want to use in this case we will use the plms sampler
* `--ckpt ../DavidAronchick.ckpt`: and then specify the path to our checkpoint
* `--n_samples 1`: no of samples we want to produce
* `--skip_grid` : skip creating a grid of images
* `--outdir ../outputs`: path to store the outputs
* `--seed $RANDOM`: The output generated on the same prompt will always be the same for different outputs on the same prompt set the seed parameter to random

When a job is submitted, Bacalhau prints out the related `job_id`. We store that in an environment variable so that we can reuse it later on.

## Checking the State of your Jobs

* **Job status**: You can check the status of the job using `bacalhau list`.

```bash
%%bash
bacalhau list --id-filter ${JOB_ID} --wide
```

When it says `Published` or `Completed`, that means the job is done, and we can get the results.

* **Job information**: You can find out more information about your job by using `bacalhau describe`.

```bash
%%bash
bacalhau describe ${JOB_ID}
```

* **Job download**: You can download your job results directly by using `bacalhau get`. Alternatively, you can choose to create a directory to store your results. In the command below, we created a directory and downloaded our job output to be stored in that directory.

```bash
%%bash
rm -rf results && mkdir -p results
bacalhau get $JOB_ID --output-dir results
```

## Viewing your Job Output

To view the file, run the following command:

View the outputs:

```python
import IPython.display as display
display.Image("results/outputs/samples/00001.png")
```

![png](../../../examples/model-inference/Stable-Diffusion-CKPT-Inference/index\_files/index\_19\_0.png)