# MedGuard VM Setup

It's essentially impossible to setup a development environment correctly the first time, especially given we don't have full information about the hardware setup. If we tried to be prescriptive about this then it's overwhelmingly likely we'd need to go round the loop which would be painful, and take a while.

What this README intends to do is define a minimum set of dependencies and tests to enable faster iteration within the Trusted Data Environment. This will require an initial set of dependencies to exist in the VM, after which we will create Docker images locally which are uploaded to the VM to perform analysis on the patient data.

## Fundamental Requirements

To successfully conduct an evaluation of MedGuard we require:
1. Pre-installed set of core dependencies
2. Downloaded model weights for several Large Language Models, with the ability to download more down the line.
3. Ability to upload files into the environment.

### 1 - Core Dependencies

- [Nvidia GPU Driver](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/n-series-driver-setup)
- [SSH](https://learn.microsoft.com/en-us/azure/virtual-machines/linux/mac-create-ssh-keys)
- [Windows Subsystem for Linux](https://learn.microsoft.com/en-us/windows/wsl/install)
- [Docker](https://docs.docker.com/engine/install/)
- [vLLM Docker Image](https://hub.docker.com/r/vllm/vllm-openai/tags)

#### Nvidia GPU Drivers

Nvidia GPU Drivers are best installed with the initial Azure image. Otherwise things can break in subtle and annoying ways. 

- To verify a correct CUDA driver run `nvidia-smi`. You'll see the GPUs and must have `CUDA >= 12.4`.

#### WSL
WSL is needed to run the LLMs locally in a performant environment. The vLLM Docker image is run via WSL without which it cannot use the GPUs. To verify installation run `wsl -l -v` to list the installed Linux distributions.

- To verify the Nvidia drivers are working within WSL. Enter wsl by running `wsl` and run `nvidia-smi`. You should see a similar output to before.

#### SSH

We will need to be able to SSH into the VM using Remote SSH with VSCode. There are several VSCode extensions that will need to be installed on the VM, which should happen while being in the permissive data landscape.

- To verify the SSH setup, show that you can SSH into the VM from your local machine.

#### Docker

Docker is required to run the vLLM docker image, and subsequent docker images that will be uploaded.

- To verify Docker is installed, run `docker --version` with in WSL
- To verify the Docker Daemon is setup correctly, run the hello world image `docker run --rm hello-world`


#### vLLM Docker Image

The vLLM Docker image contains the necessary dependencies for us to run LLMs locally whtin the environment. Pull the `v0.8.5` image using `docker pull vllm/vllm-openai:v0.8.5`

Verify the image runs using the code below:
```bash
docker run --rm --gpus all -p 8000:8000 vllm/vllm-openai:v0.8.5
```

Ensure the logs show that vLLM has detected the cuda runtime.

#### Core Analysis Environment

A core analysis environment should be setup with python. We don't currently know how we can get patient data to talk to code but iteration of this step will be much faster if there is a basic environment setup locally within the VM. Install the core analysis packages below into a virtual environment using `uv` within WSL.

```bash
pip install uv
uv sync --al-groups
```

This will spin up a core environment you can run. Let's verify this runs by running an UnSloth script. This is taken from an online notebook and has been shown to run on a Linux VM. Run this using `uv run example_script.py`. We'll see lots of outputs saying that we've connected to CUDA, and ultimately the training loop will kick off. We'll begin to see individual rewards as well.

### 2 - Downloaded model weights
We can't access external Language Model APIs within the VM. We therefore require model weights to be downloaded to the VM so we can use vLLM to load and run the model there.

Initially we require the following models to be downloaded:
1. [Llama-3.1-8b-Instruct](https://huggingface.co/meta-llama/Llama-3.1-8B-Instruct) - for rapid experimentation using a lightweight model.
2. [Llama-3.3-70b-Instruct](https://huggingface.co/meta-llama/Llama-3.3-70B-Instruct) - larger model for genuine output
3. [Llama-3.3-Nemotron-49b](https://huggingface.co/nvidia/Llama-3_3-Nemotron-Super-49B-v1) - reasoning model with leading open source performance
4. [Qwen3-32B](https://huggingface.co/Qwen/Qwen3-32B) - small reasoning model
5. [Gemma3 27b](google/gemma-3-27b-it) - leading smaller model
6. [MedGemma 27b](google/medgemma-27b-text-it) - leading medical model, identical size to above

It's recommended to download these with the Huggingface CLI. See the documetation [here](https://huggingface.co/docs/huggingface_hub/en/guides/cli).
