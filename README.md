# GPTune History Database Documentation

This repository contains source files for the GPTune history database documentation at https://gptune.lbl.gov
We use Sphinx/Readthedocs tools for generating online document web pages.

If you want to generate the HTML pages from your local system, you can use the following steps.

- Install Sphinx

```
$ pip install sphinx
```

- Compilation

```
$ cd docs/
$ make html
```

This will generate HTML pages in the *_build* directory.

- Open webpage

```
$ open _build/html/index.html
```

## Acknowledgement

GPTune Copyright (c) 2021, The Regents of the University of California, through
Lawrence Berkeley National Laboratory (subject to receipt of any required approvals
from the U.S.Dept. of Energy) and the University of California, Berkeley.
All rights reserved.

If you have questions about your rights to use or distribute this software,
please contact Berkeley Lab's Intellectual Property Office at IPO@lbl.gov.

NOTICE.  This Software was developed under funding from the U.S. Department
of Energy and the U.S. Government consequently retains certain rights.  As
such, the U.S. Government has been granted for itself and others acting on
its behalf a paid-up, nonexclusive, irrevocable, worldwide license in the
Software to reproduce, distribute copies to the public, prepare derivative
works, and perform publicly and display publicly, and to permit other to do so.
