# History DB API

This section explains the Python API to use the history database features.

## Storing/Loading Function Evaluation Data

To run GPTune, the user first needs to write a Python GPTune driver code for their tuning problem, as explained in the [GPTune Users Guide](https://gptune.lbl.gov/documentation/gptune-user-guide/).
To use history database, the user needs to add several lines of code to define the history database options and send the machine/software configuration information.
The below listing shows an example Python code that runs GPTune's multi-task learning autotuning (MLA) with the history database.
First, the user needs to import the history database module and create an instance (lines 8 and 30).
The user also needs to set tuning problem name (line 31); the GPTune history database will then create a JSON file using that name.
It is also possible to optionally define the path of the database file by setting *history_db_path* (line 32).

As shown in lines 34--51, users can optionally send the information about their machine and software configuration.
For the machine configuration, users can send the machine name and the number of nodes/cores used (lines 34--41).
As shown in lines 43--51, the software versions are passed as dictionaries which can contain a string of the software version and an array of the version split numbers (e.g. major, minor, and revision numbers).
Users can also use a different identifier such as Git commit information (line 49).
This can be helpful if the software does not have a specific version number.

The history database can also load previous performance data when starting autotuning.
As shown in lines 53--71, users can define conditions (load_condition) using dictionaries to selectively load previous performance data.
For each load condition, the user can use an array to allow multiple configurations for loading.
For example, line 59 allows to load performance data obtained when using 15, 16, and 17 core counts.
Finally, users need to register the history database instance when creating the GPTune instance (line 74) and run MLA_HistoryDB (line 80) to start autotuning.
GPTune will then run with the history database mode by loading/storing the performance data from/into the user's local storage.

Example Python application-GPTune driver code:

```Python
 1:    from autotune.search import *
 2:    from autotune.space import *
 3:    from autotune.problem import *
 4:    from gptune import GPTune
 5:    from data import Data
 6:    from options import Options
 7:    from computer import Computer
 8:    from historydb import HistoryDB
 9:    
10:    task_space = Space([Categorical(['a','b','c'], name="pb")])
11:    input_space = Space([Integer(0, 10, name="x")])
12:    output_space = Space([Real(0.0, inf, name="time")])
13:    
14:    def objective(point):
15:        from math import exp
16:        return exp(point['x'])
17:    
18:    cst1 = "x >= .5"
19:    def cst2(point):
20:        return (point['x'] < 1.5)
21:        
22:    constraints = {'cst1': cst1, 'cst2': cst2}
23:    
24:    problem = TuningProblem(task_space, input_space, output_space, objective, constraints, None) # no analytical model
25:    
26:    computer = Computer(nodes=1, cores=16)
27:    option = Options()
28:    
29:    # setting to use the history database
30:    history_db = HistoryDB()
31:    history_db.application_name = 'PDGEQRF' # database file name
32:    history_db.history_db_path = './' # default location is $PWD
33:    
34:    # optional information to store into the history database
35:    history_db.machine_configuration = {
36:        "machine":"cori",
37:        "haswell": {
38:            "nodes":1,
39:            "cores":16
40:        }
41:    }
42:
43:    history_db.software_configuration = {
44:        "openmpi":{
45:            "version":"4.0.0",
46:            "version_split":[4,0,0],
47:        },
48:        "scalapack":{
49:            "git":"bc6cad585362aa58e05186bb85d4b619080c45a9"
50:        },
51:    }
52:            
53:    # conditions for loading previous performance data
54:    history_db.load_condition = {
55:        "machine_configuration": {
56:            "machine":['cori'], # load only if the data's machine name is cori
57:            "haswell": {
58:                "nodes":[1], # load only if the data's node count is 1
59:                "cores":[15,16,17] # load if the data's core count is 15, 16, or 17.
60:            }
61:        },
62:        "software_configuration": {
63:            "openmpi":{
64:                "version_from":[4,0,0],
65:                "version_to":[5,0,0]
66:            },
67:            "scalapack":{
68:                "git":"bc6cad585362aa58e05186bb85d4b619080c45a9"
69:            }
70:        }
71:    }
72:    
73:    # add the history db module into the GPTune module
74:    gt = GPTune(problem, computer=computer, data=data, options=options, history_db=history_db)
75:    
76:    ntask = 2
77:    nruns = 20
78:    giventask = [[1],[2]]
79:    
80:    (data, model, stats) = gt.MLA_HistoryDB(NS=nruns, Igiven=giventask, NI=ntask, NS1=max(nruns/2, 1))
```

## Storing/Loading Surrogate Models

In addition to the ability to re-use function evaluation results, the history database also supports storing and loading trained GP surrogate models.
Re-using pre-trained surrogate models can be useful because the modeling phase of GPTune can require a significant amount of computational resources and time.

**Storing surrogate model data.**
Storing model data is done automatically by GPTune if the user runs GPTune with the history database mode as described in [Storing/Loading Function Evaluation Data](./userguide_api.md).
The history database stores every modeling data during the autotuning process.

**Run MLA with Pre-Trained Surrogate Models.**
GPTune provides a method called *MLA_LoadModel* which runs MLA with a pre-trained surrogate model.
Users can invoke *MLA_LoadModel* as follows.

```Python
# create a history db module instance
history_db = HistoryDB()
history_db.tuning_name = 'example'
history_db.machine_configuration = {}
history_db.software_configuration = {}
history_db.load_condition = {}

# add the history db module into the GPTune module
gt = GPTune(problem, computer, data, options, history_db)

# new method named "Load Model" in the GPTune module
gt.MLA_LoadModel(NS=nruns, Igiven=giventask)
```

Unlike the *MLA* method, *MLA_LoadModel* does not require the information about the number of initial samples (*NS1*).
*MLA_LoadModel* only requires the number of additional samples to be tuned after loading the model (*NS*) and the task parameter information (*Igiven*).
Note that trained surrogate models may or may not be meaningful for different problem spaces.
Currently, the history database loads trained models only if they match the problem space of the given optimization problem.

**Model selection method.**
In *MLA_LoadModel*, it is important to select the best model data from all available model data.
By default, *MLA_LoadModel* selects the model data that contains most function evaluation results, assuming that we can build a better model with more function evaluation data.
*MLA_LoadModel* also provides several more model selection methods, and users can choose one model selection method when calling *MLA_LoadModel*, as follow.

```Python
gt.MLA_LoadModel(NS=nruns, Igiven=giventask, method="max_evals"))
```

**Model selection parameters.**
*method="max_evals"*: choose the model that has most function evaluation data.
*method="MLE"* or *"mle"*: choose the model that has the highest likelihood.
*method="AIC"* or *"aic"*: choose the model based on Akaike Information Criterion (AIC).
*method="BIC"* or *"bic"*: choose the model based on Bayesian Information Criterion (BIC).

The (current) modeling method does not leverage any previous modeling information (e.g. pre-trained models) to (efficiently) update the surrogate model.
One possible and useful user scenario is to find out if there is a good enough surrogate model and use the model for optimization (with no additional updates or fewer updates).

By default, *MLA_LoadModel* does update the model after collecting an additional function evaluation result.
Some users may want to adjust the number of additional samples to further update the surrogate model.
*MLA_LoadModel* provides an additional argument *update:int* which represents the number of additional samples needed for the model to be updated further.

```Python
gt.MLA_LoadModel(NS=nruns, Igiven=giventask, update=5)
```
For example, with the above command, after loading the initial surrogate model, GPTune updates the model every time it gets five more samples.
If users want to update the model with other patterns, they can use both MLA\_LoadModel and MLA\_HistoryDB to reuse and update the model according to the user-defined pattern.

Although the provided model selection methods are easy to use, some users probably want to use more sophisticated methods based on the model statistics information.
Therefore, the GPTune history database provides a method called **ReadModelData** that reads all model data (that match the given tuning problem) as an array of dictionaries.

```Python
# Read all model data as an array of dictionaries
model_data = history_db.ReadModelData(problem=problem, Igiven=Igiven)
```

As shown in Listing in [JSON Format](./overview.md), the loaded model data contains the model statistics (e.g. (neg) log likelihood, last gradients, the number of iterations of the modeling algorithm, and the number of function evaluation data).
Users can define their own method to select the best model data (or discard the model data) based on the loaded model data.
Then, users can invoke *MLA_LoadModel* by specifying the selected surrogate model using its unique ID (UID), as follows.

```Python
model_index = UserDefinedCriterion(model_data)
model_uid = model_data[model_index]["uid"]
gt.MLA_LoadModel(NS=nruns, Igiven=giventask, model_uid=model_uid))
```

