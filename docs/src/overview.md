# Motivation and Goals

[GPTune](https://github.com/gptune/GPTune) is an autotuner for high-performance computing codes, relying on multitask learning to help solve the underlying black-box optimization problem.
GPTune is part of the [xSDK4ECP](https://xsdk.info/ecp) project supported by the Exascale Computing Project (ECP).

GPTune is designed to tune high-performance application codes as "black-boxes", running them for carefully chosen tuning parameter values and building a performance model (i.e. surrogate model) based on the measured performance (i.e. function evaluation data).
One of the main costs with this approach is the expensive black-box objective function (i.e. run and measure the application on a parallel machine).
To reduce this cost, the history database aims to provide the following features:

**Re-using autotuning data.**
The proposed history database can re-use performance data (e.g. function evaluation data and trained surrogate models) obtained from previous autotuning.
This allows the user to continue autotuning without recollecting data or rebuilding the surrogate model.

**Harnessing the power of crowdtuning.**
We provide a public shared database, where users can store their performance data or download the performance data provided by other users.
With the shared database, everyone can benefit from (expensive) runs of widely used high-performance computing codes.

**Reproducible autotuning.**
Users may want to reproduce performance data from the same or different users.
We aim to provide a portable workflow automation framework to help users reproduce performance data that exist in our shared database.

# History Database Features

GPTune users can invoke the history database either by manual Python coding in the application-GPTune driver code or by using a command line interface provided by the workflow automation tool called CK-GPTune.
If the history database is invoked, GPTune can store and load performance data to and from performance data files in the user's local storage; each application (code) will have a data file that contains all performance data (obtained by the user and/or downloaded from the shared public database) of the application in JavaScript Object Notation ([JSON](https://www.json.org/json-en.html)) format.

In what follows, we present what you can do with the GPTune history database.

### Re-using Function Evaluation Data

Users can allow GPTune to store function evaluation data obtained from autotuning into data files.
Each data file contains the function evaluation results obtained from the GPTune's Bayesian optimization model.
Autotuning of GPTune relies on three information spaces: *input space* (IS), *parameter space* (PS), and *output space* (OS) ([GPTune Users Guide](https://gptune.lbl.gov/documentation/gptune-user-guide/)).
IS contains all the input problems (i.e. tasks) that the application may encounter (e.g. the sizes of matrices, pointers to input files) and PS contains all the tuning parameter configurations to be optimized (e.g. size of row/column blocks).
OS is the output space for each of the scalar objective functions (e.g. measured runtime).
GPTune saves the function evaluation result for each parameter configuration.
The history database stores these performance data into the JSON file right after evaluating each parameter configuration.
This practice ensures that no data is lost, in the case where  a long run with many parameter configurations does not complete.
If GPTune is run in parallel and multiple processes need to update performance data simultaneously, to keep the consistency of the data, the history database allows one process to update the data file at a time based on simple file access control.
GPTune users can choose to load the previous function evaluation data when starting a new autotuning.
This feature will be useful whether or not a user wants to share data to or from other users.

## Re-using GP Surrogate Model

GPTune uses Bayesian optimization to iteratively build a Gaussian Process (GP) surrogate model by running the application at carefully chosen tuning parameter values.
Refer to ([GPTune Users Guide](https://gptune.lbl.gov/documentation/gptune-user-guide/)) for more details about how GPTune builds surrogate models.
In addition to the ability to reuse previous function evaluation data, the history database also supports storing and loading trained GP surrogate models; users can use this feature to save the GP surrogate model after finishing some autotuning and load a trained model to continue autotuning afterward (with no additional model updates or fewer updates).
Again, this feature will be useful whether or not a user wants to share data to or from other users.
GPTune provides a Python interface to run autotuning after loading a GP surrogate model from the database.
While it supports several model selection criterion to select the model such as MLE (Maximum Likelihood Estimation), AIC (Akaike Information Criterion), and BIC (Bayesian Information Criterion), users can also select a model based on their own method based on the provided statistics information; the history database provides some statistics (e.g. likelihood, gradients) of the model.

## Define Machine/Software Dependencies

In addition to performance data obtained from autotuning, the history database also records the machine configuration (e.g. the number of nodes/cores used) and software information (e.g. which software libraries are used for that application) into the JSON file.
Users can provide this information manually when calling GPTune, but they can also leverage a workflow automation tool called [CK-GPTune](https://github.com/yhcho614/ck-gptune/) to manage the information automatically.
With CK-GPTune, users need to define the application's software dependencies with a meta-description file, then CK-GPTune detects the software packages/libraries that have dependencies, based on the [Collective Knowledge (CK)](https://cknowledge.io) technology.
Users can therefore determine which performance data are relevant for re-using (learning) from a possibly different machine or software versions or configurations.

## Reproducing Performance Data

CK-GPTune not only provides automatic software detection, but also allows you to define/run automated workflows (e.g. experiment, analysis) using a command line interface and meta description files.
In CK-GPTune, we currently provide workflow automation for four example codes (e.g. [ScaLAPACK's PDGEQRF](http://www.netlib.org/scalapack/slug/)) that users can install/run/autotune using simple commands, and plan to provide more examples.
Users can also automate their program workflows and share them with other users with the public database.
This will allow users to reproduce performance data from the same and different users.
For more information about CK-GPTune, please refer to our Github web pages [here](https://github.com/yhcho614/ck-gptune/).

## Public Shared Repository

To harness the power of crowdtuning, we provide a public shared database which allows users to upload their performance data or download performance data provided by other users.

Here are the main design points of our public database:

* Storage: To store all performance data from multiple sites, we use [Cori](https://docs.nersc.gov/systems/cori/)'s storage system provided by NERSC.

* Web service: The public repository uses web resources provided by NERSC (also known as [Science Gateways](https://www.nersc.gov/assets/Uploads/19-Science-Gateways.pdf) for user-DB connection.
We provide the web service at [https://gptune.lbl.gov/](https://gptune.lbl.gov) to access the public database.

* DBMS: Since our performance data is managed as JSON files, we use [MongoDB](https://mongodb.com) internally to manage performance data. User data is managed with [SQLite](https://www.sqlite.org/index.html).

* User interface: Users can download/upload performance data from/to the public repository using the web interface.

* User identification: The public database requires login credentials for users to submit their performance data, but we allow anyone to browse and download data if the data is public data.

