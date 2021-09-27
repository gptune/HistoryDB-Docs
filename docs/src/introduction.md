# Motivation and Goals

The success of performance tuning depends on collecting a sufficient number of performance data samples, but collecting performance data is an expensive task especially when tuning HPC codes.
To address this challenge, we designed and developed a shared database for autotuning called *GPTune History Database*.
The history database allows users to share performance data with other users at different sites.
Ultimately, the history database would enable the idea of crowd-tuning, which means that mulitple users can collaborate on tuning challenging and expensive problems.

This history database project is part of an autotuning project called [GPTune](https://github.com/gptune/GPTune) [[5](references.md)][[6](references.md)].
GPTune is part of the [xSDK4ECP](https://xsdk.info/ecp) project supported by the Exascale Computing Project (ECP) and designed to tune high-performance computing codes based on multitask and transfer learning to help solve the underlying black-box optimization problem.
While collected data in the history database is compatible with GPTune, the history database can also be used for other autotuners and optimization frameworks.

# Use Scenarios

Here, we present several possible use scenarios in which users can use the GPTune history database.

**Checkpointing and restarting.**
The history database allows the user to reuse performance data (e.g. function evaluation data and trained surrogate models) obtained from previous autotuning.
This allows the user to continue autotuning (checkpointing and restarting) without recollecting data or rebuilding the surrogate model.
This is particularly useful if the user's autotuning process can possibly be long and there is the possibility of machine failure, limited job allocation time, etc.

**Using pre-trained tuning results obtained by other users.**
Supercomputing resources are costly for many users.
With our shared repository, users can use already tuned parameter configurations obtained by other users.
Everyone can therefore benefit from (expensive) runs of widely used high-performance computing codes.
Each performance data can also contain the machine and software configuration information, hence users can determine which performance data are relevant for re-using.

**Re-using GP surrogate model.**
The history database also supports storing and loading trained GP surrogate models along with some model statistics information such as likelihood values of the model.
Users can reuse pre-trained models for sensitivity analysis or performance prediction.

**Harnessing the power of the crowd for performance tuning.**
Users can post their tuning problems on the GPTune shared repository.
Based on the tuning problem information, multiple users at different sites can run autotuning for the same tuning problem while sharing performance data.
This feature is particularly useful if the HPC code is expensive to run, and/or there are multiple users who want to use the same HPC code.

**Using reproducible autotuning workflows with history database.**
Some users may want to automate their tuning process using some automation tools such as [CK](https://cknowledge.io).
GPTune History Database can also be used for storing performance data from CK-based autotuning workflows.
The automation with CK would allow other users to reproduce performance data easily, which makes the crowdtuning effort more feasible.

