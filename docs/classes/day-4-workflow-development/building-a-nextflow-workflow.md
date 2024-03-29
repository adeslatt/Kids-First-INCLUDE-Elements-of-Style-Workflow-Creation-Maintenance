
## Building A Nextflow Workflow

If you have arrived here from the front page and have not built your *`fastqc-docker`* and *`multiqc-docker`* containers, please proceed to the [preamble to building a Nextflow Workflow](preamble.md)

**Main outcome:** 

During the first session you will build a [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) & [MultiQC](https://multiqc.info/) pipeline to learn the basics of Nextflow including:

- [Parameters](https://www.nextflow.io/docs/latest/getstarted.html?highlight=parameters#pipeline-parameters)
- [Processes](https://www.nextflow.io/docs/latest/process.html) (inputs, outputs & scripts)
- [Channels](https://www.nextflow.io/docs/latest/channel.html)
- [Operators](https://www.nextflow.io/docs/latest/operator.html)
- [Configuration](https://www.nextflow.io/docs/latest/config.html)
    
## In the Google Shell

We are going to continue use [Google shell Cloud](https://shell.cloud.google.com/) to walk through the building of a Nextflow script and in the next session building the containers we used to do so.

Inside the shell we have the following:

* `Docker` is installed

* `conda` has been installed -- which we did in the previous lesson.

* The standard workflow language for Nextflow `nextflow` can be installed

* The standard workflow language for the Common Workflow Language (CWL) `cwltool` can be installed

### What is [Docker](https://www.docker.com/) <img src="/../../img/Moby-Logo.png" width=50 align=left>?

Just to recap a bit. Docker is the application we use to build our containerized images.   It turns our code into an image that can be run interactively or be placed upon a virtual machine that we spin up on CAVATICA as part of a workflow.   The logo that is associated with `Docker` is a whale but looks like a containership.   Containers revolutionized the shipping industry by creating a uniform entity that could permits disparate items to be packaged in the same manner allowing devices that do not know what they contain to carry those items.   Much in the same way that `packets` revolutionized communication with the internet.  This is why we concern ourselves with containers. 

### Install Nextflow

Now we can install `nextflow` package.   To get the installation command, we again search `Anaconda` for the package details.   We see that this is a pretty popular package, there have been at the time of this writing, that this has been downloaded 162,780 times and that the last upload was 2 months and 6 days ago and we have the GitHub location of the software. 

Importantly, we now see how to install the `nextflow` application.

```bash
conda install -c bioconda nextflow
```

The package installer manages the items that need to be installed, telling us the following packages will be installed and asking if we would like to proceed.

```bash
The following NEW packages will be INSTALLED:

  coreutils          bioconda/linux-64::coreutils-8.25-1
  java-jdk           bioconda/linux-64::java-jdk-8.0.92-1
  libgcc             pkgs/main/linux-64::libgcc-7.2.0-h69d50b8_2
  nextflow           bioconda/linux-64::nextflow-0.24.2-0

The following packages will be UPDATED:

  ca-certificates    conda-forge::ca-certificates-2021.10.~ --> pkgs/main::ca-certificates-2022.3.18-h06a4308_0


Proceed ([y]/n)?
```

We say `y` and the package is installed.

Confirming we type the following:

```bash
which nextflow
```

And because we have activated our environment `eos`, you will see that the `nextflow` application is installed in the `bin` which is short for `binary` location within `eos`

```bash
(eos) adeslat@cloudshell:~$ which nextflow
/home/adeslat/miniconda3/envs/eos/bin/nextflow
```

### a) Getting Course material.

Let's use your forked version of the repository.

First lets make sure it is up-to-date.

To do that do the following.

Follow the directions for [keeping your repository in sync](https://github.com/NIH-NICHD/Kids-First-Elements-of-Style-Workflow-Creation-Maintenance/blob/main/classes/Intro-to-Git-Github/why-git-and-setup.md#keeping-your-repository-fork-in-sync)

At this time do not worry about making a pull request.  If your repository has commits ahead at this time, do not concern yourself.

Then as we did yesterday, if you have not checked out your own version.  Do so now but in your username subdirectory.

After synchronizing, navigate to your subdirectory and clone the updated respository

```bash
cd adeslatt
```
And clone

```bash
git clone https://github.com/NIH-NICHD/Building-A-Nextflow-Script.git
```

### b) Parameters (otherwise known as inputs)

Now that we have Nextflow & Docker installed we're ready to run our first script and learn how we bring in data into a file.

Let's open the file `params_reads.nf`

You see the following content:

```nextflow
// params_reads.nf
params.reads = false

println "My reads: ${params.reads}"
```

The first line is a comment.  Comments in Nextflow can begin with `//`.   Every language has a different way of indicating that this is for the human to read and not the machine.   This comment may seem silly, but when you have many files open, it is uesful to have an indication of where you are.

The second line creates something for the computer.  Think of it as an empty vessel.  We can call it what we want, but in this case we are actually going to use something that our program, `nextflow` understands, `params` and create an arbitrary term _reads_ Together they make what you could imagine is an element of a paragraph.  You can define many things that are of a `params` type, but for now we are just creating one thing.  Something we are calling `reads`.

Finally we are setting this `params.reads` to `false`.

The third line instructs the program `nextflow` to print out the value of this `parameter` upon execution of the workflow.

This minimal workflow can now be executed by the `nextflow` application we have installed.

We will do this and we will provide the workflow a value to `params.reads`.

In this repository are two `fastq` files in a `directory` within the freshly checked out repository as follows.

```bash
nextflow run params_reads.nf --reads testdata/test.20k_reads_1.fastq.gz
```

The run returns the name of our file.

### c) Processes (inputs, outputs & scripts)

Well, we have run our first `nextflow` workflow, congratulations!.

Nextflow allows the execution of any command or user script by using a `process` definition. 

A process is defined by providing three main declarations: 
the process [inputs](https://www.nextflow.io/docs/latest/process.html#inputs), 
the process [outputs](https://www.nextflow.io/docs/latest/process.html#outputs)
and finally the command [script](https://www.nextflow.io/docs/latest/process.html#script).

Let's look at the next workflow, `fastqc.nf`, we see the following:
```nextflow
//fastqc.nf
reads = file(params.reads)

process fastqc {

    publishDir "results", mode: 'copy'

    input:
    file(reads) from reads

    output:
    file "*_fastqc.{zip,html}" into fastqc_results

    script:
    """
    fastqc $reads
    """
}
```



Here we created the variable `reads` which is a `file` from the command line input.

We can then create the process `fastqc` including:
 - the [directive](https://www.nextflow.io/docs/latest/process.html#directives) `publishDir` to specify which folder to copy the output files to 
 - the [inputs](https://www.nextflow.io/docs/latest/process.html#inputs) where we declare a `file` `reads` from our variable `reads`
 - the [output](https://www.nextflow.io/docs/latest/process.html#outputs) which is anything ending in `_fastqc.zip` or `_fastqc.html` which will go into a `fastqc_results` channel
 - the [script](https://www.nextflow.io/docs/latest/process.html#script) where we are running the `fastqc` command on our `reads` variable

We can then run our workflow with the following command:
```bash
nextflow run fastqc.nf --reads testdata/test.20k_reads_1.fastq.gz -with-docker fastqc
```

We are using a docker image I previously had pushed to the Seven Bridges Image repository -- in the next session I will walk through how that is done.

By running Nextflow using the `with-docker` flag we can specify a Docker container to execute this command in. This is beneficial because it means we do not need to have `fastqc` installed locally on our laptop. We just need to specify a Docker container that has `fastqc` installed.


### d) Channels

Channels are the preferred method of transferring data in Nextflow & can connect two processes or operators.

<!--
There are two types of channels:
1. [Queue channels](https://www.nextflow.io/docs/latest/channel.html#queue-channel) can be used to connect two processes or operators. They are usually produced from factory methods such as [`from`](https://www.nextflow.io/docs/latest/channel.html#from)/[`fromPath`](https://www.nextflow.io/docs/latest/channel.html#frompath) or by chaining it with methods such as [`map`](https://www.nextflow.io/docs/latest/operator.html#operator-map). **Queue channels are consumed upon being read.**
2. [Value channels](https://www.nextflow.io/docs/latest/channel.html#value-channel) a.k.a. singleton channel are bound to a single value and can be read unlimited times without consuming there content. Value channels are produced by the value factory method or by operators returning a single value, such us first, last, collect, count, min, max, reduce, sum.
-->

Here we will use the method [`fromFilePairs`](https://www.nextflow.io/docs/latest/channel.html#fromfilepairs) to create a channel to load paired-end FASTQ data, rather than just a single FASTQ file.


To do this we will replace the code from [1c](https://github.com/NIH-NICHD/Kids-First-Elements-of-Style-Workflow-Creation-Maintenance/edit/main/classes/Building-A-Nextflow-Script/README.md#c-processes-inputs-outputs--scripts) with the following 

```nextflow
//fastqc.nf
reads = Channel.fromFilePairs(params.reads, size: 2)

process fastqc {

    tag "$name"
    publishDir "results", mode: 'copy'
    container 'pgc-images.sbgenomics.com/deslattesmaysa2/fastqc:v1.0'

    input:
    set val(name), file(reads) from reads

    output:
    file "*_fastqc.{zip,html}" into fastqc_results

    script:
    """
    fastqc $reads
    """
}
```

The `reads` variable is now equal to a channel which contains the reads prefix & paired-end FASTQ data. Therefore, the input declaration has also changed to reflect this by declaring the value `name`. This `name` can be used as a tag for when the pipeline is run. Also, as we are now declaring two inputs the `set` keyword has to be used. Finally, we can specify the container name within the processes as a directive.

To run the pipeline:
```bash
nextflow run fastqc.nf --reads "testdata/test.20k_reads_{1,2}.fastq.gz" -with-docker pgc-images.sbgenomics.com/adeslat/fastqc:v0.11.9
```

### e) Operators

Operators are methods that allow you to manipulate & connect channels.

Here we will add a new process `multiqc` & use the [`.collect()`](https://www.nextflow.io/docs/latest/operator.html#collect) operator

Add the multiqc process after `fastqc`:
```nextflow
//fastqc_multiqc_wf.nf
reads = Channel.fromFilePairs(params.reads, size: 2)

process fastqc {

    tag "$name"
    publishDir "results", mode: 'copy'
    container 'pgc-images.sbgenomics.com/deslattesmaysa2/fastqc:v1.0'

    input:
    set val(name), file(reads) from reads

    output:
    file "*_fastqc.{zip,html}" into fastqc_results

    script:
    """
    fastqc $reads
    """
}
process multiqc {

    publishDir "results", mode: 'copy'
    container 'pgc-images.sbgenomics.com/deslattesmaysa2/multiqc:v1.0'

    input:
    file ('fastqc/*') from fastqc_results.collect()

    output:
    file "*multiqc_report.html" into multiqc_report
    file "*_data"

    script:
    """
    multiqc . -m fastqc
    """
}
```

Here we have added another process `multiqc`. We have used the `collect` operator here so that if `fastqc` ran for more than two pairs of files `multiqc` would collect all of the files & run only once.

The pipeline can be run with the following:
```bash
nextflow run fastqc_multiqc_wf.nf --reads "testdata/test.20k_reads_{1,2}.fastq.gz" -with-docker pgc-images.sbgenomics.com/deslattesmaysa2/fastqc:v1.0
```

### f) Configuration

Configuration, such as parameters, containers & resources eg memory can be set in `config` files such as [`nextflow.config`](https://www.nextflow.io/docs/latest/config.html#configuration-file).

For example our `nextflow.config` file might look like this:
```
docker.enabled = true
params.reads = false

process {
  cpus = 1
  memory = "2.GB"

  withName: fastqc {
    container = "pgc-images.sbgenomics.com/adeslat/fastqc:v0.11.9"
  }
  withName: multiqc {
    container = "pgc-images.sbgenomics.com/adeslat/multiqc:v1.0dev0"
  }
}
```

which are in fact the images as I stored them in the CAVATICA repository.

Here we have enabled docker by default, initialised parameters, set resources & containers. It is best practice to keep these in the `config` file so that they can more easily be set or removed. Containers & `params.reads` can then be removed from `main.nf`.

The pipeline can now be run with the following:
```bash
nextflow run fastqc_multiqc_wf.nf --reads "testdata/test.20k_reads_{1,2}.fastq.gz"
```

## Proceed to the next lesson

[Return to the Agenda](day-4-workflow-development.md)