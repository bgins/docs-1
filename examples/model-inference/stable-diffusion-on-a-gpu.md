# Stable Diffusion on a GPU

This example tutorial demonstrates how to use Stable Diffusion on a GPU and run it on the [Bacalhau](https://www.bacalhau.org/) demo network. [Stable Diffusion](https://github.com/CompVis/stable-diffusion) is a state of the art text-to-image model that generates images from text and was developed as an open-source alternative to [DALL·E 2](https://openai.com/dall-e-2/). It is based on a [Diffusion Probabilistic Model](https://arxiv.org/abs/2102.09672) and uses a [Transformer](https://arxiv.org/abs/1706.03762) to generate images from text.

## TL;DR[​](http://localhost:3000/examples/model-inference/stable-diffusion-gpu/#tldr) <a href="#tldr" id="tldr"></a>

```bash
bacalhau docker run \
    --id-only \
    --gpu 1 \
    ghcr.io/bacalhau-project/examples/stable-diffusion-gpu:0.0.1 \
    -- python main.py --o ./outputs --p "meme about tensorflow"
```

## Prerequisite[​](http://localhost:3000/examples/model-inference/stable-diffusion-gpu/#prerequisite) <a href="#prerequisite" id="prerequisite"></a>

To get started, you need to install the Bacalhau client, see more information [here](http://localhost:3000/getting-started/installation)

## Quick Test[​](http://localhost:3000/examples/model-inference/stable-diffusion-gpu/#quick-test) <a href="#quick-test" id="quick-test"></a>

Here is an example of an image generated by this model.

```bash
bacalhau docker run \
    --gpu 1 \
    ghcr.io/bacalhau-project/examples/stable-diffusion-gpu:0.0.1 \
    -- python main.py --o ./outputs --p "cod swimming through data"
```

<figure><img src="../../.gitbook/assets/image0-97d9cc068b5879d6cc88f7ec68535eea.png" alt=""><figcaption></figcaption></figure>

## Development[​](http://localhost:3000/examples/model-inference/stable-diffusion-gpu/#development) <a href="#development" id="development"></a>

This stable diffusion example is based on the [Keras/Tensorflow implementation](https://github.com/fchollet/stable-diffusion-tensorflow). You might also be interested in the Pytorch oriented [diffusers library](https://github.com/huggingface/diffusers).

### Installing dependencies[​](http://localhost:3000/examples/model-inference/stable-diffusion-gpu/#installing-dependencies) <a href="#installing-dependencies" id="installing-dependencies"></a>

{% hint style="info" %}
When you run this code for the first time, it will download the pre-trained weights, which may add a short delay.
{% endhint %}

Based on the requirements [here](https://github.com/fchollet/stable-diffusion-tensorflow), we will install the following:

```bash
pip install git+https://github.com/fchollet/stable-diffusion-tensorflow --upgrade --quiet
pip install tensorflow tensorflow_addons ftfy --upgrade --quiet
pip install tqdm --upgrade
apt install --allow-change-held-packages libcudnn8=8.1.0.77-1+cuda11.2
```

### Testing the Code[​](http://localhost:3000/examples/model-inference/stable-diffusion-gpu/#testing-the-code) <a href="#testing-the-code" id="testing-the-code"></a>

We have a sample code from this the [Stable Diffusion in TensorFlow/Keras](https://github.com/fchollet/stable-diffusion-tensorflow) repo which we will use to check if the code is working as expected. Our output for this code will be a _DSLR photograph of an astronaut riding a horse_.

{% hint style="info" %}
When you run this code for the first time, it will download the pre-trained weights, which may add a short delay.
{% endhint %}

```python
from stable_diffusion_tf.stable_diffusion import Text2Image
from PIL import Image

generator = Text2Image( 
    img_height=512,
    img_width=512,
    jit_compile=False,  # You can try True as well (different performance profile)
)
img = generator.generate(
    "DSLR photograph of an astronaut riding a horse",
    num_steps=50,
    unconditional_guidance_scale=7.5,
    temperature=1,
    batch_size=1,
)
pil_img = Image.fromarray(img[0])
display(pil_img)
```

When running this code, if you check the GPU RAM usage, you'll see that it's sucked up many GBs, and depending on what GPU you're running, it may OOM (Out of memory) if you run this again.

You can try and reduce RAM usage by playing with batch sizes (although it is only set to 1 above!) or more carefully controlling the TensorFlow session.

To clear the GPU memory we will use **numba**. This won't be required when running in a single-shot manner.

```bash
pip install numba --upgrade
```

```python
# clearing the GPU memory 
from numba import cuda 
device = cuda.get_current_device()
device.reset()
```

### Write the Script[​](http://localhost:3000/examples/model-inference/stable-diffusion-gpu/#write-the-script) <a href="#write-the-script" id="write-the-script"></a>

You need a script to execute when we submit jobs. The code below is a slightly modified version of the code we ran above which we got from [here](https://github.com/fchollet/stable-diffusion-tensorflow), however, this includes more things such as argument parsing [argument parsing](https://github.com/fchollet/stable-diffusion-tensorflow/blob/master/text2image.py) to be able to customize the generator.

```python
#content of the main.py file

import argparse
from stable_diffusion_tf.stable_diffusion import Text2Image
from PIL import Image
import os
parser = argparse.ArgumentParser(description="Stable Diffusion")
parser.add_argument("--h",dest="height", type=int,help="height of the image",default=512)
parser.add_argument("--w",dest="width", type=int,help="width of the image",default=512)
parser.add_argument("--p",dest="prompt", type=str,help="Description of the image you want to generate",default="cat")
parser.add_argument("--n",dest="numSteps", type=int,help="Number of Steps",default=50)
parser.add_argument("--u",dest="unconditionalGuidanceScale", type=float,help="Number of Steps",default=7.5)
parser.add_argument("--t",dest="temperature", type=int,help="Number of Steps",default=1)
parser.add_argument("--b",dest="batchSize", type=int,help="Number of Images",default=1)
parser.add_argument("--o",dest="output", type=str,help="Output Folder where to store the Image",default="./")

args=parser.parse_args()
height=args.height
width=args.width
prompt=args.prompt
numSteps=args.numSteps
unconditionalGuidanceScale=args.unconditionalGuidanceScale
temperature=args.temperature
batchSize=args.batchSize
output=args.output

generator = Text2Image(
    img_height=height,
    img_width=width,
    jit_compile=False,  # You can try True as well (different performance profile)
)

img = generator.generate(
    prompt,
    num_steps=numSteps,
    unconditional_guidance_scale=unconditionalGuidanceScale,
    temperature=temperature,
    batch_size=batchSize,
)
for i in range(0,batchSize):
  pil_img = Image.fromarray(img[i])
  image = pil_img.save(f"{output}/image{i}.png")
```

{% hint style="info" %}
For a full list of arguments that you can pass to the script, see more information [here](../../references/cli-reference/all-flags.md)
{% endhint %}

### Run the Script[​](http://localhost:3000/examples/model-inference/stable-diffusion-gpu/#run-the-script) <a href="#run-the-script" id="run-the-script"></a>

After writing the code the next step is to run the script.

```bash
python3 main.py
```

As a result, you will get something like this:

<figure><img src="../../.gitbook/assets/index_18_0-cfecae1fe624c50676c9406eb9b637d7.png" alt=""><figcaption></figcaption></figure>

The following presents additional parameters you can try:

1. `python main.py --p "cat with three eyes` - to set prompt
2. `python main.py --p "cat with three eyes" --n 100` - to set the number of iterations to 100
3. `python stable-diffusion.py --p "cat with three eyes" --b 2` to set batch size to 2 (№ of images to generate)

## Containerize Script using Docker[​](http://localhost:3000/examples/model-inference/stable-diffusion-gpu/#containerize-script-using-docker) <a href="#containerize-script-using-docker" id="containerize-script-using-docker"></a>

Docker is the easiest way to run TensorFlow on a GPU since the host machine only requires the [NVIDIA® driver](https://github.com/NVIDIA/nvidia-docker/wiki/Frequently-Asked-Questions#how-do-i-install-the-nvidia-driver). To containerize the inference code, we will create a `Dockerfile`. The Dockerfile is a text document that contains the commands that specify how the image will be built.

```docker
FROM tensorflow/tensorflow:2.10.0-gpu

RUN apt-get -y update

RUN apt-get -y install --allow-change-held-packages libcudnn8=8.1.0.77-1+cuda11.2 git

RUN python3 -m pip install --upgrade pip

RUN python -m pip install regex tqdm Pillow tensorflow tensorflow_addons ftfy  --upgrade --quiet

RUN pip install git+https://github.com/fchollet/stable-diffusion-tensorflow --upgrade --quiet

ADD main.py main.py

# Run once so it downloads and caches the pre-trained weights
RUN python main.py --n 1
```

The Dockerfile leverages the latest official TensorFlow GPU image and then installs other dependencies like `git`, `CUDA` packages, and other image-related necessities. See [the original repository](https://github.com/fchollet/stable-diffusion-tensorflow/blob/master/requirements.txt) for the expected requirements.

{% hint style="info" %}
See more information on how to containerize your script/app [here](https://docs.docker.com/get-started/02\_our\_app/)
{% endhint %}

### Build the container[​](http://localhost:3000/examples/model-inference/stable-diffusion-gpu/#build-the-container) <a href="#build-the-container" id="build-the-container"></a>

We will run `docker build` command to build the container;

```bash
docker build -t <hub-user>/<repo-name>:<tag> .
```

Before running the command replace following:

1. **hub-user** with your docker hub username, If you don’t have a docker hub account follow [these instructions](https://docs.docker.com/docker-id/) to create a Docker account, and use the username of the account you created
2. **repo-name** with the name of the container, you can name it anything you want
3. **tag** this is not required but you can use the latest tag

In our case:

```bash
docker build -t ghcr.io/bacalhau-project/examples/stable-diffusion-gpu:0.0.1 .
```

### Push the container[​](http://localhost:3000/examples/model-inference/stable-diffusion-gpu/#push-the-container) <a href="#push-the-container" id="push-the-container"></a>

Next, upload the image to the registry. This can be done by using the Docker hub username, repo name or tag.

```bash
docker push <hub-user>/<repo-name>:<tag>
```

In our case:

```bash
docker push ghcr.io/bacalhau-project/examples/stable-diffusion-gpu:0.0.1 .
```

## Running a Bacalhau Job[​](http://localhost:3000/examples/model-inference/stable-diffusion-gpu/#running-a-bacalhau-job) <a href="#running-a-bacalhau-job" id="running-a-bacalhau-job"></a>

### Structure of the command[​](http://localhost:3000/examples/model-inference/stable-diffusion-gpu/#structure-of-the-command) <a href="#structure-of-the-command" id="structure-of-the-command"></a>

{% hint style="warning" %}
Some of the jobs presented in the Examples section may require more resources than are currently available on the demo network. Consider [starting your own network](../../getting-started/create-private-network.md) or running less resource-intensive jobs on the demo network
{% endhint %}

To submit a job run the Bacalhau command with following structure:

1. `export JOB_ID=$( ... )` exports the job ID as environment variable
2. The `--gpu 1` flag is set to specify hardware requirements, a GPU is needed to run such a job
3. The `--id-only` flag is set to print only job id
4. `ghcr.io/bacalhau-project/examples/stable-diffusion-gpu:0.0.1`: the name and the tag of the docker image we are using
5. `-- python main.py --o ./outputs --p "meme about tensorflow"`: The command to run inference on the model. It consists of:
   1. `main.py` path to the script
   2. `--o ./outputs` specifies the output directory
   3. `--p "meme about tensorflow"` specifies the prompt

```bash
export JOB_ID=$(
    bacalhau docker run \
    --id-only \
    --gpu 1 \
    ghcr.io/bacalhau-project/examples/stable-diffusion-gpu:0.0.1 \
    -- python main.py --o ./outputs --p "meme about tensorflow")
```

The Bacalhau command passes a prompt to the model and generates an image in the outputs directory. The main difference in the example below compared to all the other examples is the addition of the `--gpu X` flag, which tells Bacalhau to only schedule the job on nodes that have `X` GPUs free. You can [read more about GPU support](../../setting-up/running-node/gpu.md) in the documentation.

{% hint style="success" %}
This will take about 5 minutes to complete and is mainly due to the cold-start GPU setup time. This is faster than the CPU version, but you might still want to grab some fruit or plan your lunchtime run.

Furthermore, the container itself is about 10GB, so it might take a while to download on the node if it isn't cached.
{% endhint %}

When a job is submitted, Bacalhau prints out the related `job_id`. We store that in an environment variable so that we can reuse it later on.

## Checking the State of your Jobs[​](http://localhost:3000/examples/model-inference/stable-diffusion-gpu/#checking-the-state-of-your-jobs) <a href="#checking-the-state-of-your-jobs" id="checking-the-state-of-your-jobs"></a>

**Job status**: You can check the status of the job using `bacalhau job list`.

```bash
bacalhau job list --id-filter ${JOB_ID}
```

When it says `Completed`, that means the job is done, and we can get the results.

**Job information**: You can find out more information about your job by using `bacalhau job describe`.

```bash
bacalhau job describe ${JOB_ID}
```

**Job download**: You can download your job results directly by using `bacalhau job get`. Alternatively, you can choose to create a directory to store your results. In the command below, we created a directory and downloaded our job output to be stored in that directory.

```bash
rm -rf results && mkdir -p results
bacalhau job get $JOB_ID --output-dir results
```

After the download has finished you should see the following contents in results directory

## Viewing your Job Output[​](http://localhost:3000/examples/model-inference/stable-diffusion-gpu/#viewing-your-job-output) <a href="#viewing-your-job-output" id="viewing-your-job-output"></a>

Now you can find the file in the `results/outputs`folder:

<figure><img src="../../.gitbook/assets/index_35_0-98ebebd818f711d58152af5f358c7ab4.png" alt=""><figcaption></figcaption></figure>