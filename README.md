# GOT-10k Python Toolkit

> UPDATE:<br>
> 1. The toolkit can be directly installed via `pip install got10k` !<br>
> 2. Check our light-weighted repository [SiamFC](https://github.com/got-10k/siamfc) for a minimal example of training and evaluation using GOT-10k toolkit!

This repository contains the official python toolkit for running experiments and evaluate performance on [GOT-10k](http://got-10k.aitestunion.com/) benchmark. The code is written in pure python and is compile-free. Although we support both python2 and python3, we recommend python3 for better performance.

For convenience, the toolkit also provides unofficial implementation of dataset interfaces and tracking pipelines for [OTB (2013/2015)](http://cvlab.hanyang.ac.kr/tracker_benchmark/index.html), [VOT (2013~2018)](http://votchallenge.net), [DTB70](https://github.com/flyers/drone-tracking), [TColor128](http://www.dabi.temple.edu/~hbling/data/TColor-128/TColor-128.html), [NfS](http://ci2cv.net/nfs/index.html) and [UAV123](https://ivul.kaust.edu.sa/Pages/pub-benchmark-simulator-uav.aspx) benchmarks.

[GOT-10k](http://got-10k.aitestunion.com/) is a large, high-diversity and one-shot database for training and evaluating generic purposed visual trackers. If you use the GOT-10k database or toolkits for a research publication, please consider citing:

```Bibtex
"GOT-10k: A Large High-Diversity Benchmark for Generic Object Tracking in the Wild."
L. Huang, X. Zhao and K. Huang,
arXiv:1810.11981, 2018.
```

&emsp;\[[Project](http://got-10k.aitestunion.com/)\]\[[PDF](https://arxiv.org/abs/1810.11981)\]\[[Bibtex](http://got-10k.aitestunion.com/bibtex)\]

## Table of Contents

* [Installation](#installation)
* [Quick Start: A Concise Example](#quick-start-a-concise-example)
* [Quick Start: Jupyter Notebook for Off-the-Shelf Usage](#quick-start-jupyter-notebook-for-off-the-shelf-usage)
* [How to Define a Tracker?](#how-to-define-a-tracker)
* [How to Run Experiments on GOT-10k?](#how-to-run-experiments-on-got-10k)
* [How to Evaluate Performance?](#how-to-evaluate-performance)
* [How to Loop Over GOT-10k Dataset?](#how-to-loop-over-got-10k-dataset)
* [Issues](#issues)

### Installation

Install the toolkit using `pip` (recommended):

```bash
pip install --upgrade got10k
```

Or, alternatively, clone the repository and install dependencies:

```
git clone https://github.com/got-10k/toolkit.git
cd toolkit
pip install -r requirements.txt
```

Then directly copy `got10k` folder to your workspace to use it.

### Quick Start: A Concise Example

Here is a simple example on how to use the toolkit to define a tracker, run experiments on GOT-10k and evaluate performance.

```Python
from got10k.trackers import Tracker
from got10k.experiments import ExperimentGOT10k

class IdentityTracker(Tracker):
    def __init__(self):
        super(IdentityTracker, self).__init__(name='IdentityTracker')
    
    def init(self, image, box):
        self.box = box

    def update(self, image):
        return self.box

if __name__ == '__main__':
    # setup tracker
    tracker = IdentityTracker()

    # run experiments on GOT-10k (validation subset)
    experiment = ExperimentGOT10k('data/GOT-10k', subset='val')
    experiment.run(tracker, visualize=True)

    # report performance
    experiment.report([tracker.name])
```

To run experiments on [OTB](http://cvlab.hanyang.ac.kr/tracker_benchmark/index.html), [VOT](http://votchallenge.net) or other benchmarks, simply change `ExperimentGOT10k`, e.g., to `ExperimentOTB` or `ExperimentVOT`, and `root_dir` to their corresponding paths for this purpose.

### Quick Start: Jupyter Notebook for Off-the-Shelf Usage

Open [quick_examples.ipynb](https://github.com/got-10k/toolkit/tree/master/examples/quick_examples.ipynb) in [Jupyter Notebook](http://jupyter.org/) to see more examples on toolkit usage.

### How to Define a Tracker?

To define a tracker using the toolkit, simply inherit and override `init` and `update` methods from the `Tracker` class. Here is a simple example:

```Python
from got10k.trackers import Tracker

class IdentityTracker(Tracker):
    def __init__(self):
        super(IdentityTracker, self).__init__(
            name='IdentityTracker',  # tracker name
            is_deterministic=True    # stochastic (False) or deterministic (True)
        )
    
    def init(self, image, box):
        self.box = box

    def update(self, image):
        return self.box
```

### How to Run Experiments on GOT-10k?

Instantiate an `ExperimentGOT10k` object, and leave all experiment pipelines to its `run` method:

```Python
from got10k.experiments import ExperimentGOT10k

# ... tracker definition ...

# instantiate a tracker
tracker = IdentityTracker()

# setup experiment (validation subset)
experiment = ExperimentGOT10k(
    root_dir='data/GOT-10k',    # GOT-10k's root directory
    subset='val',               # 'train' | 'val' | 'test'
    result_dir='results',       # where to store tracking results
    report_dir='reports'        # where to store evaluation reports
)
experiment.run(tracker, visualize=True)
```

The tracking results will be stored in `result_dir`.

### How to Evaluate Performance?

Use the `report` method of `ExperimentGOT10k` for this purpose:

```Python
# ... run experiments on GOT-10k ...

# report tracking performance
experiment.report([tracker.name])
```

When evaluated on the __validation subset__, the scores and curves will be directly generated in `report_dir`.

However, when evaluated on the __test subset__, since all groundtruths are withholded, you will have to submit your results to the [evaluation server](http://got-10k.aitestunion.com/submit_instructions) for evaluation. The `report` function will generate a `.zip` file which can be directly uploaded for submission. For more instructions, see [submission instruction](http://got-10k.aitestunion.com/submit_instructions).

See public evaluation results on [GOT-10k's leaderboard](http://got-10k.aitestunion.com/leaderboard).

### How to Loop Over GOT-10k Dataset?

The `got10k.datasets.GOT10k` provides an iterable and indexable interface for GOT-10k's sequences. Here is an example:

```Python
from PIL import Image
from got10k.datasets import GOT10k
from got10k.utils.viz import show_frame

dataset = GOT10k(root_dir='data/GOT-10k', subset='train')

# indexing
img_file, anno = dataset[10]

# for-loop
for s, (img_files, anno) in enumerate(dataset):
    seq_name = dataset.seq_names[s]
    print('Sequence:', seq_name)

    # show all frames
    for f, img_file in enumerate(img_files):
        image = Image.open(img_file)
        show_frame(image, anno[f, :])
```

To loop over `OTB` or `VOT` datasets, simply change `GOT10k` to `OTB` or `VOT` for this purpose.

### Issues

Please report any problems or suggessions in the [Issues](https://github.com/got-10k/toolkit/issues) page.
