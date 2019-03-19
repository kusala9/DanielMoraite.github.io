![TensorFlow](/images/robo4.png)

# Tensorflow and NN


## Keras

Let have a look at Python library Keras since I find it useful for later projects, as in detecting ships in satellite imagery.


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


      
     

#### Sources: 

[Keras](https://keras.io)

[Deep Learning with python](https://www.manning.com/books/deep-learning-with-python?a_aid=keras&a_bid=76564dff)

[A first look at a neural network](githhub ... )

---------------

###### Coming soon: 

> some of my favorite projects.

----------------
----------------
