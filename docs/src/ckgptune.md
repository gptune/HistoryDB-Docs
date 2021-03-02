# CK-GPTune

## What is CK-GPTune?

In this section, we introduce [CK-GPTune](https://github.com/yhcho614/ck-gptune) which is a workflow automation framework for GPTune.
CK-GPTune helps users install GPTune and provides an interface to run GPTune using simple commands.
CK-GPTune also provides some example tuning problems that users can install/autotune using the workflow automation technology.
CK-GPTune is built based on the [Collective Knowledge (CK)](https://cknowledge.org) technology that provides a lot of useful functions to automate (experiment) workflows and to define/detect software dependencies to compile and run applications automatically.

## Background of CK

CK is a tool which helps organize research workflows.
CK can manage many different things, such as research data, programs or scripts analyzing this data, as well as the research data. In our CK-GPTune, it is used for recording and  organizing the history data of simulations. 

In CK, there are three important concepts: *entries*, *repositories*, *modules*. Here, we provide a brief introduction of these concepts.

* CK Entries: Each entry represents a research component (e.g. data, program, script) that you want to manage. Each entry has *meta.json* that contains meta information (e.g. run command, compiler/runtime dependencies) about the entry.

* CK Repositories: Each CK repository is a collection of entries which are meant to be shared with other people. For example, to install a CK repository *ck-autotuning*, we can use the following commands:
    
    ```
    $ ck pull repo:ck-autotuning
    $ ck pull repo --url=https://github.com/ctuning/ck-autotuning
    ```

* CK Modules: Modules group entries with "actions" to operate on these entries. Each module has *module.py* in that we can define its customized actions.
    
    ```
    $ ck [action] [module:entry]
    ```

CK provides many useful modules such as "program", "package", and "experiment". For example, the program module [CK Problem Module](https://cknowledge.io/c/module/program/) offers actions to compile and run programs.
    
```
$ ck compile program:gemm
$ ck run program:gemm
```

For more information, [the CTuning community](https://ctuning.org) provides a lot of documentation about CK including a detailed user manual [CK:Manual](https://ck.readthedocs.io/_/downloads/en/latest/pdf/).
Also, there is a blog at [https://github.com/michel-steuwer/About-CK](https://github.com/michel-steuwer/About-CK) that provides a good overview of CK.

## Installation

***Installation of CK-GPTune.***
To use CK-GPTune, users first need to install the CK framework using the following command.
```Bash
$ pip install ck --user
```
This will create a directory named *CK* in the home directory and install the CK framework.

We provide a repository of CK-GPTune on Github ([https://github.com/yhcho614/ck-gptune](https://github.com/yhcho614/ck-gptune)).
Users can install CK-GPTune using the following command.

```Bash
$ ck pull repo --url=https://github.com/yhcho614/ck-gptune
```

CK-GPTune will be installed in *$HOME/CK/ck-gptune*.
CK-GPTune relies on several other CK repositories (e.g. ck-autotuning, ctuning-programs).
These repositories will be automatically installed when installing CK-GPTune.

***Installation and Detection of GPTune.***
With CK-GPTune, users have the option to install GPTune ([https://github.com/gptune/GPTune/tree/history_db](https://github.com/gptune/GPTune/tree/history_db)) automatically using the following command.

```Bash
$ ck install ck-gptune:package:gptune
```

GPTune relies on several software packages such as OpenMPI, BLAS, LAPACK, [ScaLAPACK](http://www.netlib.org/scalapack/), [MPI4PY](https://mpi4py.readthedocs.io/en/stable/), [Scikit-optimize](https://scikit-optimize.github.io/stable) and [Autotune](https://pypi.org/project/autotune/).
The command automatically detects if these software packages are available on your system.
If there are missing software packages, CK-GPTune will print out a message to let you know which software packages need to be installed. If there are multiple software versions in your computer, CK-GPTune will ask you to choose one.
After resolving all the dependencies, CK-GPTune installs the GPTune library in *$HOME/CK-TOOLS*
(e.g. *$HOME/CK-TOOLS/lib-gptune-1.0.0-gcc-9.3.0-compiler.python-3.8.5-linux-64*).

To use CK-GPTune, users need to detect the installed GPTune software with the following command.
The command detects the GPTune installation path and prepares an executable environment by setting all the environment variables needed by GPTune.

```Bash
$ ck detect soft:lib.gptune
```

Some users may want to use already installed GPTune or install GPTune manually by following the GPTune [UsersGuide](https://gptune.lbl.gov/documentation/gptune-user-guide/).
In this case, users can simply use the above command to detect the installed GPTune and continue using CK-GPTune.


## Automation Examples in CK-GPTune

CK-GPTune currently provides four example programs including [gptune-demo](https://gptune.lbl.gov/documentation/gptune-user-guide/), [PDGEQRF (ScaLAPACK)](http://www.netlib.org/scalapack/), [PDDSpawn (SupeLU_DIST)](https://portal.nersc.gov/project/sparse/superlu/) and  [IJ (Hypre)](https://computing.llnl.gov/projects/hypre-scalable-linear-solvers-multigrid-methods).
Each program has its own working directory (e.g. *$HOME/CK/ck-gptune/program/scalapack-pdqrdriver/*).

These example programs are managed as components (i.e. CK entries) of [the CK's program module](https://cknowledge.io/c/module/program/) that provides a unified way for program compilation and execution workflow.
With the CK program module, for example, you can compile and run these benchmarks with a simple command line interface (CLI).
To compile the scalapack-pdqrdriver example, you can use the following command.

```Bash
$ ck compile ck-gptune:program:scalapack-pdqrdriver
```

This will compile and install the software for the scalapack-pdqrdriver example in *$HOME/CK/ck-gptune/program/scalapack-pdqrdriver/tmp*.
While installing the software, CK-GPTune will detect the installation paths and the versions of the required software packages (e.g. OpenMPI, scalapack).
These software dependencies are defined in *meta.json* in the *.cm* directory of each benchmark program, and the detected versions of software dependencies are stored in the *tmp-deps.json* file in the same directory.

Then, you can also run the example using the following command.

```Bash
$ ck run ck-gptune:program:scalapack-pdqrdriver
```

This command will run the example as defined by the *run\_cmd* in the *meta.json* file.
Each of our four example programs has a Python script to run autotuning with GPTune.
The run command will execute the Python script and stores the output into the *stdout.out* file in the *tmp* directory.
Currently the run command does not pass additional arguments to the script, and the Python script runs with a default setting.

CK-GPTune provides a CK module called *gptune* to run these examples with more functionalities (e.g. run with history database, passing additional arguments).
In CK modules, we can define and implement our own actions using Python.
The *gptune* module currently offers two actions *autotune* and *crowdtune*.
The *autotune* action will run GPTune for the example without using the history database, which is equivalent to command *$ ck run ck-gptune:program:[example]*.

```Bash
$ ck autotune gptune --bench=scalapack-pdqrdriver
```

With the *gptune* module, we can also pass additional arguments as shown in the below command; the module internally uses environment variables to pass the arguments values.
If the argument values are not given by this command line, the example will run with the default setting.
```Bash
$ ck autotune gptune --bench=scalapack-pdqrdriver --ntask=10 --nruns=10
```

The *crowdtune* action, on the other hand, automatically invokes the history database while running autotuning.

```Bash
$ ck crowdtune gptune --bench=scalapack-pdqrdriver --ntask=10 --nruns=10 --machine=cori
```

With this command, the scalapack pdqrdriver example loads existing performance data from the performance data file.
The output and the performance data file are stored in the working directory (*tmp* directory) of the example.
CK-GPTune supports storing the information about machine specifications and the software dependencies.
The machine related information (e.g. machine name, number of cores/nodes) needs to be passed as arguments.
```Bash
$ ck crowdtune gptune --bench=scalapack-pdqrdriver --ntask=10 --nruns=10 --machine=cori -nnodes=1 -ncores=16
```
The software dependency information, on the other hand, is automatically parsed from the example program directory (there is tmp-deps.json file that stores the information about software dependencies).

Users can also use CK-GPTune to invoke \texttt{MLA\_LoadModel} automatically.
Users can use the following command to run MLA with pre-trained surrogate models.

```Bash
$ ck MLA_LoadModel gptune --bench=scalapack-pdqrdriver --method="max_evals" --nruns=50 --machine=cori --nodes=1 --cores=16
```

## Workflow Automation with CK

In CK-GPTune, programs are managed as components of the [CK's program](https://cknowledge.io/c/module/program/) module that provides a unified way for program compilation and workflows (and automatic detection of software dependencies) using a CLI and meta description files.
For these programs, CK-GPTune can automatically run GPTune with the history database.
Users therefore need to add their program and the Python interface code that calls GPTune into the CK-GPTune's program list to take advantage of CK-GPTune.

Users can create an entry of a new program using the following command.

```Bash
$ ck add ck-gptune:program:my_test_program
```

The command then prints the available templates as shown in the below, and users can select one of these template and extend it for their applications.

```Bash
0) C program "Hello world" (--template=template-hello-world-c)
1) C program "Hello world" with compile and run scripts (--template=template-hello-world-c-compile-run-via-scripts)
2) C program "Hello world" with jpeg dataset (--template=template-hello-world-c-jpeg-dataset)
3) C program "Hello world" with output validation (--template=template-hello-world-c-output-validation)
4) C program "Hello world" with xOpenME interface and pre/post processing (--template=template-hello-world-c-openme)
5) C++ program "Hello world" (--template=template-hello-world-cxx)
6) Fortran program "Hello world" (--template=template-hello-world-fortran)
7) Java program "Hello world" (--template=template-hello-world-java)
8) Python program "Hello world" (--template=template-hello-world-python)
9) bench-julia-sin (--template=bench-julia-sin)
10) cbench-automotive-susan (--template=cbench-automotive-susan)
11) milepost-codelet-mibench-automotive-susan-s-src-susan-codelet-1-1 (--template=milepost-codelet-mibench-automotive-susan-s-src-susan-codelet-1-1)
12) polybench-cpu-2mm (--template=polybench-cpu-2mm)
13) polybench-cuda-gemm (--template=polybench-cuda-gemm)
14) Empty entry

Select template for the new entry (or press Enter for 0):
```

The command will create a directory for the program at $HOME/CK/ck-gptune/program/my_test_program.
Instead of using these templates, users can also copy content from an existing program and modify/extend the content for their program.

```Bash
$ ck copy ck-gptune:program:scalapack-pdqrdriver ck-gptune:program:my_test_program
```

After creating the entry, users may add source codes, run/compile scripts, and datasets into the program's directory.
Then, users need to define their actions (e.g. how to compile and run, what are the required software packages, etc.) in the meta description file ($HOME/CK/ck-gptune/program/my_test_program/.cm/meta.json).

The below listing shows an example meta description file.
In lines 4--22, the meta description file describes how to compile the program and its compile-level dependencies.
The software packages needed by the program are automatically detected based on their tags during program compilation.
CK internally uses the [CK soft](https://cknowledge.io/c/module/soft/) module which is able to detect many software packages.
However, if CK does not support detecting a certain software package, the user has to add a new detection plugin by following the [CK manuals](https://ck.readthedocs.io/_/downloads/en/latest/pdf/).
CK-GPTune automatically stores the detected software versions into the history database.
As shown in lines 23--30, the user can also define runtime-level dependencies.
Lines 53--66, on the other hand, define how to run their program for autotuning.
The user probably just needs to set *run_cmd_main* as a runnable command; the command should run the Python interface code which calls GPTune for for the user's optimization probelm (there is no need to manually invoke the history database).
Also, as shown in lines 31--52, the user can define some rules using JSON format to selectively load previous performance data.

After completing the meta description file, the user can run the program with the history database using CK-GPTune commands, as shown in the examples above.
First, the user needs to compile and install the program using the following command.
```Bash
$ ck compile program:my_test_program
```
The command detects software dependencies defined in the meta description file and stores the detected software information in the program's working directory.
CK-GPTune will send the detected software versions to the history database and stores the information into the performance data file.
After installing the program, the following command runs GPTune with history database.
```Bash
$ ck crowdtune gptune --bench=my_test_program
```
The line executes the command described in the meta description file and automatically invokes the history database.
GPTune loads existing performance data from the performance data file according to the load rules in the meta description file.
The machine related information (machine name, number of cores/nodes) can be passed as arguments, as follows.
```Bash
$ ck crowdtune gptune --bench=my_test_program --machine=cori --nodes=1 --cores=16
```
In addition to the purpose of passing machine information to CK-GPTune, the user may require additional arguments to pass to user programs or scripts.
Arguments of the above command line are treated as environment variables.
Hence, to receive arguments through the command line, the user needs to modify the user program to get the argument values through environment variables.

Example meta description file

```Json
{
  "backup_data_uid": "8bf9aa0ad04427bb",
  "build_compiler_vars": {},
  "use_compile_script": "yes"
  "compile_cmds": {
    "default": {
      "cmd": "bash ../install_scalapack$#script_ext#$"
    }
  },
  "compile_deps": {
    "openmpi": {
      "local": "yes",
      "name": "OpenMPI library",
      "tags": "lib,mpi,openmpi",
      "version_from": [4,0,1]
    },
    "scalapack": {
      "local": "yes",
      "name": "SCALAPACK",
      "tags": "lib,scalapack"
    }
  },
  "run_deps": {
    "lib-gptune": {
      "local": "yes",
      "name": "GPTune library",
      "tags": "lib,gptune",
      "version_from": [1,0,0]
    }
  },
  "load_deps": {
    "machine_deps": {
        "machine":['cori'],
        "nodes":[1],
        "cores":[15,16,17]
    },
    "compile_deps": {
        "mpi":[
            {
                "name":"openmpi",
                "version_from":[4,0,0],
                "version_to":[5,0,0]
            }
        ],
        "scalapack":[
            {
                "name":"scalapack",
                "version_to":[2,1,0]
            }
        ]
    }
  },
  "run_cmds": {
    "default": {
      "ignore_return_code": "no",
      "run_time": {
        "run_cmd_main": "$MPIRUN -n 1 $<<CK_ENV_COMPILER_PYTHON_FILE>>$ ..$#dir_sep#$run_autotuner.py",
        "run_cmd_out1": "stdout.log",
        "run_cmd_out2": "stderr.log",
        "run_output_files": [
          "stdout.log",
          "stderr.log"
        ]
      }
    }
  },
  "process_in_tmp": "yes"
}
```

