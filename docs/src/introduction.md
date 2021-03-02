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

Here, we present several possible use scenarios in which users can use the GPTune history database.

**Checkpointing and restarting of autotuning.**
The GPTune history database stores every function evaluation result into the database.
The user can use this feature for checkpointing and restarting of an autotuning process.
This is particularly useful if the user's autotuning process can possibly be long and there is the possibility of machine failure, limited job allocation time, etc.

**Using pre-trained tuning results obtained by other users.**
Supercomputing resources are costly for many users.
With our shared repository, users can use already tuned parameter configurations obtained by other users.
Submitted performance data can also contain the machine and software configuration information, hence users can determine which performance data are relevant for re-using.

**Open tuning-problem for crowdtuning.**
Users can define their tuning problem on the GPTune shared repository.
Based on the open tuning problem information, multiple users can run autotuning for the same tuning problem while sharing performance data.
This feature is particularly useful if the HPC code is expensive to run, and there are multiple users who want to use the same HPC code.

**Sharing reproducing autotuning workflows.**
Users can automate their tuning process using the [CK](https://cknowledge.io) technology and can upload the automation information into the database.
The automation allows other users to reproduce performance data easily, and makes the crowdtuning effort more feasible.

**Re-using GP surrogate model.**
The history database also supports storing and loading trained GP surrogate models along with some model statistics information such as likelihood values of the model.
Users can use this feature to not only re-use pre-trained models for autotuning, but also analyze model data for research purposes.

