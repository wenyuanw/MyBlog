# Keras

Keras实现CNN

```python
#CNN build by Keras
# 2020.3.7
#Guangdong
import numpy as np
np.random.seed(1337) #for reproducibility
from keras.datasets import mnist
from keras.utils import np_utils
from keras.models import Sequential
from keras.layers import Dense,Activation,Convolution2D,MaxPooling2D,Flatten
from keras.optimizers import Adam

#download the mnist to the path '~/keras/datasets/' if it is the first to be called
#X shape(60,000 28x28),y shape(10,000,)
(X_train,y_train),(X_test,y_test)=mnist.load_data()

#data pre-processing
X_train=X_train.reshape(-1,1,28,28)
X_test=X_test.reshape(-1,1,28,28)
y_train=np_utils.to_categorical(y_train,num_classes=10)
y_test=np_utils.to_categorical(y_test,num_classes=10)

#Another way to build your CNN
model=Sequential()

#Conv layer 1 output shape (32,28,28)
model.add(Convolution2D(
    filters=32,
    kernel_size=(5, 5),
    input_shape=(1,     #channels
                 28,28),#height&width
    padding="same",  # padding method
))
model.add(Activation('relu'))

#Pooling layer 1 (Max pooling) output shape (32,14,14)
model.add(MaxPooling2D(
    pool_size=(2,2),
    strides=(2,2),
    padding="same",#padding method
))

#Conv layer 2 output shape (64,14,14)
model.add(Convolution2D(64,(5, 5),padding='same'))
model.add(Activation('relu'))

#Pooling layer 2 (Max pooling) output shape (64,7,7)
model.add(MaxPooling2D(pool_size=(2,2),padding='same'))

#Fully connected layer 1 input shape (64*7*7)=(3136), output shape (1024)
model.add(Flatten())
model.add(Dense(1024))
model.add(Activation('relu'))

#Fully connected layer 2 to shape (10) for 10 classes
model.add(Dense(10))
model.add(Activation('softmax'))

#Another way to define your optimizer
adam=Adam(lr=1e-4)

#We add metrics to get more results you want to see
model.compile(optimizer=adam,
              loss='categorical_crossentropy',
              metrics=['accuracy'])

print('Training-------')

#Another way to train the model
model.fit(X_train,y_train,epochs=1,batch_size=32,)

print('\nTesting-------')
#Evaluate the model witg the metrics we defined earlier
loss,accuracy=model.evaluate(X_test,y_test)

print('\ntest loss: ',loss)
print('\ntest  accuracy: ',accuracy)
```

```html
<script src="//unpkg.com/docsify"></script> 
<script src="//unpkg.com/prismjs/components/prism-bash.js"></script> 
<script src="//unpkg.com/prismjs/components/prism-php.js"></script>
```
