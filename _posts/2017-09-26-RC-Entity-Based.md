---
layout: page
title : An Entity Based Based Memory Network! 
---

_“As I entered the room I saw there was a green apple kept on the table...”_

As you were reading this sentence somewhere down the line you started imagining yourself entering a room and looking at a green apple kept on a table, you did not remember the sentence and the words, yet somewhere down the line, you created an imaginary ‘existence’ of whatever was happening!  

Memory networks as we saw in our previous post did something inline to this. However, they remember stuff as vectors which are formed out of sentences. Often it's hard to explore the information contained in the text especially in case of long and complicated sentences. How often have you said, “In essence the paragraph says….!” Can we train a network to do something analogous to that?    

So last year in December [Xun Wang](http://www.kecl.ntt.co.jp/icl/lirg/members/wang/index.html) came up with [Entity Based Memory Networks](https://arxiv.org/abs/1612.03551) in order to implement something cognate to this idea. This blog is just an attempt to explain the idea in a plain lucid manner :) 

## What's the difference! 
<img width="493" alt="screen shot 2017-09-27 at 11 42 08 am" src="https://user-images.githubusercontent.com/11302053/30898432-f6f0b3be-a378-11e7-8755-ea8d4321eeac.png">

As you can see in the figure the memory isn't storing information directly from the sentences contrary to the previosu implementations of the memory network. It's extracting entities out of it and then pooling them into the memory updating the entities as well as the memory as and when it get's new information.

So each sentence is considered one by one. There are some extracted entities and the new sentence. Both of them are used to create what is referred to as the state of the entities. Now based on the question, some information from the entity is focussed upon and used for generating the output feature which is further converted into the appropriate response.  


### Recap

Just to recapitulate the memory network has 4 main modules:

- I: Input module.
- G: Generalization module 
- O: Output feature module. 
- R: Response module. 

If you feel you wanna read memory networks again you can read this [post](https://harsh-agarwal.github.io/Memory-Networks/). 

## How do we define this mathematically? 

Let’s see how can we implement such a learning model to capture entities. 

Section 2.1 of the original paper explains the idea pretty well! Just for convenience I am putting it here :)

_"Firstly we use an example to illustrate how the model works. Table 1 shows a piece of text which contains 4 sentences and 2 questions. There are 7 entities in total, all of them are in bold._
 
1. **Mary** moved to the **bathroom**.
2. **John** went to the **hallway**.
3. Where is Mary? Bathroom. _Entities same as in 1_
4. **Daniel** went back to the **hallway**. 
5. **Sandra** moved to the **garden**.
6. Where is Daniel? Hallway. _Entities same as in 4_

 _An example from bAbI, a toy dataset for question answering_

_This text is elaborated around the 7 entities. It describes how their states change (i.e., the change of a character’s location) when the story goes on. Note that here all the entities are concrete concepts that exist in reality. It is also possible to talk about abstract concepts._
_The core of the proposed model are entities. We take Sentence 1 (S1) as input and extract the entities it contains {Mary, bathroom}. Vectors representing the states of these entities are initialized using some pre-learned word embeddings {**Mary**, **bathroom**} and stored in a memory pool. Meanwhile, we turn S1 into a vector **S1** using an autoencoder model .Then we use the sentence vector S1 toupdate the entities’states {**Mary**,**bathroom**}. The goal is to reconstruct **S1** solely from {**Mary**, **bathroom**}. In the same way, we process the following text (S2) and its containing entities (John, hallway) until encounter a question (S3). S3 is converted into a vector **S3** following the same method that processes previous input text. Then taking **S3** as input, we retrieve related entities from the memory which now stores all the entities (Mary, bathroom, John, hallway) that appear before S3. The related entities’ states are then used to produce a feature vector. In this case, (Mary and bathroom) are related to the question and their states are used for constructing the feature vector. Note the current states of the two entities (Mary and bathroom) are different from their initial values due to S1. Based on the feature vector, we then use another neural network model to predict the answer to S3._
_The model monitors the entities involved in text and keeps updating their states according to the input. Whenever we have a question with regard to the text, we check the states of entities and predict an answer accordingly. The proposed model comprises of 4 modules, as is shown in Fig. 1. Each module is designed for a unique purpose and together they construct the entity-based memory network model."_

## Making the model! 

<img width="456" alt="screen shot 2017-10-01 at 1 54 01 pm" src="https://user-images.githubusercontent.com/11302053/31052992-18478fe4-a6b0-11e7-9c27-58b62e88701f.png">

Let’s look at the working of each module in detail:

**Input module:** This module converts the sentences into a vector.  The module takes as  an input a sentence and converts it into a vector. Meanwhile, it also extracts all the entities it contains. The paper assumes that we have the entities annotated for each sentence. The question is also processed using this module. The authors of the paper used an autoencoder to convert the sentence into a vector representation.

**Generalisation Module:** Every time as you are looking at new sentences there are two possible choices.
1. _You might get some information about the existing entities_  
2. _You might catch hold of some new entities itself._

The update in entities takes place using the generalisation module. The following figure schematically shows how the module works. 

<img width="499" alt="screen shot 2017-10-01 at 1 59 04 pm" src="https://user-images.githubusercontent.com/11302053/31053012-a2bc10fa-a6b0-11e7-947f-039ce75a2a67.png">

For the _first choice_ discussed when it get’s _entities that are not contained in the memory pool, the model would create a new memory slot for them and initialize these slots using pre-learned word embedding._

For the _second choice_ if it looks at new information about the present entities, it needs to update their state. How can that be done? The model asks _us to construct the sentence solely from its entities. So we define a function **f2** which will take one by one the entities of sentence as input and output the sentence vector (**S2**). The **f2** model is trained by comparing the output (**S2**) with **S** and minimizing the loss using stochastic gradient descent as it is trained._

The equation: 

<div style="text-align:center" ><img width="491" alt="screen shot 2017-10-01 at 2 38 05 pm" src="https://user-images.githubusercontent.com/11302053/31053213-1540feec-a6b6-11e7-9747-e347419b6a84.png"></div>

The authors of the paper have used a [Gated Recurrent Unit](http://www.wildml.com/2015/10/recurrent-neural-network-tutorial-part-4-implementing-a-grulstm-rnn-with-python-and-theano/) (GRU) is used as f2 which update the sentence by taking entities as input and then entities are updated by comparing S1 and S2. This way we make sure entity contains sufficient context and information regarding the sentences being read.

**Output feature module:** The output module is triggered whenever the model stumbles upon a question. It retrieves the related entities according to the question asked and retrieves the output feature. It retrieve related entities according to the input question and then produce an output feature vector accordingly.

If you are a bit familiar with GRU's you will be able to make sense out of the following equations: 

<div style="text-align:center" ><img width="525" alt="screen shot 2017-10-01 at 2 59 36 pm" src="https://user-images.githubusercontent.com/11302053/31053353-164c9bd6-a6b9-11e7-92bf-1880a71fc041.png"></div>

Using a GRU the equations would become: 
<div style="text-align:center" ><img width="525" alt="screen shot 2017-10-01 at 3 05 18 pm" src="https://user-images.githubusercontent.com/11302053/31053400-ec86d69e-a6b9-11e7-9b63-cd9d65a5a382.png"></div>

A figure to that explains the output module: 

<div style="text-align:center" ><img width="561" alt="screen shot 2017-10-01 at 3 08 00 pm" src="https://user-images.githubusercontent.com/11302053/31053412-3f4a7926-a6ba-11e7-8573-6a36b0c8da1c.png"></div>

And finally 

**Respone Module:** Generate the response according to the output feature vector. To generate the final answer, we use a simple neural network which takes the feature vector O as input and predict a word as output._The word with the highest probability is selected._

<div style="text-align:center" ><img width="387" alt="screen shot 2017-10-01 at 3 13 31 pm" src="https://user-images.githubusercontent.com/11302053/31053459-13ad50bc-a6bb-11e7-9471-9a936d9bd5d3.png"></div>

So now we understand the fundamentals behind solving an RC. We could use this or similar ideas in various other ways to come up with our own model to solve a Reading Comprehension using Neural Networks! If you have any ideas regarding the same please mail us at [DeepLearningIsLife](mailto:deeplearningislife@gmail.com)

Written by,

Harsh Agarwal and Shubham Gautam 


