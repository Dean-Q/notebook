---
description: some install commands
---

# pip & conda

we use Anacoda prompt as python command line. in Anacoda command line you can execute python/pip/conda command

### Conda

1. create virtual env    `conda create -n langchain-env python=3.9`
2. active virtual env    `conda activate langchain-env`
3. deactive virtual env  `conda deactivate`
4. list all virtual env   `conda env list`/`conda info --envs`
5. delete unused virtual env  `conda remove -n old-env --all`

### pip

1. install a package   `pip install -U langchain`
2. uninstall a package `pip uninstall -y langchain`
3. list all realted package `pip list | findstr langchain`
4. clean the cache  `pip cache purge`

