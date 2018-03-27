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
and supporting most of the data types for visualization in TensorBoard.
We adapted the low-level logging components, `FileWriter`, `EventFileWriter`,
`EventsWriter`, `RecordWriter`,
and `_EventLoggerThread`, from their Python and C++
implementations in [TensorFlow](https://github.com/tensorflow/tensorflow),
and user-level logging APIs defined in `SummaryWriter` from
[tensorboard-pytorch](https://github.com/lanpa/tensorboard-pytorch).
The encoding algorithm used in writing protobuf objects into event files
is directly borrowed from
[TeamHG-Memex/tensorboard_logger](https://github.com/TeamHG-Memex/tensorboard_logger).

### How to install MXBoard from PyPI
```bash
pip install mxboard
```

### How to build MXBoard from source and install locally
```bash
git clone https://github.com/awslabs/mxnet-tensorboard.git
cd mxnet-tensorboard/python
python setup.py install
```

### How to install TensorBoard
To launch TensorBoard for visualization, make sure you have the
[official release of TensorBoard](https://pypi.python.org/pypi/tensorboard) installed.
You can type `pip install tensorflow && pip install tensorboard`
on you machine to install TensorBoard.

### How to launch TensorBoard
After you installed the TensorBoard Python package, type the following command in the terminal
to launch TensorBoard:
```
tensorborad --logdir=/path/to/your/log/dir --host=your_host_ip --port=your_port_number
```
As an example of visualizing data using the browser on your machine, you can type
```
tensorborad --logdir=/path/to/your/log/dir --host=127.0.0.1 --port=8888
```
Then in the browser, type address `127.0.0.1:8888`. Note that in some situations,
the port number `8888` may be occupied by other applications and launching TensorBoard
may fail. You may choose a different port number that is available in those situations.


### How to use TensorBoard GUI for data visualization
Please find the tutorials on
[TensorFlow website](https://www.tensorflow.org/programmers_guide/summaries_and_tensorboard)
for details.

### What other packages are required for using MXBoard
Please make sure the following Python packages have been installed before using
the logging APIs:
- [MXNet](https://pypi.python.org/pypi/mxnet)
- [protobuf3](https://pypi.python.org/pypi/protobuf)
- [six](https://pypi.python.org/pypi/six)
- [Pillow](https://pypi.python.org/pypi/Pillow)


### What data types in TensorBoard GUI are supported by MXNet logging APIs
We currently support the following data types that you can find on the TensorBoard GUI:
- SCALARS
- IMAGES
- [HISTOGRAMS](https://www.tensorflow.org/programmers_guide/tensorboard_histograms)
- [PROJECTOR/EMBENDDING](https://www.tensorflow.org/programmers_guide/embedding)
- AUDIO
- TEXT
- PR CURVES

### What are logging APIs
MXBoard provides the logging APIs through the `SummaryWriter` class.

```python
    mxboard.SummaryWriter
    mxboard.SummaryWriter.add_audio
    mxboard.SummaryWriter.add_embedding
    mxboard.SummaryWriter.add_graph
    mxboard.SummaryWriter.add_histogram
    mxboard.SummaryWriter.add_image
    mxboard.SummaryWriter.add_pr_curve
    mxboard.SummaryWriter.add_scalar
    mxboard.SummaryWriter.add_text
    mxboard.SummaryWriter.close
    mxboard.SummaryWriter.flush
    mxboard.SummaryWriter.get_logdir
    mxboard.SummaryWriter.reopen
```

## Examples
Let's take a look at several simple examples demonstrating the use of MXBoard logging APIs.


### Graph
Graphs are visual representations of neural networks. MXBoard supports visualizing MXNet neural
networks in terms of [Symbol](https://github.com/apache/incubator-mxnet/blob/master/python/mxnet/symbol/symbol.py#L53)
and [HybridBlock](https://github.com/apache/incubator-mxnet/blob/master/python/mxnet/gluon/block.py#L376).
The following code would present the visualization of a toy network defined using symbols.
Users can double click the node block to expand or collapse the node for
exposing or hiding extra sub-nodes of an operator.
```python
import mxnet as mx
from mxboard import SummaryWriter

data = mx.sym.Variable('data')
weight = mx.sym.Variable('weight')
bias = mx.sym.Variable('fc1_bias', lr_mult=1.0)
conv1 = mx.symbol.Convolution(data=data, weight=weight, name='conv1', num_filter=32, kernel=(3, 3))
conv2 = mx.symbol.Convolution(data=data, weight=weight, name='conv2', num_filter=32, kernel=(3, 3))
conv3 = conv1 + conv2
bn1 = mx.symbol.BatchNorm(data=conv3, name="bn1")
act1 = mx.symbol.Activation(data=bn1, name='relu1', act_type="relu")
sum1 = act1 + conv3
mp1 = mx.symbol.Pooling(data=sum1, name='mp1', kernel=(2, 2), stride=(2, 2), pool_type='max')
fc1 = mx.sym.FullyConnected(data=mp1, bias=bias, name='fc1', num_hidden=10, lr_mult=0)
fc2 = mx.sym.FullyConnected(data=fc1, name='fc2', num_hidden=10, wd_mult=0.5)
sc1 = mx.symbol.SliceChannel(data=fc2, num_outputs=10, name="slice_1", squeeze_axis=0)

with SummaryWriter(logdir='./logs') as sw:
    sw.add_graph(sc1)
```
![png](https://github.com/reminisce/web-data/blob/tensorboard_doc/mxnet/tensorboard/doc/summary_graph_symbol.png)

Users can try the following code to visualize a much more sophisticated network:
[Inception V3](https://arxiv.org/abs/1512.00567) defined in MXNet Gluon model zoo.
```python
from mxboard import SummaryWriter
from mxnet.gluon.model_zoo.vision import get_model

net = get_model('inceptionv3')

with SummaryWriter(logdir='./logs') as sw:
    sw.add_graph(net)
```

### Scalar
Scalar values are often plotted in terms of curves, such as training accuracy as time evolves. Here
is an example of plotting the curve of `y=sin(x/100)` where `x` is in the range of `[0, 2*pi]`.
```python
import numpy as np
from mxboard import SummaryWriter

x_vals = np.arange(start=0, stop=2 * np.pi, step=0.01)
y_vals = np.sin(x_vals)
with SummaryWriter(logdir='./logs') as sw:
    for x, y in zip(x_vals, y_vals):
        sw.add_scalar(tag='sin_function_curve', value=y, global_step=x * 100)
```
![png](https://github.com/reminisce/web-data/blob/tensorboard_doc/mxnet/tensorboard/doc/summary_scalar_sin.png)


### Histogram
We can visulize the value distributions of tensors by logging `NDArray`s in terms of histograms.
The following code snippet generates a series of normal distributions with smaller and smaller standard deviations.
```python
import mxnet as mx
from mxboard import SummaryWriter


with SummaryWriter(logdir='./logs') as sw:
    for i in range(10):
        # create a normal distribution with fixed mean and decreasing std
        data = mx.nd.normal(loc=0, scale=10.0/(i+1), shape=(10, 3, 8, 8))
        sw.add_histogram(tag='norml_dist', values=data, bins=200, global_step=i)
```
![png](https://github.com/reminisce/web-data/blob/tensorboard_doc/mxnet/tensorboard/doc/summary_histogram_norm.png)


### Image
The image logging API can take MXNet `NDArray` or `numpy.ndarray` of 2-4 dimensions.
It will preprocess the input image and write the processed image to the event file.
When the input image data is 2D or 3D, it represents a single image.
When the input image data is a 4D tensor, which represents a batch of images, the logging
API would make a grid of those images by stitching them together before write
them to the event file. The following code snippet saves 15 same images
for visualization in TensorBoard.
```python
import mxnet as mx
import numpy as np
from mxboard import SummaryWriter
from scipy import misc

# get a racoon face image from scipy
# and convert its layout from HWC to CHW
face = misc.face().transpose((2, 0, 1))
# add a batch axis in the beginning
face = face.reshape((1,) + face.shape)
# replicate the face by 15 times
faces = [face] * 15
# concatenate the faces along the batch axis
faces = np.concatenate(faces, axis=0)

img = mx.nd.array(faces, dtype=faces.dtype)
with SummaryWriter(logdir='./logs') as sw:
    # write batched faces to the event file
    sw.add_image(tag='faces', image=img)
```
![png](https://github.com/reminisce/web-data/blob/tensorboard_doc/mxnet/tensorboard/doc/summary_image_faces.png)


### Embedding
Embedding visualization enables people to get an intuition on how data is clustered
in 2D or 3D space. The following code takes 2,560 images of handwritten digits
from the [MNIST dataset](http://yann.lecun.com/exdb/mnist/) and log them
as embedding vectors with labels and original images.
```python
import numpy as np
import mxnet as mx
from mxnet import gluon
from mxboard import SummaryWriter


batch_size = 128


def transformer(data, label):
    data = data.reshape((-1,)).astype(np.float32)/255
    return data, label

# training dataset containing MNIST images and labels
train_data = gluon.data.DataLoader(
    gluon.data.vision.MNIST('./data', train=True, transform=transformer),
    batch_size=batch_size, shuffle=True, last_batch='discard')

initialized = False
embedding = None
labels = None
images = None

for i, (data, label) in enumerate(train_data):
    if i >= 20:
        # only fetch the first 20 batches of images
        break
    if initialized:  # after the first batch, concatenate the current batch with the existing one
        embedding = mx.nd.concat(*(embedding, data), dim=0)
        labels = mx.nd.concat(*(labels, label), dim=0)
        images = mx.nd.concat(*(images, data.reshape(batch_size, 1, 28, 28)), dim=0)
    else:  # first batch of images, directly assign
        embedding = data
        labels = label
        images = data.reshape(batch_size, 1, 28, 28)
        initialized = True

with SummaryWriter(logdir='./logs') as sw:
    sw.add_embedding(tag='mnist', embedding=embedding, labels=labels, images=images)
```
![png](https://github.com/reminisce/web-data/blob/tensorboard_doc/mxnet/tensorboard/doc/summary_embedding_mnist.png)


### Audio
The following code generates audio data uniformly sampled in range `[-1, 1]`
and write the data to the event file for TensorBoard to playback.
```python
import mxnet as mx
from mxboard import SummaryWriter


frequency = 44100
# 44100 random samples between -1 and 1
data = mx.random.uniform(low=-1, high=1, shape=(frequency,))
max_abs_val = data.abs().max()
# rescale the data to the range [-1, 1]
data = data / max_abs_val
with SummaryWriter(logdir='./logs') as sw:
    sw.add_audio(tag='uniform_audio', audio=data, global_step=0)
```
![png](https://github.com/reminisce/web-data/blob/tensorboard_doc/mxnet/tensorboard/doc/summary_audio_uniform.png)


### Text
TensorBoard is able to render plain text as well as text in the markdown format.
The following code demonstrates these two use cases.
```python
from mxboard import SummaryWriter


def simple_example(sw, step):
    greeting = 'Hello MXNet from step {}'.format(str(step))
    sw.add_text(tag='simple_example', text=greeting, global_step=step)


def markdown_table(sw):
    header_row = 'Hello | MXNet,\n'
    delimiter = '----- | -----\n'
    table_body = 'This | is\n' + 'so | awesome!'
    sw.add_text(tag='markdown_table', text=header_row+delimiter+table_body)


with SummaryWriter(logdir='./logs') as sw:
    simple_example(sw, 100)
    markdown_table(sw)
```
![png](https://github.com/reminisce/web-data/blob/tensorboard_doc/mxnet/tensorboard/doc/summary_text.png)


### PR Curve
Precision-Recall is a useful metric of success of prediction when the categories are imbalanced.
The relationship between recall and precision can be visualized in terms of precision-recall curves.
The following code snippet logs the data of predictions and labels for visualizing
the precision-recall curve in TensorBoard. It generates 100 numbers uniformly distributed in range `[0, 1]` representing
the predictions of 100 examples. The labels are also generated randomly by picking either 0 or 1.
```python
import mxnet as mx
import numpy as np
from mxboard import SummaryWriter

with SummaryWriter(logdir='./logs') as sw:
    predictions = mx.nd.uniform(low=0, high=1, shape=(100,), dtype=np.float32)
    labels = mx.nd.uniform(low=0, high=2, shape=(100,), dtype=np.float32).astype(np.int32)
    sw.add_pr_curve(tag='pseudo_pr_curve', predictions=predictions, labels=labels, num_thresholds=120)
```
![png](https://github.com/reminisce/web-data/blob/tensorboard_doc/mxnet/tensorboard/doc/summary_pr_curve_uniform.png)


## References
1. https://github.com/TeamHG-Memex/tensorboard_logger
2. https://github.com/lanpa/tensorboard-pytorch
3. https://github.com/dmlc/tensorboard
4. https://github.com/tensorflow/tensorflow
5. https://github.com/tensorflow/tensorboard

## License

This library is licensed under the Apache 2.0 License. 
