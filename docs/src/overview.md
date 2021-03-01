# Design

![History database design](../static/history_db.png)

The figure illustrates the design of the history database.
GPTune users can invoke the history database either by manual Python coding in the application-GPTune driver code or by using a command line interface provided by the workflow automation tool called CK-GPTune.
GPTune allows users to store performance data obtained from autotuning into data files.
Users can store performance data into their local storage.
In the local storage, each tuning problem has a data file that contains all the performance data of previous runs of the application.
The obtained performance data are saved as dictionaries in JavaScript Object Notation (JSON) format.
The user can also upload their performance data into the public shared repository.
In the repository, all performance data is managed with MongoDB.

Each data file contains the function evaluation results obtained from the GPTune's Bayesian optimization model.
The multitask learning autotuning (MLA) of GPTune relies on three information spaces: *input space* (IS), *parameter space* (PS), and *output space* (OS).
IS contains all the input problems (i.e. tasks) that the application may encounter (e.g. the sizes of matrices) and PS contains all the tuning parameter configurations to be optimized.
GPTune saves the function evaluation result of each parameter configuration into OS.
The history database stores these performance data into the JSON file right after evaluating each parameter configuration.
This practice ensures that no data is lost, in the case where  a long run with many parameter configurations does not complete.
If GPTune is run in parallel and multiple processes need to update performance data simultaneously, the history database allows one process to update the data file at a time based on simple file access control.

In addition to the information of IS, PS, and OS, the history database also records the meta-description like machine configuration and software information (e.g. which software libraries are used for that application) into the JSON file.
Users can provide this information manually when calling GPTune, but they can also leverage a workflow automation tool called CK-GPTune to manage the information automatically.
With CK-GPTune, users can define the application's software dependencies with a meta-description file, then CK-GPTune detects the software packages/libraries that have dependencies, using the Collective Knowledge (CK) technology.
Based on this information, users will be able to determine which data are relevant for learning from a possibly different machine or software versions or configurations.
Moreover, the workflow automation support from CK-GPTune will allow users to reproduce performance data from the same and different users.

# JSON File Format For Describing Performance Data

## User-Side JSON File Format

```Json
{
    "name": "application/tuning-problem name",
    "func_eval": [
        {
            ...
        },
        {
            ...
        },
        ...
    ],
    "model_data": [
        {
            ...
        },
        {
            ...
        },
        ...
    ]
}
```

## Function Evaluation Data

Listing shows a performance data file for the QR factorization routine of ScaLAPACK for two different tasks \{m: 816, n: 599\} and \{m: 669, n: 164\}.
In this example, we evaluated runtime of one parameter configuration for each task (two parameter configurations in total).
In Listing~\ref{listing:pdqrdriver.json}, \texttt{I} contains the information about each task parameter, and all the performance data of that task are stored in \texttt{func\_eval} as an array where each array item contains the tuning parameter configuration \texttt{P} and its evaluation result \texttt{O}.
In GPTune, there is a Python class named \texttt{data} to store sampled tasks, tuning parameters and outputs~\cite{GPTuneUserGuide}.
After finishing each function evaluation, the history database queries the contents of the \texttt{data} class and updates the performance data file.

Each function evaluation data in *func\_eval* can also contain the information about the machine and software configuration to run the application.
The information related to the machine configuration includes the machine name (e.g. Cori) and the number of cores/nodes used.
The software information contains the versions of software packages that are used for compiling/installing the application.
The machine and software configurations are stored in *machine\_deps* and *compile\_deps*, respectively.
Unlike task and tuning parameters, the machine and software information is not available in GPTune and needs to be given by the user.
Users can use CK-GPTune to automatically detect the software dependencies or provide the information manually in the application-GPTune driver code.

Example JSON performance data

```Json
{
  "I": {
    "m": 30000,
    "n": 30000
  },
  "P": {
    "mb": 5,
    "nb": 13,
    "nproc": 136,
    "p": 9
  },
  "machine_deps": {
    "machine": "cori",
    "type": "haswell",
    "nodes": 8,
    "cores": 32
  },
  "software_deps": {
    "openmpi": {
      "version": "4.0.0",
      "version_split": [
        4,
        0,
        0
      ],
      "tags": "lib,mpi,openmpi"
    },
    "scalapack": {
      "version": "2.1.0",
      "version_split": [
        2,
        1,
        0
      ],
      "tags": "lib,scalapack"
    }
  },
  "O": {
    "r": 15.148637
  },
  "time": {
    "tm_year": 2021,
    "tm_mon": 1,
    "tm_mday": 27,
    "tm_hour": 21,
    "tm_min": 22,
    "tm_sec": 22,
    "tm_wday": 2,
    "tm_yday": 27,
    "tm_isdst": 0
  },
  "uid": "cc7c03ec-6128-11eb-a40f-85c4081a47e2"
}
```

<br>



## Surrogate Model


```Json
{
  "hyperparameters": [
    0.5140214197473143,
    1.037247070366763,
    337.6636254330382,
    15.04072992869631,
    5.107732628800839,
    2.9558180040483086,
    8.696644421366367,
    26.085771260561174,
    25.125605700207174,
    4.608683353484095,
    3.511325043265344,
    1.994878529737146,
    29.597204778514428,
    2.3554709171760972,
    9.999999586881094e-06,
    69.65076357886942
  ],
  "model_stats": {
    "log_likelihood": -35.22869969421778,
    "neg_log_likelihood": 35.22869969421778,
    "gradients": [
      -0.5379014374164104,
      0.02347984513751207,
      3.1845619569323703e-06,
      0.0019392480264629336,
      0.022834797398354992,
      -0.03018478674355551,
      0.022739720419626193,
      -0.00010746831896165282,
      0.002468829248110913,
      0.01631023320692664,
      0.030788540386404495,
      -0.024631175060550625,
      -0.27274355043218274,
      -6.198242202047159e-05,
      -3.262995507641375e-05,
      -0.2253219864474227
    ],
    "iteration": 65
  },
  "iteration": 65,
  "func_eval": [
    "09aab368-612d-11eb-8bf3-bbda784b918d",
    "09aae608-612d-11eb-8bf3-bbda784b918d",
    "09ab0c32-612d-11eb-8bf3-bbda784b918d",
    "09ab30fe-612d-11eb-8bf3-bbda784b918d",
    "09ab54ee-612d-11eb-8bf3-bbda784b918d"
  ],
  "task_parameters": [
    [
      200,
      200,
      200
    ]
  ],
  "problem_space": {
    "IS": [
      {
        "type": "int",
        "lower_bound": 20,
        "upper_bound": 1024
      },
      {
        "type": "int",
        "lower_bound": 20,
        "upper_bound": 1024
      },
      {
        "type": "int",
        "lower_bound": 20,
        "upper_bound": 1024
      }
    ],
    "PS": [
      {
        "type": "int",
        "lower_bound": 1,
        "upper_bound": 31
      },
      {
        "type": "int",
        "lower_bound": 1,
        "upper_bound": 31
      },
      {
        "type": "int",
        "lower_bound": 30,
        "upper_bound": 31
      },
      {
        "type": "real",
        "lower_bound": 0,
        "upper_bound": 1
      },
      {
        "type": "real",
        "lower_bound": 0,
        "upper_bound": 1
      },
      {
        "type": "int",
        "lower_bound": 1,
        "upper_bound": 12
      },
      {
        "type": "categorical",
        "categories": [
          "0",
          "1",
          "2",
          "3",
          "4",
          "6",
          "8",
          "10"
        ]
      },
      {
        "type": "categorical",
        "categories": [
          "-1",
          "0",
          "6",
          "8",
          "16",
          "18"
        ]
      },
      {
        "type": "categorical",
        "categories": [
          "5",
          "6",
          "7",
          "8",
          "9"
        ]
      },
      {
        "type": "int",
        "lower_bound": 0,
        "upper_bound": 5
      },
      {
        "type": "categorical",
        "categories": [
          "0",
          "3",
          "4",
          "5",
          "6",
          "8",
          "12"
        ]
      },
      {
        "type": "int",
        "lower_bound": 0,
        "upper_bound": 5
      }
    ],
    "OS": [
      {
        "type": "real",
        "lower_bound": -Infinity,
        "upper_bound": Infinity
      }
    ]
  },
  "modeler": "Model_LCM",
  "objective_id": 0,
  "time": {
    "tm_year": 2021,
    "tm_mon": 1,
    "tm_mday": 27,
    "tm_hour": 21,
    "tm_min": 52,
    "tm_sec": 47,
    "tm_wday": 2,
    "tm_yday": 27,
    "tm_isdst": 0
  },
  "uid": "0c5acce2-612d-11eb-8bf3-bbda784b918d"
}
```
