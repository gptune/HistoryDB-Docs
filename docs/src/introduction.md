# Motivation and Goals

[GPTune](https://github.com/gptune/GPTune) is an autotuner for high-performance computing codes, relying on multitask learning to help solve the underlying black-box optimization problem.
GPTune is part of the [xSDK4ECP](https://xsdk.info/ecp) project supported by the Exascale Computing Project (ECP).

GPTune is designed to tune high-performance application codes as "black-boxes", running them for carefully chosen tuning parameter values and building a performance model (i.e. surrogate model) based on the measured performance (i.e. function evaluation data).
One of the main costs with this approach is the expensive black-box objective function (i.e. run and measure the application on a parallel machine).

The history database aims to reduce the cost by providing the following features:

**Re-using autotuning data.**
The history database allows the user to re-use performance data (e.g. function evaluation data and trained surrogate models) obtained from previous autotuning.
This allows the user to continue autotuning without recollecting data or rebuilding the surrogate model.

**Harnessing the power of crowdtuning.**
We provide a public shared database, where users can store their performance data or download the performance data provided by other users.
Everyone can therefore benefit from (expensive) runs of widely used high-performance computing codes.

**Reproducible autotuning.**
Some users may want to reproduce performance data from the same or different users.
We aim to provide a portable workflow automation framework to help users reproduce performance data that exist in our shared database.

# Use Scenarios

Here, we present some use scenarios in which users can use the GPTune history database.

**Re-using function evaluation data.**
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

**Re-using GP surrogate model.**
GPTune uses Bayesian optimization to iteratively build a Gaussian Process (GP) surrogate model by running the application at carefully chosen tuning parameter values.
Refer to ([GPTune Users Guide](https://gptune.lbl.gov/documentation/gptune-user-guide/)) for more details about how GPTune builds surrogate models.
In addition to the ability to reuse previous function evaluation data, the history database also supports storing and loading trained GP surrogate models; users can use this feature to save the GP surrogate model after finishing some autotuning and load a trained model to continue autotuning afterward (with no additional model updates or fewer updates).
Again, this feature will be useful whether or not a user wants to share data to or from other users.
GPTune provides a Python interface to run autotuning after loading a GP surrogate model from the database.
While it supports several model selection criterion to select the model such as MLE (Maximum Likelihood Estimation), AIC (Akaike Information Criterion), and BIC (Bayesian Information Criterion), users can also select a model based on their own method based on the provided statistics information; the history database provides some statistics (e.g. likelihood, gradients) of the model.

In addition to the ability to reuse previous function evaluation data, the history database also supports storing and loading trained GP surrogate models.
Reusing pre-trained surrogate models can be useful for autotuning users because the modeling phase of GPTune can require a significant amount of time.
%In addition to the information we currently store (task/tuning parameters and the evaluation results, machine/software configuration), we are currently considering storing some sort of data (e.g.\ trained hyperparameter values, etc.) of the GP surrogate model so that the same or other users can re-use the trained model.
%In GPTune~\cite{GPTuneUserGuide}, once initial sampling is completed, the GP surrogate model is updated each time GPTune evaluates an additional sample parameter configuration.
%When the surrogate model is updated, the history database stores the surrogate model data into the database.
Users can use this feature to save the GP surrogate model after some autotuning and load a trained model to continue autotuning afterward.
Note that trained surrogate models may not be meaningful for different problem spaces.
The history database therefore loads trained models only if they match the problem space of the given optimization problem.



**Define machine/software dependencies.**
In addition to performance data obtained from autotuning, the history database also records the machine configuration (e.g. the number of nodes/cores used) and software information (e.g. which software libraries are used for that application) into the JSON file.
Users can provide this information manually when calling GPTune, but they can also leverage a workflow automation tool called [CK-GPTune](https://github.com/yhcho614/ck-gptune/) to manage the information automatically.
With CK-GPTune, users need to define the application's software dependencies with a meta-description file, then CK-GPTune detects the software packages/libraries that have dependencies, based on the [Collective Knowledge (CK)](https://cknowledge.io) technology.
Users can therefore determine which performance data are relevant for re-using (learning) from a possibly different machine or software versions or configurations.

**Reproducing performance data.**
CK-GPTune not only provides automatic software detection, but also allows you to define/run automated workflows (e.g. experiment, analysis) using a command line interface and meta description files.
In CK-GPTune, we currently provide workflow automation for four example codes (e.g. [ScaLAPACK's PDGEQRF](http://www.netlib.org/scalapack/slug/)) that users can install/run/autotune using simple commands, and plan to provide more examples.
Users can also automate their program workflows and share them with other users with the public database.
This will allow users to reproduce performance data from the same and different users.
For more information about CK-GPTune, please refer to our Github web pages [here](https://github.com/yhcho614/ck-gptune/).

**Public Shared Repository.**
To harness the power of crowdtuning, we provide a public shared database which allows users to upload their performance data or download performance data provided by other users.

