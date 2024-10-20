# 斯坦福 GPT／Transformer 原理介绍 (中英文双字幕) - P10：10.Represent part-whole hierarchies in a neural network, Geoff Hinton - life_code - BV1X84y1Q7wV

![](img/a6e82a7e40b8f10bedf5eedaad6d0492_0.png)

Before we start I gave the same talk at Stanford quite recently。

 I suggested to the people inviting me I could just give one talk and both audiences come but they will prefer it as two separate talks so if you went to this talk recently I suggest you leave now you won't learn anything new。



![](img/a6e82a7e40b8f10bedf5eedaad6d0492_2.png)

Okay。嗯。What I'm going to do is combine some recent ideas in neural networks。

To try to explain how a neural network could represent parthole hierarchies。

Without violating any of the basic principles of how neurons work。And I'm going to。

IEx these ideas in terms of an imaginary system。I started writing a design document for a system and in the end I decided the design document by itself was quite interesting。

 so this is just vapourware stuff that doesn't exist little bits of it not exist but。

Somehow I find it easy to explain the ideas in the context of an imaginary system。

So most people now studying neural networks are doing engineering and they don't really care if it's exactly how the brain works。

 they're not trying to understand how the brain works， they're trying to make cool technology。

And so 100 layers is fine in a ressonnet， weight sharing is fine in the convolutionary neuralette。

Some researchers， particularly computational neuroscientists， investigate neural networks。

 artificial neural networks in an attempt to understand how the brain might actually work。

I think weve still got a lot to learn from the brain。

And I think it's worth remembering that for about half a century。

 the only thing that kept research on neural networks going was the belief that it must be possible to make these things learn complicated things because the brain does。

So。Every image has a different pass tree， that is the structure of the holes and the parts in the image。

And in a real neural network， you can't dynamically allocate。

 you can't just grab a bunch of neurons and say， okay， you now represent this。

Because you don't have random excess memory， you can't just set the weights of the neurons to be whatever you like What a neuron does is determined by its connections and they only change slowly。

At least probably mostly the change slightly。嗯。So the question is if you can't change what neurons do quickly。

How can you represent a dynamic past tree？In symbolic AI it's not a problem。

 you just grab a piece of memory that's what it normally amounts to and say this is going to represent a node in the past and I'm going to give it pointers to other nodes。

 other bits of memory that represent other nodes， so there's no problem。For about five years。

 I played with a theory called capsules。Where。You say because you can't allocate neurons on the fly。

 you're going to allocate them in advance， so we're going to take groups of neurons and we're going to allocate them to different possible nodes in a poitory。

And most of these groups of neurons for most images are going to be silent。

 a few are going to be active。And then the ones that are active。

 we have to dynamically hook them up into a past tree。

 so we have to have a way of roing between these groups of neurons。So that was the capsules theory。

And I had some very competent people working with me who actually made it work。

 but it was tough going。My view is the side ideas want to work and some ideas don't want to work and capsules were sort of in between things like back propagation just want to work you try them and they work there's other ideas I've had that just don't want to work capsules were sort of in between and we got it working。

But I now have a new theory that could be seen as a funny kind of capsules model in which each capsule is universal。

 that is instead of a capsule being dedicated to a particular kind of thing。

Each capsule can represent any kind of thing。But hardware still comes in capsules。

Which are also called embedding sometimes。So。The imaginary system I'll talk about is called Gm。

And in Gam。Hardware gets allocated to columns。And each column contains multiple levels of representation of what's happening in a small patch of the image。

So within a column， you might have a lower level representation that says it's a nostril。

And the next level up might say it's a nose and the next level up might say a face。

 the next level up a person on the top level might say it's a party， that's what the whole scene is。

And the idea for representing part hollow hierarchies is to use islands of agreement between the embeddings at these different levels。

So at the scene level， at the top level， you'd like the same embedding for every patch of the image because that patch is a patch of the same scene everywhere。

At the object level， you'd like the embeddings of all the different patches that belong to the object to be the same。

So as you go up this hierarch， you're trying to make things more and more the same。

And that's how you're squeezing redundancy out。The embedding vectors are the things that act like pointers and the embedding vectors are dynamic。

 they're neural activations rather than neural weights。

 so it's fine to have different embedding vectors for every image。

So here's a little picture if you had a one dimensional row of patches。

These are the columns for the patches。And。You'd have something like a convolution on neuralness as the front end。

And then after the front end you produce your lowest level embeddings to say what's going on in each particular patch and so that bottom layer of black arrows they all different Of course these embeddings are thousands of dimensions。

Maybe hundreds of thousands in your brain。And so a two dimensional vector。Isn't right。

 but at least I can represent whether two vectors are the same by using the orientation。

So at the lowest level， all the patches will have different representations。But the next level up。

The first two patches， they might be part of a nostril， for example。And so。嗯。Yeah。

 they'll have the same embedding。But the next level up。

The first three patches might be part of a nose。And so they'll all have the same embedding。

Notice that even though what's in the image is quite different。At the part level。

 those three red vectors are all meant to be the same。

So what we're doing is we're getting the same representation for things that are superficially very different。

😊，We're finding spatial coherence in an image by giving the same representation to different things。

😊，And at the object level， you might have a nose and then a mouse。And they're the same face。

 they're part of the same face and so all those vectors are the same and this network hasn't yet settled down to produce on the unseen level。

So the islands of agreement are what capture the past tree。

 now they're a bit more powerful than a past tree， they can capture things like shut the heck up。

You can have shut an up can be different vectors at one level， but at a higher level。

 shut an up can have exactly the same vector， namely the vector for shut up。

And they can be disconnected so you can do things a bit more powerful than a context free grammar here。

 but basically it's a past true。If you're a physicist。

You can think of each of these levels as an icing model。

With real valued vectors rather than binary spins。And you can think of them being coordinate transforms between levels。

 which makes it much more complicated， and then this is a kind of multi level icing model。

But with complicated interactions between the levels， because， for example。

 between the red arrows and the black arrows above them。

 you need the coordinate transform between a nose and a face， but we'll come to that later。

If you're not a physicist， ignore all that because it won't help。

So I want to start and this is I guess is particularly relevant for a natural language course where you're some of you are not vision people。

By trying to prove to you that coordinate systems are not just something invented by Descarte。

 coordinate systems were invented by the brain a long time ago。

 and we use coordinate systems in understanding what's going on in an image。

I also want to demonstrate the psychological reality of past trees for an image。

So I'm going to do this with a task that I invented a long time ago。

In the 1970s when I was a grad student， in fact。And you have to do this task to get that full benefit from it。

So I want you to imagine on the tabletop in front of you， there's a wireframe cube。

And it's in the standard orientation for a cube is resting on the tabletop。

And from your point of view。There's a front bottom right hand corner。And a top back left hand corner。

 here we go。ok。The front bottom right hand corner is resting on the tabletop along with the four other corners。

And the top back left hand corner is at the other end of a diagonal that goes through the center of the cube。

Okay， so far so good。Now what we're going to do is rotate the cube so that this finger stays on the tabletop。

And the other finger is vertically above it like that。😊，This finger shouldn't have moved。ok。

So now we've got the cube in an orientation where that thing that was a body diagon is now vertical。

And all you've got to do is take the bottom finger because that's still on the tabletop and point with the bottom finger to where the other corners of the cube。

So I want you to actually do it off you go， take your bottom finger。

 hold your top finger at the other end of that diagonal that's now be made political and just point to where the other corners are。

And。Luckily zoom so most of you， other people won't be able to see what you did and I can see that some of you aren't pointing and that's very bad。

So most people。Point out four other corners and the most common response is to say they're here。

 here， here and here， they point out four corners in a square halfway at that axis。嗯。That's wrong。

 as you might imagine， and it's easy to see that it's wrong because if you imagine the cube。

 the normal orientation。And camp the corners， there's eight of them。And these were two corners。

So where did the other two corners go？So one theory is that when you rotated the cube。

 the centpetal forces made them fly off into your unconscious， that's not a very good theory。So。

What's happening here is you have no idea where the other corners are unless you're something like a crystallographer。

You can sort of imagine bits of the cube， but you just can't imagine this structure of the other corners。

 what structure they form。And this common response that people give are four corners in a square。

Is doing something very weird。Is trying to is saying well okay I don't know I don't know whether it's of a cube bar but I know something about cubes。

 I know the corners come in fours， I know a cube has this fourfold rotational symmetry or two planes of bilateral symmetry but right angle s rather。

And so what people do is they preserve the symmetries of the cube in their response。

They give four corners in a square。Now， what they've actually pointed out if they do that is two pyramids。

Each of which has a square base， one's upside down， and they're stuck base to base。

So you can visualize that quite easily， a square based pyramid with another one stuck underneath it。

And so now you get your two fingers as the vertices of those two pyramids。And。

What's interesting about that is。You've preserved the symmetries of the cube at the cost of doing something pretty radical。

 which is changing faces to vertices and vertices to faces。

The thing you pointed out if you did that was an octtoahedron。It has eight faces and six vertices。

 the cube has six faces and eight vertices。😊，So in order to preserve the symmetries you know about of the cube。

You've if you did that， you've done something really radical， which has changed faces forversities。

 andversities for faces。嗯。I should show you what the answer looks like。

 so I'm going to step back and try and get enough light and maybe you can see this cube。

So this is a queue。And。You can see that the other edges。

Former coding of Zigza ring around the middle。So I got a picture of it。

So the colored rods here are the other edges of the cube， the ones that don't touch your fingertips。

And your top finger connected to the three vertices of those flaps。

And your bottom fingers connected to the lowest three vertices there。

And that's what a cube looks like is's something you had no idea about this is just a completely different model of a cube it's so different I'll give it a different name I call it a hexahahedron。

😊，And。The thing to notice is a hexahahedron and a cube are just conceptually utterly different。

 you wouldn't even know one was the same as the other if you think about one as hegen and one as a cube。

It's like the ambiguity between a tilted square and an upright diamond。

 but more powerful because you're not familiar with it。嗯。

And that's my demonstration that people really do use coordinate systems and if you use a different coordinate system to describe things。

 and here I force you to use a different coordinate system by making the diagonal be vertical and asking you to describe it relative to that vertical axis。

Then familiar things become completely unfamiliar。😡。

And when you do see them relative to this new frame， they're just a completely different thing。

Notice that things like convolutional neural nets don't have that。

 they can't look at something and have two utterly different internal representations of the very same thing。

I'm also showing you that you do parsing， so here I've colored it so you pass it into what I call the crown。

 which is three triangular flaps that slope upward withs and outwards。Here's a different policy。

The same green flap sloping upwards and outwards， now we have a red flap sloping downwards and outwards。

And we have a central rectangle， and you just have the two ends of the rectangle。N。

 if you perceive this。And now close your eyes and ask you， were there any parallel edges there？

You're very well aware that those two blue edges were parallel。

And you're typically not aware of any other paralleles， even though you know by symmetry。

 there must be other pairs。Similarly with the crown， if you see the crown。

 and then I ask you to close your eyes and ask you where the para is， you don't see any parallelurgs。

And that's because the coordinate systems you're using for those flaps don't line up with the edges and you only notice parallels if they line up with the coordinate system you're using so here for the rectangle the parallelurgs align line with the coordinate system for the flaps they don't。

So you're aware that those two blue edges are parallel。

 but you're not aware that one of the green edges and one of the red edges is parallel。嗯。

So this isn't like the Necker cube ambiguity where when it flips。

 you think that what's out there in reality is different， things are at a different depth。

This is like next weekend we should be visiting relatives。

So if you take the sentence next weekend we shall be visiting relatives。

 it can mean next weekend what we will be doing is visiting relatives。

 or it can mean next weekend what we will be is visiting relatives。

Now those are completely different senses， they happen to have the same truth conditions。

 they mean the same thing in the sense of truth conditions because if you're visiting relatives。

 what you are is visiting relatives。And it's that kind of ambiguity。

 no disagreement around what's going on in the world。

 but two completely different ways of seeing the sentence。So。This is this was drawn in the 1970s。

 this is what AI was like in the 1970s。This is a sort of structural description of the crown interpretation。

So you have nodes for the all various parts in the hierarchy。

I've also put something on the arcs that RWx is the relationship between the crown and the flap。

And that can be represented by a matrix is really the relationship between the intrinsic frame of reference of the chrome and the intrinsic frame of reference of the flap。

😊，And notice that。If I change my viewpoint， that doesn't change at all。😡。

So that kind of relationship will be a good thing to put in the weights of a neural network because you'd like a neural network to be able to recognize shapes independently viewpoint。

And that RWX is knowledge about this shape， that's independent of viewpoint。

Here's the zigzag interpretation。And here's something else where I've added。

The things in the heavy blue boxes。They're the relationship between。😡，The aode and the viewer。

That is to be more explicit， the coordinate transformation between the intrinsic frame of reference of the crown and the intrinsic frame of reference of the viewer。

 your eyeball is that R WV。And that's a different kind of thing altogether because as you change viewpoint that changes。

 in fact， as you change viewpoint all those things in blue boxes all change together in a consistent way。

And there's a simple relationship， which is that if you take RWV， and you multiply it by RWx。

 you get Rx v。So you can easily propagate viewpoint information over a structural description。

And that's what I think a mental image is rather than a bunch of pixels。

It's a structural description with Associative viewpoint information。嗯。

That makes sense of a lot of properties of mental images。

 like if you want to do any reasoning with things like RWX， you form a mental image。

That is you fill in that you choose a viewpoint。And I want to do one more demo to convince you you always choose a viewpoint when you're solving mental imagery problems。

So I'm going give you another very simple mental imagery problem at the risk of running overtime。

Imagine that。You're at a particular point and you travel a mile east and then you travel a mile north and then you travel a mile east again。

 what's your direction back to your starting point？This isn't a very hard problem。

 it's sort of a bit south and quite a lot west， right？It's not exactly southwest。

 but it's sort of southwest。Now， when you did that task。

 what you imagined from your point of view is you went to mile East and then you went to mile north and then you went to mile East again。

I'll tell you what you didn't imagine， you didn't imagine that you went to my East and then you went to my north and then you went to my East again。

You could have solved the problem perfectly well with North not being up， but you had north A。

 you also didn't imagine this， you go a mile east and then a mile north and then a mile east again。

And you didn't imagine this， you go mile east and then a mile north and so on。

 you imagined it at a particular scale in a particular orientation and in a particular position。😊。

That's。And you can answer questions about roughly how big it was and so。

 so that's evidence that to solve these tasks that involve using relationships between things。

 you form a mental image okay， and that form mental imagery。

So I'm now going to give you a very brief introduction to contrastive learning。

So where this is a complete。Disconnect in the talk， but I'll come back together soon。So。

In contrast selfive wise learning， what we try and do is make two different crops of an image have the same representation。

嗯。There's a paper a long time ago by Becker and Hinton where we were doing this to discover low level coherence in an image。

 like the continuity of surfaces。我。The depth of surfaces。

It's been improved a lot since then and it's been used for doing things like classification。

 that is you take an image that has one prominent object in it。And you say。

If I take a crop of the image that contains sort of any part of that object。

It should have the same representation as some other crop of the image containing part of that object。

And。This has been developed a lot in the last few years。

 I'm going to talk about a model developed a couple of years ago of my group in Toronto called Sinclair but there's lots of other models and since then things have improved。

So in Simclair， you're taking an image X。You take two different crops and you also do color distortion of the crops。

 different color distortions of each crop。And that's to prevent it from using color histograms to say they're the same。

So you mess with the color so it can't use color。In a simple way。And。

That gives you Xi tilde and Xj tilde。You then put those through the same neural network F。

Then you get a representation H。And then you take your representation H and you put it through another neural network。

 which compresses it a bit。It goes to low dimensionality。

That's an extra complexity I'm not going to explain， but it makes it work a bit better。

You can do it without doing that。And you get two embedding Z and Zj。

And your aim is to maximize the agreement between those vectors。

And so you start off doing that and you say， okay， let's start off with random neural networks。

 random weights in the neural networks， and let's take two patches and let's put them through these transformations and let's try and make ZI be the same as ZJ so let's back propagate the squared difference between components of I and components of J。

And hey， Presto， what you discover is when everything collapses。For every image。

 it will always produce the same ZI and Zj。And then you realize， well。

 that's not what I meant by agreement， I meant they should be the same。

When you get two crops of the same image and different when you get two crops of different images。

Otherwise， there's not really agreement， right？嗯。So you have to have negative examples。

 you have to show crops from different images and say those should be different。

If they're already different， you don't try and make them a lot more different。

It's very easy to make things very different， but that's not what you want you just want to be sure they're different enough so crop from different images aren't taken to be from the same image。

 so if they happen to be very similar you push them apart。

And that stops your representations clapsing that's called contrastive learning。

And it works very well。So。What you can do is do unsupervised learning。

By trying to maximize agreement between the。Representations you get from two image patches from the same image。

And after you've done that， you just take your representation of the image patch。

And you feed it to a linear classifier， a bunch of weights so that you multiply the representation by a weight matrix。

 put it through a softmax and get class labels。And then you train that by。Great descent。And。

What you discover is that that's just about as good as training on label data。

 so now the only thing you trained on label data is that last linear classifier。

The previous layers were trained on unlabeled data。

And you've managed to train your representations without needing labels。

Now there's a problem with this。He works very nicely。

But it's really confounding objects and whole seas。

So it makes sense to say two different patches from the same scene。Should get the same。

Vectctor label at the seam level because they're from the same scene。

But what if one of the patches contains bits of objects A and B and another patch contains bits of objects A and C。

 you don't really want those two patches to have the same representation at the object level。

So we have to distinguish these different levels of representation。And for contrastive learning。

 if you don't use any kind of gating or attention， then what's happening is you're really doing learning at the seam level。

What we'd like is that the representations you get at the object level。Should be the same。

If both patches are patches from J A， but should be different if one patches from JA and one patches from J B。

 and to do that we're going to need some form of attention to decide whether they really come from the same thing。

And so Glom is designed to do that， it to I take contrastive learning。

And to introduce attention of the kinds you get in transformers in order not to try and say things are the same when they're not。

I should mention at this point that most of you will be familiar with Bert。

And you could think of the word fragments that are fed into Bert as like the image patches I'm using here。

And in Bt， you have that whole column of representations of the same word fragment。In book。

 what's happening presumably as you go up is you're getting。Semanically richer representations。

But in Burt， there's no attempt to get representations of larger things like whole phrases。嗯。

This one I'm going to talk about will be a way to modify Bch， so as you go up。

 you get bigger and bigger islands of agreement。So for example， after a couple of levels。

 then things like New and York will have the different fragments of York。

 I suppose it's got two different fragments， will have exactly the same representation if it was done in the G right。

And then as you go another level。The fragments of new or news probably are thin in its own right。

 but the fragments of York would all have exactly the same representation。

That had this island of agreement and that will be a representation of a compound thing and as you go up you're going to get these islands of agreement that represent bigger and bigger things and that's going to be a much more useful kind of bird because instead of taking vectors that represent word fragments and then sort of muning them together by taking the max of each for example the max of each component for example。

 which is just a crazy thing to do you'd explicitly as you're learning form representations of larger parts and the parthole hierarchy。

ok。So what we're going after in Glom is a particular kind of spatial coherence that's more complicated than the spatial coherence caused by the fact that surfaces tend to be at the same depth and same orientation in nearby patches of an image。

We're going after the spatial coherence。UThat says that if you find a mouth in an image and you find a nose in an image and then the right spatial relationship to make a face。

 then that's a particular kind of coherence。And we want to go after that unsupervised。😊。

And we want to discover that kind of coherence in images。So before I go into more details of Alom。

 I want to disclaim that。啱。For years， computer vision treated vision as you've got a static image a uniform resolution and you want to say what's in it。

That's not how vision works in the real world in the real world。

 this is actually a loop where you decide where to look。If you're a person or a robot。

You better do that intelligently。And。That gives you a sample of the objectic array。

 it turns the objectic array， the incoming light。Into a retal image and on your retina。

 you have high resolution in the middle and low resolution around the edges。

And so you're focusing on particular details and you never。

 ever process the whole image a uniform resolution。

 you're always focusing on something and processing where you taking at high resolution and everything else at much lower resolution。

 particularly around the edges。So I'm going to ignore all the complexity of how you decide where to look and all the complexity of how you put together the information you get from different extensions by saying let's just talk about the very first fixation on a novel image so you look somewhere and now what happens on that first fixation。

We know that the same hardware in the brain is going to be reused for the next fixation。

 but let's just think about the first fixation。So finally， here's a picture of the architecture。

And this is。😊，The architecture。For a single location， so like for a single word fragment in Bt。And。

It shows you what's happening for multiple frames， so Gom is really designed for video。

 but I only talk about applying it to static images。

Then you should think of a static image as a very boring video in which the frames are all the same as each other。

So。I'm showing you three adjacent levels in the hierarchy。And I'm showing you what happens over time。

So if you look at the middle level。Maybe that's the sort of major part level。

And look at that box that says level L。And that's at frame four。So the right hand level L box。

And let's ask how the state of that box， the state of that embedding is determined。So inside the box。

 we're going to get an embedding。嗯。And the embedding is going to be the representation of what's going on at the major part level for that little patch of the image。

And level L in this diagram， all of these embeddings will always be devoted to the same patch of the retinal image。

ok。The level L embedding。On the right hand side。You can see there's three things determining it there。

 there's the green arrow。And for static images， the greenarrow are rather boring。

 it's just saying you should sort of be similar to the previous state of level L。

 so it's just doing temporal integration。第一。😊，Blue arrow is actually a neural net with a couple of hidden layers in it。

I'm just showing you the embeddings here， not all the layers of the neural net。

We need a couple of hidden layerss to do the coordinate transforms that are required。

And the blue arrow。Is basically taking information at the level below at the previous time step。

So level L minus1 on frame three might be representing that I think I might be a nostril。Well。

 if you think you might be an nostril， what you predict at the next level up is a nose。What's more。

 if you have a coordinate frame for the nostril， you can predict the coordinate frame for the nose。

 maybe not perfectly， but you have a pretty good idea of the orientation position and scale of the nose。

So that bottom up neural net。😡，Is。A netch that can take any kind of part at level on mine can take an nostril。

 but it could also take a steering wheel and predict the car from the steering wheel。

And predict what you've got at the next level up。😡，The red arrow is a top down you're all that。So。

 the red arrow。Is predicting。The nose from the whole face。

 and again it has a couple of hidden layers due coordinate transforms。

Because if you know the co frame of the face and you know the relationship between a face and a nose and that's going to be in the weights of that top down you're on net。

Then you can predict that it's a nose and what the pose of the nose is。

And that's all going to be in activities in that embedding better。Okay。

Now there's all of that is what's going on in one column of hardware that's all about a specific patch of the image。

 so that's very， very like what's going on for one word fragment in Bt you have all these levels of representation。

嗯。It's a bit confusing exactly what the ratio of this is to Bt and I'll give you the reference to a long archive paper at the end that has a whole section on how this relates to Bt。

But it's confusing because this has time steps。And that makes it a little more complicated， okay。

So those are three things that determine the level andbedding， but there's one fourth thing。

Which is in black at the bottom there。And that's the only way in which different locations interact。

And that's a very simplified form of a transformer。If you take a transformer as in Bt， and you say。

 let's make the embeddings and the keys and the queries and the values all be the same as each other。

We just have this one vector。So now all you're trying to do。

Is make the level L embedding in one column。Be the same as the level L embedding in nearby columns。

But it's going to be gated， you're only to try and make it be the same。If。It's already quite similar。

So here's how the attention works。You take the level L embedding in location X， that's Alex。

And you take the level only em bedding in the nearby location Y， that or why。

You take the scalr product。You expiate。And you normalize， in other words， you do a softm。

And that gives you the weight to use。😡，In。Your desire to make。LX， be the same as L Y。

So the input produced。By this from neighbors。Is an attention weighted average of the level level embedding of nearby columns？

And that's an extra input that you get is trying to make you agree with nearby things and that's what's going to cause you to get these islands of agreement。

So back to this picture。I think。Yeah。This is what we'd like to see。😡，And the reason。

We get those that big island of agreement at the object level。

Is because we're trying to get agreement there， we're trying to learn the coordinate transform。

From the red arrows to the level above and from the green arrows to the level above。

 such that we get agreement。ok。Now， one thing we need to worry about。

Is that the difficult thing in perception。嗯。It's not so bad in language。

 it's probably worse than visual perception， is that there's a lot of ambiguity。

If I'm looking at a line drawing， for example， I see a circle。

Well a circle could be the right eye of a face or it could be the left eye of a face or it could be the front wheel of a car or the back wheel of a car there's all sorts of things that that circle could be。

And we'd like to disambiguate the circle。And there's a long line of work。

usingsing things like markco random fields， here we need a variational mark random field。

 which I'll call a transformational random field。Because the interaction between， for example。

 something that might be an eye and something that might be a mouse。

Needs to be gated by corner transforms。You know， for the。

 let's take a nose on our mouth because that's my standard thing。

If you take something that might be a nose and you want to ask。

 does anybody out there support the IR nose？Well， what you'd like to do is send to everything nearby a message saying。

😡，Um， do you have the right kind of pose and right kind of identity to support the idea that I'mknows？

And so you'd like， for example， to send out a message from the nose。

You'd send out a message to all nearby locations saying does anybody have a mouth with the pose that I predict by taking the pose of the nose。

 multiplying by the coordinate transform between a nose and a mouth and now I can predict the pose of the mouth is there anybody out there with that pose who thinks they might be a mouth？

And I think you can see you're going to have to send out a lot of different messages。😊。

For each kind of other thing that might support you， you're going to send a different message。

 so you're going to need a multi headed。transformformer and it's going to be doing these coordinate transforms and you have to corner transform the inverse transform on the way back because if the mouse supports you what it needs to support is a nose not with the pose of the mouth。

 but with the appropriate pose。So that's going to get very complicated。

 you're going have n squared interactions all with coordinate transforms。

There's another way of doing it that's much simpler， that's called a half transform。😊。

At least it's much simpler if you have a way of representing ambiguity。

So instead of these direct interactions between parts like a nose and a mouth。

What you're going to do is you're going to make each of the parts。Predict the whole。

So the nose can predict the face， and it can predict the pose of the face。

 and the mouth can also predict the face。😊，Now these will be in different columns of G。

 but in one column of G， you'll have a nose preing face。In a nearby column。

 you'll have a mouth predicting a face。And those two faces should be the same if this really is a face。

So when you do this attention weighted averaging with nearby things。

 what you're doing is you're getting confirmation。That the support。For the hypothesis you've got。

 I mean， suppose to in one column make the hypothesis， it's a face with this poems。

That gets supported by nearby columns that derived the very same embedding from quite different data。

 one derived it from the nose and one derived it from the mouth。

And this doesn't require any dynamic routing。Because the embeddings are always referring to what's going on in the same small patch of the image。

We in a column there's no routing。And between columns。There's something a bit like rooting。

 but it's just the standard transformer kind of attention。

 you're just trying to agree with things that are similar。And。Okay， so that's how Gs meant to work。

And the big problem is。😡，That if I see a circle， it might be a left eye， it might be a right eye。

 it might be a。From wheel of a car might be the battery wheel of a car because my embedding for a particular patch at a particular level has to be able to represent anything。

When I get an ambiguous thing I have to do with all these possibilities of what whole it might be part of so instead of trying to resolve ambiguity at the part level what I can do is jump to the next level up and resolve the ambiguity there just by saying things are the same。

 which is an easier way to resolve ambiguity。But the cost of that is I have to be able to represent all the ambiguity I get at the next level up。

Now it turns out you can do that we've done a little toy example where you can actually preserve this ambiguity。

But it's difficult， it's the kind of thing neural nets are good at。

So if you think about the embedding of the next level up。

You've got a whole bunch of neurons whose activities are that embedding。

And you want to represent a highly multimodal distribution。

 like it might be a car with this pose or a car with that pose or a face with this pose or a face with that pose。

All of these are possible predictions for finding a circle。And so you have to represent all that。

And the question is， can you let's do that？And I think the way they must be doing it is。

Each neuron in the embedding。Stands for an unnormalized log probability distribution over this huge space of possible identities and possible poses。

 the sort of cross product of identities impose poses。嗯。

And so the neuron is this rather the log probability distribution over that space。

And when you activate the neuron， what it's saying is add in that log probability distribution to what you've already got。

And so now if you have a whole bunch of load probability distributions。And you add them all together。

You can get a much more peaky log probability distribution。

And when you expentiate to get a probability distribution， it gets very peaky。

And so very vague basis functions。In this joint space of pose and identity and basis functions in the log probability in that space。

Can be combined to produce sharp conclusions。So。I think that's how neurons are representing things most people think about neurons as they think about the thing that they're representing。

But obviously in perception， you have to deal with uncertainty and so neurons have to be good at representing multimodal distributions。

And this is the only way I can think of that's good at doing it。That's a rather weak argument。

 I mean it's the argument that led Chomsky to believe that language wasn't learned because he couldn't think of how it was learned。

My view is neurons must be using this representation because I can't think of any other way of doing it。

ok。I just said all that because I got ahead of myself because I got excited。

Now the reason you can get away with this， the reason you have these very vague distributions in the unormalized low probability space。

Is because these neurons are all dedicated to a small patch of image and they're all trying to represent the thing that's happening that patch of image so you're only trying to represent one thing。

 you're not trying to represent some set of possible objects if you're trying to represent some set of possible objects you have a horrible binding problem and you couldn't use these very vague distributions but so long as you know that all of these neurons。

 all of the active neurons refer to the same thing then you can do the intersection。

 you can add the low probability distribution together and intersect the sets of things they represent。

Okay， I'm getting near the end， how would you train a system like this？Well， obviously。

 you could train it the way you train but you could do deep end to end training。And for Gm。

 what that will consist of and the way we trained a toy example。Is you。Take an image。

You leave out some patches of the image。You then let Gom settle down for about 10 iterations。

And is trying to fill in。The lowest level representation of the bo in the image。

The lowest level embedding。And it fills them in role and so you know back propagate that error and you're back propagating it through time in this network。

 so it will also back propagate up and down through the levels。灯。

So you're basically just doing back proation through time of the error。

Due to filling in things incorrectly， that's basically how B is trained and you could train G the same way。

But I also want to include an extra bit in the training。To encourage islands。嗯。

We want to encourage big islands of identical vectors at high levels。

And you can do that by using conrusive learning。So。If you think how the next。At the next time step。

 you think how an embedding is determined。Is determined by combining。

A whole bunch of different factors， what was going on in the previous time step。

At this level of representation in this location。What was going on at the previous time step in this location。

 but to the next level down？Of the next little a。And also what was going on at the previous time step at nearby locations。

At the same level。And the weighted average of all those things I'll call the consensus embedding。

 and that's what you use for the next embedding。And I think you can see that if we try and make the bottom up neural net on the top down neural net。

 if we try and make the predictions agree with the consensus。

The consensus has folded in information from nearby locations。

That already roughly agree because of the attention waiting。

And so by trying to make the top down and bottom up neural networks， agree with the consensus。

You're trying to make them agree with what's going on nearby locations that are similar。

And say you'll betraying it to four islands。This is more interesting to neuroscientists than to people who do natural language。

 so I'm going to ignore that。嗯。You might think it's wasteful to be replicating。

All these embeddings at the object level， so the idea is at the object level there'll be a large number of patches that all have exactly the same vector representation。

And that seems like a waste， but actually biology is full of things like that。

 all your cells have exactly the same DNA and all the parts of an organ have pretty much the same vector of protein expressions so there's lots of replication goes on in to keep things local。

😊，And it's the same here and actually that replication is very useful when you're settling on an interpretation because before you settle down you don't know which things should be the same as which other things。

 so having separate vectors in each location to represent what's going on there at the object level gives you the flexibility to gradually segment things as you settle down in a sensible way。

😊，It allows you to hedge your bets and what you're doing is not quite like clustering you're creating clusters of identical vectors rather than discovering clusters in fixed data。

 so clustering you're given the data and it's fixed and you find the clusters here the embeddings at every level they vary over time they're determined by the top down and bottom up inputs and by inputs coming from nearby locations so what you're doing is forming clusters rather than discovering them in fixed data。

And that's got a somewhat different flavor and can't settle down faster。

And one other advantage this replication is。What you don't want is to have much more work in your transformer as you go to higher levels。

But you do need longer range interactions at higher levels。

 presumably for the lowest levels you want fairly short range interactions in your transformer and they could be dense as you go to high levels you want much longer range interactions so you could make them sparse。

And people have done things like that for。But like systems。

Here it's easy to make them sparse because you're expecting big islands so all you need to do is see one patch of a big island to know what the vector representation of that island is and so sparse representations will work much better if you have these big islands of agreement as you go up so the idea is you have longer range and sparse connections as you go up so the amount of computation is the same at every level。

And just to summarize。嗯。I showed how to combine three important advances of neural networks in Gm I didn't actually talk about neural fields and that's important for the top down network maybe since I've got two minutes to spare i'm going to go back and mention neural fields very briefly。

Yeah， when I train that top down neural network。I have a problem。And the problem is。😰。

If you look at those red arrows and those green arrows。They're quite different。😡。

But if you look at the level above the object level。All those vectors are the same。And， of course。

In an engineered system， I want to replicate the neural nets in every location。

 so he's exactly the same top down and bottom up neural nets everywhere。And so the question is。

 how can the same neural net be given a black arrow？

And sometimes produce a red arrow and sometimes produce a green arrow。

 which have quite different orientations。How can it produce a nose where there's nose and a mouth where there's mags。

 even though the face vector is the same everywhere？

And the answer is the top down neural network doesn't just get the face vector。

 it also gets the location of the patch for which is producing the PA vector。

So the three patches that should get the red vector are different from different locations from the three patches that should get the green vector。

So if I use a neural network and the guess the location is input as well， here's what it can do。

 it can take the pose that's encoded in that black vector， the pose of the face。

It can take the location。In the image for which is predicting the vector of the level below。

And the pose is relative to the image too， so knowing the location in the image and knowing the pose of the whole face it can figure out which bit of the face it needs to predict at that location and so in one location it can predict okay there should be nose there and it gives you the red vector in another location it can predict from where that image patch is there should be mouth there so it can give you the green arrow。

😊，So you can get the same vector at the level above to predict different vectors in different places at the level below by giving it the place that it's predicting for and that's what's going on in neural fields。

😊，Okay now this was quite a complicated talk there's a long paper about it on archive that goes into much more detail。

And you could view this talk as just an encouragement to read that paper when I'm done。

Exactly on time。thank you。 a lot。

![](img/a6e82a7e40b8f10bedf5eedaad6d0492_4.png)