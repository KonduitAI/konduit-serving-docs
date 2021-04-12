---
description: Running and IRIS dataset classifier through CUSTOM endpoints
---

# Pytorch \(IRIS\)

### Overview

This documentation shows Konduit-Serving can serve a custom model and include post-processing in the pipeline to give a direct output label understood by a human. Iris model is used in this example to deploy on the server as a classifier through custom endpoints.

### Adding package to the classpath

First of all we need to add the main package to the classpath so that the notebook can load all the necessary libraries from Konduit-Serving into the Jupyter Notebook kernel.

Classpaths can be considered similar to `site-packages` in the python ecosystem where each library that's to be imported to your code is loaded from.

We package almost everything you need to get started with the `konduit.jar` package so you can just start working on the actual code, without having to care about any boilerplate configuration.

```bash
%classpath add jar ../../konduit.jar
```

Let's ensure the working directory is correct and list all the file available in the directory.

```bash
%%bash
echo "Current directory $(pwd)" && tree
```

You'll be able to view as the following.

```text
Current directory /root/konduit/demos/1-pytorch-onnx-iris
.
├── dataset
│   └── iris.csv
├── iris.onnx
├── onnx-iris.ipynb
├── onnx.yaml
└── train.py

1 directory, 5 files
```

### Main model script code

We're creating a Pytorch model from scratch here and then converting that into ONNX format.

```bash
%%bash
less train.py
```

You'll be able to browse the source code how the training takes places.

```text
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, precision_score, recall_score

import torch
import torch.nn as nn
import torch.nn.functional as F
from torch.autograd import Variable


class Net(nn.Module):
    # define nn
    def __init__(self):
        super(Net, self).__init__()
        self.fc1 = nn.Linear(4, 100)
        self.fc2 = nn.Linear(100, 100)
        self.fc3 = nn.Linear(100, 3)
        self.softmax = nn.Softmax(dim=1)

    def forward(self, X):
        X = F.relu(self.fc1(X))
        X = self.fc2(X)
        X = self.fc3(X)
        X = self.softmax(X)

        return X


# load IRIS dataset
dataset = pd.read_csv('dataset/iris.csv')

# transform species to numerics
dataset.loc[dataset.species == 'Iris-setosa', 'species'] = 0
dataset.loc[dataset.species == 'Iris-versicolor', 'species'] = 1
dataset.loc[dataset.species == 'Iris-virginica', 'species'] = 2

train_X, test_X, train_y, test_y = train_test_split(dataset[dataset.columns[0:4]].values,
                                                    dataset.species.values, test_size=0.8)

# wrap up with Variable in pytorch
train_X = Variable(torch.Tensor(train_X).float())
test_X = Variable(torch.Tensor(test_X).float())
train_y = Variable(torch.Tensor(train_y).long())
test_y = Variable(torch.Tensor(test_y).long())

net = Net()

criterion = nn.CrossEntropyLoss()  # cross entropy loss

optimizer = torch.optim.SGD(net.parameters(), lr=0.01)

for epoch in range(1000):
    optimizer.zero_grad()
    out = net(train_X)
    loss = criterion(out, train_y)
    loss.backward()
    optimizer.step()

    if epoch % 100 == 0:
        print('number of epoch', epoch, 'loss', loss.item())

predict_out = net(test_X)
_, predict_y = torch.max(predict_out, 1)

print('prediction accuracy', accuracy_score(test_y.data, predict_y.data))

print('macro precision', precision_score(test_y.data, predict_y.data, average='macro'))
print('micro precision', precision_score(test_y.data, predict_y.data, average='micro'))
print('macro recall', recall_score(test_y.data, predict_y.data, average='macro'))
print('micro recall', recall_score(test_y.data, predict_y.data, average='micro'))

# Input to the model
x = torch.randn(1, 4, requires_grad=True)

# Export the model
torch.onnx.export(net,  # model being run
                  x,  # model input (or a tuple for multiple inputs)
                  "iris.onnx",  # where to save the model (can be a file or file-like object)
                  export_params=True,  # store the trained parameter weights inside the model file
                  opset_version=10,  # the ONNX version to export the model to
                  do_constant_folding=True,  # whether to execute constant folding for optimization
                  input_names=['input'],  # the model's input names
                  output_names=['output'],  # the model's output names
                  dynamic_axes={'input': {0: 'batch_size'},  # variable length axes
                                'output': {0: 'batch_size'}})
```

### Viewing the configuration file

The configuration for the custom endpoint is as follow:

```bash
%%bash
less onnx.yaml
```

The output shows configurations in YAML, in which you can see two steps in the pipeline, serving a model and post-processing to make it directly understood by a human.

```text
---
host: "localhost"
port: 0
protocol: "HTTP"
pipeline:
  steps:
  - '@type': "ONNX"
    modelUri: "iris.onnx"
    inputNames:
    - "input"
    outputNames:
    - "output"
  - '@type': "CLASSIFIER_OUTPUT"
    input_name: "output"
    labels:
      - Setosa
      - Versicolor
      - Virginica
```

### Starting the server

`konduit serve` can be used together with ID's name \(use any name\) and configuration file to start the server.

```bash
%%bash
konduit serve -id onnx-iris -c onnx.yaml -rwm -b
```

You'll get the following message.

```text
Starting konduit server...
Using classpath: /root/konduit/bin/../konduit.jar
INFO: Running command /root/miniconda/jre/bin/java -Dkonduit.logs.file.path=/root/.konduit-serving/command_logs/onnx-iris.log -Dlogback.configurationFile=/tmp/logback-run_command_80a3902b721c4c3f.xml -jar /root/konduit/bin/../konduit.jar run --instances 1 -s inference -c onnx.yaml -Dserving.id=onnx-iris
For server status, execute: 'konduit list'
For logs, execute: 'konduit logs onnx-iris'
```

To view the logs of the running server, use `konduit logs` commands.

```bash
%%bash
konduit logs onnx-iris -l 100
```

You'll be able to see the following from logging.

```text
15:01:50.334 [main] INFO  a.k.s.c.l.command.KonduitRunCommand - Processing configuration: /root/konduit/demos/1-pytorch-onnx-iris/onnx.yaml
.
.
.

####################################################################
#                                                                  #
#    |  /   _ \   \ |  _ \  |  | _ _| __ __|    |  /     |  /      #
#    . <   (   | .  |  |  | |  |   |     |      . <      . <       #
#   _|\_\ \___/ _|\_| ___/ \__/  ___|   _|     _|\_\ _) _|\_\ _)   #
#                                                                  #
####################################################################

.
.
.
15:01:50.919 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - Inference HTTP server is listening on host: 'localhost'
15:01:50.919 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - Inference HTTP server started on port 35761 with 2 pipeline steps

```

### Sending inputs

Now we can send our inputs through `cURL` for inference

```text
%%bash
konduit predict onnx-iris "{\"input\":[[5.1,3.5,1.4,0.2]]}"
```

So, the server will print out the output with a label.

```text
{
  "output" : [ [ 0.99312085, 0.0068791825, 6.1220806E-9 ] ],
  "prob" : 0.9931208491325378,
  "index" : 0,
  "label" : "Setosa"
}
```

Try once again with `--input-type` flag.

```text
%%bash
konduit predict onnx-iris --input-type multipart "input=[[5.1,3.5,1.4,0.2]]"
```

You'll see the output of the prediction.

```text
{
  "output" : [ [ 0.99312085, 0.0068791825, 6.1220806E-9 ] ],
  "prob" : 0.9931208491325378,
  "index" : 0,
  "label" : "Setosa"
}
```

### Stopping the server

Now after we're done with the server, we can stop it through the `konduit stop` command

```text
%%bash
konduit stop onnx-iris
```

You'll receive this once the server is terminated. 

```text
Stopping konduit server 'onnx-iris'
Application 'onnx-iris' terminated with status 0
```

Let's take a look at the following example, where we are going to give an image input and doing pre-processing step before fetching it into the model.

