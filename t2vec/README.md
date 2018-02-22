
This repository contains the code used in our ICDE-18 paper *Deep Representation Learning for Trajectory Similarity Computation*.


## Preprocessing

The preprocessing step will generate all data required in the training stage, and the generated training and validation dataset will be saved in `data`. The preprocessing code is written in [Julia Language](https://julialang.org/), please refer to the code and install all required packages.

Two parameters can be set at this step, `cellsize` in `preprocessing/preprocess.jl` and denoising `radius` in `preprocessing/utils.jl:disort()`, you can leave them as their default values which are the ones used in our paper.

```shell
$ curl http://archive.ics.uci.edu/ml/machine-learning-databases/00339/train.csv.zip -o data/porto.csv.zip
$ unzip data/porto.csv.zip
$ mv train.csv data/porto.csv
$ cd preprocessing
$ julia preprocess.jl
```

## Training

```shell
$ python t2vec.py -data data -vocab_size 18866 -criterion_name "KLDIV" -knearestvocabs "preprocessing/porto-vocab-dist-cell100.h5"
```

The training produces two model `checkpoint.pt` and `best_model.pt`, `checkpoint.pt` contains the latest trained model and `best_model.pt` saves the model which has the best performance on the validation data.

In our original experiment, the model was trained with a Tesla K40 GPU about 14 hours so you can just terminate the training after 14 hours if you use a GPU that is as good as or better than K40, the above two models will be saved automatically.


## Embedding

Please refer to the code in `experiment/experiment.jl` seeing how to generate the test file `trj.t` and then run

```shell
$ python t2vec.py -data experiment -vocab_size 18866 -checkpoint "best_model.pt" -mode 2
```

It will encode the trajectories in file `experiment/trj.t` into vectors which will be saved into file `experiment/trj.vec`.

In our experiment we train a three-layers model and the last layer outputs are used as the trajectory representations, see the code in `experiment/experiment.jl`:

```julia
vecs = h5open(joinpath("", vecfile), "r") do f
    read(f["layer3"])
end
```

## Miscellaneous

The code was written in Julia 0.5 and PyTorch 0.1.12 along with Python 3.6, and since then both Julia and PyTorch have evolved rapidly to a higher version which may bring some breaking changes, hence the code here only serves as an illustrating examples of our original prototype.
