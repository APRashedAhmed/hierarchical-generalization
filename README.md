<div align="center">

# Hierarchical Generalization and Learning

</div>

Code that implements the task described in Collins, Frank (2016) task in
[Neural signature of hierarchically structured expectations predictsclustering and transfer of rule sets in reinforcement learning][1].
The package can be installed for general reuse and extension.

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->
## Table of Contents

  - [Getting Started](#getting-started)
    - [Installation](#installation)
    - [Usage](#usage)
  - [API](#api)
    - [Generating Datasets](#generating-datasets)

<!-- markdown-toc end -->

## Getting Started

### Installation

To use the package, first clone it:

```bash
git clone https://github.com/APRashedAhmed/hierarchical-generalization.git
```

Install the requirements:

```bash
# For pip
pip install requirements.txt

# For conda
conda install `cat requirements.txt` -c conda-forge
```

And then install the repo using `pip`:
```bash
# Local install
pip install .
```

Note: If planning on making changes to the code, it is better to install using
the `setup.py` file directly:
```bash
python setup.py develop
```
Installing in this way will allow code changes to be reflected when importing,
whereas installing with `pip` does not.

### Usage

Once installed, the repo functions similarly to any package installed by `pip`
or `conda` and can be imported:

```python
# Import and alias the whole package
import hierarchical_generalization as hg

# Import a specific module
from hierarchical_generalization import taskset as ts

# Import a specific function
from hierarchical_generalization.make_datasets import generate_task_data
```
<br>

## API

### Generating Datasets

The main way to use the package is through the high-level data generation
functions in [`make_datasets.py`](hierarchical_generalization/make_datasets.py). 

1. `generate_phase_train_test_data` - Generates the training and testing data
as dictionaries keyed by the different phases.

```python
from hierarchical_generalization.make_datasets import generate_phase_train_test_data

# Outputs the training and testing
train_data, test_data = generate_phase_train_test_data() 
```

2. `generate_taskset_test_data` - Generates auxiliary an testing data 
dictionary that is seprated by the different task sets.

```python
from hierarchical_generalization.make_datasets import generate_taskset_test_data

# Outputs the taskset test data
ts_test_data = generate_taskset_test_data()
```

3. `generate_task_data` - Generates the phase train, phase test, and taskset
test datasets and ensures the arguments are consistent between the three.

```python
from hierarchical_generalization.make_datasets import generate_task_data

# Outputs the taskset test data
phase_train_data, phase_test_data, ts_test_data = generate_task_data()
```

See the docstrings for each of the functions for more details including their
call signatures.

<!-- Markdown References -->

[1]: https://pubmed.ncbi.nlm.nih.gov/27082659/
