# Questions for the Reader

To improve our approach of incorporating a history database, we kindly ask for feedback from our users, especially on the following issues.

**Balancing user convenience with the reliability of data.**
Users can use the history database either by using [Python API](./userguide_api.md) or by using the workflow automation tool [CK-GPTune](./ckgptune.md).
Both approaches have advantages and disadvantages.

* [CK-GPTune](./ckgptune.md): this tool is able to automatically detect the versions of the software packages required by the application and store the information into the database. Users can also automate program workflows and share them with other users. However, there is a burden of installing and learning the CK technology.

* [Manual coding](./userguide_api.md): it may be easier to use the manual approach thanks to the simple Python interface, however, users need to provide all information (e.g. machine and software configuration) by hand. Hence, there is a possibility that users may provide incorrect information. Also, it is more difficult for other users to reproduce the experiments.

In this regard, we would like to ask the following questions:

* [Q1.] We are wondering if users would like to use CK-GPTune for their optimization problems. If so, what do you find useful in CK-GPTune, e.g. workflow automation, reproducing experiments possibly from other users, software version detection?.

* [Q2.] If you want to use CK-GPTune, are you considering uploading your CK workflows and programs to the public repository so that you and other users can run and reproduce your results?

* [Q3.] CK-GPTune may require a lot of effort for users to define/add a new program, but it is relatively easy to use programs that have already registered in CK-GPTune.
CK-GPTune currently provides four example programs including [gptune-demo](https://gptune.lbl.gov/documentation/gptune-user-guide/), [PDGEQRF (ScaLAPACK)](http://www.netlib.org/scalapack/), [pddspawn (SuperLU)](https://portal.nersc.gov/project/sparse/superlu/), and [IJ (Hypre)](https://computing.llnl.gov/projects/hypre-scalable-linear-solvers-multigrid-methods) that users can install/run/autotune using simple commands.
Are you willing to use it if we provide more programs (e.g. more ScaLAPACK routines) in CK-GPTune?

**Re-using GP surrogate models.**
The history database also stores the trained GP surrogate model (e.g. model hyperparameters, statistics) so that the same or other users can re-use the trained model.
Regarding re-using trained models, our questions are as follows:

* [Q4.] Are you interested in accessing trained models from other users and uploading models to our public repository? If so, what would you like to use the model for, e.g. re-using/reproducing the model, analyzing the model (for research purposes)?

* [Q5.] Do you have plans to use different modeling algorithms or use any analytical performance models (in GPTune, the surrogate model can also be guided by existing (approximate) performance models) instead of using the GP and Linear Coregionalization Model (LCM) currently used by GPTune?

**Using the public repository.**
Regarding the public data repository, please let us know if you have concerns about the security/reliability issues of public performance data or if you have any feature requests about the public repository.

[TODO] we might want to have a (Google) survey form for the reader to submit feedback.

# Feature Request

Please contact us at gptune-dev@lbl.gov if you have any other feedback and/or feature requests about our history database.

