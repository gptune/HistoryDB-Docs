# Design

![History database design](../static/history_db.png)

The figure illustrates the design of the history database.
Users can use the history database either by using [Python API](./historydb_localdb.md) in their GPTune driver code or by using a command line interface (CLI) provided by the workflow automation tool called [CK-GPTune](./ckgptune.md).
The GPTune history database can store and load performance data to and from performance data files with JavaScript Object Notation ([JSON](https://json.org)) [[4](references.md)] format in the user's local storage.
In the user's local storage, each tuning problem has a separate data file that contains all performance data (obtained by the user and/or downloaded from the shared public database) of the tuning problem.

Each data file contains the function evaluation results obtained from the GPTune's Bayesian optimization model.
The multitask learning autotuning (MLA) of GPTune relies on three information spaces: **input space (IS)**, **parameter space (PS)**, and **output space (OS)**.
IS contains all the input problems (i.e. tasks) that the application may encounter (e.g. the sizes of matrices, pointers to input files) and PS contains all the tuning parameter configurations to be optimized (e.g. size of row/column blocks).
OS is the output space for each of the scalar objective functions (e.g. measured runtime).
The history database stores these performance data into the JSON file right after evaluating each parameter configuration.
This practice ensures that no data is lost, in the case where  a long run with many parameter configurations does not complete.
If GPTune is run in parallel and multiple processes need to update performance data simultaneously, the history database allows one process to update the data file at a time based on simple file access control.

In addition to the information of IS, PS, and OS, the history database also records the meta-description like machine configuration and software information (e.g. which software libraries are used for that application) into the JSON file.
We offer two ways for users to provide machine/software configuration information.

* [HistoryDB API](./historydb_localdb.md): Users can provide machine/software configuration information manually using our Python API. In case the software is installed using [Spack](https://spack.io/) [[3](references.md)], the user can inform the database about which software is installed with Spack, so that our database can automatically parse the software configuration and record it in the database.
* [CK-GPTune](./ckgptune.md): Users can define the application's software dependencies with a meta-description file, then CK-GPTune detects the software information using the [Collective Knowledge (CK)](https://cknowledge.io) [[2](references.md)] technology. Machine configuration still needs to be provided manually by the user.

Based on the provided information, users will be able to determine which data are relevant for learning from a possibly different machine or software versions or configurations.
Moreover, workflow automation will allow users to reproduce performance data from the same and different users.

We provide a public shared repository at [https://gptune.lbl.gov](https://gptune.lbl.gov) through [NERSC](https://www.nersc.gov)'s [Science Gateways](https://docs.nersc.gov/services/science-gateways), where users can upload their performance data obtained from GPTune and download performance data provided by other users.
In the shared repository, all submitted performance data is stored in a storage provided by [NERSC](https://www.nersc.gov) and internally managed by using [MongoDB](https://mongodb.com) [[1](references.md)].
The shared database requires login credentials for users to submit their performance data.
Every submitted performance data can have multiple accessibility options: publicly available data, private data, or data that can be shared with specific users.
In other words, the shared repository allows anyone to browse and download publicly available data.
More detailed user manuals of these interfaces are given at [History DB Repository](./historydb_repository.md) tab.

In our recent paper "Enhancing Autotuning Capability with a History Database" [[7](references.md)] about the history database presents some more detailed information and benefits of using the history database.


# JSON Format

In this section, we explain the JSON format to store performance data from GPTune.
Each tuning problem has a separate data file (e.g. *tuning_problem_name.json*) that contains all performance data (obtained by the user and/or downloaded from the shared public database) of the tuning problem.
Each JSON file has two labels *func_eval* and *model_data*.
As the name indicates, *func_eval* contains the list of all function evaluation results, and *surrogate_model* contains the list of each trained surrogate model's meta-data.

```Json
{
  "tuning_problem_name": "my_tuning_problem_name",
  "func_eval": [
    {
      /* function evaluation result */
    },
    {
      /* function evaluation result */
    },

    ...

    {
      /* function evaluation result */
    }
  ],
  "surrogate_model": [
    {
      /* surroagte model meta-data */
    },
    {
      /* surroagte model meta-data */
    },

    ...

    {
      /* surroagte model meta-data */
    }
  ]
}
```

<br>

## Function Evaluation Result

The following listing shows a function evaluation result of the QR factorization routine of [ScaLAPACK](http://www.netlib.org/scalapack/) for a given task/parameter configuration.
In the listing, **task_parameter** contains the information about the task parameter, and **tuning_parameter** contains the tuning parameter configuration, and its evaluation result is stored in **output**.
These information is collected automatically in GPTune if the user invoke the history database.

Each function evaluation data can also contain the information about the machine and software configuration to run the application (or the tuning problem).
The information related to the machine configuration includes the machine name (e.g. Cori) and the number of cores/nodes used.
The software information contains the versions of software packages that are used for compiling/installing the application.
The machine and software configurations are stored in **machine_configuration** and **software_configuration**, respectively.
Unlike task and tuning parameters, the machine and software information is not available in GPTune and needs to be given by the user.
Users can use [CK-GPTune](./ckgptune.md) to automatically detect the software dependencies or provide the information [manually](./userguide_api.md) in the application-GPTune driver code.

Also, when saving a function evaluation result, the data-creation time and a unique ID of the function evaluation are automatically generated and appended by GPTune.
This information can be useful if the user uploads the data into our shared repository.
If different users submit function evaluation results for the same task and parameter configurations, the database can differentiate between different function evaluation results based on their UIDs.

Example function evaluation result:

```Json
{
    "tuning_problem_name": "PDGEQRF",
    "task_parameter": {
        "m": 10000,
        "n": 10000
    },
    "tuning_parameter": {
        "mb": 14,
        "nb": 8,
        "npernode": 3,
        "p": 5
    },
    "constants": {
        "bunit": 8,
        "cores": 32,
        "nodes": 64
    },
    "evaluation_result": {
        "r": 0.88476
    },
    "machine_configuration": {
        "haswell": {
            "cores": 32,
            "nodes": 64
        },
        "machine_name": "Cori"
     },
    "software_configuration": {
        "gcc": {
             "version_split": [8, 3, 0]
         },
         "openmpi": {
             "version_split": [4, 0, 1]
         },
         "scalapack": {
             "version_split": [2, 1, 0]
         }
    },
    "source": "measure",
    "time": {
        "tm_hour": 0,
        "tm_isdst": 1,
        "tm_mday": 22,
        "tm_min": 15,
        "tm_mon": 6,
        "tm_sec": 1,
        "tm_wday": 1,
        "tm_yday": 173,
        "tm_year": 2021
    },
    "uid": "8f33fefe-d329-11eb-b2e9-0137b026b11b"
}
```

## Surrogate Model

This section explains what information is stored by GPTune for a GP surrogate model.
The below listing shows the information of a surrogate model for the IJ routine of [Hypre](https://computing.llnl.gov/projects/hypre-scalable-linear-solvers-multigrid-methods) for five different function evaluation results for task \{i: 200, j: 200, k: 200\}.

Label **hyperparameters** contains the hyperparameters values which are required to reproduce the surroagate model.
Here, we assume the GPTune's default modeling scheme, based on [Linear Coregionalization Model (LCM)](https://dl.acm.org/doi/abs/10.1145/3437801.3441621) [[6](references.md)], is used.
Recall that, for a set of correlated objective functions for all the given tasks \\( \{ y_{i}(x) \}, {i\in 1..\delta}\\), LCM builds a joint model of the target functions \\( \{ f_{i}(x) \}, {i \in 1..\delta} \\), through the underlying assumption of linear dependence on latent functions \\( \{ u_{q} \} \\), each of which is an independent GP:
<script type="text/javascript" async
  src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.7/MathJax.js?config=TeX-MML-AM_CHTML">
</script>
$$ f_{i}(x) = \sum_{q=1}^{Q}{a_{i,q}u_{q}(x)}$$
where \\( a_{i,q} \\) are hyperparameters, and \\( \{ u_{q} \}\\) are latent functions each of which is an independent GP:

$$ k_{q}(x,x') = \sigma_{q}^{2} exp(-\sum_{j=1}^{\beta}\frac{(x_{j}-{x_{j}}')^{2}}{I_{j}^{q}}) $$

where \\( \sigma_{q} \\) and \\( I_{j}^{q} \\) are also hyperparameters to be learned.
In addition to the aforementioned hyperparameters \\( a_{i,q} \\), \\( \sigma_{q} \\), \\( I_{j}^{q} \\), we use additional diagonal regularization parameters \\( b_{i,q} \\) and \\( d_{i} \\) for the covariance matrix during the LCM (for the details about the covariance matrix and the parameter search algorithm, please find [the GPTune paper](https://dl.acm.org/doi/abs/10.1145/3437801.3441621) in PPoPP 2021).
Hence, in the below example which considers one task (\\( Q=1 \\)) for 12 tuning parameters (\\( \beta=12 \\)), we store 16 hyperparameters in total (12 for \\( I_{j}^{q} \\) and 1 for each of \\( a_{i,q} \\), \\( \sigma_{q} \\), \\( b_{i,q} \\), \\( d_{i} \\)).
As another example, considering two tasks (\\( Q=2 \\)) for 12 tuning parameters (\\( \beta=12 \\)),, we need to store 36 hyperparameters in total (24 for \\( I_{j}^{q} \\), 4 for \\( a_{i,q} \\), 2 for \\( \sigma_{q} \\), 4 for \\( b_{i,q} \\), 2 for \\( d_{i} \\)).

**model_stats** stores the model's statistics information.
For the GPTune's LCM, we can store some statistics information such as *log_likelihood*, *neg_log_likelihood*, *gradients*, and *iteration* (how many iterations were required to converge the model).
Note that, trained surrogate models may or may not be meaningful for different problem spaces.
Therefore, the JSON data also contains task parameter information (**task_parameters**) and which function evaluation results were used (**func_eval**) to build the surrogate model, by containing the list of the UIDs of the function evaluation results.
The history database can load trained models only if they match the problem space of the given optimization problem.
Similar to function evaluation results, the data generation time and a unique ID of each surrogate model are also automatically appended by GPTune.

Example surroagte model data:

```Json
{
    "tuning_problem_name": "PDGEQRF",
    "hyperparameters": [100000000.0,
                        0.19226454690325398,
                        0.1539474196726214,
                        0.12115729284533584,
                        1.0,
                        1.2772596408973604e-07,
                        1e-10,
                        3.9912970870393],
    "model_stats": {"gradients": [5.261526635521777e-16,
                                  1.0041584825870586e-06,
                                  -4.1709510778531467e-07,
                                  -2.099933034571677e-06,
                                  0.0,
                                  -5.6577754176681755e-15,
                                  -1.0061870303119767e-08,
                                  -1.4113176689534157e-06],
                    "iterations": 100,
                    "log_likelihood": -11.382183474421357,
                    "neg_log_likelihood": 11.382183474421357},
    "function_evaluations": ["38313b9e-d329-11eb-b2e9-0137b026b11b",
                             "38314b52-d329-11eb-b2e9-0137b026b11b",
                             "383155c0-d329-11eb-b2e9-0137b026b11b",
                             "38315fe8-d329-11eb-b2e9-0137b026b11b",
                             "38316a10-d329-11eb-b2e9-0137b026b11b",
                             "383174ba-d329-11eb-b2e9-0137b026b11b",
                             "38317f6e-d329-11eb-b2e9-0137b026b11b",
                             "383189c8-d329-11eb-b2e9-0137b026b11b",
                             "383193d2-d329-11eb-b2e9-0137b026b11b",
                             "38319e36-d329-11eb-b2e9-0137b026b11b"],
    "input_space": [{"lower_bound": 128,
                     "name": "m",
                     "transformer": "normalize",
                     "type": "int",
                     "upper_bound": 10000},
                    {"lower_bound": 128,
                     "name": "n",
                     "transformer": "normalize",
                     "type": "int",
                     "upper_bound": 10000}],
    "modeler": "Model_LCM",
    "num_func_eval": 10,
    "num_func_eval_per_task": 10,
    "num_task_parameters": 1,
    "objective": {"lower_bound": -inf,
                  "name": "r",
                  "objective_id": 0.0,
                  "transformer": "identity",
                  "type": "real",
                  "upper_bound": inf},
    "objective_id": 0,
    "output_space": [{"lower_bound": -inf,
                      "name": "r",
                      "transformer": "identity",
                      "type": "real",
                      "upper_bound": inf}],
    "parameter_space": [{"lower_bound": 1,
                         "name": "mb",
                         "transformer": "normalize",
                         "type": "int",
                         "upper_bound": 16},
                        {"lower_bound": 1,
                         "name": "nb",
                         "transformer": "normalize",
                         "type": "int",
                         "upper_bound": 16},
                        {"lower_bound": 0,
                         "name": "npernode",
                         "transformer": "normalize",
                         "type": "int",
                         "upper_bound": 5},
                        {"lower_bound": 1,
                         "name": "p",
                         "transformer": "normalize",
                         "type": "int",
                         "upper_bound": 2048}],
    "task_parameters": [[10000, 10000]],
    "time": {"tm_hour": 0,
             "tm_isdst": 1,
             "tm_mday": 22,
             "tm_min": 12,
             "tm_mon": 6,
             "tm_sec": 38,
             "tm_wday": 1,
             "tm_yday": 173,
             "tm_year": 2021},
    "uid": "39c09946-d329-11eb-b2e9-0137b026b11b"
}
```
