---
title: "Part 1: Building Deep Learning Models by using Keras and Tensorflow"
description: Learning how to build simple to complex architecture of deep learning model.
comments: true
toc: false
layout: post
categories: [tensorflow, keras]
author: josua naiborhu
---

This is just some random notes  by reading `Deep Learning with Python` by  Francois Chollet.This book focusses on developing Deep Learning Models using Keras and Tensorflow.
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
## Subcalssing Model class
This is the advanced of building model pattern by subclassing Layer class to create custom layers.
This is like subclassing nn.Model in pytorch(if you experienced in pytorch)

-__init__ for defining the layers the model will use 
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
 -- To Be Continued








