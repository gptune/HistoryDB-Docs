# GPTune Local DB

This section explains the Python API to use the history database features using users' local database.

## Reusing Function Evaluation Data

To run GPTune, the user first needs to write a Python code (driver code) to invoke GPTune for the tuning problem, as explained in the [GPTune Users Guide](https://gptune.lbl.gov/documentation/gptune-user-guide/) [[5](references.md)].
To use history database, the user needs to add several lines of code in the driver code to define the history database options and send the machine/software configuration information.
As another interface, the user can provide a meta description file in JSON to define the history database option with the path of $driver_directory/.gptune/meta.json ($driver_directory is the directory where the GPTune driver code is located).

The below listing shows an example Python code that runs GPTune's multi-task learning autotuning (MLA) with the history database.
First, the user needs to provide a dictionary to describe the tuning meta information, as shown by *tuning_metadata* in the code (lines 35--76).
The user also needs to set tuning problem name; the GPTune history database will then create a JSON file using that name.
<!--It is also possible to optionally define the path of the database file by setting *history_db_path*.-->

In the dictionary, users can send the information about their machine and software configuration.
For the machine configuration (*machine_configuration*), users can send the machine name and the number of nodes/cores used.
The software versions are also passed as dictionaries (*software_configuration*) which can contain a string of the software version and an array of the version split numbers (e.g. major, minor, and revision numbers).
Users can also use a different identifier such as Git commit information using *version_text* label instead of *version_split* label.
This can be helpful if the software does not have a specific version number.

The history database can also load previous performance data when starting autotuning.
As shown by lines 56--75, users can define load conditions (*lodable_machine_configurations* and *lodable_software_configurations*) using dictionaries to selectively load previous performance data.
For each load condition, the user can use an array to allow multiple configurations for loading.

Then, the user needs to import the history database module and create an instance (line 99).
Users need to register the history database instance when creating the GPTune instance and run MLA to start autotuning.
GPTune will then run with the history database mode by loading/storing the performance data from/into the user's local storage.

The following example is a snippet of GPTune driver code using the history database.
The full code can be found at [here](https://github.com/gptune/GPTune/blob/master/examples/Scalapack-PDGEQRF/scalapack_MLA.py).

```Python
001: from autotune.search import *
002: from autotune.space import *
003: from autotune.problem import *
004: from gptune import *
005:
006: def objectives(point):
007:     nodes = point['nodes']
008:     cores = point['cores']
009:     bunit = point['bunit']
010:     m = point['m']
011:     n = point['n']
012:     mb = point['mb']*bunit
013:     nb = point['nb']*bunit
014:     p = point['p']
015:     npernode = 2**point['npernode']
016:     nproc = nodes*npernode
017:     nthreads = int(cores / npernode)
018:     q = int(nproc / p)
019:     nproc = p*q
020:     params = [('QR', m, n, nodes, cores, mb, nb, nthreads, nproc, p, q, 1., npernode)]
021:
022:     elapsedtime = pdqrdriver(params, niter=2, JOBID=0)
023:
024:     return elapsedtime
025:
026: def cst1(mb,p,m,bunit):
027:     return mb*bunit * p <= m
028: def cst2(nb,npernode,n,p,nodes,bunit):
029:     return nb * bunit * nodes * 2**npernode <= n * p
030: def cst3(npernode,p,nodes):
031:     return nodes * 2**npernode >= p
032:
033: def main():
034:
035:     """ Define tuning metadata for history database """
036:     tuning_metadata = {
037:         "tuning_problem_name": "PDGEQRF",
038:         "machine_configuration": {
039:             "machine_name": "Cori",
040:             "haswell": {
041:                 "nodes": 1,
042:                 "cores": 32
043:             }
044:         },
045:         "software_configuration": {
046:             "openmpi": {
047:                 "version_split": [4,0,1]
048:             },
049:             "scalapack": {
050:                 "version_split": [2,1,0]
051:             },
052:             "gcc": {
053:                 "version_split": [8,3,0]
054:             }
055:         },
056:         "loadable_machine_configurations": {
057:             "Cori" : {
058:                 "haswell": {
059:                     "nodes":1,
060:                     "cores":32
061:                 }
062:             }
063:         },
064:         "loadable_software_configurations": {
065:             "openmpi": {
066:                 "version_from":[4,0,1],
067:                 "version_to":[5,0,0]
068:             },
069:             "scalapack":{
070:                 "version_split":[2,1,0]
071:             },
072:             "gcc": {
073:                 "version_split": [8,3,0]
074:             }
075:         }
076:     }
077:
078:     (machine, processor, nodes, cores) = GetMachineConfiguration(meta_dict = tuning_metadata)
079:     print ("machine: " + machine + " processor: " + processor + " num_nodes: " + str(nodes) + " num_cores: " + str(cores))
080:
081:     """ Define search space """
082:     m = Integer(128, 10000, transform="normalize", name="m")
083:     n = Integer(128, 10000, transform="normalize", name="n")
084:     mb = Integer(1, 16, transform="normalize", name="mb")
085:     nb = Integer(1, 16, transform="normalize", name="nb")
086:     npernode = Integer(1, int(math.log2(cores)), transform="normalize", name="npernode")
087:     p = Integer(1, nodes*cores, transform="normalize", name="p")
088:     r = Real(float("-Inf"), float("Inf"), name="r")
089:
090:     IS = Space([m, n])
091:     PS = Space([mb, nb, npernode, p])
092:     OS = Space([r])
093:
094:     constraints = {"cst1": cst1, "cst2": cst2, "cst3": cst3}
095:     constants={"nodes":nodes,"cores":cores,"bunit":8}
096:     print(IS, PS, OS, constraints)
097:
098:     problem = TuningProblem(IS, PS, OS, objectives, constraints, None, constants=constants)
099:     historydb = HistoryDB(meta_dict=tuning_metadata)
100:     computer = Computer(nodes=nodes, cores=cores, hosts=None)
101:     options = Options()
102:     data = Data(problem)
103:
104:     gt = GPTune(problem, computer=computer, data=data, options=options, historydb=historydb, driverabspath=os.path.abspath(__file__))
105:
106:     """ Building MLA with the given list of tasks """
107:     giventask = [[2000, 2000]]
108:     NI = len(giventask)
109:     NS = 20
110:     (data, model, stats) = gt.MLA(NS=NS, Igiven=giventask, NI=NI, NS1=max(NS//2, 1))
111:
112: if __name__ == "__main__":
113:     main()
```

## Reusing Trained Surrogate Models

In addition to the ability to re-use function evaluation results, the history database also supports storing and loading trained GP surrogate models.
Re-using pre-trained surrogate models can be useful because the modeling phase of GPTune can require a significant amount of computational resources and time.

Storing model data is done automatically by GPTune if the user runs GPTune with the history database mode. 
An example of JSON data for recording a surrogate model can be found at [here](overview.html#json-format).
By default, if LCM is used, the history database stores every modeling data during the autotuning process.

<!--

### Run MLA with a Pre-Trained Surrogate Model

GPTune provides a method called *MLA_LoadModel* which runs MLA with a pre-trained surrogate model.
Users can invoke *MLA_LoadModel* as follows [TODO: update].

```Python
# create a history db module instance
history_db = HistoryDB()
history_db.tuning_problem_name = 'example'
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

-->

Users can read a surrogate model from the database and use it as a black-box function.
Here, a black-box function means a callable function (from the user side) which returns the mean value predicted by the surrogate model for the given task and parameter information. The task and parameter information is given by the user as function arguments.

The following is a snippet of code to load a model as a black-box function and call the function for a given task/parameter set for our PDGEQRF example.

**Listing 1**
```Python
gt = GPTune( ... )
model_function = gt.LoadSurrogateModel(Igiven = [[4000,2000]], method="max_evals")

ret = model_function({
        "m": 4000,
        "n": 2000,
        "mb": 16,
        "nb": 16,
        "npernode": 5,
        "p": 13})
      })
print (ret) # output is also a dictionary e.g. { "r": 0000 }
```

Similar to MLA\_LoadModel (run MLA with a pre-trained surrogate model), there are several model selection methods:

**Model selection parameters.**
*method="max_evals"*: choose the model that has most function evaluation data.
*method="MLE"* or *"mle"*: choose the model that has the highest likelihood.
*method="AIC"* or *"aic"*: choose the model based on Akaike Information Criterion (AIC).
*method="BIC"* or *"bic"*: choose the model based on Bayesian Information Criterion (BIC).

If the user has trained LCM models for multiple tasks (MLA), the user may want to use them as well.
To do so, the user can simply pass all the task information used to train the model when calling *LoadSurrogateModel*, as follows.

**Listing 2**
```Python
gt = GPTune( ... )
model_function = gt.LoadSurrogateModel(Igiven = [[4000,2000],[2000,1000],[1000,500],[500,250]], method="max_evals")

ret = model_function({
        "m": 4000,
        "n": 2000,
        "mb": 16,
        "nb": 16,
        "npernode": 5,
        "p": 13})
      })
print (ret) # output is also a dictionary e.g. { "r": 0000 }
```

To reproduce the surrogate model and return its black-box function, other important information is the task/parameter/output space, because we want to check if the model assumes the same task/parameter/output space that the user is interested in.
Currently, the user can use the *autotune*'s space declaration in the user's Python code and pass it to the GPTune instance, as we write a driver code for GPTune autotuner.
The following is a simplified code for this.

**Listing 3**
```Python
m = Integer(mmin, mmax, transform="normalize", name="m")
n = Integer(nmin, nmax, transform="normalize", name="n")
mb = Integer(1, 16, transform="normalize", name="mb")
nb = Integer(1, 16, transform="normalize", name="nb")
npernode = Integer(int(math.log2(nprocmin_pernode)), int(math.log2(cores)), transform="normalize", name="npernode")
p = Integer(1, nprocmax, transform="normalize", name="p")
r = Real(float("-Inf"), float("Inf"), name="r")

IS = Space([m, n])
PS = Space([mb, nb, npernode, p])
OS = Space([r])

problem = TuningProblem(IS, PS, OS, ...)
gt = GPTune(problem, ...)
model_function = gt.LoadSurrogateModel(Igiven = giventask, method = "max_evals")

ret = model_function({
        "m": 4000,
        "n": 2000,
        "mb": 16,
        "nb": 16,
        "npernode": 5,
        "p": 13})
      })
print (ret) # output is also a dictionary e.g. { "r": 0000 }
```

The feature to reuse a pre-trained surrogate model can be used for several useful scenarios: in-depth analysis using the model function, use the model function to guide autotuning and/or to tune a new problem, to be used for transfer learning, etc.

**Transfer learning.**
We can treat transfer learning as just like running MLA with pre-trained results. Due to the GPTune's core computational logic, we want to assume that every task always have the same number of sample function evalutaion results. The idea is to use this model function to obtain additional samples of the tasks that cannot be run in the transfer learning method (e.g. result from other machine).
Some examples of this transfer learning approach can be found at [here](https://github.com/gptune/GPTune/tree/master/examples/ScaLAPACK-PDGEQRF-ATMG).

**Make prediction.**
After we reproduce the surrogate model, the surrogate model predicts the output (mean and variance) for a given numpy array of the parameters (normalized floating point values) without information about the parameter space. When the user wants to load a model function, we assume the user defines the parameter space (e.g. PS in the above Listing 3) with the same parameter order used to build the surrogate model. This limitation can be relaxed with more efforts.

**Sensitivity analysis.**
Another use case of surrogate models is conducting a surrogate model-based sensitivity analysis to determine how different values of individual tuning parameters could affect the output results.
GPTune currently offers an interface for sensitivity analysis tool based on [the Sobol method](https://www.sciencedirect.com/science/article/abs/pii/S0378475400002706) [[8](references.md)] and using the implementation of a Python module called [SALib](https://joss.theoj.org/papers/10.21105/joss.00097.pdf) [[9](references.md)].
The Sobol analysis requires (1) samples drawn from the function directly, (2) evaluating the model using the generated sample inputs and saving the model output, and (3) conducting a variance-based mathematical analysis to compute the sensitivity indices.
In the GPTune interface, we use a pre-trained surrogate model to draw and evaluate samples in this sensitivity analysis approach.

