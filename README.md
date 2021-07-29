<div align="center">

# Hierarchical Generalization and Learning

</div>

Code that implements the task described in Collins, Frank (2016) task in
[Neural signature of hierarchically structured expectations predicts clustering and transfer of rule sets in reinforcement learning][1].
The package can be installed for general reuse and extension.

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->
## Table of Contents

  - [Description](#description)
  - [Getting Started](#getting-started)
	  - [Installation](#installation)
	  - [Usage](#usage)
  - [Generating Datasets](#generating-datasets)
  - [Example Use-Cases and Prior Work](#example-use-cases-and-prior-work)
	  - [Training and Testing on Phase Data](#training-and-testing-on-phase-data)
	  - [More Samples Per Phase](#more-samples-per-phase)
	  - [Testing On Task-Set Data](#testing-on-task-set-data)

<!-- markdown-toc end -->

## Description

The original task is structured in three phases where participants were presented
with visual stimuli that comprised combinations of shapes and colors. They were
then asked to select one of four keys corresponding to the correct action for
that particular shape and color combination. For all phases, below is figure 1C
showing the mapping from shapes and colors to actions:

<!-- Turns out centering images in GitHub `README.md`s is not that straight -->
<!-- forward. See this SO post:  -->
<!-- https://stackoverflow.com/questions/12090472/how-do-i-center-an-image-in-the-readme-md-file-on-github -->
<p align="center" width="100%">
    <img src="images/fig_1c.png">
</p>

Here the shapes correspond to `S1` through `S4`, the colors correspond to `C0` to `C4`,
and the actions correspond to `A1` to `A4`. Participants were shown the stimuli in
phases, but the stimuli can also be grouped into different task-sets.

This repo implements the task computationally using horizontal and vertical bars
on a numpy array, corresponding to color and shape in the original task.

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

## Generating Datasets

The main way to use the package is through the high-level data generation
functions in [`make_datasets.py`](hierarchical_generalization/make_datasets.py). 

1. `generate_phase_train_test_data` - Generates the training and testing data
as dictionaries keyed by the different phases.

```python
from hierarchical_generalization.make_datasets import generate_phase_train_test_data

# Outputs the training and testing
train_data, test_data = generate_phase_train_test_data() 
```

2. `generate_taskset_test_data` - Generates an auxiliary testing set 
dictionary that is separated by the different task sets.

```python
from hierarchical_generalization.make_datasets import generate_taskset_test_data

# Outputs the taskset test data
ts_test_data = generate_taskset_test_data()
```

3. `generate_task_data` - Generates the phase train, phase test, and task-set
test datasets and ensures the arguments are consistent between the three.

```python
from hierarchical_generalization.make_datasets import generate_task_data

# Outputs the taskset test data
phase_train_data, phase_test_data, ts_test_data = generate_task_data()
```

## Example Use-Cases and Prior Work

The examples below will show pseudo-python of how the task has been used 
previously, and therefore how it can be used going forward. 
For illustrative purposes, training curves of a multilayer perceptron (MLP) are 
shown, along with how to interpret the graphs. The MLP has `N_COLORS * N_SHAPES` 
input units, `100` hidden units, and `N_ACTIONS` output units. For all figures,
the model is rerun `50` times with random initialization. To view the full code
written for `tensorflow=1.10`, see 
[this notebook](https://github.com/APRashedAhmed/leabra-tf/blob/master/docs/source/notebooks/0.8-Hierarchical-Learning.ipynb).

Note that in the original task, participants were shown between `40` and `120` 
trials for phases `A` and `B` (or up to a criterion of 4 out of 5 last trials correct
for each stimulus), and `80` trials for phase `C`. For simplicity, the examples 
below were run using `120` trials for all phases, but can be modified using the
`n_samples` argument for each phase (which is by default set to be `120`, `120`,
and `80` for phases `A`, `B` and `C`, respectively) when defining task.

### Training and Testing on Phase Data

The task structure calls for serial training on each phase of the data. This can
be done by looping through the resulting phases and training on each serially,
while testing on all phases after every sample to visualize how well the model
performs on stimuli from each phase as a function of the phase and number of
samples it has seen:

```python
import hierarchical_generalization.default_configuration as cfg
from hierarchical_generalization.make_datasets import generate_phase_train_test_data

# Define the data
train_phase_data, test_phase_data = generate_phase_train_test_data() 

# Define a model
model = MLP(input=cfg.N_COLORS*cfg.N_SHAPES, hidden=100, output=cfg.N_ACTIONS)

# Loop through each phase
for train_phase, test_data in train_phase_data.items():
    # Train on that particular phase
    for X, y in train_data:
        # Some training function
        train(model, X, y) 
        
        # Test the model on all the phase data
        for test_phase, test_data in test_phase_data.items():
            # Some evaluation function that keeps track of the accuracy
            metrics = evaluate_model(model, test_data)
            
# Plot the metrics with some plotting function
plot_metrics(metrics)
```

<p align="center" width="100%">
    <img src="images/phase_train_test.png">
</p>

The plot shows three curves corresponding to the testing set for phase `A`, `B`, and
`C`, as a function of samples the model is shown. The vertical dashed line 
indicates the end of one phase and the start of the next. 

As expected during the first phase (`A`), the accuracy of the phase `A` testing set 
steadily increases until the end of the phase. This pattern applies to the phase
`B` and `C` testing sets for the middle and rightmost training phases respectively.
Interestingly the performance on phase `A` remains high through phase `B` training
and only begins to drop in phase `C`. Additionally, performance on phase `C` begins
to increase by the end of phase `B`, perhaps indicating some level of 
generalization. Training on phase `C` seems to cause catastrophic interference
with what was learned in phases `A` and `B`, as indicated by the dip in performance
on those testing sets.

### More Samples Per Phase

One way to further assess the generalization effects of the model at each phase is
by presenting more samples per phase. This allows the model to better learn each
phase before moving to the next one, allowing for more robust claims on 
generalization. This can be done when defining the dataset:

```python
# Imports
...

# Set n_samples to 240 for each phase
n_samples = 240
phase_a_args, phase_b_args, phase_c_args = tuple({"n_samples": n_samples} for _ in range(3))

# Pass the phase arg dictionaries as inputs to the dataset generation function
train_phase_data, test_phase_data = generate_phase_train_test_data(
    phase_a_args,
    phase_b_args,
    phase_c_args,
) 

# Rest of the code
...
```

Alternatively, rather than sampling new points, each phase can be run for more 
than one epoch for a similar effect:

```python

# n_samples to 120 and epochs to 2 gives the same as above
n_samples = 120
n_epochs = 2

# Define the data and model as above
...

# Loop through each phase as usual
for train_phase, test_data in train_phase_data.items():

    # Loop epochs in side the phase loop
    for epoch in range(n_epochs):

        # Train on that particular phase
        for X, y in train_data:
            # Rest of the training and testing code
            ...
            
# Plot the results
...
```

Which would yield results similar to the following:

<p align="center" width="100%">
    <img src="images/phase_train_test_two_epochs.png">
</p>

The plot above is identical to the one in the previous section but with twice as
many samples as seen on the x-axis. Most of the task learning seems to happen 
within the original `120` samples per phase.


### Testing On Task-Set Data

To assess transfer learning of the task between phase `A` and `B`, the paper
examined the learning rates the `C0` and `C1` (task-set 1), and `C2` (task-set 
2) stimuli for both phases. In phase `A`, `C0` and `C1` are learned more slowly
than `C2` because `C0` and `C1` are presented half as frequently as `C2` (fig 
2C). However, this trend is reversed in phase `B` because the participants 
learned the task structure meaning they could transfer this knowledge to the new
`C0` and `C1` stimuli (fig 2D).

<p align="center" width="100%">
    <img src="images/fig_2cd.png">
</p>

We can assess this form of transfer in a model by modifying the previous 
examples to test the model on each task-set on every trial, to see if it 
exhibits the same phenomena with the task-set performance.

```python
# Imports
...

# Define the training and task-set data
phase_train_data, _, ts_test_data = generate_task_data()

# Define the model
...

# Loop through each phase
for train_phase, test_data in train_phase_data.items():
    # Train on that particular phase
    for X, y in train_data:
        # Some training function
        train(model, X, y)

        # Loop through the phase A task-set testing data
        for phase_a_ts in ['TS 1 Phase A', 'TS 2 Phase A']:
            # Use them to evaluate the model
            metrics = evaluate_model(model, ts_test_data[phase_a_ts])
            
# Plot the results
...
```

Which would yield results similar to the following:

<p align="center" width="100%">
    <img src="images/phase_a_ts1_ts2.png">
    <img src="images/phase_a_ts1_ts2_2_epochs.png">
</p>

Note: plots for the same experiments but with twice as many samples are plotted 
on the right. 

Focusing on the left third of the graphs, it seems that the model learns 
task-set `2` before learning task-set `1` since the model consistently reaches
peak performance on that task-set before the other just like in the paper
(fig 2C).

We can then repeat this for the phase `B` task-sets:

```python
# All prior code
...

        # Loop through the phase B task-set testing data
        for phase_b_ts in ['TS 1 Phase B', 'TS 2 Phase B']:
            # Use them to evaluate the model
            metrics = evaluate_model(model, ts_test_data[phase_b_ts])
            
# Plot the results
...
```
Giving the following:

<p align="center" width="100%">
    <img src="images/phase_b_ts1_ts2.png">
    <img src="images/phase_b_ts1_ts2_2_epochs.png">
</p>

Focusing now on the middle third of the graphs, we see that the model is still
learning task-set `2` faster than task-set `1` implying that the model does not
transfer the underlying structure from phase `A` when learning the task in phase
`B`.

<!-- Markdown References -->

[1]: https://pubmed.ncbi.nlm.nih.gov/27082659/
