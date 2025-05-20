---
toc_depth: 3
---

# Writing Slurm Batch Jobs

This Slurm tutorial serves as a hands-on guide for users to create Slurm batch scripts based on their specific software needs and apply them for their respective usecases.  It covers basic examples for beginners and advanced ones, including sequential and parallel jobs, array jobs, multithreaded jobs, GPU utilization jobs, and MPI (Message Passing Interface) jobs. To know which type of batch jobs are suitable for your pipeline/usecase, please refer to the [User Guide](#slurm-batch-job-user-guide) section.

## Structure of a Slurm Batch Job

Below is the template for a typical Slurm job submission in the Cheaha high-performance computing (HPC) system. The script begins with `#!/bin/bash`, indicating it is a bash script. The next step would be to declare Slurm configuration options, specifying the required resources for job execution. This section typically comprises parameters such as CPU count, partition, memory allocation, time limit, etc. Following the configuration, the script may include sections for loading necessary software or libraries required for the job.

```bash
#!/bin/bash
# Declaring Slurm configuration options and specifying required resources
...
# Loading Software/Libraries
...
# Running Code
...
```

The last portion is running the actual code or software. Here, the computational task or program intended for execution is launched using specific commands and processes, which depends on the software used and overall computational workflow. For more detailed specification, refer to [Slurm job submission](../slurm/submitting_jobs.md). The following sections present practical examples for writing a Slurm batch script to specific use cases, and prerequisites to start with the tutorial.

## Prerequisites

If you're new to using Unix/Linux commands and bash scripting, we suggest going through the software carpentry lesson, [The Unix Shell](https://swcarpentry.github.io/shell-novice/). Also, we recommend reviewing the [Cheaha Hardware Information](../../cheaha/hardware.md) to help guide you in choosing appropriate partition and resources.

## Slurm Batch Job User Guide

This user guide provides comprehensive insight into different types of batch jobs, facilitating in identifying the most suitable job type for your specific tasks. With clear explanations and practical examples, you will gain a deeper understanding of sequential, parallel, array, multicore, GPU, and multi-node jobs, assisting to make informed decisions when submitting jobs on the Cheaha system.

1. [A Simple Slurm Batch Job](#example-1-a-simple-slurm-batch-job) is ideal for Cheaha users who are just starting with Slurm batch job submission. It uses a simple example to introduce new users to requesting resources with `sbatch`, printing the `hostname`, and monitoring batch job submission.

1. [Sequential Job](#example-2-sequential-job) is used when tasks run one at a time sequentially. Adding more CPUs does not make a sequential job run faster. If you need to run many such sequential jobs simultaneously, you can submit it as an single [array job](#example-4-array-job). For instance, a Python or R script that executes a series of steps—such as data loading, extraction, analysis, and output reporting—where each step must be completed before the next can begin.

1. [Parallel Jobs](#example-3-parallel-jobs) is suitable for executing multiple independent tasks/jobs simultaneously and efficiently distributing them across resources. This approach is particularly beneficial for small-scale tasks that cannot be split into parallel processes within the code itself. For example, consider a Python script that operates on different data set, in such a scenario, you can utilize `srun` to execute multiple instances of the script concurrently, each operating on a different dataset and on different resources.

1. [Array Job](#example-4-array-job) is used for submitting and running multiple large number of identical tasks in parallel. They share the same code and execute with similar resource requirements. Instead of submitting multiple [sequential job](#example-2-sequential-job), you can submit a single array job, which helps to manage and schedule a large number of similar tasks efficiently. This improves efficiency, resource utilization, scalability, and ease of debugging. For instance, array jobs can be designed for executing multiple instances of the same task with slight variations in inputs or parameters such as perform [FastQC](https://home.cc.umanitoba.ca/~psgendb/doc/fastqc.help) processing on 10 different samples.

1. [Mutlithreaded or Multicore Job](#example-5-multithreaded-or-multicore-job) is used when software inherently support multithreaded parallelism i.e  run independent tasks simultaneously on multicore processors. For instance, there are numerous software such as [MATLAB](https://www.mathworks.com/help/matlab/ref/parfor.html), [FEBio](https://help.febio.org/FebioUser/FEBio_um_3-4-Section-2.6.html), [Xplor-NIH](https://nmr.cit.nih.gov/xplor-nih/doc/current/helperPrograms/options.html) support running multiple tasks at the same time on multicore processors. Users or programmers do not need to modify the code; you can simply enable multithreaded parallelism by configuring the appropriate options.

1. [GPU Job](#example-6-gpu-job) utilizes the parallel GPUs, which contain numerous cores designed to perform the same mathematical operations simultaneously. GPU job is appropriate for pipelines and software that are designed to run on GPU-based systems and efficiently distribute tasks across cores to process large datasets in parallel. Example includes [Tensorflow](https://www.tensorflow.org/guide/gpu), [Parabricks](../../education/case_studies.md), [PyTorch](https://pytorch.org/tutorials/prototype/ios_gpu_workflow.html#prototype-use-ios-gpu-in-pytorch), etc.

1. [Multinode Job](#example-7-multinode-job) is for pipeline/software that can be distributed and run across multiple nodes. For example, MPI based applications/tools such as [Quantum Expresso](https://www.quantum-espresso.org/Doc/user_guide/node20.html), [Amber](https://usc-rc.github.io/tutorials/amber), [LAMMPS](https://docs.lammps.org/Run_basics.html), etc.

### Example 1: A Simple Slurm Batch Job

Let us start with a simple example to print `hostname` of the node where your job is submitted. You will have to request for the required resources to run your job using Slurm parameters (lines 5-10). To learn more about individual Slurm parameters given in the example, please refer to [Slurm flag and environment variables](../slurm/submitting_jobs.md#slurm-flags-and-environment-variables) and the official [Slurm documentation](https://slurm.schedmd.com/).
o
To test this example, copy the below script in a file named `hostname.job`. This job executes the `hostname` command (line 15) on a single node, using one task, one CPU core, 1 gigabyte of memory, with a time limit of 10 minutes. The output and error logs are directed to separate files with names based on their job name and ID (line 11 and 12). For a more detailed understanding of the individual parameters used in this script, please refer to the section on [Simple Batch Job](../slurm/submitting_jobs.md#a-simple-batch-job). The following script includes comments, marked with `###`, describing their functions. We will utilize this notation for annotating comments in subsequent examples.

```bash linenums="1"
#!/bin/bash

### Declaring Slurm configuration options and specifying required resources
#SBATCH --job-name=hostname     ### Name of the job
#SBATCH --nodes=1               ### Number of Nodes
#SBATCH --ntasks=1              ### Number of Tasks
#SBATCH --cpus-per-task=1       ### Number of Tasks per CPU
#SBATCH --mem=1G                ### Memory required, 1 gigabyte
#SBATCH --partition=express     ### Cheaha Partition
#SBATCH --time=00:10:00         ### Estimated Time of Completion, 10 minutes
#SBATCH --output=%x_%j.out      ### Slurm Output file, %x is job name, %j is job id
#SBATCH --error=%x_%j.err       ### Slurm Error file, %x is job name, %j is job id

### Running the command `hostname`
hostname
```

#### Submitting and Monitoring the Job

Now submit the script `hostname.job` for execution on Cheaha cluster using `sbatch hostname.job`. Slurm processes the job script and schedules the job for execution on the cluster. The output you see, "Submitted batch job 26035322," indicates that the job submission was successful, and Slurm has assigned a unique job ID `26035322`.

```bash
$sbatch hostname.job

Submitted batch job 26035322
```

After submitting the job, Slurm will create the output and error files with job name `hostname` and id `26035322`  as,

```bash
$ ls

hostname_26035322.err  hostname_26035322.out  hostname.sh
```

The submitted job will be added to the Slurm queue and will wait for available resources based on the specified job configuration and the current state of the cluster. You can use `squeue -j job_id` to monitor the status of your job.

```bash
$squeue -j 26035322

JOBID      PARTITION    NAME        USER    ST       TIME  NODES NODELIST(REASON)
26035322   express      hostname    USER    CG       0:01      1 c0156
```

The above output provides a snapshot of the job's status, resource usage,  indicating that it is currently running on one node (c0156). The term `CG` refers to completing its execution. For more details refer to [Managing Slurm jobs](../slurm/job_management.md). If the job is successful, the `hostname_26035322.err` file will be empty/without error statement. You can print the result using,

```bash
$ cat hostname_26035322.out
c0156
```

### Example 2: Sequential Job

This example illustrate a Slurm job that runs a Python script involving [NumPy](https://numpy.org/) operation. This python script is executed sequentially using the same resource configuration as [Example 1](../slurm/slurm_tutorial.md#example-1-a-simple-slurm-batch-job). Let us name the below script as `numpy.job`.

```bash linenums="1"
#!/bin/bash
#SBATCH --job-name=numpy            ### Name of the job
#SBATCH --nodes=1                   ### Number of Nodes
#SBATCH --ntasks=1                  ### Number of Tasks
#SBATCH --cpus-per-task=1           ### Number of Tasks per CPU
#SBATCH --mem=4G                    ### Memory required, 4 gigabyte
#SBATCH --partition=express         ### Cheaha Partition
#SBATCH --time=01:00:00             ### Estimated Time of Completion, 1 hour
#SBATCH --output=%x_%j.out          ### Slurm Output file, %x is job name, %j is job id
#SBATCH --error=%x_%j.err           ### Slurm Error file, %x is job name, %j is job id

### Loading Anaconda3 module to activate `pytools-env` conda environment
module load Anaconda3
conda activate pytools-env

### Run the script `python_test.py`
python python_test.py
```

 The batch job requires an input file `python_test.py` (line 17) for execution. Copy the input file from the [Containers page](../../workflow_solutions/getting_containers.md#create-your-own-docker-container). Place this file in the same folder as the `numpy.job`. This python script performs numerical integration and data visualization tasks, and it relies on the following packages: numpy, matplotlib, scipy for successful execution. These dependencies can be installed using [Anaconda](../../workflow_solutions/using_anaconda.md) within a `conda` environment named `pytools-env`. Prior to running the script, load the `Anaconda3` module and activate the `pytools-env` environment (line 13 and 14).  Once job is successfully completed, check the slurm output file for results. Additionally, a plot named `testing.png` will be generated.

```bash
$ ls

numpy_26127143.err  numpy_26127143.out  numpy.job  python_test.py  testing.png
```

```bash
$cat numpy_26127143.out

[ 0 10 20 30 40]
[-5.  -4.5 -4.  -3.5 -3.  -2.5 -2.  -1.5 -1.  -0.5  0.   0.5  1.   1.5
  1.   2.5  3.   3.5  4.   4.5]
[ 0.   0.5  1.   1.5  2.   2.5  3.   3.5  4.   4.5  5.   5.5  6.   6.5
  1.   7.5  8.   8.5  9.   9.5 10.  10.5 11.  11.5 12.  12.5 13.  13.5
 1.   14.5 15.  15.5 16.  16.5 17.  17.5 18.  18.5 19.  19.5 20. ]
(2.0, 2.220446049250313e-14)
```

You can review detailed information about finished jobs using `sacct` command for a specific job id as shown below. For instance, this job was allocated with one CPU and has been successfully completed. The lines with ".ba+" and ".ex+" refer to batch step and external step within a job, but we will ignore them for simplicity in this and future examples. The exit code `0:0` signifies a normal exit with no errors.

```bash
$ sacct -j 26127143

       JobID    JobName  Partition    Account  AllocCPUS      State ExitCode
------------ ---------- ---------- ---------- ---------- ---------- --------
26127143          numpy    express      USER          1  COMPLETED      0:0
26127143.ba+      batch                 USER          1  COMPLETED      0:0
26127143.ex+     extern                 USER          1  COMPLETED      0:0
```

### Example 3: Parallel Jobs

Multiple jobs or tasks can be executed simultaneously using `srun` within a single batch script. In this example, the same executable `python_script_new.py` is run in parallel with distinct inputs (line 17-19). The `&` symbol at the end of each line run these commands in background. The `wait` command (line 20) performs synchronization and ensures that all background processes and parallel tasks are completed before finishing. In Line 4, three tasks are requested as there are three executables to be run in parallel. The overall job script is allocated with three CPUs, and in lines(17-19), each `srun` script utilizes 1 CPU to perform their respective task. Copy the batch script into a file named `multijob.job`. Use the same `conda` environment `pytools-env` shown in [example2](../slurm/slurm_tutorial.md#example-2-sequential-job).

```bash linenums="1"
#!/bin/bash
#SBATCH --job-name=multijob             ### Name of the job
#SBATCH --nodes=1                       ### Number of Nodes
#SBATCH --ntasks=3                      ### Number of Tasks
#SBATCH --cpus-per-task=1               ### Number of Tasks per CPU
#SBATCH --mem=4G                        ### Memory required, 4 gigabyte
#SBATCH --partition=express             ### Cheaha Partition
#SBATCH --time=01:00:00                 ### Estimated Time of Completion, 1 hour
#SBATCH --output=%x_%j.out              ### Slurm Output file, %x is job name, %j is job id
#SBATCH --error=%x_%j.err               ### Slurm Error file, %x is job name, %j is job id

### Loading Anaconda3 module to activate `pytools-env` conda environment
module load Anaconda3
conda activate pytools-env

### Runs the script `python_test.py` in parallel with distinct inputs and ensures synchronization
srun --nodes=1 --ntasks=1 python python_script_new.py 1 100000 &
srun --nodes=1 --ntasks=1 python python_script_new.py 100001 200000 &
srun --nodes=1 --ntasks=1 python python_script_new.py 200001 300000 &
wait
```

Copy the following python script and call it as `python_script_new.py`. The input file takes two command-line arguments i.e. the `start` and `end` values. The script uses these values to creates an array and compute the sum of its elements using numpy. The above batch script runs three parallel instances of this Python script with different inputs.

```bash linenums="1"
import sys
import numpy as np

### Verify if correct number of command-line arguments is provided
if len(sys.argv) != 3:
  print("Usage: python python_script_new.py <start> <end>")
  sys.exit(1)

### Passing start and end values from command-line arguments
start = int(sys.argv[1])
end = int(sys.argv[2])

### Create an array from start to end using numpy
input_array = np.arange(start, end)

### Perform addition on the array elements using numpy's sum function
sum_result = np.sum(input_array)

### Print Input Range and Sum
print("Input Range: {} to {}, Sum: {}".format(start, end, sum_result))
```

The below output shows that each line corresponds to the output of one parallel execution of python script with specific input ranges. Note that the results are in out of order. This is because each `srun` script runs independently, and their completion times may vary based on factors such as system load, resource availability, and the nature of their computations. If the results must be in order to be correct, you will need to modify your script to explicitly collect and organize them. One possible approach can be found in the section [srun for running parallel jobs](../slurm/submitting_jobs.md#srun-for-running-parallel-jobs) (refer to example 2).

```bash
$cat multijob_27099591.out

Input Range: 1 to 100000, Sum: 4999950000
Input Range: 200001 to 300000, Sum: 24999750000
Input Range: 100001 to 200000, Sum: 14999850000
```

The `sacct` report indicates that three CPUs have been allocated. The python script executes with unique task IDs 27099591.0,27099591.1,27099591.2.

```bash
$ sacct -j 27099591

       JobID    JobName  Partition    Account  AllocCPUS      State ExitCode
------------ ---------- ---------- ---------- ---------- ---------- --------
27099591       multijob    express      USER          3  COMPLETED      0:0
27099591.ba+      batch                 USER          3  COMPLETED      0:0
27099591.ex+     extern                 USER          3  COMPLETED      0:0
27099591.0       python                 USER          1  COMPLETED      0:0
27099591.1       python                 USER          1  COMPLETED      0:0
27099591.2       python                 USER          1  COMPLETED      0:0
```

### Example 4: Array Job

Array jobs are more effective when you have a larger number of similar tasks to be executed simultaneously with varied input data, unlike `srun` parallel jobs which are suitable for running a smaller number of tasks concurrently (e.g. less than 5). Array jobs are easier to manage and monitor multiple tasks through unique identifiers. However, with the increased power come more moving parts, so the example here requires some setup.

#### Environment Setup for Slurm Array Execution

To keep things organized, let us first structure the directories and implement necessary preprocessing step required for the subsequent Slurm array examples.

**(i) Setup**: This part of the section covers creating the necessary directories and input files. Since this setup is common for all the following examples, we will do it once before moving on.

Before executing the Slurm array examples [parallel-python-tasks](#example-41-running-parallel-python-tasks-with-dynamic-input-ranges), [word-count-line-by-line](#example-42-line-by-line-word-count) and [dynamic-word-count-multiple-files](#example-43-dynamically-reading-and-counting-words-in-multiple-files) and [word-count-from-file-list](#example-44-counting-words-in-multiple-files-from-a-file-list), let us set up the environment. This includes creating a main directory, `array_example`, to store all job files and scripts. Within it, we will create a directory `logs` to organize output and error logs. Let us create and structure these directories.

```bash
$mkdir array_example
$cd array_example
$mkdir logs
```

**(ii) Input File Generation**: For the slurm array examples [word-count-line-by-line](#example-42-line-by-line-word-count) and [dynamic-word-count-multiple-files](#example-43-dynamically-reading-and-counting-words-in-multiple-files) and [word-count-from-file-list](#example-44-counting-words-in-multiple-files-from-a-file-list), let us create subdirectory `input_files` within the `array_example` folder to store a set of generated input text files.

```bash
$cd array_example
$mkdir input_files
$cd input_files
```

The next step involves generating input files using a pseudo-random text generation script. Save the following script as `generate_input.sh` within the `input_files` folder. This script generates five random text files, each containing a different number of randomly selected words. The script creates a file with 1 to 10 randomly generated lines, where each line contains 5 to 20 randomly selected words from `/usr/share/dict/words`. The output is saved in files named random_file_1.txt, random_file_2.txt, ..., random_file_5.txt, with each file containing a unique, randomly determined number of lines and words per line. For more details refer to [script concepts](../../workflow_solutions/shell.md#script-concepts) and [bash scripting example](../../cheaha/software/modules.md#best-practice-for-loading-modules).

```bash
#!/bin/bash

### Loop to create 5 random files
for i in {1..5}; do
    ### Generate a random number of lines (1-10)
    num_lines=$(($RANDOM % 10 + 1))
    for _ in $(seq 1 $num_lines); do
    ### Generate a random words per line (5-20)
        num_words=$(($RANDOM % 20 + 5))
        ### Use 'shuf' to pick $num_words random words from the dictionary file.
        ### The pipe ('|') sends these selected words to 'paste', which combines them into a single line,
        ### separating each word with a space (using the '-d " "' option).
        shuf -n $num_words /usr/share/dict/words | paste -s -d " "
    done > "random_file_$i.txt"
done
```

You can execute the `generate_input.sh` script as shown below. The first command `chmod` grants execute permission to the script so it can be run directly. The second command runs the script to generate the required input files. For more details on usage of bash scripting, refer to [Script Concepts](../../workflow_solutions/shell.md/#script-concepts)

```bash
$chmod +x generate_input.sh
$./generate_input.sh
```

Now that the general environment and preprocessing setup are complete, the following sections will walk you through four detailed Slurm array job examples.

#### Example 4.1: Running Parallel Python Tasks with Dynamic Input Ranges

The following Slurm script is an example of how you might convert the previous [parallel job example](#example-3-parallel-jobs) script to an array job. To start, copy and save the below script to a file named, `slurm_array.job` within the `array_example` folder. The script requires the input file `python_script_new.py` and the `conda` environment `pytools-env`, similar to those used in [example2](../slurm/slurm_tutorial.md#example-2-sequential-job) and [example 3](../slurm/slurm_tutorial.md#example-3-parallel-jobs). Line 11 specifies the script as an array job, treating each task within the array as an independent job. For each task, lines 18-19 calculates the input range. `SLURM_ARRAY_TASK_ID` identifies the task executed using indexes, and is automatically set for array jobs. The python script (line 24) runs individual array task concurrently on respective input range. The command `awk` is used to prepend each output line with the unique task identifier and then append the results to the file, `output_all_tasks.txt`. For more details on on parameters of array jobs, please refer to [Batch Array Jobs](../slurm/submitting_jobs.md#batch-array-jobs-with-known-indices) and [Practical Batch Array Jobs](../slurm/practical_sbatch.md#).

<!-- markdownlint-disable MD046 -->
!!! important

    For large array jobs, implementing [throttling](./submitting_jobs.md#throttling-in-slurm-array-jobs) helps control the number of concurrent jobs, preventing resource contention across the Cheaha cluster. Running too many jobs at once can cause competition for CPU, memory, or I/O, which may negatively impact performance.
<!-- markdownlint-enable MD046 -->

```bash linenums="1"
#!/bin/bash
#SBATCH --job-name=slurm_array       ### Name of the job
#SBATCH --nodes=1                    ### Number of Nodes
#SBATCH --ntasks=1                   ### Number of Tasks
#SBATCH --cpus-per-task=1            ### Number of Tasks per CPU
#SBATCH --mem=4G                     ### Memory required, 4 gigabyte
#SBATCH --partition=express          ### Cheaha Partition
#SBATCH --time=01:00:00              ### Estimated Time of Completion, 1 hour
### Slurm Output file, %x is job name, %A is array job id, %a is array job index
#SBATCH --output=logs/%x_%A_%a.out
### Slurm Error file, %x is job name, %A is array job id, %a is array job index
#SBATCH --error=logs/%x_%A_%a.err
#SBATCH --array=0-2                  ### Number of Slurm array tasks, 3 tasks

### Loading Anaconda3 module to activate `pytools-env` conda environment
module load Anaconda3
conda activate pytools-env

### Calculate the input range for each task
start=$((($SLURM_ARRAY_TASK_ID) * 100000 + 1))
end=$((($SLURM_ARRAY_TASK_ID + 1) * 100000))

### Run the python script with input arguments and append the results to a .txt file for each task
python python_script_new.py $start $end 2>&1 \
  | awk -v task_id=$SLURM_ARRAY_TASK_ID '{print "array task " task_id, $0}' \
  >> output_all_tasks.txt
```

Submit the script `slurm_array` for execution using the command,

```bash
$cd array_example
$sbatch slurm_array.job
```

The output shows the sum of different input range computed by individual task, making it easy to track using a task identifier, such as array task 1/2/3. While each task is clearly labeled (e.g., "array task 1"), the order in which these outputs appear in the file may not be in order. This is because Slurm array tasks run in parallel and write to the output file concurrently, so the write order is not guaranteed.

```bash
$ cat $HOME/array_example/output_all_tasks.txt

array task 2 Input Range: 200001 to 300000, Sum: 24999750000
array task 1 Input Range: 100001 to 200000, Sum: 14999850000
array task 0 Input Range: 1 to 100000, Sum: 4999950000
```

The `sacct` report indicates that the job `33509888` consists of three individual tasks, namely `33509888_0`, `33509888_1`, and `33509888_2`. Each task was allocated one CPU resource.

```bash
$ sacct -j 33509888

       JobID    JobName  Partition    Account  AllocCPUS      State ExitCode
------------ ---------- ---------- ---------- ---------- ---------- --------
33509888_2   slurm_arr+    express      prema          1  COMPLETED      0:0
33509888_2.+      batch                 prema          1  COMPLETED      0:0
33509888_2.+     extern                 prema          1  COMPLETED      0:0
33509888_0   slurm_arr+    express      prema          1  COMPLETED      0:0
33509888_0.+      batch                 prema          1  COMPLETED      0:0
33509888_0.+     extern                 prema          1  COMPLETED      0:0
33509888_1   slurm_arr+    express      prema          1  COMPLETED      0:0
33509888_1.+      batch                 prema          1  COMPLETED      0:0
33509888_1.+     extern                 prema          1  COMPLETED      0:0
```

In the following three examples, we will explore additional Slurm array job scenarios that are commonly used in scientific simulations.

#### Example 4.2: Line-by-Line Word Count

In the examples [word-count-line-by-line](#example-42-line-by-line-word-count), [dynamic-word-count-multiple-files](#example-43-dynamically-reading-and-counting-words-in-multiple-files), and [word-count-from-file-list](#example-44-counting-words-in-multiple-files-from-a-file-list), let us explore how to use a Slurm Array Job to process text files in parallel.

The following Slurm job script processes a text file line by line using an array job. Save the following Slurm script as `line_word_count.job` within `array_example` folder. Prior to running this job make sure to complete the [input file generation](#example-4-array-job) step. This SLURM job ensures that each task in the job array processes a single line from an input file and counts the number of words in that line.

```bash linenums="1"
#!/bin/bash
#SBATCH --job-name=line_word_count  ### Name of the job
#SBATCH --cpus-per-task=1           ### Number of Tasks per CPU
#SBATCH --mem=2G                    ### Memory required, 2 gigabyte
#SBATCH --partition=express         ### Cheaha Partition
#SBATCH --time=00:10:00             ### Estimated Time of Completion, 10 minutes
### Slurm Output file, %x is job name, %A is array job id, %a is array job index
#SBATCH --output=logs/%x_%A_%a.out
### Slurm Error file, %x is job name, %A is array job id, %a is array job index
#SBATCH --error=logs/%x_%A_%a.err

### Define the input file, for instance, random_file_2.txt
INPUT_FILE="$HOME/array_example/input_files/random_file_2.txt"

### `sed` is a shell command and stream editor for manipulating text.
### It extracts the line from $INPUT_FILE matching the task ID (adjusted for zero-based indexing)
LINE=$(sed -n "$((SLURM_ARRAY_TASK_ID + 1))p" "$INPUT_FILE")

### The command `wc -w` counts the number of words in the extracted line ($LINE),
### while echo "$LINE" prints the line of text. The pipe (|) then passes this text from echo "$LINE"
### to wc -w, which counts the words in the line for each task.
WORD_COUNT=$(echo "$LINE" | wc -w)
echo "Task $SLURM_ARRAY_TASK_ID: $WORD_COUNT words"
```

Before running the above SLURM job, determine the number of lines in the input file (e.g., random_file_2.txt) using the following command:

```bash
$MAX_TASKS=$(wc -l < $HOME/array_example/input_files/random_file_2.txt)
```

Next, submit the array job using the command below. It creates an array of tasks starting from 0 up to (MAX_TASKS – 1), matching the zero-based indexing used in the script.

```bash
$sbatch --array=0-$(($MAX_TASKS - 1)) line_word_count.job
```

The below output comes from the SLURM job array script (line_word_count.job), where each task processes one line from random_file_2.txt. Since the file has 3 lines, SLURM created 3 tasks (Task 0, Task 1, Task 2), each counting words in its respective line.

```bash
### counts the number of lines in the file, for instance, random_file_2.txt
$ wc -l $HOME/array_example/input_files/random_file_2.txt
3 $HOME/array_example/input_files/random_file_2.txt
```

```bash
### Print and verify the output using the `cat` command
$ cat $HOME/array_example/logs/line_word_count_$JOB_ID_*.out
Task 0: 8 words
Task 1: 19 words
Task 2: 6 words
```

#### Example 4.3: Dynamically Reading and Counting Words in Multiple Files

This example job script performs the same function as example [word-count-line-by-line](#example-42-line-by-line-word-count), but instead of counting the number of words in a single file line by line, it is designed to dynamically count the number of words across multiple files. Save the below script as `dynamic_file_word_count.job` within the `array_example` folder. It utilizes the same input files generated from the [Input File Generation](#example-4-array-job) section.

```bash linenums="1"
#!/bin/bash
#SBATCH --job-name=dynamic_file_word_count   ### Name of the job
#SBATCH --cpus-per-task=1                    ### Number of Tasks per CPU
#SBATCH --mem=4G                             ### Memory required, 4 gigabyte
#SBATCH --partition=express                  ### Cheaha Partition
#SBATCH --time=00:15:00                      ### Estimated Time of Completion, 15 minutes
### Slurm Output file, %x is job name, %A is array job id, %a is array job index
#SBATCH --output=logs/%x_%A_%a.out
### Slurm Error file, %x is job name, %A is array job id, %a is array job index
#SBATCH --error=logs/%x_%A_%a.err

### Define working directory
WORKDIR="$HOME/array_example/input_files"

### Find all files in the working directory ($WORKDIR) that match the pattern "random_file_*.txt"
### Pass the list of these files to the 'sort' command via the pipe ('|')
### The sorted file paths are then captured and stored in the FILES array.
FILES=($(find "$WORKDIR" -type f -name "random_file_*.txt" | sort))

### Selects the file corresponding to the current Slurm Array task ID.
### The task IDs are assigned starting from 1, but in bash, array indexing starts from 0.
### To align the task ID with bash’s 0-based array indexing, we subtract 1 from the task ID.
FILE="${FILES[$SLURM_ARRAY_TASK_ID-1]}"

### Get the full path to the directory containing the file
DIRNAME=$(dirname "$FILE")

### Extract the base name of the file (without the extension)
BASENAME=$(basename "$FILE" .txt)

### Check if file exists and count words from file
if [[ -f "$FILE" ]]; then
    ### If the file exists, count the number of words in the file using 'wc -w'
    ### The result is saved to a file with the same name as the original file,
    ### but with a '.wordcount' extension in the same directory as the original file.
    wc -w "$FILE" > "$DIRNAME/${BASENAME}.wordcount"
    echo "Processed $FILE"
else
    echo "File not found: $FILE"
fi
```

The script begins by setting up the Slurm array job with required resources. The working directory is defined, and the script uses the `find` command to search for files matching the pattern random_file_*.txt within that working directory and its subdirectories, passing the resulting list of file paths to the sort command to ensure they are processed in order. The file corresponding to the current Slurm array task ID is selected, and its directory and base name are extracted. If the file exists, the script counts the words in the file using the `wc -w` command and saves the word count to a new file with a `.wordcount` extension in the same directory. If the file is not found, an error message is displayed.

Before you run the script `dynamic_file_word_count.job`, determine the number of files in the directory and its subdirectories using the following command. The command counts the number of files matching the pattern random_file_*.txt in the path`$HOME/array_example/input_files` directory and its subdirectories, and stores the result in the variable `MAX_TASKS`.

```bash
$MAX_TASKS=$(find $HOME/array_example/input_files -type f -name "random_file_*.txt" | wc -l)
```

Next, submit the array job using the command below, which creates an array of tasks from 1 to the value of MAX_TASKS, where each task corresponds to processing a different file listed in the array.

```bash
$sbatch --array=1-"$MAX_TASKS" dynamic_file_word_count.job
```

In the output below, each file was processed independently by a Slurm job array task. The task IDs 1, 2, 3, 4, and 5 correspond to the five different files being processed. For instance, here the Slurm Job ID is `31934540`. Each output file contains the word count for a specific text file handled by its respective Slurm array task.

The `find` command in the output shows the word counts for each of the random_file*.txt files, where each `.wordcount` file contains the word count for its corresponding input file.

```bash
### Listing all the output files generated by the Slurm from the `logs` directory
$ cd $HOME/array_example/logs/
$ ls dynamic_file_word_count*.out
dynamic_file_word_count_31934540_1.out  dynamic_file_word_count_31934540_3.out
dynamic_file_word_count_31934540_2.out  dynamic_file_word_count_31934540_4.out
dynamic_file_word_count_31934540_5.out

### Finds all files matching "random_file*.wordcount" in the specified directory and subdirectories.
### For each file, display contents with word count and file path.
$ find $HOME/array_example/input_files/ -type f -name "random_file*.wordcount" -exec cat {} \;
81 /home/$USER/Tutorial/slurm_tutorial/example4/input_files/random_file_1.txt
117 /home/$USER/Tutorial/slurm_tutorial/example4/input_files/random_file_5.txt
33 /home/$USER/Tutorial/slurm_tutorial/example4/input_files/random_file_2.txt
104 /home/$USER/Tutorial/slurm_tutorial/example4/input_files/random_file_3.txt
114 /home/$USER/Tutorial/slurm_tutorial/example4/input_files/random_file_4.txt
```

#### Example 4.4: Counting Words in Multiple Files from a File List

This example job script is similar to example [dynamic-word-count-multiple-files](#example-43-dynamically-reading-and-counting-words-in-multiple-files), but instead of dynamically reading files from the directory, it counts the number of words in multiple files in parallel using a SLURM job array, with the files listed in a separate file list. This example utilizes the same input files from [Example 4](#example-4-array-job) as described in the `Input File Generation` section.

To create a file list by tracking all the files in the `input_files` directory, use the find command along with globbing to list all generated files in the current directory and its subdirectories. The input files along with its absolute path are tracked in `file_list.txt`.

```bash
### Change to the "array_example" directory.
cd array_example
### Search for all files in the "input_files" directory matching the pattern "random_file_*.txt".
### For each matching file, the 'realpath' command is used to get the absolute path of the file.
### The output is then sorted and written to "file_list.txt".
find $HOME/array_example/input_files -type f \
  -name "random_file_*.txt" \
  -exec realpath {} \; | \
  sort > file_list.txt
```

Copy the following SLURM array job script to a file naming `file_list_word_count.job` within the `array_example` folder. This Slurm array job script count the number of words across multiple files listed in a file list `file_list.txt`.

```bash linenums="1"
#!/bin/bash
#SBATCH --job-name=file_list_word_count   ### Name of the job
#SBATCH --cpus-per-task=1            ### Number of Tasks per CPU
#SBATCH --mem=4G                     ### Memory required, 4 gigabyte
#SBATCH --partition=express          ### Cheaha Partition
#SBATCH --time=00:15:00              ### Estimated Time of Completion, 15 minutes
### Slurm Output file, %x is job name, %A is array job id, %a is array job index
#SBATCH --output=logs/%x_%A_%a.out
### Slurm Error file, %x is job name, %A is array job id, %a is array job index
#SBATCH --error=logs/%x_%A_%a.err

### Define working directory
WORKDIR="$HOME/array_example/input_files"
FILELIST="file_list.txt"

### Get the file corresponding to the current task ID
### 'sed' picks the file from the $FILELIST based on the task ID (SLURM_ARRAY_TASK_ID).
FILE=$(sed -n "${SLURM_ARRAY_TASK_ID}p" $FILELIST)

### Extract the base name of the file (without the .txt extension)
BASENAME=$(basename "$FILE" .txt)

### Check if file exists and count words from file
if [[ -f "$FILE" ]]; then
    ### `wc -w` counts the words, saving the result in a new file with `.wordcount` appended.
    wc -w "$FILE" > "$WORKDIR/${BASENAME}.wordcount"
    echo "Processed $FILE"
else
    echo "File not found: $FILE"
fi
```

The above SLURM job script runs a word count operation in parallel on multiple files using a job array (1-5). It reads a list of filenames along with its path from `file_list.txt` located in `$HOME/array_example/input_files`. Each task in the job array processes a different file based on its task ID (SLURM_ARRAY_TASK_ID). If the file exists, it counts the number of words using `wc -w` and saves the output as `<filename>.wordcount`, and logs standard output and errors for each task separately.

Before running the above SLURM job, determine the number of files in the directory and its subdirectories using the following command. The command counts the number of files matching the pattern random_file_*.txt in the path `$HOME/array_example/input_files` directory and its subdirectories, and stores the result in the variable `MAX_TASKS`.

```bash
### Get the total number of tasks by counting the lines in 'file_list.txt'
### 'wc -l' counts the number of lines in the file, which represents the number of tasks.
$MAX_TASKS=$(wc -l < $HOME/array_example/file_list.txt)
```

Next, submit the array job using the command below, which creates an array of tasks from 1 to the value of MAX_TASKS, where each task corresponds to processing a different file listed in the array.

```bash
### Submit a job array with tasks ranging from 1 to MAX_TASKS
$sbatch --array=1-"$MAX_TASKS" file_list_word_count.job
```

The output generated will be similar to example [dynamic-word-count-multiple-files](#example-43-dynamically-reading-and-counting-words-in-multiple-files), as the functionality is the same, but the method of handling the input data differs.

### Example 5: Multithreaded or Multicore Job

This Slurm script illustrates execution of a MATLAB script in a multithread/multicore environemnt. Save the script as `multithread.job`. The `%` symbol in this script denotes comments within MATLAB code. Line 16 runs the MATLAB script `parfor_sum_array`, with an input array size `100` passed as argument, using 4 CPU cores (as specified in Line 5).

```bash linenums="1"
#!/bin/bash
#SBATCH --job-name=multithread          ### Name of the job
#SBATCH --nodes=1                       ### Number of Nodes
#SBATCH --ntasks=1                      ### Number of Tasks
#SBATCH --cpus-per-task=4               ### Number of Tasks per CPU
#SBATCH --mem=16G                       ### Memory required, 16 gigabyte
#SBATCH --partition=express             ### Cheaha Partition
#SBATCH --time=01:00:00                 ### Estimated Time of Completion, 1 hour
#SBATCH --output=%x_%j.out              ### Slurm Output file, %x is job name, %j is job id
#SBATCH --error=%x_%j.err               ### Slurm Error file, %x is job name, %j is job id

### Loading required MATLAB module
module load rc/matlab/R2023b

### Executing the matlab script with input arguments
matlab -nosplash -nodesktop -r "parfor_sum_array(100); quit;"
```

Copy the below MATLAB script as `parfor_sum_array.m`. At the beginning, the script defines a function `sum_array` and variable `array_size` is passed as an input argument. This function uses multithreading with the `parfor` option to calculate the sum of elements in an array. On Line 10, the number of workers (`num_workers`) is set to the value of the environment variable `SLURM_CPUS_PER_TASK` i.e. 4. The script then creates a parallel pool using lines 13-17, utilizing the specified number of workers. The parallel computation of summing up of array elements is performed using a `parfor` loop in lines 23-27. By using `parfor` with a pool of workers, operations are run in parallel for improved performance. More insights on usage of `parfor` can be found in the official [MATLAB page](https://www.mathworks.com/help/matlab/ref/parfor.html).

<!-- markdownlint-disable MD046 -->
!!! important

    Make sure that the `SLURM_CPUS_PER_TASK > 1` in order to take advantage of multithreaded performance. It is important that the  `SLURM_CPUS_PER_TASK` does not exceed the number of workers and physical cores (i.e. CPU cores) available on the node. This is to prevent high context switching, where individual CPUs are constantly switching between multiple running processes, which can negatively impact job performance of all jobs running on the node. It may also lead to overhead during job execution and result in poorer performance. Please refer to our [Hardware page](../hardware.md#hardware-information) to learn more about resource limits and selecting appropriate resources.
<!-- markdownlint-disable MD046 -->

<!-- markdownlint-disable MD046 -->
!!! bug

    There is a known issue with `parpool` and other related multi-core parallel features such as `parfor` affecting R2022a and earlier. See our [Modules Known Issues section](../software/modules.md#matlab-issues) for more information.
<!-- markdownlint-enable MD046 -->

```bash linenums="1"
% Function to calculate the sum of an array in parallel. This function takes array_size as input from command-line arguments
function sum_array(array_size)
    % Check if input array size is provided
    if nargin < 1
        disp('error: pass input array size as arguments');
        return;
    end

    % The number of workers is set based on Slurm parameter i.e. number of CPUS per task
    num_workers = str2double(getenv('SLURM_CPUS_PER_TASK'));

    % Create parallel pool
    poolobj = gcp('nocreate');
    if isempty(poolobj) || poolobj.NumWorkers ~= num_workers
        delete(poolobj);
        parpool(num_workers);
    end

    % Initialization of Array
    A = zeros(1, array_size);

    % Perform parallel computation for each element of the array
    sum_result = 0;
    parfor i = 1:array_size
        A(i) = i;
        sum_result = sum_result + A(i);
    end

    % Display the total sum
    disp(['Sum of array is: ' num2str(sum_result)]);
end
```

The below result summarizes the parallel pool initialization and its utilization of 4 workers for  computation of sum of an array. Followed by, the `sacct` report illustrates that the multithreaded job was allocated with 4 CPUs and was successfully completed.

```bash
$ cat multithread_27105035.out

MATLAB is selecting SOFTWARE OPENGL rendering.

                            < M A T L A B (R) >
                  Copyright 1984-2023 The MathWorks, Inc.
             R2023b Update 6 (23.2.0.2485118) 64-bit (glnxa64)
                             December 28, 2023

To get started, type doc.
For product information, visit www.mathworks.com.

Starting parallel pool (parpool) using the 'Processes' profile ...
Connected to parallel pool with 4 workers.
Sum of array is: 5050
Parallel pool using the 'Processes' profile is shutting down.
```

```bash
$ sacct -j 27105035

       JobID    JobName  Partition    Account  AllocCPUS      State ExitCode
------------ ---------- ---------- ---------- ---------- ---------- --------
27105035     multithre+    express      USER          4  COMPLETED      0:0
27105035.ba+      batch                 USER          4  COMPLETED      0:0
27105035.ex+     extern                 USER          4  COMPLETED      0:0
```

### Example 6: GPU Job

This slurm script shows the execution of Tensorflow job using GPU resources. Let us save this script as `gpu.job`. The Slurm parameter `--gres=gpu:2` in line 6, requests for 2 GPUs. In line 8, note that in order to run GPU-based jobs, either the `amperenodes` or `pascalnodes` partition must be used (please refer to our [GPU page](../slurm/gpu.md) for more information). Lines 14-15 loads the necessary CUDA modules, while lines 18-19 load the Anaconda module and activate a `conda` environment called `tensorflow`. Refer to [Tensorflow official page](https://www.tensorflow.org/) for installation. The last line executes a python script that utilizes Tensorflow library to perform matrix multiplication across multiple GPUs.

```bash linenums="1"
#!/bin/bash
#SBATCH --job-name=gpu              ### Name of the job
#SBATCH --nodes=1                   ### Number of Nodes
#SBATCH --ntasks=1                  ### Number of Tasks
#SBATCH --cpus-per-task=1           ### Number of Tasks per CPU
#SBATCH --gres=gpu:2                ### Number of GPUs, 2 GPUs
#SBATCH --mem=16G                   ### Memory required, 16 gigabyte
#SBATCH --partition=amperenodes     ### Cheaha Partition
#SBATCH --time=01:00:00             ### Estimated Time of Completion, 1 hour
#SBATCH --output=%x_%j.out          ### Slurm Output file, %x is job name, %j is job id
#SBATCH --error=%x_%j.err           ### Slurm Error file, %x is job name, %j is job id

### Loading the required CUDA and cuDNN modules
module load CUDA/12.2.0
module load cuDNN/8.9.2.26-CUDA-12.2.0

### Loading the Anaconda module and activating the `tensorflow` environment
module load Anaconda3
conda activate tensorflow

### Executing the python script
python matmul_tensorflow.py
```

Let us now create a file named `matmul_tensorflow.py` and copy the following script into it. This python script demonstrates the utilization of Tensorflow library to distribute computational tasks among multiple GPUs, in order to perform matrix multiplication in parallel (Lines 11-19). Lines 8-9 retrieve the logical GPUs and enable device placement logging, which helps to analyze which device is used for each operation. The final results are aggregated and the sum is computed on the CPU device (lines 22-23).

```bash linenums="1"
import tensorflow as tf

### Print Tensorflow version and check for available number of GPUs
print("TensorFlow version:", tf.__version__)
print("Num GPUs Available: ", len(tf.config.list_physical_devices('GPU')))

### Get the logical GPUs and enable device placement
gpus = tf.config.list_logical_devices('GPU')
tf.debugging.set_log_device_placement(True)

if gpus:
    ### Create tensors on each GPU and Perform matrix multiplication on multiple GPUs
    c = []
    for gpu in gpus:
        with tf.device(gpu.name):
            a = tf.random.uniform(shape=(4, 3))
            b = tf.random.uniform(shape=(3, 4))
            c.append(tf.matmul(a, b))
            print(f"Computation on GPU: {gpu.name}")

    ### Calculate the Sum on CPU device
    with tf.device('/CPU:0'):
        matmul_sum = tf.add_n(c)

    ### Print the result
    print(matmul_sum)
```

The results indicate that the Tensorflow version utilized is 2.15. The segments `/device:GPU:0` and `/device:GPU:1` specify that the computations were executed on two GPUs. The final results is a 4x4 matrix obtained by summing the matrix multiplication results. In the `sacct` report, the column `AllocGRES` shows that 2 GPUs are allocated for this job.

```bash
$ cat gpu_27107694.out

TensorFlow version: 2.15.0
Num GPUs Available:  2
Computation on GPU: /device:GPU:0
Computation on GPU: /device:GPU:1
tf.Tensor(
[[1.6408134 0.9900811 1.3046092 0.9307438]
 [1.5603762 1.6812123 1.8867838 1.0662912]
 [2.481688  1.8107605 2.0444224 1.5500932]
 [2.415476  1.9280369 2.020216  1.4872619]], shape=(4, 4), dtype=float32)
```

```bash
$ sacct -j 27107694 --format=JobID,JobName,Partition,Account,AllocCPUS,allocgres,State,ExitCode

       JobID    JobName  Partition    Account  AllocCPUS    AllocGRES      State ExitCode
------------ ---------- ---------- ---------- ---------- ------------ ---------- --------
27107694            gpu amperenod+      USER          1        gpu:2  COMPLETED      0:0
27107694.ba+      batch                 USER          1        gpu:2  COMPLETED      0:0
27107694.ex+     extern                 USER          1        gpu:2  COMPLETED      0:0
```

### Example 7: Multinode Job

The below Slurm script runs a Quantum Expresso job using the `pw.x` executable on multiple nodes. In this example, we request for 2 nodes on `amd-hdr100` partition in lines 4 and 7. The suitable Quantum Expresso module is loaded in line 13. The last line is configured for a parallel computation of Quantum Expresso simulation across 2 nodes `N 2` and 4 MPI processes `-nk 4` for the input parameters in `pw.scf.silicon.in`. The input file `pw.scf.silicon.in` and psuedo potential file is taken from the [github page](https://pranabdas.github.io/espresso/hands-on/scf/). However this input is subject to change, hence according to your use case you can change the inputs.

```bash linenums="1"
#!/bin/bash

#SBATCH --job-name=mpijob               ### Name of the job
#SBATCH --nodes=2                       ### Number of Nodes
#SBATCH --ntasks 4                      ### Number of Tasks
#SBATCH --mem=64G                       ### Memory required, 64 gigabyte
#SBATCH --partition=amd-hdr100          ### Cheaha Partition
#SBATCH --time=12:00:00                 ### Estimated Time of Completion, 12 hour
#SBATCH --output=%x_%j.out              ### Slurm Output file, %x is job name, %j is job id
#SBATCH --error=%x_%j.err               ### Slurm Error file, %x is job name, %j is job id

### Load the suitable Quantum Expresso module
module load QuantumESPRESSO/6.3-foss-2018b

### Executes the executable "pw.x" across 2 nodes and 4 processes/CPU cores for the input `pw.scf.silicon.in`
srun --mpi=pmix_v3 -N 2 pw.x -nk 4 -i pw.scf.silicon.in
```

The below output shows that the workflow has been distributed across 2 nodes, with a total of 4 pools. The computations are performed based on these above-mentioned parallel execution configuration. Also, displays the metrics such as parallelization, overall performance, and successful job completion status. Note that the results only display essential information to aid in understanding the execution of this multi-node job. And, the `sacct` report indicates that the job is allocated with 4 CPUs across 2 nodes, and was completed successfully.

```bash
$ cat multinode_27108398.out

Program PWSCF v.6.3MaX starts on  8Mar2024 at 13:18:37

     This program is part of the open-source Quantum ESPRESSO suite
     for quantum simulation of materials; please cite
         "P. Giannozzi et al., J. Phys.:Condens. Matter 21 395502 (2009);
         "P. Giannozzi et al., J. Phys.:Condens. Matter 29 465901 (2017);
          URL http://www.quantum-espresso.org",
     in publications or presentations arising from this work. More details at
     http://www.quantum-espresso.org/quote

     Parallel version (MPI & OpenMP), running on       4 processor cores
     Number of MPI processes:                 4
     Threads/MPI process:                     1

     MPI processes distributed on     2 nodes
     K-points division:     npool     =       4
     Reading input from pw.scf.silicon.in

     Current dimensions of program PWSCF are:
     Max number of different atomic species (ntypx) = 10
     Max number of k-points (npk) =  40000
     Max angular momentum in pseudopotentials (lmaxx) =  3
     .....
     .....
          Parallel routines

     PWSCF        :     1.17s CPU         1.36s WALL

   This run was terminated on:  13:18:38   8Mar2024
=------------------------------------------------------------------------------=
   JOB DONE.
=------------------------------------------------------------------------------=
```

```bash
$ sacct -j 27108398 --format=JobID,JobName,Partition,Account,AllocCPUS,AllocNodes,State,ExitCode

       JobID    JobName  Partition    Account  AllocCPUS AllocNodes      State ExitCode
------------ ---------- ---------- ---------- ---------- ---------- ---------- --------
27108398      multinode amd-hdr100      USER          4          2  COMPLETED      0:0
27108398.ba+      batch                 USER          3          1  COMPLETED      0:0
27108398.ex+     extern                 USER          4          2  COMPLETED      0:0
27108398.0         pw.x                 USER          4          2  COMPLETED      0:0
```
