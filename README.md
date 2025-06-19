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

### 2 - Core Analysis Environment

A core analysis environment should be setup with python. We don't currently know how we can get patient data to talk to code but iteration of this step will be much faster if there is a basic environment setup locally within the VM. Install the core analysis packages below into a virtual environment using `uv` within WSL.

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
uv sync
```

This will spin up a core environment you can run. Let's verify this runs by using a combinat