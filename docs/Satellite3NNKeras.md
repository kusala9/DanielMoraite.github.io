![# Welcome to my adventure](/images/Sat2.jpg)

# Ships in Satellite Imagery

-------------------------

## Classify ships in San Franciso Bay using Planet satellite imagery


Satellite imagery is one of the most abundant data sources that a data scientist can play with. 
This is part for which I have chosen to look into it in the first place, since it reduce my work of colecting data or even collateral research for personal projects. 
There is a downside to it as well: my personal computer storage size and computational power(.. a MacBook Air). I will soon look into AWS Amazon Web Services to compensate for it. 

Meanwhile I have discovered a data set that is quite small and I can run it on my machine: uses one of my favorite providers, Planet satellite imagery.  

About the data:
- includes 4000 80x80 RGB images labeled with either a "ship" or "no-ship" classification, valued 1 or 0.  
- image chips are orthorectified to a 3 meter pixel size
- dataset as .png image chips, image filename follows a specific format: {label} __ {scene id} __ {longitude} _ {latitude}.png
- longitude_latitude: longitude and latitude coordinates of the image center point
- dataset is also distributed as a JSON formatted text file, contains: data, label, scene_ids, and location lists
- pixel value data for individual images is stored as a list of 19200 integers: first 6400 contain the red channel, next 6400 the green, and last 6400 the blue. 
- list values at index i in labels, scene_ids, and locations each correspond to the i-th image in the data list
- class labels: "ship" class includes 1000 images, near-centered on the body of a single ship. 
             "no-ship" class includes 3000 images, 1/3 are a random sampling of different landcover features. - that do not include any portion of an ship. 
             next 1/3 are "partial ships", and 1/3 are images that have previously been mislabeled by machine learning models(because of strong linear features). 

What we want to achieve:
Detect the location of ships in satellite images, which can be applied to solving problems as: monitoring port activity and supply chain analysis.

----------------------------------

### First Part 
   
#### Reading and Preparing Data

We make sure we import all the libraries and modules that we need, apart from regular Keras: Sequential, Dense, Flatten, Activation and Dropout will also use Conv2D and MaxPooling2D (see full notebook end of article).
Now let`s download and study the data set: 

      f = open(r'../ships-in-satellite-imagery/shipsnet.json')
      dataset = json.load(f)
      f.close()
      input_data = np.array(dataset['data']).astype('uint8')
      output_data = np.array(dataset['labels']).astype('uint8')
      input_data.shape
      (4000, 19200)

      # and since I was currios to see how the tupple of arrays of arrays look like:
      input_data
      array([[ 82,  89,  91, ...,  86,  88,  89],
             [ 76,  75,  67, ...,  54,  57,  58],
             [125, 127, 129, ..., 111, 109, 115],
             ...,
             [171, 135, 118, ...,  95,  95,  85],
             [ 85,  90,  94, ...,  96,  95,  89],
             [122, 122, 126, ...,  51,  46,  69]], dtype=uint8)
      # now we realize that this is not a photo format that we can visualize, in order to be able to read an image we need to reshape the array/input_data:
      n_spectrum = 3 # the number of color chanels: RGB 
      weight = 80
      height = 80
      X = input_data.reshape([-1, n_spectrum, weight, height])
      X[0].shape
     
      # let`s pick one channel 
      pic = X[3]

      red_spectrum = pic[0]
      green_spectrum = pic[1]
      blue_spectrum = pic[2] 

and the fun part: ploting the photo in all 3 chanels: 

    plt.figure(2, figsize = (5*3, 5*1))
    plt.set_cmap('jet')

    #show each channel
    plt.subplot(1, 3, 1)
    plt.imshow(red_spectrum)

    plt.subplot(1, 3, 2)
    plt.imshow(green_spectrum)

    plt.subplot(1, 3, 3)
    plt.imshow(blue_spectrum)

    plt.show()

![# Welcome to my adventure](/images/ships1.png)

Just don`t get discouraged if some of the photos as X[0] might have all the 3 bands RGB the same, just try another one X[3]. 

A vector of 4000 elements is our output:

      output_data
      array([1, 1, 1, ..., 0, 0, 0], dtype=uint8)
      np.bincount(output_data)
      array([3000, 1000]) 
      
Vector contains of 3000 zeros and 1000 units = 1000 images are tagged with "ship" and 3000 images with "not ship".


#### Preparing the data for keras

First categorically encoding the labels:

      # output encoding
      y = np_utils.to_categorical(output_data, 2)

Second shuffle all indexes:

      indexes = np.arange(4000)
      np.random.shuffle(indexes)

Pick X_train, y_train:

      X_train = X[indexes].transpose([0,2,3,1])
      y_train = y[indexes]

And of course normalization:

      X_train = X_train / 255 
      # images are type uint8 with values in the [0, 255] interval and we would like to contain values between 0 and 1

-------------------------------

### Second Part 

#### Training the model/ neural network 

      np.random.seed(42)

      # network design
      model = Sequential()

      model.add(Conv2D(32, (3, 3), padding='same', input_shape=(80, 80, 3), activation='relu'))
      model.add(MaxPooling2D(pool_size=(2, 2))) #40x40
      model.add(Dropout(0.25))

      model.add(Conv2D(32, (3, 3), padding='same', activation='relu'))
      model.add(MaxPooling2D(pool_size=(2, 2))) #20x20
      model.add(Dropout(0.25))

      model.add(Conv2D(32, (3, 3), padding='same', activation='relu'))
      model.add(MaxPooling2D(pool_size=(2, 2))) #10x10
      model.add(Dropout(0.25))

      model.add(Conv2D(32, (10, 10), padding='same', activation='relu'))
      model.add(MaxPooling2D(pool_size=(2, 2))) #5x5
      model.add(Dropout(0.25))

      model.add(Flatten())
      model.add(Dense(512, activation='relu'))
      model.add(Dropout(0.5))

      model.add(Dense(2, activation='softmax'))

> for details on: relu, softmax and dropout, please see my previous article on [Tensorflow Keras](https://danielmoraite.github.io/docs/fifth.html)


      # optimization setup
      sgd = SGD(lr=0.01, momentum=0.9, nesterov=True)
      model.compile(
          loss='categorical_crossentropy',
          optimizer=sgd,
          metrics=['accuracy'])

      # training
      model.fit(
          X_train, 
          y_train,
          batch_size=32, # 32 photos at once
          epochs=18,
          validation_split=0.2,
          shuffle=True,
          verbose=2)

Get and grab a nice cup of tea or matcha cause this might take a few minutes (or at least it took a few on my machine as you can see bellow):

      Train on 3200 samples, validate on 800 samples
      Epoch 1/18
       - 67s - loss: 0.4076 - acc: 0.8219 - val_loss: 0.2387 - val_acc: 0.9025
      Epoch 2/18
       - 89s - loss: 0.2227 - acc: 0.9034 - val_loss: 0.1767 - val_acc: 0.9150
      Epoch 3/18
       - 74s - loss: 0.1809 - acc: 0.9278 - val_loss: 0.1481 - val_acc: 0.9425
      Epoch 4/18
       - 72s - loss: 0.1444 - acc: 0.9428 - val_loss: 0.1201 - val_acc: 0.9600
      Epoch 5/18
       - 48s - loss: 0.1334 - acc: 0.9522 - val_loss: 0.1126 - val_acc: 0.9513
      Epoch 6/18
       - 42s - loss: 0.1221 - acc: 0.9591 - val_loss: 0.0879 - val_acc: 0.9637
      Epoch 7/18
       - 40s - loss: 0.1068 - acc: 0.9625 - val_loss: 0.0846 - val_acc: 0.9738
      Epoch 8/18
       - 45s - loss: 0.0820 - acc: 0.9716 - val_loss: 0.0808 - val_acc: 0.9675
      Epoch 9/18
       - 41s - loss: 0.0851 - acc: 0.9728 - val_loss: 0.0626 - val_acc: 0.9838
      Epoch 10/18
       - 40s - loss: 0.0799 - acc: 0.9709 - val_loss: 0.0662 - val_acc: 0.9762
      Epoch 11/18
       - 42s - loss: 0.0672 - acc: 0.9800 - val_loss: 0.0599 - val_acc: 0.9812
      Epoch 12/18
       - 41s - loss: 0.0500 - acc: 0.9813 - val_loss: 0.0729 - val_acc: 0.9738
      Epoch 13/18
       - 42s - loss: 0.0570 - acc: 0.9784 - val_loss: 0.0625 - val_acc: 0.9788
      Epoch 14/18
       - 43s - loss: 0.0482 - acc: 0.9828 - val_loss: 0.0526 - val_acc: 0.9775
      Epoch 15/18
       - 42s - loss: 0.0510 - acc: 0.9822 - val_loss: 0.0847 - val_acc: 0.9762
      Epoch 16/18
       - 44s - loss: 0.0440 - acc: 0.9841 - val_loss: 0.0615 - val_acc: 0.9800
      Epoch 17/18
       - 41s - loss: 0.0411 - acc: 0.9862 - val_loss: 0.0559 - val_acc: 0.9775
      Epoch 18/18
       - 42s - loss: 0.0515 - acc: 0.9834 - val_loss: 0.0597 - val_acc: 0.9775
      Out[31]:
      <keras.callbacks.History at 0xb47f7bfd0>

> for details on: categorical_crossentropy, 'accuracy', as well the Loss Function please see my previous [article on NN](https://danielmoraite.github.io/docs/fifth.html)
 

-------------------------------

### Third Part

#### Apply the model and Search on the image

      # download image
      image = Image.open(r'../ships-in-satellite-imagery/scenes/sfbay_1.png')
      pix = image.load()
      
'plt.imshow(image)' if you want to have a quick look, though in order to be able to properly make use of it, we need to create a vector:

      n_spectrum = 3
      width = image.size[0]
      height = image.size[1]

      # creat vector
      picture_vector = []
      for chanel in range(n_spectrum):
          for y in range(height):
              for x in range(width):
                  picture_vector.append(pix[x, y][chanel])

      picture_vector = np.array(picture_vector).astype('uint8')
      picture_tensor = picture_vector.reshape([n_spectrum, height, width]).transpose(1, 2, 0)

      plt.figure(1, figsize = (15, 30))

      plt.subplot(3, 1, 1)
      plt.imshow(picture_tensor)

      plt.show()
      
![# Welcome to my adventure](/images/ships2.png)

#### Now let's search for ships on the image

      picture_tensor = picture_tensor.transpose(2,0,1)

      # Search on the image

      def cutting(x, y):
          area_study = np.arange(3*80*80).reshape(3, 80, 80)
          for i in range(80):
              for j in range(80):
                  area_study[0][i][j] = picture_tensor[0][y+i][x+j]
                  area_study[1][i][j] = picture_tensor[1][y+i][x+j]
                  area_study[2][i][j] = picture_tensor[2][y+i][x+j]
          area_study = area_study.reshape([-1, 3, 80, 80])
          area_study = area_study.transpose([0,2,3,1])
          area_study = area_study / 255
          sys.stdout.write('\rX:{0} Y:{1}  '.format(x, y))
          return area_study

      def not_near(x, y, s, coordinates):
          result = True
          for e in coordinates:
              if x+s > e[0][0] and x-s < e[0][0] and y+s > e[0][1] and y-s < e[0][1]:
                  result = False
          return result
        
      def show_ship(x, y, acc, thickness=5):   
          for i in range(80):
              for ch in range(3):
                  for th in range(thickness):
                      picture_tensor[ch][y+i][x-th] = -1

          for i in range(80):
              for ch in range(3):
                  for th in range(thickness):
                      picture_tensor[ch][y+i][x+th+80] = -1

          for i in range(80):
              for ch in range(3):
                  for th in range(thickness):
                      picture_tensor[ch][y-th][x+i] = -1

          for i in range(80):
              for ch in range(3):
                  for th in range(thickness):
                      picture_tensor[ch][y+th+80][x+i] = -1

Of course you can pick more steps instead of 10 or maybe less: just make sure you have patience cause this might take as well a while. 

      step = 10; coordinates = []
      for y in range(int((height-(80-step))/step)):
          for x in range(int((width-(80-step))/step) ):
              area = cutting(x*step, y*step)
              result = model.predict(area)
              if result[0][1] > 0.90 and not_near(x*step,y*step, 88, coordinates):
                  coordinates.append([[x*step, y*step], result])
                  print(result)
                  plt.imshow(area[0])
                  plt.show()


![# Welcome to my adventure](/images/ships5.png)


As you can see: It did classify as ship images that have straight lines and bringht pixels 
- which I guess is the next step in finding a way to polish the model - though that's for another time. 

Or if you give it a second run: 

![# Welcome to my adventure](/images/ships7.png)

#### Now let's make sense of tags and find them on the image:

    for e in coordinates:
        show_ship(e[0][0], e[0][1], e[1][0][1])

    picture_tensor = picture_tensor.transpose(1,2,0)
    picture_tensor.shape
    (1777, 2825, 3)

    plt.figure(1, figsize = (15, 30))

    plt.subplot(3,1,1)
    plt.imshow(picture_tensor)

    plt.show()


![# Welcome to my adventure](/images/ships4.png)

-------------------------

Of course you can re-train the model and give it another run or with the current model give a second search and see what you might get. 

Good Luck and maybe next time will spot some cars .. if we find some nice labeled data. 
          
-------------------------

##### Sources: 

- Planet data [you can check how to extract images from my article](https://danielmoraite.github.io/docs/satellite1.html)
- my Tensorflow article on [Keras](https://danielmoraite.github.io/docs/fifth.html)
- [Kaggle competition data download](https://www.kaggle.com/rhammell/ships-in-satellite-imagery)
- find my full Jupyter notebook on [GitHub](https://github.com/DanielMoraite/DanielMoraite.github.io/blob/master/assets/Keras%20for%20search%20ships%20in%20satellite%20image.ipynb)

- and special support from and thanks to my dear friend [Laurens Geffert](https://janlauge.github.io)

---------------------------
---------------------------

