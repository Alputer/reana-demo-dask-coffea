# REANA example - Dask and Coffea

[![image](https://github.com/reanahub/reana-demo-dask-coffea/workflows/CI/badge.svg)](https://github.com/reanahub/reana-demo-dask-coffea/actions)
[![image](https://img.shields.io/badge/discourse-forum-blue.svg)](https://forum.reana.io)
[![image](https://img.shields.io/github/license/reanahub/reana-demo-dask-coffea.svg)](https://github.com/reanahub/reana-demo-dask-coffea/blob/master/LICENSE)
[![image](https://www.reana.io/static/img/badges/launch-on-reana-at-cern.svg)](https://reana.cern.ch/launch?url=https%3A%2F%2Fgithub.com%2Freanahub%2Freana-demo-dask-coffea&specification=reana.yaml&name=reana-demo-dask-coffea)

## About

## Analysis structure

Making a research data analysis reproducible basically means to provide "runnable
recipes" addressing (1) where is the input data, (2) what software was used to analyse
the data, (3) which computing environments were used to run the software and (4) which
computational workflow steps were taken to run the analysis. This will permit to
instantiate the analysis on the computational cloud and run the analysis to obtain (5)
output results.


### 1. Input data

In this example, we are using the file whose url is given below which is hosted at eospublic.
- `root://eospublic.cern.ch//eos/root-eos/benchmark/Run2012B_SingleMu.root`

### 2. Analysis code

The analysis code consists of a single python file called `analysis.py` which connects to a dask cluster and then conducts the analysis.

### 3. Compute environment

In order to be able to rerun the analysis even several years in the future, we need to "encapsulate the current compute environment". We shall achieve this by preparing a [Docker](https://www.docker.com/)  container image for our analysis steps.

This example makes use of the coffea platform and the specific image for the platform we are using in this example can be found [here](https://hub.docker.com/r/coffeateam/coffea-dask-cc7).

### 4. Analysis workflow

The analysis workflow is simple and consists of a single step. We simply run the script `python analysis.py` to run the example. However, realize that the actual analysis is relatively heavy and parallelized by dask behind the scenes. As a user, the task graphs and the parallel steps are hidden to us.

### 5. Output results

The example produces the given histogram as an output.
![](https://github.com/user-attachments/assets/e52c2391-626d-4556-90ca-75248516cc95)


## Running the example on REANA cloud

There are two ways to execute this analysis example on REANA.

If you would like to simply launch this analysis example on the REANA instance at CERN
and inspect its results using the web interface, please click on the following
badge:

[![Launch with Serial on REANA@CERN badge](https://www.reana.io/static/img/badges/launch-with-serial-on-reana-at-cern.svg)](https://reana.cern.ch/launch?url=https://github.com/reanahub/reana-demo-dask-coffea&specification=reana.yaml&name=reana-demo-dask-coffea)

If you would like a step-by-step guide on how to use the REANA command-line client to
launch this analysis example, please read on.

We start by creating a [reana.yaml](reana.yaml) file describing the above analysis
structure with its inputs, code, runtime environment, computational workflow steps and
expected outputs:

```yaml
version: 0.9.3
inputs:
  files:
    - codes/analysis.py
workflow:
  type: serial
  resources:
    dask:
      image: docker.io/coffeateam/coffea-dask-cc7:0.7.22-py3.10-g7f049
      cores: 2
      memory: "4Gi"
      single_worker_cores: 0.5
      single_worker_memory: "1Gi"
  specification:
    steps:
      - name: mystep
        environment: docker.io/coffeateam/coffea-dask-cc7:0.7.22-py3.10-g7f049
        commands:
        - python codes/analysis.py
outputs:
  files:
    - histogram.png
```

In this example we are using a simple Serial workflow engine to represent our sequential
computational workflow steps.

We can now install the REANA command-line client, run the analysis and download the
resulting plots:

```console
$ # create new virtual environment
$ virtualenv ~/.virtualenvs/reana
$ source ~/.virtualenvs/reana/bin/activate
$ # install REANA client
$ pip install reana-client
$ # connect to some REANA cloud instance
$ export REANA_SERVER_URL=https://reana.cern.ch/
$ export REANA_ACCESS_TOKEN=XXXXXXX
$ # create new workflow
$ reana-client create -n myanalysis
$ export REANA_WORKON=myanalysis
$ # upload input code, data and workflow to the workspace
$ reana-client upload
$ # start computational workflow
$ reana-client start
$ # ... should be finished in about 5 minutes
$ reana-client status
$ # list workspace files
$ reana-client ls
$ # download output results
$ reana-client download
```

Please see the [REANA-Client](https://reana-client.readthedocs.io/) documentation for
more detailed explanation of typical `reana-client` usage scenarios.