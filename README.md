# genomedk-tutorial

## Introduction

We have all been in the same situation, where we quickly want to sort a bam
file, or maybe just want to check a new tool published. Maybe we check the load
on the frontend of genomedk. It does not look to be used that much; the coast
is clear, and we run our software directly on the frontend, which ends up
taking 8 CPUs, and suddenly it is slow for everyone.

This leads to annoyed users where a simple `ls` command is slow, and in
absolute worst case scenario it can be dangerous. Maybe a clinical interpreter
is looking at some data for a patient that needs an urgent answer. But now it
is not possible, due to the heavy load on the frontend.

Is there a better way to run your jobs? How should you run a single line of
code? What if it is a complex workflow? Below are listed different ways to to
run software on genomedk.

Below the examples will refer to an account. This can for example be MomaDiagnostics,
Improve, etc. Use the one appropriate for your project and affiliation. It is
possible to read more [here](https://genome.au.dk/docs/interacting-with-the-queue/).


## `srun`

In the case where you want to test how a software works, it can easily be
solved with the `srun` command

```bash
  srun --pty --account=<my-account> /bin/bash
```
This will allocate 4 hours with 1GB memory on 1 CPU. If you need more
resources, then do
```bash
  srun --pty --mem=16g --cpus-per-task=4 \
       --time=120 --account=<my-account> /bin/bash
```
Here are allocated 16GB memory, 4 CPUs, and 120 minutes. For more informaton,
just run
```bash
  srun --help
```
to see all available options.

It might take a few moments before you enter a node depending on the resources
you request, and the current load on the cluster.

`srun` is a great tool for exploring a new tool without putting a load on the
frontend.


## `sbatch`

While `srun` is good for a more interactive workflow, it is probably not
what you want once you are settled on a standard way to run your new tool. This
is where `sbatch` enters. You can make a bash script like

```bash
  #!/bin/bash
  #SBATCH --account <my-account>
  #SBATCH --mem 4g
  #SBATCH --time 12:00:00
  #SBATCH -c 4

  echo This is my first script
```
and simple run like
```bash
  sbatch example.sh
```
This will give you a jobid, which you can inspect with the `jobinfo`
command. You can also run sbatch more directly without creating a script like

```bash
  sbatch --account <my-account> --mem 4g --time 12:00:00 -c 4 \
      echo This is my first script
```
However, creating a script is preferred so you are able to document in the
future how you obtained the results.



## `gwf`
[`gwf`](https://gwf.app/) can be installed with conda and are great for more
complex workflows. Here output from one job can be used as input for another
job, and so forth.

An example could be
  1. Trim fastq files
  2. Map trimmed fastq files
  3. Call SNVs on resulting bam file
  4. Call CNVs on resulting bam file
  5. ...
Here item 3 and 4 can be run in parallel, since there are no dependencies
between them.

### Real example
It is easiest to find some examples on for example the github page of gwf, here
is a [good
example](https://github.com/gwforg/gwf/blob/master/examples/readmapping/workflow.py).
Otherwise you can find people at MOMA that often use gwf, for example the
clinical bioinformaticians.

For running a gwf workflow you need a file called `workflow.py` and then run
```bash
  gwf run
```
To see the status of all the jobs in your current workflow, then run
```bash
  gwf status
```
or if you are only interested in jobs `running` and `shouldrun`, then
```bash
  gwf status -s running -s shouldrun
```


## snakemake

Another tool for running complex workflows is snakemake. While many people at MOMA
uses gwf, it is a tool probably only used on genomedk, while snakemake has a large
international community.
