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

**To be Continued!**

Written by,

Harsh Agarwal and Shubham Gautam 


