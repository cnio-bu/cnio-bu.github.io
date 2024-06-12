# How to setup a conda environment for GPU-accelerated single cell analysesf
This is by no means an extensive guide as things are moving fast in the single cell field, but should give you a decent starting point to setup  your libraries and packages to accelerate single cell analyses using the CNIO's GPU nodes.

# 1. Create a new conda environment from a GPU node
This can be done automatically by running a job using sbatch and a bash script to create and install de conda packages. To launch a job using the GPU, you can just specify the **partition** with:

```bash
sbatch -p gpu --gres=gpu:A100:1 --wrap "my_script.bash"
```

However, due to the complexity of actually setting up the whole environment for the first time, I recommend asking for an interactive session in GP001 and running everything by hand like so:

```bash
srun --mem=4096 -c 1 -t 60  -p gpu --pty /bin/bash
```

The GPU node is slow when it comes to installing packages so unfortunately, expect to queue a full hour for package management. Once you are granted an interactive session in a GPU node, start by creating a new environment. DO NOT try to recycle another environment, **start fresh**.

```bash 
mamba create -n mygpu
mamba activate mygpu
```

And then, once **mygpu** is activated, install the first set of libraries

```bash
mamba install pytorch torchvision torchaudio pytorch-cuda=11.8 cuda-nvcc=11.8 -c pytorch -c nvidia
mamba install jax jaxlib -c conda-forge
```

## Important notice
Keep in mind that conda/mamba will default to a CPU based compilation of CUDA/lightning and other libraries If you try to install them from a head node or from any worker node that has not a GPU.

# 2. Verify that CUDA is working
From an interactive session inside the GPU node or from a python script, execute:

```python
import torch
print(torch.cuda.get_device_name(0))
```
The requested GPU should be available at slot 0. If this slot is empty, then:

a) You are not running your script in a node equipped with a GPU
b) CUDA was not installed properly and It's not working.



# 3. Install scvi-tools from pip
I do not know how or why, but somehow conda/mamba manages to pull an ancient version
of scvi-tools after installing the correct libraries, so we'll install scvi with pip

```bash
pip install scvi-tools
```

# 4. Verify that everything is working
From an interactive session inside the GPU node or from a python script, execute:

```python3
import torch
import scvi
import scanpy as sc
```

These three libraries should load flawlessly. scvi-tools, when installed with pip, should also install a compatible version of scanpy. If the latter is not working, It is safe to install it with conda/mamba once scvi-tools is already installed.