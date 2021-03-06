# Nextflow workshop

[https://github.com/cellgeni/nf-workshop](https://github.com/cellgeni/nf-workshop)

Vladimir Kiselev

Head of the Cellular Genetics Informatics

Sanger Institute

## Basic concepts

* Nextflow is designed around the idea that the Linux platform is the lingua franca of data science. 
* Linux provides many simple but powerful command-line and scripting tools that, when chained together, facilitate complex data manipulations
* Nextflow extends this approach, adding the ability to define complex program interactions and a high-level parallel computational environment based on the dataflow programming model

## Processes and channels

* Nextflow pipeline script is made by joining together many different processes. Each process can be written in any scripting language that can be executed by the Linux platform (Bash, Perl, Ruby, Python, etc.)
* Processes are executed independently and are isolated from each other, i.e. they do not share a common (writable) state
* The only way they can communicate is via asynchronous FIFO queues, called channels 
* Any process can define one or more channels as input and output

## Execution and abstraction

* While a process defines what command or script has to be executed, the executor determines how that script is actually run on the target environment
* If not otherwise specified, processes are executed on the local computer. The local executor is very useful for pipeline development and test purposes, but for real world computational pipelines an HPC or cloud platform is required
* Nextflow provides an abstraction between the pipeline's functional logic and the underlying execution environment
* Thus it is possible to write a pipeline once and to seamlessly run it on your computer, a grid platform, or the cloud, without modifying it

## Clone the workshop repo and install Nextflow
```
git clone https://github.com/cellgeni/nf-workshop.git
cd nf-workshop
curl -s https://get.nextflow.io | bash
# use wget if curl is not available
# wget -qO- https://get.nextflow.io | bash
ls
```

## `Hello world!` pipeline
* The [Hello world! pipeline](hello-world.nf) defines two processes
* `splitLetters` splits a string in file chunks containing 6 characters
* `convertToUpper` receives these files and transforms their contents to uppercase letters
* The resulting strings are emitted on the result channel and the final output is printed by the subscribe operator

## Run the pipeline
```
> ./nextflow run hello-world.nf
N E X T F L O W  ~  version 0.28.0
Launching `hello-world.nf` [exotic_bartik] - revision: 361b274147
[warm up] executor > local
[e7/7d678f] Submitted process > splitLetters
[5e/fe9bf6] Submitted process > convertToUpper (2)
[bb/75ef46] Submitted process > convertToUpper (1)
WORLD!
HELLO
```

* The first process is executed once, and the second twice
* The result string is printed
* `convertToUpper` is executed in parallel, so it is possible that you will get the final result printed out in a different order:

```
HELLO
WORLD!
```

## `work` directory
```
> tree -a work
work
├── 66
│   └── 5422cf0adc07c4662eaaa04b5c1700
│       ├── .command.begin
│       ├── .command.err
│       ├── .command.log
│       ├── .command.out
│       ├── .command.run
│       ├── .command.sh
│       ├── .exitcode
│       ├── chunk_aa
│       └── chunk_ab
├── 7b
│   └── 86f99a5e06fe183daee815b2cf09c7
│       ├── .command.begin
│       ├── .command.err
│       ├── .command.log
│       ├── .command.out
│       ├── .command.run
│       ├── .command.sh
│       ├── .exitcode
│       └── chunk_aa -> /Users/vk6/nf-workshop/work/66/5422cf0adc07c4662eaaa04b5c1700/chunk_aa
└── f5
    └── 56dcd57618fe720c4ed4602055f68a
        ├── .command.begin
        ├── .command.err
        ├── .command.log
        ├── .command.out
        ├── .command.run
        ├── .command.sh
        ├── .exitcode
        └── chunk_ab -> /Users/vk6/nf-workshop/work/66/5422cf0adc07c4662eaaa04b5c1700/chunk_ab

6 directories, 25 files
```

## `work` directory

* `work` directory contains sub-directories where Nextflow executes its processes
* The names of the directories are randomly generated
* `splitLetters` was executed in the `66` sub-directory
* `convertToUpper` was executed in `7b` and `f5` sub-directories
* For each process Nextflow generates some system scripts and the outputs
* The original process command is in .command.sh
* `.command.run` is the script submitted to the cluster environment
* `.command.err`, `.command.log`, `.command.out` are the standard outputs
* `convertToUpper` depends on the output of `splitLetters`, a link has been created (no copying)

## Modify and resume
Let's modify the script block in the `convertToUpper` process (now will run `rev $x`):
```
process convertToUpper {
    input:
    file x from letters

    output:
    stdout result

    script:
    """
    rev $x
    """
}
```

## Modify and resume
Now rerun Nextflow with the `-resume` flag:
```
> ./nextflow run hello-world.nf -resume
N E X T F L O W  ~  version 0.28.0
Launching `hello-world.nf` [mighty_goldstine] - revision: 0fa0fd8326
[warm up] executor > local
[66/5422cf] Cached process > splitLetters
[00/fc01e0] Submitted process > convertToUpper (1)
[b2/816523] Submitted process > convertToUpper (2)
olleH
!dlrow
```

Note that the first process `splitLetters` was cached and was not run at all!
## Pipeline parameters
Nextflow allows to define parameters inside the pipeline, e.g. in the [Hello, world!](hello-world.nf) pipeline there is a `str` parameter defined:
```
params.str = 'Hello world!'
```

We can use it in the command line to redefine the default value:
```
> ./nextflow run hello-world.nf --str 'Hola mundo'
N E X T F L O W  ~  version 0.28.0
Launching `hello-world.nf` [elated_hamilton] - revision: b0857ec305
[warm up] executor > local
[b3/924952] Submitted process > splitLetters
[d8/727942] Submitted process > convertToUpper (2)
[5a/3b7252] Submitted process > convertToUpper (1)
odnu
m aloH
```
## Local software
By default, Nextflow can use all the software available in your bash environment. 

## `conda` environments
However, if you need some specific software installation that you don't want to be bothered with, we recommend using `conda` environments. To install `conda`, please follow the steps on [this page](https://conda.io/docs/user-guide/install/index.html).

There are [multiple ways](https://conda.io/docs/user-guide/tasks/manage-environments.html) of creating and managing `conda` environments. We recommend using `environment.yml` file. For more details please see [here](https://conda.io/docs/user-guide/tasks/manage-environments.html#creating-an-environment-from-an-environment-yml-file).

## `conda` environments
Here we will show an example of setting up a conda environment for our [RNAseq pipeline](https://github.com/cellgeni/rnaseq) using the `environment.yml` file:
```
name: rnaseq
channels:
  - conda-forge
  - bioconda
dependencies:
  - fastqc=0.11.7
  - bedops=2.4.30
  - cutadapt=1.15
  - trim-galore=0.4.5
  - star=2.5.4a
  - hisat2=2.1.0
  - rseqc=2.6.4
  - picard=2.17.6
  - samtools=1.7
  - preseq=2.0.2
  - subread=1.6.0
  - stringtie=1.3.3
  - multiqc=1.4
```

## `conda` environments
To create an environment with the all the listed software one needs to run:
```
conda env create -f environment.yml
```

Once the environment is ready it can be activated inside a Nextflow process using these lines:
```
beforeScript "set +u; source activate rnaseq"
afterScript "set +u; source deactivate"
```

> We do not recommend to add any R-related packages to your conda environment, in out experience it never worked well.

## Docker images
You conda environments can be further dockerized if needed using a `Dockerfile`:
```
FROM continuumio/miniconda
ADD environment.yml
RUN conda env create -f environment.yml
```

You can then add any other softwared you need for your pipeline to the docker image.

## `SC3` clustering Nextflow pipeline
Now we will create a more real-life example pipeline. We will run [SC3](https://bioconductor.org/packages/release/bioc/html/SC3.html) clustering of a small single-cell RNAseq dataset (which is included in the `SC3` package). `SC3` is a stochastic clustering algorithm and with this pipeline we would like to check its stability. To do that we will run `SC3` multiple times in parallel every time changing an initial random seed. After that we will merge all the results into one matrix, which is useful for further downstream analysis.

## `main.nf`
In order to publish your Nextflow pipeline to GitHub (or any other supported platform) and allow other people to use it, you only need to create a GitHub repository containing all your project script and data files.

Nextflow only requires that the main script in your pipeline project to be called `main.nf`. We will use this name for our `SC3` pipeline.

In the `main.nf` we have two processes (`run_sc3` and `merge_results`) and one parameter (`params.n`) which is the number of times we would like to `SC3` with different random seeds.

__Exercise__ Have a look at [main.nf](main.nf) and notice how it is different from [hello-world.nf](hello-world.nf).

## Third-party scripts
Since `SC3` is an R package, preferably we would like to have an R script with all the `SC3` commands in a separate file. Nextflow allows you to store all third-party scripts in the `bin` folder in the root directory of your project repository. Nextflow will automatically add this folder to the `PATH` environment variable, and the scripts will automatically be accessible in your pipeline without the need to specify an absolute path to invoke them.

> Note that you have to grant these scripts the execute permission (`chmod +x bin/*`) and add a shebang to all of your third-party scripts. In the case of `R` scripts we will add `#!/usr/bin/env Rscript` shebang. The scripts then should be called __just by their name__ in the process scripts.

In our pipeline we have to `R` scripts in the `bin` folder (`sc3.R` and `merge.R`) corresponding to the two pipeline processes.

## Run `SC3`
Now, let's run our `SC3` pipeline:
```
> ./nextflow run main.nf
N E X T F L O W  ~  version 0.28.0
Launching `main.nf` [elegant_carson] - revision: d5c6c10c98
[warm up] executor > local
[0a/9ccd50] Submitted process > run_sc3 (3)
[b2/a0bbd7] Submitted process > run_sc3 (1)
[9d/9a91db] Submitted process > run_sc3 (2)
[7a/1bb0d2] Submitted process > merge_results
```

The first process was run 3 times (default value), but this number can be controlled by using `--n` flag when executing `main.nf`.

__Exercise__ What is the result of the pipeline?

__Exercise__ Explore the newly created folders in the `work` directory.

## Run `SC3`

__Exercise__ Check Nextflow integration with GitHub by changing your directory to some temporary one and running:
```
curl -s https://get.nextflow.io | bash
./nextflow run cellgeni/nf-workshop
```

Note that the pipeline has automatically been pulled from GitHub.

## Configuration file
When a pipeline script is launched Nextflow looks for a file named `nextflow.config` in the current directory and in the script base directory (if it is not the same as the current directory). Finally it checks for the file `$HOME/.nextflow/config`.

`nextflow.config` is a configuration file that is used to define parameters required by your computational environment. If you need to run your pipeline on different environments, you can make use of configuration __profile__s. A __profile__ is a set of configuration attributes that can be activated/chosen when launching a pipeline execution by using the `-profile` command line option.

__Exercise__ Have a look at our [nextflow.config](nextflow.config) file. Did you notice the cloud profile?!

## Sanger farm settings
We have tested Nextflow on the Sanger farm and found parameters that are required to be present in the configuration file to be able to run the pipeline successfully:
```
// this is required by bsub on farm3: selected[mem] should = rusage[mem]
// http://mediawiki.internal.sanger.ac.uk/wiki/index.php/Submitting_large_memory_jobs
process.clusterOptions = { "-R \"select[mem>${task.memory.toMega()}]\"" }
// https://www.nextflow.io/docs/latest/executor.html#lsf
executor.perJobMemLimit = true
```

The first requirement is the memory limit and second one is giving memory to the job and not to a single core inside the job (in case of multicore jobs).

## Run Nextflow on the Sanger farm

Here is what we recommend you to do to be able to successfully run Nextflow on the Sanger farm:
```
# login to the farm
ssh -Y farm3-head2

# create a dedicated bash session for your pipeline
tmux new -s sc3_pipeline

# this will add Nextflow executable to your path (together with other
# CellGen IT software and R packages, including SC3)
source /nfs/cellgeni/.cellgenirc

# Java is not permitted on the head nodes, but it is allowed on the 
# slave nodes, therefore we need to start an interactive job to be 
# able to run Nextflow
bsub -Is -q long -R"select[mem>5000] rusage[mem=5000]" -M5000 bash

# (optional)
# your session in the long queue will be active for 2 days, if your
# pipeline requires more time to finish, you can use rhweek queue which
# will be active for 1 week:
# bsub -Is -q rhweek -R"select[mem>5000] rusage[mem=5000]" -M5000 bash
```

## Run Nextflow on the Sanger farm
__Exercise__ Now try to run our `SC3` pipeline on the farm using:
```
nextflow run cellgeni/nf-workshop -profile farm
```

If you open another terminal and look at your jobs using `bjobs`, you will see that Nextflow has started 3 jobs corresponding to the first process in our pipeline running with three different random seeds:
```
vk6@farm3-head2:~$ bjobs
JOBID   USER    STAT  QUEUE      FROM_HOST   EXEC_HOST   JOB_NAME   SUBMIT_TIME
9525703 vk6     RUN   long       farm3-head2 bc-27-3-04  bash       Apr 17 11:38
9525942 vk6     RUN   normal     bc-27-3-04  bc-32-1-09  *n_sc3_(2) Apr 17 11:39
9525949 vk6     RUN   normal     bc-27-3-04  bc-32-1-09  *n_sc3_(3) Apr 17 11:39
9525953 vk6     RUN   normal     bc-27-3-04  bc-32-1-09  *n_sc3_(1) Apr 17 11:39
```

## More resources

* [Nextflow publication](https://www.nature.com/articles/nbt.3820)
* [More Nextflow presentations](https://speakerdeck.com/pditommaso/)
* [Nextflow examples](https://github.com/nextflow-io/examples)
* [Another Nextflow workshop](https://github.com/nextflow-io/hack17-tutorial)
* [awesome-nextflow](https://github.com/nextflow-io/awesome-nextflow)

## Acknowledgements

* Paolo Di Tommaso
* [Nextflow documentation](https://www.nextflow.io/docs/latest/index.html)
* [markdown-to-slides](https://github.com/partageit/markdown-to-slides)
