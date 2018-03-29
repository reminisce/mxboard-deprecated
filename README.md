# Logging MXNet Data for Visualization in TensorBoard

## Overview

MXNet TensorBoard (MXBoard) provides a set of Python APIs logging
[MXNet](http://mxnet.incubator.apache.org/) data for visualization in
[TensorBoard](https://www.tensorflow.org/programmers_guide/summaries_and_tensorboard). 
The idea of this project comes from the discussion with [Zihao Zheng](https://github.com/zihaolucky),
the author of
[dmlc/tensorboard](https://github.com/dmlc/tensorboard),
on delivering a visualization solution for MXNet users.
We aim at providing the logging APIs that could process MXNet data efficiently
and supporting most of the data types for visualization in TensorBoard GUI.
We adapted the low-level logging components, `FileWriter`, `EventFileWriter`,
`EventsWriter`, `RecordWriter`,
and `_EventLoggerThread`, from their Python and C++
implementations in [TensorFlow](https://github.com/tensorflow/tensorflow),
and user-level logging APIs defined in `SummaryWriter` from
[tensorboard-pytorch](https://github.com/lanpa/tensorboard-pytorch).
The encoding algorithm used in writing protobuf objects into event files
is directly borrowed from
[TeamHG-Memex/tensorboard_logger](https://github.com/TeamHG-Memex/tensorboard_logger).

MXBoard supports logging the following data types listed on the TensorBoard GUI:
- [GRAPHS](https://www.tensorflow.org/versions/r1.1/get_started/graph_viz)
- SCALARS
- IMAGES
- [HISTOGRAMS](https://www.tensorflow.org/programmers_guide/tensorboard_histograms)
- [PROJECTOR/EMBENDDING](https://www.tensorflow.org/programmers_guide/embedding)
- AUDIO
- TEXT
- PR CURVES

The corresponding Python APIs are accessible through a class called `SummaryWriter` as below.
```python
    mxboard.SummaryWriter.add_audio
    mxboard.SummaryWriter.add_embedding
    mxboard.SummaryWriter.add_graph
    mxboard.SummaryWriter.add_histogram
    mxboard.SummaryWriter.add_image
    mxboard.SummaryWriter.add_pr_curve
    mxboard.SummaryWriter.add_scalar
    mxboard.SummaryWriter.add_text
```

## Installation

### Install MXBoard from PyPI
```bash
pip install mxboard
```

### Install MXBoard from source
```bash
git clone https://github.com/awslabs/mxnet-tensorboard.git
cd mxnet-tensorboard/python
python setup.py install
```

### Install TensorBoard from PyPI
MXBoard is a logger for writing MXNet data to event files.
To visualize those data in browsers, users still have to install
[TensorBoard](https://www.tensorflow.org/versions/r1.1/get_started/summaries_and_tensorboard)
separately.
```bash
pip install tensorflow && pip install tensorboard
```
Type
```bash
tensorboard --help
```
to verify that the TensorBoard binary has been installed correctly.

### Other required packages
MXBoard relies on the following packages for logging data.
- [MXNet](https://pypi.python.org/pypi/mxnet)
- [protobuf3](https://pypi.python.org/pypi/protobuf)
- [six](https://pypi.python.org/pypi/six)
- [Pillow](https://pypi.python.org/pypi/Pillow)


## Visualizing MXNet data in 30 seconds
After installing all the required packages, let's walk through a simple example demonstrating how
MXBoard enables visualizing MXNet `NDArray`s in terms of histograms in TensorBoard.

1. Prepare a Python script for writing data generated by the `normal` operator to an event file.
The data is generated ten times with decreasing standard deviation and written to the event
file each time. It's expected to see the data distribution gradually becomes more centered around
the mean value. Note that here we specify the creating the event file in the folder `logs`
under the current directory. We will need to pass the folder path to the TensorBoard binary.
```python
import mxnet as mx
from mxboard import SummaryWriter


with SummaryWriter(logdir='./logs') as sw:
    for i in range(10):
        # create a normal distribution with fixed mean and decreasing std
        data = mx.nd.normal(loc=0, scale=10.0/(i+1), shape=(10, 3, 8, 8))
        sw.add_histogram(tag='norml_dist', values=data, bins=200, global_step=i)
```

2. Run TensorBoard by typing the following command under the same directory as `logs`.
```bash
tensorboard --logdir=./logs --host=127.0.0.1 --port=8888
```
Note that in some situations,
the port number `8888` may be occupied by other applications and launching TensorBoard
may fail. You may choose a different available port number.

3. In the browser, enter the address `127.0.0.1:8888`, and click the tab **HISTOGRAMS**
in the TensorBoard GUI. You will see data distribution change over time as the following.
![png](https://github.com/reminisce/web-data/blob/tensorboard_doc/mxnet/tensorboard/doc/summary_histogram_norm.png)

## More tutorials
- [Logging various data types](https://github.com/reminisce/mxnet-tensorboard/blob/tensorboard_logging/demo.md)
- [Training an MNIST model with MXBoard](https://github.com/reminisce/mxnet-tensorboard/tree/tensorboard_logging/examples/mnist)

## References
1. https://github.com/TeamHG-Memex/tensorboard_logger
2. https://github.com/lanpa/tensorboard-pytorch
3. https://github.com/dmlc/tensorboard
4. https://github.com/tensorflow/tensorflow
5. https://github.com/tensorflow/tensorboard

## License
This library is licensed under the Apache 2.0 License. 
