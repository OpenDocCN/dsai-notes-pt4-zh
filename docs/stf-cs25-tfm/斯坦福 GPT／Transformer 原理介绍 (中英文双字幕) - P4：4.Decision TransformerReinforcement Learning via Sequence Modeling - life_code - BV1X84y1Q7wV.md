# 斯坦福 GPT／Transformer 原理介绍 (中英文双字幕) - P4：4.Decision TransformerReinforcement Learning via Sequence Modeling - life_code - BV1X84y1Q7wV

![](img/041a2ae7aae080f075f80cd7c4c33a68_0.png)

So I'm excited to talk today about our recent work on using Transers for reinforcement learning and this is joint work with a bunch of really exciting collaborators。

 most of them at UC Berkeley and some of them at Facebook and Google。

 I should mention this work was led by way to talented undergrads， Li Chen and Kevin Blue。



![](img/041a2ae7aae080f075f80cd7c4c33a68_2.png)

And I'm excited to present the results we had， so let's try to motivate why we even care about this problem。

So we have seen in the last。Three or four years， that transformers since the introduction in 2017 have taken over lots and lots of different fields of artificial intelligence。

 so we saw them having a big impact for language processing， we saw them being used for vision。

 the vision transformer very recently they were in nature trying to solve protein folding and very soon they might just replace as computer scientists by having automatically generate code。

So with all of these advances， it seems like we are getting closer to having a unified model for decision making for artificial intelligence。

But artificial intelligence is much more about not just having perception。

 but also using the perception knowledge to make decisions。

And this is what this talk is going to be about。But before I go into actually thinking about how we will use these models decision making。

 here is a motivation for why I think it is important to ask this question。So unlike models for RL。

 when we look at transformers for perception modalities like I showed in the previous slide。

 we find that these models are very scalable and have very stable training dynamics so you can keep as long as you have enough computation and you have more and more data that can be sourced。

 you can train bigger and bigger models and you'll see very smooth reductions in the loss。

And the overall training dynamics are very stable， and this makes it very easy for practitioners and researchers to build these models and learn richer and richer distributions。

So like I said， all of these advances have so far occurred in perception what we've been interested in this talk is to think about how we can go from perception。

 looking at images， looking at text and all these kinds of sensory signals to then going into the field of actually taking actions and making our agents do interesting things in the world。

😊，啊。And here throughout the talk， we should be thinking about why this perspective is going to enable us to do scalable learning。

 like I showed on the previous slide， as well as brings stability into the whole procedure。😊。

So sequential decision making is a very broad area and what I'm specifically going to be focusing on today is the one route to sequential decision making that's reinforcement learning。

So just as a brief background， what is reinforcement learning。

 so we are given an agent who' is in a current state and the agent is going to interact with the environment by taking actions。

And by taking these actions， the environment is going to return to it a reward。

For how good that action was as well as next state into which the agent will transition and this whole feedback loop will continue。

啊。The goal here for an intelligent agent is to then using trial and error。

 so try out different actions， see what rewards will lead to learn a policy which maps your states to actions such that the policy maximizes the agent's cumulative rewards over time horizon。

So you take a sequence of actions and then based on the reward you accumulate for that sequence of actions。

 will' judge how good your policy is。This talk is also going to be specifically focused on a form of reinforcement learning that goes by the name of offline reinforcement learning。

So the idea here is that what changes from the previous picture where I was talking about online learning。

 online reinforcement learning is that here now instead of doing actively interacting with the environment。

 you have a collection of log data of interactions so think about some robot that's going out in the fields and it collects a bunch of sensory data and you have all logged it and using that log data you now want to train another agent。

 it could be another robot to then learn something interesting about that environment just by looking at the log data。

G so theres no trial and error component。Which is currently one of the extensions of this framework。

 which would be very exciting， so I'll talk about this towards the end of the talk why it's exciting to think about how we can extend this framework to include an exploration component and have trial and error。

Okay， so not to go more concretely into what the motivating challenge of the stock was now that they have introduced ArL。

 so let's look at some statistics。So large language models。

I have billions of parameters and they have today they have roughly about 100 layers in Transformer。

 they are very stable to train using supervised learning， style losses。

 which are the building blocks of autoregressive generation， for instance。

 or for mass language modeling as in B。And this is like a field that's growing every day and there's a course on it at Stanford that we're all taking just because it has had such a monumental impact on AI。

嗯。RL policies， on the other hand， and I'm talking about deep RL。

 the maximum they would extend to is maybe millions of parameters at 20 layers。嗯。

And what's really unnerving is that they're very unstable to train so the current algorithms for reinforcement learning they build on mostly dynamic programming which involves solving an inter loop optimization problem that's very unstable and it's very common to see practitioners in RL looking at reward reward code that look like this so what I really want you to see here is the variance in the returns that we tend to get in RL it's really huge even after doing multiple rounds of experimentation and that's often that is really at the code got to done with the fact that。

Our algorithms learning checks need better improvements so that the performance can be stably achieved by Asians。

In complex environments。So what this work is hoping to do is it's meant to introduce transformers。

And I'll first show in one slide what exactly that model looks like and we're going to go into deeper details of each of the components。

I have good question。Yeah， I can ask a question real quick。Yes。

 and thank you what I'm curious to know is is the what is the cause for why RL typically has several orders of magnitude fewer parameters？

That's a great question， so typically when you think about reinforcement learning algorithms。😊。

In deep art in particular， so the most common algorithms， for example。

 have different networks playing different roles in the task so you'll have a network for instance。

 playing the role of an actor so trying to figure out a policy and then therell a different network that's playing the role of a critic。

And these networks are trained on data that's adaptively gathered so unlike perception where you will have a huge data set of interactions on which you can train your models in this case the architectures and even the environments to some extent。

 are very simplistic because of the fact that we are trying to train very small components the functions that we're training and then bringing them all together and these functions are often trained in not super complex environments。

 so it's a mix of different issues I wouldn't say it's purely just got to with the fact that the learning objectives are fault。

 but it's a combination of the environments we use the combination of the targets that each of the neural networks are predicting which leads to networks which are much bigger than。

What we currently see tending to or， and that's why it's very common to see neural networks with much fewer layers being used in RL as opposed to perception。

Thank you。You want to ask a questionep yeah， I was going to is there a reason why you chose offline RL versus online RL？

That's another great question so the question is why offline R is opposed to online RL and the plain reason is because all this is a first work trying to look at reinforcement learning so offline RL avoids this problem of exploration。

You are given a log data set of interactions you're not allowed to further interact with the environment so just from this data set you're trying to unearth a policy of what the optimal agent would look like so it would right like if you do online RL wouldn't that like just give you this opportunity of exploration basically it would it would what would also do which is a technically challenging here is that the exploration would be harder to encode so offline R is the first step there is no reason why we should not study why online RL cannot be done it's just that it provides a more contained setup where ideas from transformers will directly extend。

Okay， sounds good， so let's look at the model and it's really simple on purpose。

So what we're going to do is we're going to look at our offline data。

which is essentially in the form of tra trees， so offline data would look like a sequence of states。

 actions， returns over multiple time steps。It's a sequence that's natural to think of us as directly feeding as input to a transformer in this case we use a causal transformer as it's common in GPT。

So we go from left to right and because the data set comes with the notion of time step causality here is much more well intended than the general meaning that's used for perception this is really causality how it should be in perspective of time。

What we predict out of this transformer are the actions conditioned on everything that comes before that token in the sequence。

So if you want to predict the action at this t minus one step， we'll use everything that came。

At time step D minus two， as well as the returns and states at time step D minus one。Okay。

So we will go into the details of how exactly each of these are encoded。

 but essentially this is in a one liner， it's taking the tragic pre datata from the offline data。

 treating it as a sequence of tokens， passing it through a causal transformer and getting a sequence of actions as the output。

Okay， so how exactly do we do the forward path through the network？

So one important aspect of this work， which is the U states' actions and this quantity called returns to room。

So these are not direct rewards。These are returns to go and let's see what they really mean。

So this is a trajectory that goes as input。and the returns to go are the sum of rewards starting from the current time step into until the end of the episode。

So really what we want the transformer is to get better at using a target return。

 this is how you should think of returns to go。As the input in deciding what action to take。

This perspective is going to have multiple advantages if it willll allow us to actually do much more than offline RL and generalizing to different tasks by just changing the returns to go。

And here it's very important， so at time step one we will just have the overall sum of rewards for the entire tragic dream at time step two we subtract the reward we get by taking the first action and then have the sum of rewards for the remainder of the tragic dream。

Okay， so that's how we quality turns to go like how many more rewards in accumulation you need to acquire to fulfill your。

Re your return goal that is set the beginning。What is the output。

 the output is the sequence of predicted actions， so as I showed in the previous slide。

 we use a causal transformer so we predict in sequence the desired actions。The attention， which is。

Going to be computed inside the transformer with taking an important hyperparameter key。

 which is the context length。We see that in perception as well here， and for the rest of the talk。

 I'm going to use the notation K to denote how many tokens in the past would we be attending over to predict the action and the current time step。

Okay， so again， digging a little bit deeper into code。

There are some subtle differences with how a decision transformer operates as opposed to a normal transformer。

 the first is that here the time step notion is going to be。哦。

Have a much bigger semantics that extends across three tokens。So in perception。

 you just think about the time step per word， for instance， like in NLP or per patch for vision。

 and in this case we will have a time step encapsulating three tokens， one for the states。

 one for the actions and one for the rewards。And then we embed each of these tokens and then add the pollution embedding as it common in the transformer and we feed those inputs to the transformer at the output we only care about one of these fee tokens in those default side I will say experiments where even the other tokens might be of interest as target predictions but for now let's keep it simple。

 we want to learn a policy a policy which is trying to predict actions。So when we try to decode。

 we'll only be looking at the actions from the hidden representation in the prefin layer。Okay。

 so this is the forward pass now what do we do with this network， we train it how do we train it？😊。

Sorry， just a quick question on semantics there if you go back one slide。The plus in this case。

 the syntax means that you are actually adding the values element wise and not concatenating them。

 is that right？That is correct。公。Okay， so what's the last function follow up on that。

 I thought it was concot， why are we just adding it？Sorry， can you go back？Yeah。

 I think it's a design choice， you can pattern in it， you can add it。

 it leads to different functions being encoded in our case it was editioned。O。

Why did you try the other one and it just didn't work or why is that because I think intuitively concatenating would like make more sense。

So I think both of them have different use cases for the functional of encoding like one is really mixing in the embeddings for the state and basically shifting it so and you add something if you think of the embedding of the states as a vector and you add something you are actually shifting it。

Whereas in the concatetnation case， you are actually increasing the dimensionality of this space。😊。

Yeah呀。So those are different choices which are doing very different things we found this one to be worked better i'm not sure I remember if the results was very significantly different if you would concatetnate them。

 but this is the one which we operate with。But wouldn't there because like if you're shifting it right like if you have an em bedding for a state and let's say like you perform certain actions and you like end up at the same state again。

You would want these embeddings to be the same however now you're at a different time step so like you shifted it so wouldn't that be like harder to learn？

So there's a bigger and interesting question in that what you said is basically。

 are we losing the Marco property？Because as you said that if we come back to the same state at a different time step。

 shouldn't we be doing similar operations and the answer here is yes， we are actually being nonmarco。

And this might seem very nonintuitive at first that why is nonmarkovness important here。

 and I want to refer to another paper which came very much in conjunction with this a transer that actually shows in more detail and its basically says that if you were trying to predict the transition dynamics。

 then you could have actually had a Markovvian system built in here， which would do just as good。

However， for the perspective of trying to actually predict actions。

 it does help to look at the previous time steps， even more so when you have missing observations so for instance。

 if you have the observations being a subset of the true state so looking at the previous states and actions helps you better fill in the missing pieces in some sense so this is commonly known as partial observability where you by looking at the previous tokens you can do a better job at predicting the actions that you should take at the current time step。

So nonmarkness is on purpose， and it's non intuitive。

 but I think it's one of the things that separates this framework from existing ones。

So it will basically help you because like RL usually works on like infinite like works better on like infinite horizon problems right so technically the way you formulated it it would work better on finite horizon problems I'm assuming because you want to take different actions based on like a history based on like given the fact that now you have to a different time step。

Yeah yeah so if you wanted to work on In Verorizon maybe something like discounting would work just as well to get that effect in this case we were using a discount factor of one or basically like no discounting at all。

 but you're right if I think we really want to extend it to In horizonizon you would need to change the discount factor。

Thanks。Okay， so questions。Oh。I think it was just answered in chat， but I'll ask it anyways。

 I think I might have missed this or maybe you're about to talk about it。

 the offline data that was collected， what policy was used to collect it。

So this is a very important question and it will be something I mentioned the experiments。

 so we were using the benchmarks that exist for our scenario where essentially the way these benchmarks are constructed as。

You train an agent using online RL and then you look at its replay buffer at some time step。

So while training so while it's like a medium sort of expert。

 you collect it the transition its' experience so far and make that as the offline data' it's something which is like our framework is very agnostic to what offline data that you use so I've not discussed so far but make something that in our experiments is based on traditional benchmarks got it so the reason I ask isn't I'm sure that your framework can accommodate any offline data。

 but it seems to me like the results that you're about to present are going to be heavily contingent on what that that data collection policy is。

Indeed， indeed， and also so we will， I think I have a slide where we show an experiment where the amount of data can make a difference in how we compare with baselines and essentially we will see how distant transformer especially shines when there is small amounts of offline data。

知道。Okay， cool。 thank you。Okay great questions so let let's go ahead so we have defined our model which is going to look at these tra trees and now let's see how we train it so very simple we are trying to fill actions。

 we'll try to match them to the ones we have in our data set if they are continuous using the mean square error if they are discrete and we can use across entropy。

😊，Um。But there is something very deep in here for our research。

 which is that these objectives are very stable to train and easily regularize because they've been developed for supervised learning。

In contrast， what R is more used to is dynamic programming。

 style objectives which are based on the Belman equation。

And those end up being much harder to optimize and scale。

 and that's why you see a lot of the variance in the results as well。

Okay so this is how we train the model now how do we use the model and that's the point about trying to do rollouts for the model so here again this is going to be similar to doing an autoregressive generation there was an important token here which was the returns to go。

And what we need to set during evaluation， presumably we want export level performance because that will have the highest returns。

 so we set the initial returns to go。Not based on a tragic tree because now we don't have a tragic tree we're going to generate a tragic tree so this is at entrance time。

 so we will set it to the expatory term for instance。

So in code what this whole procedure would look like is basically you set this returns to go token to as some target return and you set your initial state to one from the environment distribution of initial states。

And then you just roll out your decision transformer。So you get a new action。

 this action will also give you a state and reward from the environment。

 you append them to your sequence and you get a new returns to go and you take just the context and key because that's what's used by the transformer to making predictions and then field it back to the distant transformer。

So it's regular auto regressive generation， but the only key point to notice is how you initialize the transformer for RL。

So add one question here。So how much is it choice on the expert target written method。

 is it does it have to be like the mean expert reward or can it be like the maximum reward possible in demand？

😊，Like， does that try of the number really matter？That's a very good question。

 so we generally would set it to be slightly higher than the max return in the data so I think the fact that we use was 1。

1 times， but I think we have done a lot of experimentation in the range and it's fairly robust to what choice you use。

So for example， for hopper export returns about 3600 and we have found very stable performance from all the way from like 30。

5400 to even going to very high numbers like 5000 it works。嗯。Yeah， so however。

 I would want to point out that this is something which is not typically needed in regular RL like knowing the exposure return here we are actually going beyond regular Rl and that we can choose a return we want so we also actually need this information about what the exposure return is。

At that point。Thanks。There's another question。Yes， so yeah， it' just you。

 you cannot be on the regular oil， but I'm curious about。

 Do you also like restrict this framework to only offline oil cost。

If you wanna run this kind of framework in online oil。

 you'll have to determine the returns to go a priorary。 So this kind of framework。

 I think is kind of restricted to only offline oil。 Do you think so。Yes。

 and I think asking this question as well earlier that yes， I think for now this is the first work。

 so we were focusing on offline Rs where this information can be gathered from the offline data set。

啊。It is possible to think about strategies on how you can even get this online what you'll need is a curriculum so early on during training as you're gathering data。

You will set when you're doing rollouts， you will set your export return to whatever you see in the data set。

And then incremented as when we start seeing that the transformer can actually exceed that performance。

So you can think of specifying a curriculum from slow to high for what that export return could be for Biitzitchu rollout。

Chs the the decision transforming。I say cool。 Thank you。So yeah， this was about the model。

 so we discussed how this model is， what the input to these model are， what the outputs are。

 what the loss function is used for training this model， and how do we use this model at test time。

There is a connection to this framework as being one way to instantiate what is often known as all as probabilistic inference。

So we can formulate RL as a graphical model problem where you have these states and actions being used to determine what the next state is。

And to encode a notion of optimality， typically you would also have these additional a variables， 01。

02 and so on forth， which are implicitly saying that encoding some notion of reward。

 and conditioned on this optimality being true， RL is the task of learning a policy。

 which is the mapping from states to actions such that we get optimal behavior。

And if you really squint your eyes， you can see that these optimality variables and decision transformers are actually being encoded by the returns to go。

😊，So if then we give。A value that's high enough at this time during rollouts like the expert return。

 we are essentially saying that conditioned on this being。

The mathematical form of quantification of optimality。

Roll out your listen transformer to hopefully satisfy this condition。So。Yeah。

 so this was all I want to talk about the model' can you explain that， please？

What do you mean by optimality variables in the decision transformer and how do you mean like return to go？

Right， so optimality variables we can think in the more simplest context as legislative or binary。

 so one is if you。Solt the goal and zero as if you did not solve the goal。

And what basically in that case， you could also think of your decision transformer as at test time and we encode the returns to go。

We could set it to one， which will basically mean that conditioned on optimalities optimality here means。

Solving the goal as one。Generate me the sequence of actions such that this would be true。Of course。

 our learning is not perfect， so it's not guarantee we'll get that。

 but we have trained the transformer in a way to interpret the returns to go as some notion of optimality。

So is it is it if I'm interpreting this correctly， it's roughly like saying。

 show me what an optimal sequence of transitions look like。

Because you've learned the model has learned sort of both successful and unsuccessful transitions。

Exactly exactly and as we shall seen some experiments， we can for the binary case。

 it's either optimal or non optimal， but rarely this can be a continuous variable which it is in our experiment so we can also see what happens in between experimentally。

😊，哦。Okay， so let's jump into the experiments so there are a bunch of experiments and I've picked out a few which I think are interesting and give the key results in the paper but feel free to refer the paper for you and even more detailed analysis on some of the components of our module。

So this first we can look at how well does it do on offline RL。

 so there are benchmarks for the Atari suite of environments and the Open eye gym and we have another environment keyto door which is especially hard because it contains Pass Was and requires you to do credit assignment that I'll talk about later。

But across the board we see that thisilent transformer is competitive with the state of the art model free offline RL methods in this case。

 this was a version of Q learning designed for offline RL。

And it can do excellent when especially when there is long term assignment where traditional methods based on T learning would fail。

Yeah， so the takeaway here should not be that we should we are at the stage where we can just substitute the existing algorithms for the decision transformer。

 but this is a very strong evidence in favor that this paradigm which is building on transformers will permit us to better aerate and improve the models to hopefully surpass the existing algorithms uniformly。

 and there is some early evidence of that in harder environments which do require credit long term credit assignment。

Can I ask a question here about the baseline， specifically TD learning？

I'm curious to know because I know that a lot of TD learning agents are feed forward networks are these baselines do they have recurrence。

Yeah， yeah， so I think the conservative dual learning baselines here did have recur。

 but I'm not very sure so I can check back on this offline and get back to you on this。O k。

Will be okay。Thank you。Also another quick question。

 so just how exactly do you evaluate the decision transformer here in the experiment so because you need to supply the returns to go so do you use the optimal。

Like policy to get what's optimal reward and that in。

 so so here we basically look at the offline data that that is useful for training。

And we said whatever was the maximum return in the offline data， we set the。

Desire target are going to go as slightly higher than that， so 1。1 broke coefficient used。

I see so and the performance sorry I'm not really what we're seeing or else but how is the performance defined here it just like is how much reward you get actually from the Yes yes so you can specify target return to go。

 but there's no guarantee that the actual actions that you take will achieve that return so yeah so you you measure the true environmental return based on yeah I see but then like just curious like so what so are these performance the percentage you get like for like how much I guess your you recover from。

The next few months yeah so these are not percentages。

 these are some we have normalizing the returns so that everything costs between the 200。Yeah， yeah。

 I see then just wonder if you have a like a rough idea about how much like reward actually is recovered by the student transformers like does they say like if you specify。

 I want to get like 50 rewards， does it get 49 or is this even better sometimes or。

That's an excellent question and my next。So here we're going to answer precisely this question that will asked is like if you feed in the target return。

 it could be expert or it could also non be expert how well does the mall actually do in attaining it？

😊，So the x axis is what we specify as the target return we want。

 and the y axis is basically how much， how well do we actually get？For reference。

 we have this green line， which is the oracles， so which means。😊。

Whatever you desire that this student transformer gives it to you so this would have been the ideal case。

 so it's a diagonal。We also have， because this is offline RL we have in orange。

 what was the best project between data sets， so the offline data is not perfect。

 so we just plot what is the upper bound on the offline data performance。

And here we find that for the majority of the environments。

 there is a good fit between the target return we feed in and the actual performance of the model。

And there are some other observations which I wanted to take from this slide is that。

Because they can vary this notion of reward。We can in some sense do multitask R by reward condition return conditioning this is not the only way to do multitask parallel you can specify a task via natural language you can via goal state and so on forth。

 but this is one notion where the notion of a task could be how much reward you want。嗯。

And another thing to notice is occasionally these smallest extrapolate this is not a trend we have been seeing consistently。

 but we do see some signs of it， So if you look at for example Sequest。😊，Here。

 the highest return tri cleanerer data sets was pretty low。

And if we specify a return higher than that。For our decision transformer。

 we do find that the model is able to achieve。So it is able to generate tragic trees with returns higher than it ever saw in the day。

嗯。I do believe that future work in this space trying to improve this model should think about how can this trend be more consistent across environments because this would really achieve the goal of offine RL。

 which is given suboptimal behavior， how do you get optimal behavior out of it？

But remains to be seen how well this trend can be made consistent across environments。

Can I jump in with a question。Yes， so I think that last point is really interesting and it's cool that you guys occasionally see it。

I'm curious to know what happens。 So this is all condition you give as an input。

 what return you would like and it tries to select a sequence of actions that gives it。

 I'm curious to know what happens if you just give it ridiculous inputs like for example。

 you know here here the order of magnitude for the return is like 50 to 100。

 What happens if you put in 10000。Good question and this is something we tried early on I don't want to say if we went up to 10000 but we try like really high returns which not even an expert could get and generally we see this leveling performance so you can see hints of it in half Che arm and you know pong as well or Walker to some extent that if you look at the very end things start saturating。

😊，So if you exceed what is like certain threshold which often corresponds with the best tragic threshold。

 but not always beyond that everything is similar returns so it's not that it so at least one good thing is it does not degrade in performance so it would have been a little bit worrying if you specified a return of 10000 and gives you return which is。

Train0 or something really new。So it's good that it stabilizes。

 but it's not that it keeps increasing on and on， so there would be a point where the performance would get saturated。

Okay， thank you。I was also curious like so usually for transfer models you need a lot of data。

 so do you know how well like like how much data do you need like how where does it scale the data the performance？

Yeah， so here we use。The standard。UmData like the D4 RL benchmarks for Mexico。

 which I think have million u transitions in the order of millions。For。Atari。

 we used one person of the replay buffer， which is smaller than the one we used for the Mujoco benchmarks。

 and I actually have a result in the very next slide which shows dis transformer especially being useful when you have a little data。

So yeah， so I guess one question to ask before you move on。In the last slide。

 what do you mean again by return conditioning for like the multitask part？Yeah。

 so if you think about the returns to go at test time。

 the one you feed have to feed in as the starting token。As one girl。Specifying what policy you want。

喂。How is that multit？So it's multitask in the sense that because you can get different policies by changing your target return to go。

 you're essentially getting different behaviors encoded so think about for instance a hopper and you specify a return to go that's really low。

So you're basically saying， get me an agent which will just stick around its initial state。

And not going into unchared territory。And if you give it really， really high。

 then you're asking it to do the traditional task， which is to hop and go as far as possible without falling but can you qualify those multitask because that basically just means that your return conditioning is a cue for it to memorize right which is usually like one of the pitfalls of multitask。

So I'm not sure if it's a task identifier， that's what I'm trying to say。

So I'm not sure if it's memorization because like I think the purpose of this。I mean。

 like having an offline data set that's fixed is basically saying that it's very。

 very specific to if you had the same start state and you took the same actions and you had the same target fair turns。

 that would qualify as memorization。But here at test time we allow all of these things to change and in fact they do change so your initial state would be different。

 your target return it could be very could be a different scalar than one you ever saw during training。

It yeah and so so essentially the model has to learn to generate that behavior starting from a different initial state and maybe a different value of the target return than it saw during during during training if the dynamics are stochastic that also makes so that even if you memorize the actions you're not guaranteed to get the next the same next state。

 so you would actually have a bad correlation with the performance of the dynamics are also stochastic。

Also， I was very curious how much time does it take to train business transformer？In general。

So it takes about a few hours， so I want to say like about。45 hours。

Depending on what quality GPU use， but yeah thats that's a reasonable estimate y call it thanks。Okay。

 so actually while doing this experiment， this project。

 we thought of a baseline which we were surprised as not there in previously traditional on offine RL but makes very much sense and we thought we should also think about whether decision transformers are actually doing something very similar that baseline and the baseline is what we call as person behavioral cloning。

So behavioral clo， what it does is basically it ignores the returns。

And simply imitates the agent looking by just trying to map the actions given the current states。

This is not a good idea with an offline inter set which will have project trees of both low returns and high returns。

So traditional vehicle cloning， it's common to see that as a baseline in offline Ral methods and it is unless you have a very high quality data set。

 does it is not a good baseline for offline RL。However， there is a version that we call as Per BC。

 which actually makes quite a lot of sense and in this version。

We filter out the top tra trees from our offline set。

 once stop the ones which have the highest rewards。

 you know the rewards for each transition you calculate the returns of the tra trees and you take the tra trees with the highest returns and keep a certain percentage of them。

 which is going to be hyperparameter here。And once you keep those top fraction of your tragic trees。

 you then just ask your module to imitate them。Whi so imitation learning also uses especially when it's used in the form of behavioral planning。

 it uses supervised learning， essentially it's a supervised learning problem so you could actually also get supervised learning objective functions if you did this filtering step。

Oh。So。And what we find actually that for the moderate and highal regimes。

 the recent transform is actually very comparable to Pu and VC so it's a very strong baseline which I think all the future work in offline Rs should include there's actually an eyeC submission from last week which has a much more detailed analysis on justice this baseline that introduced in this paper。

And。What we do find is that for low data regimes the dis transformer does much better than person behavioral clo。

 so this is for the Atari benchmarks where like I previously mentioned we have a much smaller data set。

As compared to the Mojoko environments， and here we find that for even after varying the different fraction of the percentage hyperparameter here。

 we are generally not able to get the strong performance status in transformer gets。

So 10% DCc basically means that we filter out and keep the top 10% of the trajectories if you go even lower then the start data set becomes fairly small so the baseline become meaningless。

But for even the reasonable ranges， we'd never find the performance matching that of the transformers for these Atari benchmarks。

呃的铁 I may。So I notice in table three， for example， which is not this table with one just before in the paper。

 there's a report on the CQL performance， which to me also feels you know intuitively pretty similar to the percent BC in the sense of like you pick trajectories you know performing well and you try and stay roughly within sort of the same kind of policy distribution。

It state space distribution。I was curious on this one。

 do you have a sense of what the CQL performance was relative to say the percent BC performance here？

So that's a great question， question is that even for CQL you rely on this notion of pessimism where you want to pick trajectories where you're more confident in and try to make sure policy remains in that region。

 so I don't have the numbers with CQL on this table。

 but if you look at the detailed results for Atring。

Then I think should they should have the SQL for sure because that's the numbers we are reporting here so I can tell you what the sQL performance is actually pretty good and it's very competitive。

When the decision transformer for aary？So this TD learning baseline here is SeQ。

So financially by extension， I would imagine it doing better than person B。Yeah。

And apologize apologize if this was mentioned， I just missed it。

 but if you have the sense that this is basically like a failure of CQL to be able to extrapolate well or sort of stitch together different parts of trajectories。

Whereas a decision transformer can sort of make that extrapolation between you have like the first half of one trajectory is really good the second half of one trajectory is really good so you can actually piece those together a decision transformer where you can't necessarily do that with CQL because the path connecting those may not necessarily be well covered by the behavior policy。

Yeah， yeah， so and this actually goes to one of the intuitions which I did not emphasize too much。

 but we have a discussion on the paper where essentially why do we expect a transformer or any model for that matter to look at offline data that's suboptimal and get something a policy which generates optimal loadouts？

The intuition is that as Scott was mentioning， you could perhaps。Stitch together。

Good behaviors from suboptimal trajectories and that stitching could perhaps lead to a behavior that was better than anything you saw in individual tra trees in your data set。

It's something we find earlier evidence of small skill experiment for graphs。

 and that's really our hope also that something that will transformers very good at because it can attend to very long sequences so it could identify those segments of behavior。

 which when stitch together would give you optimal optimal behavior。So。

And it's very much possible that is something unique to this in transformers and something like CQL would not be able to do person BCC because it's filtering out the data is automatically being limited and not being able to do that because those segments of good behavior could be in trajectories which overall do not have a high return。

 but so if you filter them out， you are losing all of that information。Okay。

 so I said there is hyperparameter the context in K and like with most of perception。

 one of the big advantages of transformers opposed to other sequence models like LSTMs is that they can process really large sequences。

And here at a first glance， it might seem that being Markcoan would have been helpful for RL。

 which also was a question that was raised earlier。

 so we did this experiment where we did compare performance with context and K equals1 and here we had context and between 30 for the environments in 50 for pong and we find that increasing the context length is very。

 very important to get good performance。Okay。Now呃。So so far I've showed you how this in transformformer。

 which is very simple， there was no slide I had which was going into the details of dynamic programming。

 which is the crux of most R， this was just pure supervised learning in an autoregressive framework that was getting us as good performance。

嗯。What about cases where this approach actually starts outperforming some of the traditional methods are so to probe a little bit further we start looking at sparse reward environments and basically we just took our existing mojoku environments and then instead of giving it the information for reward for every transition we fed it the cumulative reward at the end of the trajectory so every transition will have a zero reward except the very end where you get the entire reward at once it's a very sparse reward perform scenario for that reason。

And here we find that。哦。Compared to the original denses results， the delayed results for Dt。

 they will deteriorate a little bit， which is expected because now you are withholding some of the more fine grain information at every time step。

 but the drop is。Not too significant compared to the original E performance here。

 whereas for something like CQL there is a drastic drop in performance so CQL suffers quite a lot in sparse reward scenarios。

 but the distant transformer does more。And just for completeness if you also have performance of behavioral training and person behavioral clo。

 which because they don't look at reward information。

 except maybe person BC looks at only for preprocessing the data set。

 these are agnostic to whether the environments as spa wordss or not。

Would you expect this to be different if you were doing online R。嗯。

What's the intuition for it being different April I would say no。

 but maybe I'm missing out on a key piece of intuition behind that question。嗯。

I think that because you're training like offline， right。

 like your're the next input will always be the correct action in that sense。

 So you don't just like deviate and like go off the rails technically because you just don't know。

 So I could see like how online would have like a really hard。

Like cold start basically because it just doesn't know and it's just happeningping in the dark until it like maybe eventually hits the jackpot。

Right， right I think I agree that so that's a good piece of intuition out there that yeah。

 I think here because offline R is really。Getting rid of the trial and error aspect of it and firstpa reward environment that would be harder so。

The drop in DT performance should be more prominent there。

I'm not sure how it would compare with the drop performance for other algorithms but it does seem like an interesting。

Set up to test DTN。Well and I did to maybe i'm wrong here。

 but my understanding with the decision transformer as well is this is this critical piece that in the training you use the rewards to go right so is it is it not the sense that essentially like。

は。For each trajectory from the initial state based on the training regime。

The model has access to whether or not the final result was a success or failure， right？

But that's sort of that's sort of the unique aspect of the training regime for decision transformers whereas in CQL。

My understanding is that it's based on sort of a per transition。

Training regime and so each transition is decoupled somewhat to what what the final reward was。

 is that correct？Yes， although like one difficulty which at a first glance you kind of imagined the discipline transform we're having is that。

That initial token will not change throughout the trajectory。

Yeah because it's this past reward scenario， so except the very last token where it will drop down to zero all of a sudden the token remains the same throughout。

But maybe that that but I think you're right that maybe just even at this start。

Peding it in a manner which looks at the future rewards that you need to get to is perhaps one part of the reason why the drop in performance is。

Not noticeable。Yeah， I mean， I guess one sort of abl experiment here would be if you change the training regime so that only the last trajectory had the award。

 but I'm trying to think about whether or not that would just be compensated for by sort of the attention mechanism anyway。

And vice versa right， if you embedded that reward information into the CQL training procedure as well。

 I'd be curious to see what would happen there。Yeah， a good experiments。Okay， so related to this。

 theres another environment we tested I gave you a brief preview of the results in one of the earlier slides。

 so this is called the keycudo environment。😊，And it has three phases。

 so in the first phase you the agent is placed in a room with the key。

A good agent will pick up the key and then they do it replaced in an empty room。And in phase three。

 it will be placed in a room at the door where it will actually use the key that it collected in phase one if it did to open the door。

😊，So essentially the agent is going to receive a binding reward corresponding to whether it reached and opened the door in phase three。

Conditioned on the fact that it。Did pick up the key in phase one。

So there is this national notion of that you want to assign credit to something that happened to an event that happened really in the past。

 so it's very challenging。And sensible scenario if you wanted to estimate models for how well they are at long term credit assignment。

And here we find that so we tested it for different amounts of project trees。

 so here the number of project trees basically see how often would you actually see this kind。

Of behavior and the listen transformer is and person behavioral to both of these actually baseline。

Do do much better than other models which struggle at this。does。哦。

There's a related experiment there which is also of interest。

 so generally a lot of oral algorithms have this notion of an actor and a critic。

 actor is basically someone that takes actions condition on the states to think of a policy。

 a critic is basically evaluating how good these actions are in terms of achieving a long term。

 in terms of the cummulative sum of rewards in the long term。

This is a good environment because we can see how how well the distant transformer would do if it was trained as a critic so here what we did is。

Instead of having the actions as the output target， what if you substituted that with the rewards？

So that's very much possible， we can again use the same causal transformer machinery to only look at transitions in the previous time step and try to play the reward。

And here we see this interesting pattern where in the three phases that we had in that key to door environment。

We do see the reward probability changing very much in how we expect so basically the three scenarios so the agent。

 the first scenario let's look at blue。In which the agent does not pick up the key in pays fund。

So the reward probability， they all start around the same。

 but as it becomes apparent that the agent is not going to pick up the key。

 the reward starts going down。And then it stays very much close to zero throughout the episode because there is no way you will have the key to open the door in the future phases。

If you pick up the key。There are two possibilities。哦。

Which remain which are essentially the same in phase two where you are in an empty room。

Which is just a distractor to make the episode really long。But at the very end。

 the two possibilities are one that you take the key and you actually reach the door。

 which is the one we see in orange and brown here where you see that the reward probability goes up。

😊，And is is this other possibility that you actually pick up the key but do not reach the door in which case。

 again， it starts seeing that the reward probability that's predicted starts going down？

So the takeaway from this experiment is that mission transformers are not just great actors。

 which is what we've been seeing so far in the results from the optimized policy。

 but they are also very impressive critics in doing this long-tered and assignment where the reward is also very sparse。

So just to be correct， are you predicting the rewards to go at each time stuff or is it is this like the the like the reward at each time stuff？

But you predicting。So this was the rewards to go and I can also check my impression was in this particular the experiment it didn't really make a difference。

 whether we were predicting rewards to go or the actual rewards。

But I think Lu returns to go further this one。Also most curious so how do you get the probability distribution on the rewards。

 is it just like you just evaluate a lot of different episodes and just part the rewards or are you explicitly predicting some sort of distribution？

Oh， so this is a binary reward so you can have a probabilistic outcome。哪的。have a question。 yeah。

 So generally， we will call like some， something predicts the state value or state actual value as critic。

 But in this case， you， you ask decision transformer to only predict the reward。

 So why should you still call it a critic。So I think the analogy here gets a bit clear with returns to go。

 like if you think what returns to go， it's really capturing that essence that you want to see the future rewards so I mean it's just going to predict the return to go instead of single step reward。

 right？Yeahや， yeah。Okay， so， so if you're gonna predict the returns to go is kind of counterintuitive to me because in phase  one。

 when the agent is still in the key room， I think you should have a like high returns to go if it pick up the K。

But in in the plot you， you know， in the key room， the agents pick up K and the agent that。

Didn't pick tea has the same kind level of returns to go。 So that's quite countertuitive to me。

I think this is reflecting on a good property， which is that your distribution。Like if you interpret。

 it turns to going the right way。In phase one， you don't know which of these three outcomes are really possible and phase one also I'm talking about the very big meaning basically。

 slowly you will learn about it， but essentially in phase one。

 if you see the returns to go as one or zero for all three possibilities are equally likely。

And all three possibilities， so if we try to evaluate the predicted。

Reward for these possibilities it shouldn't be the same。

Because we really haven't done how we don't know what's going to happen in phase three sorry it's my mistake because previous that salter green line is the agent which doesn't pick up the。

 but it turns out a blue line is agent which doesn't pick up。Okay， so yeah， it's my best take okay。

 It makes sense to me。 Thank you。Alsomosts not like fully clear from the paper。

 but did you do experiments where like predicting both the actions and both the rewards to go and like does it like。

😊，K candidate liberal performance if we didn't work together so actually we did some preliminary experiments on that and it didn't help us much however I do want to again put in a plug for a paper that came concurruently tragically transform summer which tried to predict states actions and rewards actually all three of them they were in a modelal based set up where it made sense also to。

Try to learn each of the components like the transition dynamics， the policy。

 and maybe even the critic in their setup together。We did not find any significant improvements。

 so in favor of simplicity and keeping it models free， we did not try to predict them together。Go it。

Okay， so the summary。We show dis transformers， which is。

A first work in trying to approach R based on sequence modeling。

The main advantages of previous approaches is it's simple by design。

 the hope is with further extensions， we will find it to scale much better than existing algorithms。

It is stable to train because the loss functions we are using have been tested and are traded upon a lot by。

Research and perception。And in the future， we will also hope that because of these similarities with in the architecture and the training with how perception based tasks are conducted。

 if it also be easy to integrate them within this loop so the states the actions or even the task of interest。

 they could be specified based on。Perceptual based cs。

 or you could have a target task being specified by natural language instruction and because these models can very well play with these kinds of inputs。

 the hope is that they would be easy to integrate within the decision making process。And emly。

 we saw strong performance in the range of offline R settings。

And especially with performance in scenarios which required us to do long term assignment。

So there's a lot of future work， this is definitely not the end。

 this is a first work in rethinking how do we build our agents that can scale and generalize。嗯。

A few things that I picked out， which I feel would be very exciting to extend。

The first is multimodality， so really one of our big motivations with going after these kinds of models is that we can combine different kinds of inputs both online and offline。

To really build decision making agents which work like humans。

 we process so many inputs around us in different modalities and we act on them so we do take decisions and we want the same to happen in artificial agents and maybe decision transformers is one important step in that route。

Multiitaask so I should or I described very limited form of multitasking here。

 which was based on the desired returns to go。But it could be more richer in terms of。

Specifying a command to be a robot or a desired goal state， which could be， for example， even visual。

So trying to better explore the different multitask capabilities of this model would also be an interesting extension。

Finally， multi agent。As human beings we never act in isolation。

 we are always acting within an environment that involves many， many more agents。

 things become partially observable in those scenarios。

 which plays to the strengths of distant transformformers being non-marovviian by design。

 so I think there is great possibilities of exploring even multi agentent scenarios where the fact the transformers can process very large sequences compared to existing algorithms could again help build better models of other agents in your environment and act。

So that yeah， there's some just useful links in case you're interested the project website。

 the paper and the code are all public。And I'm happy to take any more questions。UOkay， so I said。

 thanks for the good talk。 really appreciate it。 Everyone had a good time here。诶。

So I think we are like near the class limit so usually I have like a round of record five questions for the speaker that just like the students usually know。

 but if someone is hurry， you can you just like ask general questions folks。😊。

Before we stop the recording， so if anyone wants to like leave earlier at this time。

 just feel free to ask your questions。Otherwise， I would just like continue on。

So what do you think is like the future of like transformers in LL do you think they will take over like they've already taken over like language and vision。

 do you think for like model based and like model field learning。

 you think like youll see a lot more like transformers pop up in LL literature。Yes。

 I think we'll see a flurry of work， if not already。

 we have there's so many works using transformers at this year's eyeC conference。Having said that。

 I feel that an important piece of the puzzle that needs to be solved is explorationiration。

 It's non trivial， and it will。my guess is that you will have to forego some of the advantages that I talked about for transformers in terms of loss functions to actually enable explorationir。

So it remains to be seen whether those modified dose functions。

For exploration actually hurt performance significantly。

 but as long as we cannot cross that bottleneck， I think it is， I'm not。

 I do not want to commit that this is indeed the future of our。

Go it also your question Sure I'm not sure understood that point。

 so you're saying that in order to apply transformers in RL to do exploration there have to be particular loss functions and and they're tricky for some reason。

Could you explain more like what are the modified loss functions and why do they seem tricky？

So a sense in exploration。You have to do the opposite of exploitation， which is not a with you。

And there is right now no nothing inbuilt in the transformer right now。

 which encourages that sort of random behavior。Where you seek out。

Unfaili unfamiliar parts of in state space。That is something which is inbuil into traditional algorithms so usually has some sort of entropy bonus to encourage explorationir and those are the sort of modifications which one will also need to think about if one were to use this transforms for online RL So what happens if somebody I mean just naively suppose I have this exact same setup and the way that I sample the action is I sample epsilon greedily or I create a boltzman distribution and I sample from that I mean what happens it seems。

That's what OrL does so does what happens。So ArL does a little bit more than that。

 it indeed does those kinds of things where it would change the distribution， for example。

 that your postal distribution sample from it。But it's also， there are these。

 as I said the devil lies in the detail， it's also about how it controls that explorationir component with the exploitation。

😊，And it remains to be seen whether that is compatible with distant transformers。

I don't want to jump the gun， but I would say it it's I mean。

Preliminary evidence suggests that it's not directly transferable exact same setup to the online case that's what we have found。

 there has to be some adjustments to be made to make but we are still figuring it out。

So the reason why I ask is as you said the devil's in the details and so someone naively like me might just come along and try doing what's an RL。

 I want to hear more about this， so you're saying that what works in RL may not work for decision transformers。

Can you tell us why， like what pathologies emerge？What are those devils？Hiding in the details。

also remind you like the sorry we are like also over time so definitely okay turn feel free to like follow up but like to me that's really exciting and I'm sure that it's tricky。

Yeah I will just ask like two more questions and like you finish sh those so one is like Eric did you think like something like this transformer is a way to solve a k center problem in oil instead of using some sort of like discon factoror。

😊，This isSo that can you repeat the question sorry Yes。

 So I think usually in parallel we have to allow on some sort of disor to encode the rewards to go that but it in transformableer transformer is able to do this credit assignment without that。

 So do you think like something like this book is like the way you should do it like we should。

Like instead of having some discount， try to directly predict the rewards。

So I would go on to say that I feel that discount factor is an important consideration in general and it's not incompatible with discipline transformers。

 So basically what would change and I think the code actually gives that functionality where。

The returns to Google would be computed as the discounted sum of rewards。

And so it is very much compatible， so there's scenarios where our context length is still is not enough to actually capture the long term behavior we really need for credit assignment。

 maybe traditional tricks that are used could be brought in back to solve those kinds of problems。

Got it。 Yeah， also thought like when I was reading the distance transform work that the interesting thing is。

Then you don't have a fixed gamma like a gamma is like usually a hypopenimeter like。

You don't have a fixed gamma so do you think like can you also like learn this thing and could this also be like。

 can you have a different gamma for each gesture or something， possibly？

That would be interesting actually， I had not thought of that。

 but maybe learning to predict the discount factor could be another extension of this work。😊，Also。

 do you think like this like a decision transformer work is like like is it compatible with Q learning so if you have something like EQL stuff like that。

 can you also implement those sort of loss functions？😊，On top of a decision transformer。

So I think maybe I could imagine ways in which you could encode pessimism in here as well。

 which is key to how CQL works， and actually most of oftenal algorithms work。

 including the model based ones。Our focus here deliberately was to go out is simplicity because we feel that part of the reason why oural literature has been so scattered as well if you think about different sub problems that everyone tries to solve has been because everyone's tried to。

😊，Pick up on IDOs which are very well suited for that narrow problem like for example。

 you have whether you're doing offline or you're doing online or you're doing imitation。

 you're doing multitals and all these different variants。And so by design， we did not we were very。

 we did not want to incorporate exactly the components that exist in the current algorithms because then it just where it starts looking more like。

Architecture change that opposed to a more conceptual change engine to thinking about RS sequence modeling。

Very generally。cor it， yeah just。So do you think like we can use some sort of like like3 learning objectives instead of supervised learning？

诶。It's possible and maybe like I'm saying that for certain like for online RL， it might be necessary。

😊，Of， we were happy to see it was not necessary。But it remains to be seen more generally for a transformer model or any other model for that matter。

 encompassing R more broadly whether that becomes a necessity。哦也。Well， thanks for your time。

 this was great。

![](img/041a2ae7aae080f075f80cd7c4c33a68_4.png)