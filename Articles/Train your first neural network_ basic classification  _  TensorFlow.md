Train your first neural network: basic classification  |  TensorFlow

This guide trains a neural network model to classify images of clothing, like sneakers and shirts. It's okay if you don't understand all the details, this is a fast-paced overview of a complete TensorFlow program with the details explained as we go.

This guide uses [tf.keras](https://www.tensorflow.org/guide/keras), a high-level API to build and train models in TensorFlow.

    # TensorFlow and tf.kerasimport tensorflow as tffrom tensorflow import keras# Helper librariesimport numpy as npimport matplotlib.pyplot as pltprint(tf.__version__)

1.12.0

## [](https://www.tensorflow.org/tutorials/keras/#top_of_page)Import the Fashion MNIST dataset

This guide uses the [Fashion MNIST](https://github.com/zalandoresearch/fashion-mnist) dataset which contains 70,000 grayscale images in 10 categories. The images show individual articles of clothing at low resolution (28 by 28 pixels), as seen here:

Fashion MNIST is intended as a drop-in replacement for the classic [MNIST](http://yann.lecun.com/exdb/mnist/) dataset—often used as the "Hello, World" of machine learning programs for computer vision. The MNIST dataset contains images of handwritten digits (0, 1, 2, etc) in an identical format to the articles of clothing we'll use here.

This guide uses Fashion MNIST for variety, and because it's a slightly more challenging problem than regular MNIST. Both datasets are relatively small and are used to verify that an algorithm works as expected. They're good starting points to test and debug code.

We will use 60,000 images to train the network and 10,000 images to evaluate how accurately the network learned to classify images. You can access the Fashion MNIST directly from TensorFlow, just import and load the data:

    fashion_mnist = keras.datasets.fashion_mnist(train_images, train_labels), (test_images, test_labels) = fashion_mnist.load_data()

Downloading data from https://storage.googleapis.com/tensorflow/tf-keras-datasets/train-labels-idx1-ubyte.gz
32768/29515 \[=================================\] - 0s 0us/step
Downloading data from https://storage.googleapis.com/tensorflow/tf-keras-datasets/train-images-idx3-ubyte.gz
26427392/26421880 \[==============================\] - 1s 0us/step
Downloading data from https://storage.googleapis.com/tensorflow/tf-keras-datasets/t10k-labels-idx1-ubyte.gz
8192/5148 \[===============================================\] - 0s 0us/step
Downloading data from https://storage.googleapis.com/tensorflow/tf-keras-datasets/t10k-images-idx3-ubyte.gz
4423680/4422102 \[==============================\] - 0s 0us/step

Loading the dataset returns four NumPy arrays:

*   The `train_images` and `train_labels` arrays are the _training set_—the data the model uses to learn.
*   The model is tested against the _test set_, the `test_images`, and `test_labels` arrays.

The images are 28x28 NumPy arrays, with pixel values ranging between 0 and 255. The _labels_ are an array of integers, ranging from 0 to 9. These correspond to the _class_ of clothing the image represents:

| Label | Class |
| --- | --- |
| 0   | T-shirt/top |
| 1   | Trouser |
| 2   | Pullover |
| 3   | Dress |
| 4   | Coat |
| 5   | Sandal |
| 6   | Shirt |
| 7   | Sneaker |
| 8   | Bag |
| 9   | Ankle boot |

Each image is mapped to a single label. Since the _class names_ are not included with the dataset, store them here to use later when plotting the images:

    class_names = ['T-shirt/top', 'Trouser', 'Pullover', 'Dress', 'Coat',                'Sandal', 'Shirt', 'Sneaker', 'Bag', 'Ankle boot']

## [](https://www.tensorflow.org/tutorials/keras/#top_of_page)Explore the data

Let's explore the format of the dataset before training the model. The following shows there are 60,000 images in the training set, with each image represented as 28 x 28 pixels:

    train_images.shape

(60000, 28, 28)

Likewise, there are 60,000 labels in the training set:

    len(train_labels)

60000

Each label is an integer between 0 and 9:

    train_labels

array(\[9, 0, 0, ..., 3, 0, 5\], dtype=uint8)

There are 10,000 images in the test set. Again, each image is represented as 28 x 28 pixels:

    test_images.shape

(10000, 28, 28)

And the test set contains 10,000 images labels:

    len(test_labels)

10000

## [](https://www.tensorflow.org/tutorials/keras/#top_of_page)Preprocess the data

The data must be preprocessed before training the network. If you inspect the first image in the training set, you will see that the pixel values fall in the range of 0 to 255:

    plt.figure()plt.imshow(train_images[0])plt.colorbar()plt.grid(False)

![png](../_resources/a9393532cbec4e748b060f7356725f8a.png)

We scale these values to a range of 0 to 1 before feeding to the neural network model. For this, cast the datatype of the image components from an integer to a float, and divide by 255. Here's the function to preprocess the images:

It's important that the _training set_ and the _testing set_ are preprocessed in the same way:

    train_images = train_images / 255.0test_images = test_images / 255.0

Display the first 25 images from the _training set_ and display the class name below each image. Verify that the data is in the correct format and we're ready to build and train the network.

    plt.figure(figsize=(10,10))for i in range(25):    plt.subplot(5,5,i+1)    plt.xticks([])    plt.yticks([])    plt.grid(False)    plt.imshow(train_images[i], cmap=plt.cm.binary)    plt.xlabel(class_names[train_labels[i]])

![png](../_resources/8240fe4d066b4eee9f68377d573db4bb.png)

## [](https://www.tensorflow.org/tutorials/keras/#top_of_page)Build the model

Building the neural network requires configuring the layers of the model, then compiling the model.

### Setup the layers

The basic building block of a neural network is the _layer_. Layers extract representations from the data fed into them. And, hopefully, these representations are more meaningful for the problem at hand.

Most of deep learning consists of chaining together simple layers. Most layers, like [`tf.keras.layers.Dense`](https://www.tensorflow.org/api_docs/python/tf/keras/layers/Dense), have parameters that are learned during training.

    model = keras.Sequential([    keras.layers.Flatten(input_shape=(28, 28)),    keras.layers.Dense(128, activation=tf.nn.relu),    keras.layers.Dense(10, activation=tf.nn.softmax)])

The first layer in this network, [`tf.keras.layers.Flatten`](https://www.tensorflow.org/api_docs/python/tf/keras/layers/Flatten), transforms the format of the images from a 2d-array (of 28 by 28 pixels), to a 1d-array of 28 * 28 = 784 pixels. Think of this layer as unstacking rows of pixels in the image and lining them up. This layer has no parameters to learn; it only reformats the data.

After the pixels are flattened, the network consists of a sequence of two [`tf.keras.layers.Dense`](https://www.tensorflow.org/api_docs/python/tf/keras/layers/Dense) layers. These are densely-connected, or fully-connected, neural layers. The first `Dense` layer has 128 nodes (or neurons). The second (and last) layer is a 10-node _softmax_ layer—this returns an array of 10 probability scores that sum to 1. Each node contains a score that indicates the probability that the current image belongs to one of the 10 classes.

### Compile the model

Before the model is ready for training, it needs a few more settings. These are added during the model's _compile_ step:

*   _Loss function_ —This measures how accurate the model is during training. We want to minimize this function to "steer" the model in the right direction.
*   _Optimizer_ —This is how the model is updated based on the data it sees and its loss function.
*   _Metrics_ —Used to monitor the training and testing steps. The following example uses _accuracy_, the fraction of the images that are correctly classified.

    model.compile(optimizer=tf.train.AdamOptimizer(),               loss='sparse_categorical_crossentropy',              metrics=['accuracy'])

## [](https://www.tensorflow.org/tutorials/keras/#top_of_page)Train the model

Training the neural network model requires the following steps:

1.  Feed the training data to the model—in this example, the `train_images` and `train_labels` arrays.
2.  The model learns to associate images and labels.
3.  We ask the model to make predictions about a test set—in this example, the `test_images` array. We verify that the predictions match the labels from the `test_labels` array.

To start training, call the `model.fit` method—the model is "fit" to the training data:

    model.fit(train_images, train_labels, epochs=5)

Epoch 1/5
60000/60000 \[==============================\] - 5s 85us/step - loss: 0.4948 - acc: 0.8269
Epoch 2/5
60000/60000 \[==============================\] - 5s 84us/step - loss: 0.3731 - acc: 0.8659
Epoch 3/5
60000/60000 \[==============================\] - 5s 83us/step - loss: 0.3350 - acc: 0.8777
Epoch 4/5
60000/60000 \[==============================\] - 5s 84us/step - loss: 0.3104 - acc: 0.8850
Epoch 5/5
60000/60000 \[==============================\] - 5s 87us/step - loss: 0.2921 - acc: 0.8935

As the model trains, the loss and accuracy metrics are displayed. This model reaches an accuracy of about 0.88 (or 88%) on the training data.

## [](https://www.tensorflow.org/tutorials/keras/#top_of_page)Evaluate accuracy

Next, compare how the model performs on the test dataset:

    test_loss, test_acc = model.evaluate(test_images, test_labels)print('Test accuracy:', test_acc)

10000/10000 \[==============================\] - 1s 50us/step
Test accuracy: 0.8712

It turns out, the accuracy on the test dataset is a little less than the accuracy on the training dataset. This gap between training accuracy and test accuracy is an example of _overfitting_. Overfitting is when a machine learning model performs worse on new data than on their training data.

## [](https://www.tensorflow.org/tutorials/keras/#top_of_page)Make predictions

With the model trained, we can use it to make predictions about some images.

    predictions = model.predict(test_images)

Here, the model has predicted the label for each image in the testing set. Let's take a look at the first prediction:

    predictions[0]

array(\[1.1901825e-04, 1.1664556e-06, 2.7502297e-06, 5.7167264e-05,
       4.2034894e-06, 2.8958581e-02, 5.0696172e-04, 4.7597680e-02,
       2.4416966e-03, 9.2031080e-01\], dtype=float32)

A prediction is an array of 10 numbers. These describe the "confidence" of the model that the image corresponds to each of the 10 different articles of clothing. We can see which label has the highest confidence value:

    np.argmax(predictions[0])

9

So the model is most confident that this image is an ankle boot, or `class_names[9]`. And we can check the test label to see this is correct:

    test_labels[0]

9

We can graph this to look at the full set of 10 channels

    def plot_image(i, predictions_array, true_label, img):  predictions_array, true_label, img = predictions_array[i], true_label[i], img[i]  plt.grid(False)  plt.xticks([])  plt.yticks([])    plt.imshow(img, cmap=plt.cm.binary)  predicted_label = np.argmax(predictions_array)  if predicted_label == true_label:    color = 'blue'  else:    color = 'red'    plt.xlabel("{} {:2.0f}% ({})".format(class_names[predicted_label],                                100*np.max(predictions_array),                                class_names[true_label]),                                color=color)def plot_value_array(i, predictions_array, true_label):  predictions_array, true_label = predictions_array[i], true_label[i]  plt.grid(False)  plt.xticks([])  plt.yticks([])  thisplot = plt.bar(range(10), predictions_array, color="#777777")  plt.ylim([0, 1])   predicted_label = np.argmax(predictions_array)   thisplot[predicted_label].set_color('red')  thisplot[true_label].set_color('blue')

Let's look at the 0th image, predictions, and prediction array.

    i = 0plt.figure(figsize=(6,3))plt.subplot(1,2,1)plot_image(i, predictions, test_labels, test_images)plt.subplot(1,2,2)plot_value_array(i, predictions,  test_labels)

![png](../_resources/6eb6c039a62948f0b769d87c25174fc9.png)

    i = 12plt.figure(figsize=(6,3))plt.subplot(1,2,1)plot_image(i, predictions, test_labels, test_images)plt.subplot(1,2,2)plot_value_array(i, predictions,  test_labels)

![png](../_resources/ce4aa154ee47414297e6f7439f3c47e4.png)

Let's plot several images with their predictions. Correct prediction labels are blue and incorrect prediction labels are red. The number gives the percent (out of 100) for the predicted label. Note that it can be wrong even when very confident.

    # Plot the first X test images, their predicted label, and the true label# Color correct predictions in blue, incorrect predictions in rednum_rows = 5num_cols = 3num_images = num_rows*num_colsplt.figure(figsize=(2*2*num_cols, 2*num_rows))for i in range(num_images):  plt.subplot(num_rows, 2*num_cols, 2*i+1)  plot_image(i, predictions, test_labels, test_images)  plt.subplot(num_rows, 2*num_cols, 2*i+2)  plot_value_array(i, predictions, test_labels)

![png](../_resources/8d9775a97d7d431e802ff5dc67216852.png)

Finally, use the trained model to make a prediction about a single image.

    # Grab an image from the test datasetimg = test_images[0]print(img.shape)

(28, 28)

[`tf.keras`](https://www.tensorflow.org/api_docs/python/tf/keras) models are optimized to make predictions on a _batch_, or collection, of examples at once. So even though we're using a single image, we need to add it to a list:

    # Add the image to a batch where it's the only member.img = (np.expand_dims(img,0))print(img.shape)

(1, 28, 28)

Now predict the image:

    predictions_single = model.predict(img)print(predictions_single)

\[\[1.1901846e-04 1.1664511e-06 2.7502319e-06 5.7167257e-05 4.2034926e-06
  2.8958611e-02 5.0696166e-04 4.7597684e-02 2.4416940e-03 9.2031068e-01\]\]

    plot_value_array(0, predictions_single, test_labels)_ = plt.xticks(range(10), class_names, rotation=45)

![png](../_resources/e28ca4a0bc1e48b5bcefb9c0f06f96d3.png)

`model.predict` returns a list of lists, one for each image in the batch of data. Grab the predictions for our (only) image in the batch:

    np.argmax(predictions_single[0])

9

And, as before, the model predicts a label of 9.

    #@title MIT License## Copyright (c) 2017 François Chollet## Permission is hereby granted, free of charge, to any person obtaining a# copy of this software and associated documentation files (the "Software"),# to deal in the Software without restriction, including without limitation# the rights to use, copy, modify, merge, publish, distribute, sublicense,# and/or sell copies of the Software, and to permit persons to whom the# Software is furnished to do so, subject to the following conditions:## The above copyright notice and this permission notice shall be included in# all copies or substantial portions of the Software.## THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL# THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING# FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER# DEALINGS IN THE SOFTWARE.