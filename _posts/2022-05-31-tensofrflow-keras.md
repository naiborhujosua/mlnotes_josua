---
title: "Part 1: Building Deep Learning Models by using Keras and Tensorflow"
description: Learning how to build simple to complex architecture of deep learning model.
comments: true
toc: false
layout: post
categories: [tensorflow, keras]
author: josua naiborhu
---
## Introduction
> Some random notes  by reading `Deep Learning with Python by  Francois Chollet`. This book focusses on developing Deep Learning Models using Keras and Tensorflow.
I found this book is very good at explaining Deep learning.I try making notes for important chapters that i think it is essential for improving myself in developing deep learning models. 
I took some source code from the book as a reference. 

## Different ways of building models in Keras 
- Sequential model where the the layers are stacked each other to create the DL architecture.
- The functional API where focussing on graph-like model architecture that represents a nice usabilibity and flexibility. 
- Model Subclassing where a low level option to write model from scratch.

## Sequential Models

```python 
from tensorflow import keras 
from tensorflow.keras import layers 

model =keras.Sequential([
    layers.Dense(64,activation="relu),
    layers.Dense(10,activation="softmax")
])
```
or 

```python
model = keras.Sequential()
model.add(layers.Dense(64,activation="relu))
model.add(layers.Dense(10,activation="softmax")) 

```
We see that sequential model create the 64 multidimensional representation based on input data to create 10 output 
value using softmax activation function to get predicted probabilities. You can check model.summary() 
for looking at the architecture of your model. You can also name the models and layers by dding name parameters in each process in your architecture by using string. 

```python
model = keras.Sequential(name="my_example_model")
model.add(layers.Dense(64,activation="relu),name="my_first_layer")
model.add(layers.Dense(10,activation="softmax",name="my_last_layer")) 
```

## Functional API 
While sequential model constainst are for simple model that can express models with single input and single output where applying one layer after the other sequentially. 
There is a situation when we face a problem to encounter models with multiple inputs like image and its metadata and multiple output to predict different things about data. 
Imagine you are building a system to rank customer support tickets by priority and route them by departments based on 3 inputs 
- The title of the ticket(text input)
- The text body of the ticket(text input)
- Any tags added by the user(categorical input encode to one-hot encing)
 and create 2 outputs 
 - The priority score of the ticket , a scalr between 0-1(Sigmoid ouput)
 - The deparment that should handle the ticket (Softax over the set of departments)

```python 

vocabulary_size =10000
num_tags =100
num_departments =4
# Define model inputs
title =keras,Input(shape=(vocabulary_size,),name="title")
text_body = keras.Input(shape=(vocabulary_size,),name="text_body")
tags = keras.Input(shape=(num_tags),name="tags")

#Combine input features into a single tensor by concatenating them
features = layers.Concatenate()([title,text_body,tags])
features =layers.Dense(64,activation="relu")(features)

# Define Model outputs
priority = layers.Dense(1,activation="sigmoid",names="priority")(features)
department = layers.Dense(num_departments,activation="softmax",name="departments")(features)

model = keras.Model(inputs=[title,text_body,tags],
outputs=[priority,department])

```
You can see the structure of your model in a nice graph by plotting the moarchitecture by using `keras.utils.plt_model(instances of model object)`.The advantage of using functional API is enabling us to do feature extractions that reuse intermediate features from another model. If we want to add another output to the precious model by estimating how long a given issue ticket will take to ressolve by difficulty rating(quick,medium,difficult). we dont need to recreate the model from scratch. We can start fro intermediate features of previous model

```python 

features = model.layers[4].output
difficulty = layers.Dense(3,activation ="softmax",name="difficulty")(features)

new_model = keras.Model(inputs =[title,text_body,tags],
outputs=[priority,department,difficulty])

keras.utils.plot_model(new_model,"updated_ticket_classifier.png",show_shapes=True)
```
## Subclassing Model class
This is the advanced of building model pattern by subclassing Layer class to create custom layers.
This is like subclassing nn.Model in pytorch(if you experienced in pytorch)

- __init__ for defining the layers the model will use 
- call() for defining the forward pass of the model by reusing previously layers created.

```python 
class CustomerTicketModel(keras.Model):

    def __init__(self,num_departments):
        super.__init__()
        self.concat_layer = layers.Concatenate()
        self.mixing_layer = layers.Dense(64,activation="relu")
        self.priority_scorer = layers.Dense(1,activation="sigmoid")
        self.departments_classifier = layers.Dense(num_departments,activation="sofmax")

    def call(self,inputs):
        title = inputs["title"]
        text_body = inputs["text_body"]
        tags = inputs["tags"]

        features =self.concat_layer[title,text_body,tags]
        features =self.mixing_layer(features)

        priority =self.priority_scorer)(features)
        department = self.department_classifier(features)
        return priority,department


model =CustomerTicketModel(num_departments=4)
priority,department =model({"title":title_data,"text_body":text_body_data,"tags":ttags_data})

 ## Do compile(),fit(),evalueate and predict() as usual.
 
 ```


## Writing own callbacks (loss,EarlyStopping,ModelCheckPoint)

There are a few methods available in keras.callbacks.Callback class 
- on_epoch_begin(epoch,logs) for calling at the start of every epoch
- on_epoch_end(epoch,logs) for calling at the end of every epoch
- on_batch_begin(batch,logs)  for calling tight before processing each batch
- on_batch_end(batch,logs) for calling right after processing each batch
- on_train_begin(logs) for calling at the start of training 
- on_train_end(logs) for calling at the end of training

```python
from matplotlib import pyplot as plt
 
## Transform Training fit() ModelCheckPoint and EarlyStopping
```python 
from tensorflow.keras.datasets import mnist

def get_mnist_model():
    inputs = keras.Input(shape=(28*28,))
    features = layers.Dense(512,activation="relu")(inputs)
    features =layers.Dropout(0.5)(features)
    outputs =layers.Dense(10,activation="softmax")(features)
    model =keras.Model(inputs,outputs)
    return model


(images,labels),(test_images,test_labels) =mnist.load_data()
images = images.reshape((60000,28*28)).astype("float32")/255
test_images =test_images.reshape((10000,28*28)).astype("float32")/255
train_images,val_images = images[10000:],images[:10000]
train_labels,val_labels = labels[10000:],labels[:10000]

model = get_mnist_model()
model.compile(optimizer="rmsprop",loss="sparse_categorical_crossentropy",
metrics =["accuracy"])

model.fit(train_images,train_labels,epoch=3,validation_data=[test_images,test_labels])
test_metrics =model.evaluate(test_images,val_label)
predictions =model.predict(test_images)

callbacks_list =[
    keras.callbacks.EarlyStopping(
    monitor="val_accuracy",
    patience=2,),
    keras.callbacks.ModelCheckPoint(
        filepath="checkpoint_path.keras",
        monitor ="val_loss",
        save_best_only=True
    )
]
model = get_mnist_model()
model.compile(optimizer="rmsprop",
              loss ="sparse_categorical_crossentropy",
              metrics =["accuracy"])
model.fit(train_images,train_labels,epochs=10,calbacks=callbacks_list,
validation_data=[val_images,val_labels])

#Load the model 
model = keras.models.load_model("checkpoint_path.keras")

class lossHistory(keras.callbacks.Callback):
    def on_train_begin(self_logs):
        self.per_batch_losses=[]

    def on_batch_end(self,batch,logs):
        self.per_batch_losses.append(logs.get("loss"))

    def on_epoch_end(self,epoch,logs):
        plt.clf()
        plt.plot(range(len(self.per_batch_losses)),self.per_batch_losses,label="Training loss for each batch")
        plt.xlabel(f"Batch (epoch{epoch})")
        self.per_batch_losses =[]

model = model_get_mnist_model()
model.compile(optimizer="rmsprop",
                loss ="sparse_categorical_crossentropy",
                metrics=["accuracy"])
model.fit(train_images,train_labels,epochs=10,callbacks=[LossHistory()],
            validation_data=[val_images,val_labels])
            
```
## A complete training and evaluation loop from scratch(fit() and evaluate() method)
```python 
# Training Process
model = get_mnist_model()
loss_fn =keras.losses.SparseCategoricalCrossentropy()
optimizer =keras.optimizers.RMSprop()
metrics = [keras.metrics.SparseCategoricalAccuracy()]
loss_tracking_metrics =keras.metrics.Mean()

def train_step(inputs,targets):
    with tf.GradientTape() as tape:
        # training=True during training
        predictions =model(inputs,training=True)
        loss =loss_fn(targets,predictions)
    gradients = tape.gradients(loss,,model.trainable_weights)
    optimizer.apply_gradients(zip(gradients,model.trainable_weights))
    
    logs ={}
    for metric in metrics:
        metric.update_state(targets,predictions)
        logs[metric.name] =metric.result()
        
    loss_tracking_metric.update_state(loss)
    logs["loss"] = loss_tracking_metric.result()
    
    return logs
    
 def reset_metrics():
    for metric in metrics:
        metric.reset_state()
    loss_tracking_metric.reset_state()
    
 training_dataset =tf.data.Dataset.from_tensor_slices((train_images,train_labels))
 training_dataset = training_dataset.batch(32)
 epochs =3
 for epoch in range(epochs):
     reset_metrics()
     for input_batch,targets_batch in training_dataset:
         logs =train_step(input_batchs,targets_batch)
     print(f"Results at the end of epoch {epoch}")
     for key,value in logs.items():
          print(f"....{key}: {value:.4f}")
          
# Evaluation loop
def test_step(inputs,targets):
    #training=False for evaluation
    predictions= model(inputs,training=False)
    loss =loss_fn(targets,predictions)
    
    logs ={}
    for metric in metrics:
        metric.update_state(targets,predictions)
        logs["val" +metric.name] = metric.result()
        
        loss_tracking_metric.update_state(loss)
        logs["val_loss"] = loss_tracking_metrics.result()
        return logs
        
    val_dataset = tf.data.Dataset.from_tesor_slices((val_images,val_labels))
    val_dataset =val_dataset.batch(32)
    reset_metrics()
    
    for inputs_batch,targets_batch in val_dataset:
        logs = test_step(inputs_batch,targets_batch)
    print("Evaluation results.")
    for key,value in logs.items():
        print(f"...{key} : {value:.4f}")
          
```








