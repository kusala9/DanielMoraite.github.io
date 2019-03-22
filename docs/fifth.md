![TensorFlow](/images/robo4.png)

# Tensorflow and NN


## Keras

Let's have a look at Python library Keras since I find it useful for later projects, as in detecting ships in satellite imagery.

> To quote: Keras is a high-level neural networks API, written in Python and capable of running on top of TensorFlow, CNTK, or Theano. It was developed with a focus on enabling fast experimentation. Being able to go from idea to result with the least possible delay is key to doing good research.


First make sure you have your virtual environment activated and Tensorflow and Keras installed: 'pip install keras'.  

      import keras
      import matplotlib.pyplot as plt
      import numpy as np
      keras.__version__
      '2.2.4'   # my version

### Data set

Will pick the classic MNIST dataset example that will help us learn to classify hand-written digits. 
- grayscale images of handwritten digits (28 pixels by 28 pixels)
- 10 categories (0 to 9) 
- a set of 60,000 training images and 10,000 test images

      # MNIST dataset comes pre-loaded in Keras, in the form of a set of four Numpy arrays:

      from keras.datasets import mnist

      (train_images, train_labels), (test_images, test_labels) = mnist.load_data()

      imgs = np.concatenate((train_images, test_images), axis = 0 )

Let's just practice a little bit with arrays and interogate and plot them:

    imgs.shape
    (70000, 28, 28)

    type(train_images)
    numpy.ndarray

    train_images.shape 
    (60000, 28, 28)

If you are curious to visualize one of the letters: 

    plt.imshow(train_images[1, :, :])  # the index position is the first entry: let`s pick the second photo)

![TensorFlow](/images/keras1.png)

Training:

Will use the train_images and train_labels to train the model. The test_images and test_labels will be used for testing the model. Images are encoded as Numpy arrays and the labels are an array of digits ranging from 0 to 9.

Let's have a look at the training data:

      train_labels[1]
      0
      train_images.shape
      (60000, 28, 28)
      len(train_labels)
      60000
      train_labels
      array([5, 0, 4, ..., 5, 6, 8], dtype=uint8)

Let's have a look at the test data:
  
      test_images.shape
      len(test_labels)
      10000
      test_labels
      array([7, 2, 1, ..., 4, 5, 6], dtype=uint8)

Workflow: will input into the neural network the training data, train_images and train_labels, which in turn will learn to associate images and labels. Then will request the network to produce predictions for test_images, and we will verify if they match the labels from test_labels.


#### Building the network

      from keras import models
      from keras import layers

      network = models.Sequential()

      network
      <keras.engine.sequential.Sequential at 0xb3873a6a0>

      network.add(layers.Dense(512, activation='relu', input_shape=(28 * 28,)))
      network.add(layers.Dense(10, activation='softmax'))


The "layer" is a data-processing module which can be conceived as a "filter" for data. Layers extract representations out of the data fed into them. Deep learning could mainly consist of chaining together simple layers which will implement a form of progressive "data distillation". A deep learning model is made of a succession of increasingly refined data filters which are the "layers".

The network consists of a sequence of two Dense layers. The second layer is a 10-way "softmax" layer, which will return an array of 10 probability scores (summing to 1). Each score will be the probability that the current digit image belongs to one of our 10 digit classes.

To ready the network for training, as part of "compilation" step, will set:

- An optimizer: this is the mechanism through which the network will update itself based on the data it sees and its loss function.
- Metrics to monitor during training and testing. Will be interested only in the fraction of the images that were correctly classified: 'accuracy'.
- Last but not least: the loss function: the is how the network will be able to measure how good a job it is doing on its training data, and thus how it will be able to steer itself in the right direction.

Actually my dear friend Laurens, who was guiding and mentoring me, took me through the simple math of the loss function:

![TensorFlow](/images/LossFunction.png)

Which was pretty useful to cement the basics. 

![TensorFlow](/images/LossFunctionCalc.JPG) 

      network.compile(optimizer='rmsprop',
                loss='categorical_crossentropy',
                metrics=['accuracy'])


To be continued... very soon.. 






---------------------------

#### Sources: 

[Keras](https://keras.io)

[Deep Learning with python](https://www.manning.com/books/deep-learning-with-python?a_aid=keras&a_bid=76564dff)

[A first look at a neural network](https://github.com/fchollet/deep-learning-with-python-notebooks/blob/master/2.1-a-first-look-at-a-neural-network.ipynb)

[How to use deep learning on satellite imagery — Playing with the loss function](https://medium.com/earthcube-stories/techsecret-how-to-use-deep-learning-on-satellite-imagery-episode-1-playing-with-the-loss-8fc05c90a63a)

---------------

###### Coming soon: 

> some of my favorite projects.

----------------
----------------
