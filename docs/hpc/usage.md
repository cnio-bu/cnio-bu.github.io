# The CNIO HPC cluster

These are some guidelines to using our HPC cluster and the Slurm workflow system.

## Cluster resources

### Compute nodes

The cluster currently features 12 compute nodes with the following configurations:

Count | Node names | CPU cores | RAM   | GPUs
----- | ---------- | --------- | ----  | ----
1     | bc001      | 24        | 32Gb    -
6     | bc00[2-7]  | 52        | 512Gb | -
3     | bc00[8-10] | 128       | 1Tb   | -
1     | hm001      | 224       | 2Tb   | -
1     | gp001      | 112       | 768Gb | 3 x Nvidia A100 80Gb

<br />

### Storage

The cluster features 54Tb of standard storage (where user's home directories are mounted)
and 512Tb of high performance storage (for compute jobs input and output).

### Paragraph for adding to grant applications, etc.

> The CNIO HPC cluster currently consists of: two login nodes working in active/passive mode (2x 40 core/392Gb RAM); a total of 728 compute cores distributed in 9 "standard" compute nodes (6x 52 core/512Gb RAM and 3x 64core/768Gb RAM) and a high-memory node (1x 224 core/2Tb RAM); a dedicated GPU node featuring 3 x Nvidia A100 80Gb GPUs. Local storage is composed of 48 4TB disks in a dual-channel enclosure organized into a RAID-10 unit with an effective available storage of 54TB which host the home directories, and a high-performance lustre storage system with 512 TB of effective available storage for computation. The cluster runs a Slurm queuing system with fairshare priority management.

## Cluster usage

### Introduction to HPC

[See this tutorial](https://carpentries-incubator.github.io/hpc-intro/) for an introduction to HPC concepts and usage
(**highly** recommended if you have little or no experience in using HPC environments).

### Requesting access

Please fill out the [access request form](https://cnio-hpc.limequery.com/757581) in order to 
request access to the cluster.

### Accessing the cluster

The cluster's hostname is `cluster1.cnio.es` (`cluster1` for short), and can be accessed via SSH.

!!! Note

    If you'd like to SSH into the cluster through the CNIO VPN you will need to ask IT for VPN access.

### Mailing list

Important notifications about the cluster are sent to the CNIO HPC mailing
list. Make sure to [subscribe using this link](https://lists.cnio.es/wws/subscribe/hpc)
to stay up to date. The list has very low traffic.

!!! Note

    Do subscribe to the list!

### Storage

!!! Warning

    There are two types of storage in the cluster: your home directory and scratch
    space.

    Your home directory is well suited for storing software (e.g. installing conda), configuration files, etc.
    It is however **not** quick enough to store the input or output files of your analyses.
    Using your home for this will likely hang your jobs and corrupt your files. You should use the
    *scratch* space instead.

!!! Warning

    Scratch space is to be considered "volatile": you should copy your
    input files, execute, copy your output files, and delete everything.

    Home directories are not volatile, but data safety is not guaranteed in
    the long term (so keep backups).


#### Home directories

You have an allocation of 300Gb in your home directory, located in
`/home/<yourusername>/`.

#### Scratch space

##### User space

You have an allocation of 500Gb in your "scratch" directory, located at
`/storage/scratch01/users/<yourusername>/`.

This space is meant to be used for testing and ephemeral compute operations.

##### Project space

To request high-performance scratch space for a new project please fill the
[storage request form](https://cnio-hpc.limequery.com/826326).

#### Checking your quotas

You can check your current quotas with the following commands:

```bash
$ zfs get userquota@$(whoami) homepool/home # check your home quota

$ quotr quota list --long # check your scratch quotas

```

### Copying data to/from the cluster

The most efficient way of copying large amounts of data is by using `rsync`. `rsync` allows
you to resume a transfer in case something goes wrong.

To transfer a directory to the cluster using `rsync` you would do something like:

```bash
rsync -avx --progress mydirectory/ cluster1:/storage/scratch01/myuser/mydirectory/
```

!!! Note

    Pay attention to the trailing slashes, which completely change rsync's behaviour if missing.

For small transfers you could also use `scp` if you prefer:

```bash
scp -r mydirectory/ cluster1:/storage/scratch01/myuser/
```

In both cases you can revert the order of the local and remote directory to copy from
the cluster to your local computer instead.

### Submitting jobs

!!! Warning

    As a general rule, no heavy processes should be run directly on the login nodes.
    Instead you should use the "sbatch" or "srun" commands to send them to the compute
    nodes. See examples below.

The command structure to send a job to the queue is the following:

    sbatch -o <logfile> -e <errfile> -J <jobname> -c <ncores> --mem=<total_memory>G \
        -t<time_minutes> --wrap "<command>"

For example, to submit the command `bwa index mygenome.fasta` as a job: 

    sbatch -o log.txt -e error.txt -J index_genome -c 1 --mem=4G -t120 \
        --wrap "bwa index mygenome.fasta"

#### Job resources

`-c`, `--mem`, and `-t` tell the system how many cores, RAM memory, and time your job will need.

##### Thread management

Many programs can spawn additional threads beyond what you explicitly request through the `-c` parameter.
This can lead to oversubscription of CPU resources and negatively impact both your job and others running
on the same node. For example, some Python libraries (like NumPy), R packages, or bioinformatics tools
might automatically use multiple threads based on the available CPU cores, regardless of your Slurm allocation.

To prevent this:

- Look for program-specific thread control options (e.g., `--threads`, `-t`, etc.)
- If applicable, consider setting environment variables like `OMP_NUM_THREADS`, `OPENBLAS_NUM_THREADS`, `MKL_NUM_THREADS`, and `NUMEXPR_NUM_THREADS` to match your requested cores
- When using workflow managers like Snakemake or Nextflow, ensure they properly propagate resource constraints to their child processes

Please check [this message on the mailing list](https://lists.cnio.es/wws/arc/hpc/2024-12/msg00000.html) for more details and an example.

##### Time limits

!!! Note

    Ideally, you should aim at your jobs having a duration of between 1 and 8 hours.
    Shorter jobs may cause too much scheduling overhead, and longer jobs
    will make it harder for the scheduler to optimally schedule jobs into the
    queues.

If no time limit is specified, a default of 30 minutes will be assigned
automatically.

The "short" queue has a limit of 2 hours per job and the highest priority
(i.e. jobs in this queue will run sooner compared to other queues).

The "main" queue has a limit of 24 hours per job and medium priority. 

The "long" queue has a limit of 168 hours (7 days) per job and 4 concurrent
jobs, and the lowest priority.

!!! Note

    You can specify the partition (queue) you want to use with the `-p` argument
    to `sbatch` or `srun`.

##### Resources and their effect on job priority

The resources you request for a job will influence the chances that such job has to enter the queue, compared to others:
the more resources you request, the longer you may have to wait for those resources to be available.

In addition, the future priority of your jobs will also be influenced by the resources you *request* (not *use*, *request*, even if you don't use them in the end): the more you request, the less priority you'll have for future jobs.

If one of your jobs has lower priority than another one, but its running time would not
delay that higher priority one from entering the queue **and** there are enough resources for it,
your job could enter the queue first. That's why it's important to try and assign reasonably accurate
time limits (a.k.a. *walltimes*) to your jobs.

You can check how efficiently a job used its assigned resources with the `seff <jobid>` command (once the job is finished).

#### Snakemake executor for Slurm

!!! Warning

    The previous way of having Snakemake send jobs to the nodes using a profile (`--profile $SMK_PROFILE_SLURM`)
    is still functional but deprecated. You should ideally move to the native executor described below, and report
    any issues to the list. 

Snakemake features a [Slurm executor plugin](https://snakemake.github.io/snakemake-plugin-catalog/plugins/executor/slurm.html),
which will send jobs to Slurm instead of running them locally. After installing it, simply add the `--executor slurm` argument to
your `snakemake` command.

!!! Note

    In order to indicate the resources required by each Snakemake rule you should use the [threads](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#threads)
    and [resources](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#resources) options.

!!! Note

    The Snakemake command will remain active while your jobs run, so it's recommended that you launch it inside a detachable terminal emulator (e.g. [GNU Screen](https://www.nixtutor.com/linux/introduction-to-gnu-screen/))
    so you can disconnect from the cluster and keep your jobs running.

##### Getting information for jobs submitted via Snakemake

Text extracted from the [Snakemake Slurm executor docs](https://snakemake.github.io/snakemake-plugin-catalog/plugins/executor/slurm.html#inquiring-about-job-information-and-adjusting-the-rate-limiter)

The executor plugin for SLURM uses unique job names to inquire about job status. It ensures inquiring about job status for the series of jobs of a workflow does not put too much strain on the batch system’s database. Human readable information is stored in the comment of a particular job. It is a combination of the rule name and wildcards. You can ask for it with the sacct or squeue commands, for example:

```bash
sacct -o JobID,State,Comment%40
```

Note, the “%40” after Comment ensures a width of 40 characters. This setting may be changed at will. If the width is too small, SLURM will abbreviate the column with a + sign.

For running jobs, you can use the squeue command:

```bash
squeue -u $USER -o %i,%P,%.10j,%.40k
```

Here, the .<number> settings for the ID and the comment ensure a sufficient width, too.

#### Interactive sessions

During development and testing you may want to be able to run commands interactively. You can request
an interactive session, which will run on a compute node, using the `srun` command with the `--pty` argument.

For example, to request a 2-hour session with 4Gb RAM and 2 CPUs, you would do:

`srun --mem=4096 -c 2 -t 120 --pty /bin/bash`

!!! Note

    Interactive sessions are limited to a maximum of 120 minutes.

#### GPUs

!!! Warning

    GPU usage is "experimental". Please expect changes in the setup
    (which will be announced through the [mailing list](#mailing-list)).

The cluster features three Nvidia A100 GPUs with 80Gb of VRAM each.

To run a command using GPUs you need to specify the `gpu` partition, and request
the number of GPUs you require using the
`--gres=gpu:A100:<number_of_gpus>` argument of `sbatch`. Here's an example
to run a python script with one GPU:

`sbatch -p gpu --gres=gpu:A100:1 --wrap "python train_my_net.py"`

### Installing software

Software management is left up to the user, and we recommend doing it by installing
[mambaforge](https://github.com/conda-forge/miniforge#mambaforge) and the [bioconda channels](https://bioconda.github.io/user/install.html#set-up-channels).

If you're completely unfamiliar with conda and bioconda, we recommend following
[this tutorial](https://bioconda.github.io/tutorials/index.html). Notice that the tutorial 
uses miniconda, which is equivalent to the recommended mambaforge.

!!! Note

    Remember to use your home directory to install software (including conda),
    as lustre performs poorly when reading small files often.
