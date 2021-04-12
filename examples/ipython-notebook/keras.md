---
description: Example of Keras framework with CUSTOM endpoints
---

# Keras

### Adding package to the classpath <a id="Adding-package-to-the-classpath"></a>

Firstly, we require to include the main package to the classpath so that the notebook can load every one of the important libraries from Konduit-Serving into the Jupyter notebook kernel.

```text
%classpath add jar ../../konduit.jar
```

{% hint style="info" %}
Classpaths can be considered similar to `site-packages` in the python ecosystem where each library that's to be imported to your code is loaded from.
{% endhint %}

### Viewing Python script code

We're creating a Keras model from scratch here and then converting that into .h5 \(HDF5\) format.

```text
%%bash
less train.py
```

You can view and follow through the code to get more information on how the model is trained.

```text
import tensorflow as tf

from keras.datasets import mnist


tensorflow_version = tf.__version__
print(tensorflow_version)

# Load data
train_data, test_data = mnist.load_data()
x_train, y_train = train_data
x_test, y_test = test_data

# Normalize
x_train = x_train / 255.0
x_test = x_test / 255.0


def get_model():
    inputs = tf.keras.layers.Input(shape=(28, 28), name="input_layer")
    x = tf.keras.layers.Flatten()(inputs)
    x = tf.keras.layers.Dense(200, activation="relu")(x)
    x = tf.keras.layers.Dense(100, activation="relu")(x)
    x = tf.keras.layers.Dense(60, activation="relu")(x)
    x = tf.keras.layers.Dense(30, activation="relu")(x)
    outputs = tf.keras.layers.Dense(10, activation="softmax", name="output_layer")(x)
    model = tf.keras.Model(inputs=inputs, outputs=outputs)
    model.compile(
        optimizer='sgd',
        loss='sparse_categorical_crossentropy',
        metrics=['accuracy']
    )

    return model


def train(epochs=8):
    model = get_model()
    model.fit(x_train, y_train, epochs=epochs)

    model.summary()

    print("\n\n---\n"
          "Inputs: {}".format(model.inputs))
    print("Outputs: {}\n---".format(model.outputs))

    return model


train(8).save("keras.h5", save_format="h5")
```

### Starting a server

Let's start to serve the model in Konduit-Serving. 

```text
%%bash
konduit serve -id keras-mnist -c keras.json -rwm -b
```

You'll be able to see a similar output like below.

```text
Starting konduit server...
Expected classpath: /root/konduit/bin/../konduit.jar
INFO: Running command /root/miniconda/jre/bin/java -Dkonduit.logs.file.path=/root/.konduit-serving/command_logs/keras-mnist.log -Dlogback.configurationFile=/tmp/logback-run_command_4c3da934e0334efc.xml -cp /root/konduit/bin/../konduit.jar ai.konduit.serving.cli.launcher.KonduitServingLauncher run --instances 1 -s inference -c keras.json -Dserving.id=keras-mnist
For server status, execute: 'konduit list'
For logs, execute: 'konduit logs keras-mnist'
```

List the active servers available by using `konduit list` command.

```text
%%bash
konduit list
```

You'll see the following list of the active Konduit servers.

```text
Listing konduit servers...

 #   | ID                             | TYPE       | URL                  | PID     | STATUS     
 1   | keras-mnist                    | inference  | localhost:33997      | 24142   | started  
```

View the logs for the last 100 lines for a given id by using the `konduit logs` command.

```text
%%bash
konduit logs keras-mnist --lines 100
```

Logging is printed on the notebook once you run the above command.

```text
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
15:15:13.074 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - Inference HTTP server is listening on host: 'localhost'
15:15:13.074 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - Inference HTTP server started on port 33997 with 4 pipeline steps

```

### Feeding an input to test the model

View the test image before testing the model

```text
%%html
<img src="test-image.jpg" alt="title">
```

We're going to use available test image in this directory.

![](../../.gitbook/assets/test-image.jpg)

Let's predict the output from the server with the above input image.

```text
%%bash
konduit predict keras-mnist -it multipart 'image=@test-image.jpg'
```

The output of classification:

* `output_layer` : probabilities of possible outputs
* `prob` : highest probability
* `index` : the location of an item in an array
* `label` : label of image classification 

```text
{
  "output_layer" : [ [ 9.0376153E-7, 1.0595608E-8, 1.3115231E-5, 0.44657645, 6.748624E-12, 0.5524258, 1.848306E-7, 2.7652052E-9, 9.76023E-4, 7.5933513E-6 ] ],
  "prob" : 0.5524258017539978,
  "index" : 5,
  "label" : "5"
}
```

### Stopping the server

Once we're finished with the server, we can stop using the `konduit stop` command following the id's server.

```text
%%bash
konduit stop keras-mnist
```

You'll be able to see the following message.

```text
Stopping konduit server 'keras-mnist'
Application 'keras-mnist' terminated with status 0
```

