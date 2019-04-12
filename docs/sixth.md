![TensorFlow](/images/robo16.jpeg)

# Into the deep space

## Localized Computation

Lately we have witness how Internet-of-Things (IoT) is making our physical environment and infrastructures smarter.  What we also need to know is that IoT is changing the way we perceive computing and communication.  

And since billions of IoT devices will get activated in the upcoming years, we need to consider  better security, lower costs and higher performance applications - and the answer to the problem is: localized computation. 

The hardware is there or almost there(when we talk about specialized microchips) and the software that enables computations on such small devices is very fast emerging from different agile minds. Few of these minds I’ve met at Plumer.ai. 

They’ve put together Larq, which, to quote: 
> is an open source machine learning library for training Quantized Neural Networks (QNNs) with extremely low precision weights and activations (e.g. 1-bit). Existing Deep Neural Networks tend to be large, slow and power-hungry, prohibiting many applications in resource-constrained environments. Larq is designed to provide an easy to use, composable way to train QNNs (e.g. Binarized Neural Networks) based on the 'tf.keras' interface.

----------------

### Now let's give it a try: 

First things first: pip install larq

![TensorFlow](/images/plumerai1.png)

-------------------

Will run it on the well known data sets: MNIST

    import tensorflow as tf
    import larq as lq

    (train_images, train_labels), (test_images, test_labels) = tf.keras.datasets.mnist.load_data()

    train_images = train_images.reshape((60000, 28, 28, 1))
    test_images = test_images.reshape((10000, 28, 28, 1))

    # Normalize pixel values to be between -1 and 1
    train_images, test_images = train_images / 127.5 - 1, test_images / 127.5 - 1

    kwargs = dict(input_quantizer="ste_sign", kernel_quantizer="ste_sign", kernel_constraint="weight_clip")

    model = tf.keras.models.Sequential()

    model.add(lq.layers.QuantConv2D(32, (3, 3), kernel_quantizer="ste_sign", kernel_constraint="weight_clip", use_bias=False, input_shape=(28, 28, 1)))
    model.add(tf.keras.layers.MaxPooling2D((2, 2)))
    model.add(tf.keras.layers.BatchNormalization(scale=False))

    model.add(lq.layers.QuantConv2D(64, (3, 3), use_bias=False, **kwargs))
    model.add(tf.keras.layers.MaxPooling2D((2, 2)))
    model.add(tf.keras.layers.BatchNormalization(scale=False))

    model.add(lq.layers.QuantConv2D(64, (3, 3), use_bias=False, **kwargs))
    model.add(tf.keras.layers.BatchNormalization(scale=False))
    model.add(tf.keras.layers.Flatten())

    model.add(lq.layers.QuantDense(64, use_bias=False, **kwargs))
    model.add(tf.keras.layers.BatchNormalization(scale=False))
    model.add(lq.layers.QuantDense(10, use_bias=False, **kwargs))
    model.add(tf.keras.layers.BatchNormalization(scale=False))
    model.add(tf.keras.layers.Activation("softmax"))
    
---------------------------------

    lq.models.summary(model)
    
            Layer                  Outputs             # 1-bit    # 32-bit
        ---------------------  ----------------  ---------  ----------
        quant_conv2d           (-1, 26, 26, 32)        288           0
        max_pooling2d          (-1, 13, 13, 32)          0           0
        batch_normalization    (-1, 13, 13, 32)          0          96
        quant_conv2d_1         (-1, 11, 11, 64)      18432           0
        max_pooling2d_1        (-1, 5, 5, 64)            0           0
        batch_normalization_1  (-1, 5, 5, 64)            0         192
        quant_conv2d_2         (-1, 3, 3, 64)        36864           0
        batch_normalization_2  (-1, 3, 3, 64)            0         192
        flatten                (-1, 576)                 0           0
        quant_dense            (-1, 64)              36864           0
        batch_normalization_3  (-1, 64)                  0         192
        quant_dense_1          (-1, 10)                640           0
        batch_normalization_4  (-1, 10)                  0          30
        activation             (-1, 10)                  0           0
        Total                                        93088         702

        Total params: 93790
        Trainable params: 93322
        Non-trainable params: 468

---------------------------------

    model.compile(optimizer='adam',
                  loss='sparse_categorical_crossentropy',
                  metrics=['accuracy'])

    model.fit(train_images, train_labels, batch_size=64, epochs=6)

    test_loss, test_acc = model.evaluate(test_images, test_labels)

-------------------------------

Epoch 1/6
60000/60000 [==============================] - 91s 2ms/step - loss: 0.6450 - acc: 0.9082
Epoch 2/6
60000/60000 [==============================] - 88s 1ms/step - loss: 0.4708 - acc: 0.9626
Epoch 3/6
60000/60000 [==============================] - 77s 1ms/step - loss: 0.4449 - acc: 0.9693
Epoch 4/6
60000/60000 [==============================] - 88s 1ms/step - loss: 0.4326 - acc: 0.9735
Epoch 5/6
60000/60000 [==============================] - 142s 2ms/step - loss: 0.4272 - acc: 0.9758
Epoch 6/6
60000/60000 [==============================] - 107s 2ms/step - loss: 0.4243 - acc: 0.9771
10000/10000 [==============================] - 3s 344us/step

----------------------------------

    print(f"Test accuracy {test_acc * 100:.2f} %") 

Test accuracy 93.30 %

---------------------------------


#### Resources:

- [An Open Source Machine Learning Library for Training Binarized Neural Networks](https://github.com/DanielMoraite/larq)
- [Plumerai on Github](https://plumerai.github.io/larq/)
- [Binarized Convolutional Neural Networks](https://plumerai.github.io/larq/examples/mnist/)
- [Keras Model](https://www.tensorflow.org/alpha/guide/keras/overview#sequential_model)





----------------
## Coming soon: 

> Just a few random projects: 

* Object Detection - and the story of how I`ve spent quite considerable time to tag objects in a video created by me.
* OpenCV, etc. - playing with dogs photos is the best. 

----------------
----------------
