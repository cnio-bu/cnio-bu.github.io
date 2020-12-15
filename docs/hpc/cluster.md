# The CNIO HPC cluster

These are some guidelines to using our HPC cluster and the Slurm workflow system.

## Cluster resources

### Compute nodes

The cluster currently features the following compute nodes:

 Node | CPUs | RAM
----- | ---- | ----
bc001 | 24   | 32Gb
hm001 | 224  | 2Tb

<br />

## Cluster usage

### Introduction to HPC

[See this tutorial](https://hpc-carpentry.github.io/hpc-intro/) for an introduction to HPC concepts and usage
(**highly** recommended if you have little or no experience in using HPC environments).

### Requesting access

Please fill out the [access request form](https://cnio-hpc.limequery.com/757581) in order to 
request access to the cluster.

### Accessing the cluster

The cluster's hostname is `cluster1.cnio.es` (`cluster1` for short), and can be accessed via SSH.

!!! Note

    If you'd like to SSH into the cluster throught the CNIO VPN you will need to ask IT for access.

### Storage

#### Home directories

You have an initial allocation of 300Gb in your home directory.

This space is well suited for for storing software (e.g. conda), configuration files, etc.

It is however **not** quick enough to store the input or output files of your analyses.
Using your home for this will likely hang your jobs and corrupt your files, and you should use the
*scratch* space instead.

#### Scratch space

You have an inital allocation of 1Tb in your "scratch" directory, located at
`/storage/scratch01/users/<yourusername>/`.

Input and output of any computation that you do in the cluster should be stored there.

#### Data availability and security

Scratch space is to be considered "volatile": you should copy your
input files, execute, copy your output files, and delete everything.

Home directories are not volatile, but data safety is not guaranteed in
the long term (so keep backups).

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

    As a general rule, no heavy processes should be run on the login nodes. Jobs
    run there have a limit of 4Gb RAM and 2 hours.

The command structure to send a job to the queue is the following:

    sbatch -o <logfile> -e <errfile> -J <jobname> -c <ncores> --mem=<total_memory>G \
        -t<time_minutes> --wrap "<command>"

For example, to submit the command `bwa index mygenome.fasta` as a job: 

    sbatch -o log.txt -e error.txt -J index_genome -c 1 --mem=4G -t120 \
        --wrap "bwa index mygenome.fasta"

#### Job resources

`-c`, `--mem`, and `-t` tell the system how many cores, RAM memory, and time your job will need.

There is a limit of 24 hours per job. As a general rule, you should try to break down your job
into smaller jobs if it takes longer than ~8 hours.

##### Resources and their effect on job priority

The resources you request for a job will influence the chances that such job has to enter the queue, compared to others:
the more resources you request, the longer you may have to wait for those resources to be available.

In addition, the future priority of your jobs will also be influenced by the resources you *request* (not *use*, *request*, even if you don't use them in the end).
The more you request, the less priority you'll have for future jobs.

You can check how efficiently a job used its assigned resources with the `seff <jobid>` command (once the job is finished).

#### Snakemake profile

The cluster features a Snakemake profile that allows for the automatic submision and management of jobs.
To use it, just add the `--profile slurm` argument to your Snakemake command.

!!! Note

    In order to indicate the resources required by each Snakemake rule you should use the [threads](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#threads)
    and [resources](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#resources) options.

!!! Note

    The Snakemake command will remain active while your jobs run, so it's recommended that you launch it inside a detachable terminal emulator (e.g. GNU Screen)
    so you can disconnect from the cluster and keep your jobs running.

### Installing software

Software management is left up to the user, and we recommend doing it with
[conda](https://docs.conda.io/en/latest/miniconda.html),
by taking advantage of the [bioconda](http://bioconda.github.io/) repository.
