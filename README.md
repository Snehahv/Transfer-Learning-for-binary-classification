# Transfer-Learning-for-binary-classification
## Aim
To Implement Transfer Learning for Horses_vs_humans dataset classification using InceptionV3 architecture.
## Problem Statement and Dataset
Develop and train a binary classification model to distinguish between images of horses and humans using transfer learning with the InceptionV3 model. The task involves using both training and validation datasets, implementing data augmentation, and defining custom callbacks to stop training when a target accuracy is reached.
![Screenshot 2024-10-14 105606](https://github.com/user-attachments/assets/472cb4ab-8b90-47fe-b88c-6eb87c9b3888)

</br>
</br>
</br>

## DESIGN STEPS
### STEP 1:
Load and prepare image data for training and validation.
</br>

### STEP 2:
Build a model using pre-trained InceptionV3 with additional layers for binary classification.
</br>

### STEP 3:
Train the model on the data and stop early when accuracy reaches 97%.
<br/>
### Step 4:
Plot the accuracy and loss for both training and validation.

Write your own steps
<br/>

## PROGRAM
Include your code here
```python
DEVELOPED BY: SNEHA HV
REG NO : 212222040157

# Import all the necessary files!
import os
import tensorflow as tf
from tensorflow.keras import layers
from tensorflow.keras import Model
from os import getcwd

# GRADED FUNCTION: train_val_datasets

def train_val_datasets():
    """Creates training and validation datasets

    Returns:
        (tf.data.Dataset, tf.data.Dataset): training and validation datasets
    """

    ### START CODE HERE ###

    training_dataset = tf.keras.utils.image_dataset_from_directory( 
        directory='/content/horse-or-human.zip/',
        batch_size=32,
        image_size=(150,150),
        shuffle=True, 
        seed=7 
    ) 
    
    validation_dataset = tf.keras.utils.image_dataset_from_directory( 
        directory='/content/validation-horse-or-human.zip',
        batch_size=32,
        image_size=(150,150),
        shuffle=True, 
        seed=7 
    ) 

    ### END CODE HERE ###
                                                                        
    return training_dataset, validation_dataset

path_inception = '/content/inception_v3_weights_tf_dim_ordering_tf_kernels_notop (1).h5'

# Import the inception model
from tensorflow.keras.applications.inception_v3 import InceptionV3

# Create an instance of the inception model from the local pre-trained weights
local_weights_file ='/content/inception_v3_weights_tf_dim_ordering_tf_kernels_notop (1).h5'
pre_trained_model = InceptionV3(include_top = False,
                                input_shape = (150, 150, 3),
                                weights = None)

pre_trained_model.load_weights(local_weights_file)

# Make all the layers in the pre-trained model non-trainable
for layer in pre_trained_model.layers:
    layers.trainable = False

pre_trained_model.summary()
# Write Your Code
print('Name: SNEHA HV     Register Number:212222040157')

last_layer = pre_trained_model.get_layer('mixed7')
print('last layer output shape: ', last_layer.output.shape)
last_output = last_layer.output

# Expected Output:
# ('last layer output shape: ', (None, 7, 7, 768))

# Define a Callback class that stops training once accuracy reaches 99.9%
class EarlyStoppingCallback(tf.keras.callbacks.Callback):
    def on_epoch_end(self, epoch, logs=None):
        if logs['accuracy']>0.970:
            self.model.stop_training = True
            print("\nReached 97.0% accuracy so cancelling training!")

# GRADED FUNCTION: output_of_last_layer

def output_of_last_layer(pre_trained_model):
    """Fetches the output of the last desired layer of the pre-trained model

    Args:
        pre_trained_model (tf.keras.Model): pre-trained model

    Returns:
        tf.keras.KerasTensor: last desired layer of pretrained model
    """
    ### START CODE HERE ###

    last_desired_layer = pre_trained_model.get_layer('mixed7')
    last_output = last_desired_layer.output
    
    print('last layer output shape: ', last_output.shape)
    
    ### END CODE HERE ###

    return last_output

# Import all the necessary files!
import os
import tensorflow as tf
from tensorflow.keras import layers
from tensorflow.keras import Model
from os import getcwd
from tensorflow.keras.optimizers import RMSprop
def create_final_model(pre_trained_model, last_output):

 # Flatten the output layer of the pretrained model to 1 dimension
    x = tf.keras.layers.Flatten()(last_output)

    ### START CODE HERE ###

    # Add a fully connected layer with 1024 hidden units and ReLU activation
    x = tf.keras.layers.Dense(1024, activation='relu')(x)
    # Add a dropout rate of 0.2
    x = tf.keras.layers.Dropout(0.2)(x) 
    # Add a final sigmoid layer for classification
    x = tf.keras.layers.Dense(1, activation='sigmoid')(x)
    model = Model(inputs=pre_trained_model.input, outputs=x)
 # Compile the model
    model.compile( 
        optimizer=tf.keras.optimizers.RMSprop(learning_rate=0.00001), 
        loss='binary_crossentropy', # use a loss for binary classification
        metrics=['accuracy'] 
    )

    ### END CODE HERE ###
    return model

model = create_final_model(pre_trained_model, last_output)

# Get the Horse or Human dataset
path_horse_or_human = '/content/horse-or-human.zip'
# Get the Horse or Human Validation dataset
path_validation_horse_or_human = '/content/validation-horse-or-human.zip'
from tensorflow.keras.preprocessing.image import ImageDataGenerator

import os
import zipfile

local_zip = path_horse_or_human
zip_ref = zipfile.ZipFile(local_zip, 'r')
zip_ref.extractall('/tmp/training')
zip_ref.close()

local_zip = path_validation_horse_or_human
zip_ref = zipfile.ZipFile(local_zip, 'r')
zip_ref.extractall('/tmp/validation')
zip_ref.close()

model.summary()
print("Name: SNEHA HV  Reg No: 212222040157")

# Define our example directories and files
train_dir = '/tmp/training'
validation_dir = '/tmp/validation'

train_horses_dir = os.path.join(train_dir, 'horses')
train_humans_dir = os.path.join(train_dir, 'humans')
validation_horses_dir = os.path.join(validation_dir, 'horses')
validation_humans_dir = os.path.join(validation_dir, 'humans')

train_horses_fnames = os.listdir(train_horses_dir)
train_humans_fnames = os.listdir(train_humans_dir)
validation_horses_fnames = os.listdir(validation_horses_dir)
validation_humans_fnames = os.listdir(validation_humans_dir)

print(len(train_horses_fnames))
print(len(train_humans_fnames))
print(len(validation_horses_fnames))
print(len(validation_humans_fnames))

# Expected Output:
# 500
# 527
# 128
# 128

# Add our data-augmentation parameters to ImageDataGenerator
train_datagen = ImageDataGenerator(rescale = 1/255,
                                  height_shift_range = 0.2,
                                  width_shift_range = 0.2,
                                  horizontal_flip = True,
                                  vertical_flip = True,
                                  rotation_range = 0.4,
                                  shear_range = 0.1,
                                  zoom_range = 0.3,
                                  fill_mode = 'nearest'
                                  )

# Note that the validation data should not be augmented!
test_datagen = ImageDataGenerator(rescale = 1/255)

# Flow training images in batches of 20 using train_datagen generator
train_generator = train_datagen.flow_from_directory(train_dir,
                                                   target_size = (150, 150),
                                                   batch_size = 20,
                                                   class_mode = 'binary',
                                                   shuffle = True)

# Flow validation images in batches of 20 using test_datagen generator
validation_generator =  test_datagen.flow_from_directory(validation_dir,
                                                        target_size = (150, 150),
                                                        batch_size =20,
                                                        class_mode = 'binary',
                                                        shuffle = False)

# Expected Output:
# Found 1027 images belonging to 2 classes.
# Found 256 images belonging to 2 classes.

# Run this and see how many epochs it should take before the callback
# fires, and stops training at 97% accuracy

callbacks = EarlyStoppingCallback()
history = model.fit(train_generator,
    validation_data = validation_generator,
    epochs = 100,
    verbose = 2,
    callbacks = [EarlyStoppingCallback()],
)

%matplotlib inline
import matplotlib.pyplot as plt
acc = history.history['accuracy']
val_acc = history.history['val_accuracy']
loss = history.history['loss']
val_loss = history.history['val_loss']

epochs = range(len(acc))

plt.plot(epochs, acc, 'r', label='Training accuracy')
plt.plot(epochs, val_acc, 'b', label='Validation accuracy')
plt.title('Name: SNEHA HV     Register Number: 212222040157   ')
plt.title('Training and validation accuracy')
plt.legend(loc=0)
plt.figure()
plt.plot(epochs, loss, 'r', label='Training loss')
plt.plot(epochs, val_loss, 'b', label='Validation Loss')
plt.title('Name: SNEHA HV      Register Number: 212222040157    ')
plt.title('Training and validation Loss')
plt.legend(loc=0)
plt.figure()


plt.show()

```


## OUTPUT
### Training Accuracy, Validation Accuracy Vs Iteration Plot
<table>
<tr>
<td>
    
![Screenshot 2024-10-14 110633](https://github.com/user-attachments/assets/8900dc65-fa70-44c1-88b9-4319615ba3d9)

</td>
 


### Training Loss, Validation Loss Vs Iteration Plot
<td>
  
![Screenshot 2024-10-14 110648](https://github.com/user-attachments/assets/6cbbf0b1-3b72-46f3-916b-36eba2a39893)

</td>
</tr> 
</table>

### Conclusion

![image](https://github.com/user-attachments/assets/a090b7e9-2c5e-4976-bf5f-9b1c9d408a83)

</br>
</br>
</br>

## RESULT
Thus, transfer learning for classifying horses and human is implemented successfully
</br>
</br>
</br>
