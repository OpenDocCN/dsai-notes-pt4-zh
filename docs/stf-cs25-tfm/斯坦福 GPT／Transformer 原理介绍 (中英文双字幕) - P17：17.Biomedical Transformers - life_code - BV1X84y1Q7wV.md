# 斯坦福 GPT／Transformer 原理介绍 (中英文双字幕) - P17：17.Biomedical Transformers - life_code - BV1X84y1Q7wV

![](img/867d42af88a583402ff132cc4ae7c3fa_0.png)

![](img/867d42af88a583402ff132cc4ae7c3fa_1.png)

Yeah， so making you all see the speaker notes was not part of the plan， but I'm glad to be here and。

My name is Rebek Naro。And I am a research scientist in the Health I team at Google。

A little bit more about me， growing up in India， my parents always wanted me to be a doctor。

 to be precise a medical doctor， but unfortunately I was probably not good enough to memorize all the biology textbooks that you had to do in case you wanted to crack the medical entrance examinations so I ended up becoming a computer scientist instead。

诶。But as a great man once said， you can't connect the dots looking forward。

 you only join them looking backwards， so through other long winded path。

Not too dissimilar from how we actually train our neural networks I ended up working in medicine again this time armed with this magical new tool of AI and I can tell you that my parents are far more happy with my life choices right now。

嗯。But then I were truly satisfied。But take questionsians aside。

My goal for this talk is to peel back the curtains and give you a flavor of all the innovation that is happening at the intersection of AI and biomedicine and how that is being catalyzed by transformers and large language models in particular。

So we will spend the first few minutes trying to work up from first principles why transformers and large language models are a particularly good fit for biomedical data。

 and then we will deep dive into a few papers covering a bunch of different biomedical application settings。

And finally， I'll present my views on how this field is likely going to evolve in the next few years。

And even though my voice or tone may not exactly sound that way。

 I am incredibly excited by the possibilities of AI and biomedicine。

And I think we have an incredible opportunity in front of us to advance human health and human potential and my hope at the end of this talk is you all will feel the same way as I do today and perhaps join me。

😊，So yeah， let's jump straight in why transformers in biomedicine and sorry I'm going to pick people who are in person to answer。

 so maybe if one of you could volunteer。Go for， not that's a good answer。嗯。嗯。

s飞啊后有去快玩到未 sorry sorry快玩到几个打咯佢翻系 l。Yeah， sure，'s another one as well that's an important application setting。

你系接十我事啊。Yeah great one so I think all of you were on the right track and so maybe if you just look at different kinds of you know biomedical data。

 for example what are clinical notes I think it's sequence of Dr Jabberish okay I did not say that but let's just call sequence of doctor speak or doctor notes。

Similarly， if you were to look at electronic medical records。

 what are they they are essentially sequence of like a person's encounters with the medical system。嗯。

What about proteins going deeper into the biological stack。

 they are nottdding but a sequence of amino acids linked together by peptide bonds。

And does anybody know what this is？好多水。I think that's how we store。嗯系。Sorry again。

 you're getting close。系。Anyone else？So this is in the Wecome collection in London and this is actually a printout of the full human genome。

And。No they did not cheat over here the font is super small and as you can see there's a bunch of8GCs the entire printout contains I think over 130 volumes in that shelf and each page is printed on both sides and it's a  four point font with precisely 43。

000 cactus per page so that is how big the human reference genome is more than billions of base pair。

And so again， the genome is nothing but a sequence of neotide base pairs。

 so what we are essentially seeing over here is sequences are everywhere in biomedical data and what is the best neuralelectric architecture for modeling them？

嗯。And I guess since you are all in this course， I don't have to convince you that the answer is transformers。

 right？Okay that's good， but maybe I'll just offer a few reasons over here firstly as you can see the data itself is multimodal in nature and we just saw a few examples and as someone pointed out transformers have proven remarkable at you know guing up pretty much any kind of data and we are really seeing this remarkable convergence across fields whether that's speech or NLP or vision or robotics I mean pretty much everywhere we are using transformers and I think biomedicine is no different。

😊，I think secondly， transformers are far more effective at modeling complex long range interactions over sequences。

 and this property is particularly important in the biomedical domain and we will cover this in more detail later in the talk。

And finally， as again， someone pointed out， these data sets can be quite big and you can easily get into the billions of token territory and this is where transformers with all the parallelizable operations and the relative ease of training and maybe someone should try training an LSM and an RN on these kind of datas。

 you'll realize that these are much better suited for the kind of data sets that we have in this domain over here。

So yeah， I think there are a few more reasons as well。

 but I think these are the key ones as to why transformers are particularly well suited for biomedical data sets and tasks。

Any questions so far？Okay， great， so now in the next part of this talk。

 we will dive deep into a few papers applying transformers to biomedical data。

We'll start with clinical applications first and then go gradually deeper into the biology stack looking at proteins and genomic applications as well。

 and what you will observe is that while transformers and large language models by extension are a great fit。

 often you have to innovate， not just on the modeling side。

 but also on the data and evaluation side to make these applicant scenarios really work。

And so the first paper I want to talk about over here is this recent work from our team called largege La modelss encode clinical Knowled。

The motivation for this work is actually quite straightforward so if you look at medicine it is a humane endeavor and language is at the heart of it facilitating interactions between people and those who provide care for them unfortunately if you look at a lot of medical AI systems developed till date these are all narrow single task single domain models lacking interactive and expressibility capabilities。

And as a result， what has happened is there is this discordance between what these models can do and what is expected of them by patients and you know care providers and others and this in turn has I think prevented broad uptake of medical AI。

诶。And you can see that， for example， we don't really have AI and many clinics out there like helping us with diagnosis and so on and so forth。

But the recent progress with Transform based large language models。

 it offers us an opportunity to change all of this and redesign and rethink medical AI systems with language at the heart of it。

 mediating humania interactions between doctors， researchers and patients。

And I will be amazed if I don't point out that there has been a large volume of work in this space。

 particularly in the last few years， there have been various attempts to train language models in the biomedical domain with models of various different sizes on different cor of biomedical data。

And while this is exciting， the quality bar for applications in the medical domain is actually quite high。

And so what is missing is actually is that there is actually not many good evaluation benchmarks and evaluation protocols and frameworks so we don't have the equivalent of a big bench in medicine and hopefully you guys have covered big bench before and so big bench is is benchmark where you can assess large language models across a variety of task domains and settings but we don't have an equivalent of that in the medical domain。

And further， if you look at the evaluations that are typically used in these previous studies。

 they only look at objective metrics like accuracy or natural language generation metrics like blue or cider。

 but these failed to capture the nuances of real world use cases in clinical settings。

So what we essentially need was a good benchmark and a task and also a good evaluation framework for evaluating these models。

And so to address this unmet need and assess the potential of LLMs in medicine， in our team。

 we decided to focus on the medical question answering task， why。

 because answering medical questions is actually quite challenging。

 it requires reading comprehension skills， ability to accurately recall medical knowledge and also manipulate and reason about it。

And furthermore， the Q&A task is general enough and can subsume a bunch of different application settings such as summarization of clinical notes。

 clinical decision support， and also like primary cartriaging of patient concerns and so on。

So we've identified the task the next question is what data set and so when we looked at the literature over here。

 what we saw is that there were several data sets floating around assessing model capabilities in a bunch of different settings so what we decided was we should probably just unify all of them and put together in one benchmark and so we did that and we call it multim QA and so if you look at it this benchmark now covers medical question answering data sets from a bunch of different settings such as professional medical questions like the US medical license exam style questions it also includes medical research questions those based on pububM abstracts and so on and also questions from live users and consumers asking about medical information。

And also the setting changes， it could be closed domain or open domain and the model may be expected to produce long form answer in one setting and maybe a short form answer in another setting。

And finally， we saw that while the。The Q&A data sets which covered consumer course， yeah go。

 I'll come back to this。So yeah very quickly finally when we looked at cool so the question was how do we evaluate longform answers and I'll come back to this sub later and so very quickly when we looked at the data sets that actually provided consumer medical questions we found them to be quite small in size so we decided to augment them and so we went out to Google and looked at like the most frequently asked consumer medical questions and so we curated a data set and we added that to the benchmark and we call that health search QA over here。

And so yeah， again，I'll come back with the start circular I'm sure so here are a few examples so if you look at the consumer medical questions they are quite short in nature while and so they come from the health search queue and the liveQ datas whereas I think if you look at the USM style questions these are like long viignettes and so doctors have to like really really carefully read through them and come up with the right answer which often and involves a process of emination so again very very different application settings and so the model has to really like really adapt and understand the task to do well in all these settings across the board。

And live QA is interesting because the answers， the reference answers over here were actually provided by librarians。

 so that's another good comparison point for us going ahead。And so in terms of statistics。

 we had a total of seven data sets in this benchmark， as I said， we cover professional medicine。

 medical research and consumer medical questions there again of various different sizes and can be long form short form open domain and flow domain so very diverse and I think it provides a very comprehensive evaluation of models in this medicaling setting。

So we have a task on the benchmark the next question again I think asked was how do we evaluate these models and as I mentioned before automated metrics are actually deeply unsatisfactory because they fail to capture the nuances of real world clinical applications so what we did was actually heavily inspired by some of Steven's work over here was to put together a human evaluation framework for assessing these longform answerss and。

This had two parts， the first part was evaluation by clinicians and we asked them to rate the moral responses along 12 axes pertaining to factuality of the responses。

 ability to recall medical knowledge to medical reasoning。

 and also for the potential of harm and bias in these responses。

But if you look at like the potential end users of such medical Q&A systems these are likely going to be non- expertpert layer users。

 so it is also important to get these answers evaluated by them as well and so we also additionally asked a pool of lay users as to how helpful and actionable they thought the answers were。

And so that was our evaluation framework and we also have the benchmark fix。

 so now we've moved a fun part of building and aligning LLM to the medical domain task。

So in this work we decide to build on the Palm family of language models has that been covered in the coast before。

 okay great， so but very quickly， I believe this is still the largest publicly announced densely activated decodrenly large language model with the largest one being fine 40 billion parameters in total。

A few more details， the model strain on SA 40 billion tokens， 25% of which is multilingual。

 the data come from a bunch of different sources， including social media conversations， web pages。

 books， GitHub， and Wikipedia and so on and so forth。And at that time of release。

 the model was shared the art on many NLP reasoning benchmarks and also was the first model to exceed the average human performance on Big Bennch。

Further over the last year， palm derived models were shown to be super useful in a bunch of different application settings。

 including for cogen， which was the palmM code model and robotics。

 the palm Saan model and also for answering math and science questions， which was the Minva models。

 and so we thought PAM was a very good foundation model for us to build on and use it in the medical domain as well。

And overall， I think Palmm is a true magic of engineering。

 but I will refer you all back to Acsha's paper on this for more details I think it's a must read。

And again in late October last year， Jason Way and a few others at Google Bra came out with the F palm variant of these of the palmM model and this is basically the instruction tune counterpart and this model was even better than PM and I believe it is still the sort of sort of yard on many benchmarks such as MMLu。

 TiDQA and I think it exceeds palm performance by an average of 9。4% across big bench tasks。So。

So we decided to build on the F palm model and we applied a combination of prompting strategies。

 including few short prompting， chain of thought reasoning and also cell consistency to the54 billion parameter variant。

 and we evaluated it on the multimQA data sets that had these short form MCQ questions。

And we found that this model was really really good at the time of publicationation。

 this model on the USM Me data set exceeded the previous state of the art by over 17%。

It's only for the USMLE Me area that's right and so you see that the accuracy over the previous year of Y at the time of publication went up by over 17% and I believe this was the first LLM basedDI system to obtain like a passing equivalent score which was 60% or above on this benchmark。

And similarly， when we looked at other MCQ data sets in the benchmark， for example， MCQA。

 which is a data set of Indian medical entrance examination questions。

 the model was again the state of the art on PubMC which was question answering based on Pubmed abstracts。

 that was again the model was state of the art at the time of publication and same story on MMLU clinical topics as well which include genetics。

 anatomy professional medicine， clinical knowledge and a bunch of other topics in there。So。

All this was great and then when we started looking at the scaling plots。

 what we saw was that the performance seemed to be improving as we scaled the model from 8 billion to 62 billion to 5 and40 billion。

And so what this basically suggested that these general purpose large language models trained on public internet seem to encode clinical knowledge pretty well and their medical reasoning abilities tend to scale with model parameter size。

We also did another experiment when we looked at selective prediction and we used the self-consency votes to determine when to differ。

 and this is important in clinical settings because doctors communicate when they don't know about something and if our AI systems are going to be used in clinical settings for example for diagnosis。

 they should be able to tell you when they don't know something。

And so what we observed here was this fairly crude metric we were getting like a linear improvement in performance as we changed the defral threshold and this was quite nice but in practice it is actually quite ineffient because you are generating like you know multiple decoding samples to be able to compute this metric so we need a better method ask yeah it's basically says I'm uncertain around one and that's determined based on the cell consistency works so exactly same exactly。

So theNo because they're just trained on this expert prediction task and that depends on the data the PubM QA has some answers which are maybe but again we don't explicitly find tune in the models over here so no the models are not trained。

Yeah， so that is where we have this medical runs in the long zero lot。No。

 so this is primarily based on the reference of in the data sets which is so this is all accuracy matrix so we already know between the four options of I options which one's the right one and so we just do the classification yeah so I'll come back to the condition evaluation aitrator。

So maybe is something how are you naturally uncertainty so if you know what selfconency prompting what we do is we generate multiple decos from the same model and then we see the number of times the highest ranking answer is voted and based on that you can fix a threshold and say if it's below this number I'm going to differ so if say the majority answer comes up in your selfconsency decode only like n times out of k or whatever then that if that n is too small then it's very likely the model's uncertain so that's how we differ。

嗯。嗯。So we don't really see like a paper off in this plot。

 so like it's actually that that what the rest would look like。

I think if you plot it further to flatline， but again that's not useful I mean if you're saying no to every question that's not useful at also you want to have a reasonable deferral percentage of I think that's high I think that's still high 50% is quite high but again that this is a very contrived setting but in real world use cases is probably I think that number should be much lower。

might and the data addicts， some medical products went to more important than others。

 so more question。嗯。That's right I think balanced accuracy might be a better metric but we looked at some of these data sets and one data set the s was pretty bad the Pub data set and I think no one should use it so if anyone's reporting sort of numbers on that data you should just discussrus them and'm talking about very specific people but again I think as I mentioned these accuracy metrics are good for you know publicity and pushing of benchmark numbers and so on and so forth but the real evaluation is human evaluation of the long performances and that's what I'll come to in the next one。

嗯。So so far so good right I mean we we were getting solar results on these benchmarks and we were very happy and so what we did was I mean one thing you'll observe that I have so far only reported results on multiple choice questions shortformances so what was left for us to do was to take these answers take these models and generate longform answerss to the other data sets that we had and get them human evaluated and I think that is where the real project began when we looked at the evolvevals by experts on lay people it revealed very like key gaps and limitations in the fl farm responses。

 we were often seeing that these models were hallucinating or producing incomp responses and when we asked experts like whether they preferred clinician generated answers or these model generated answerss they almost always preferred clinician generated answerss。

嗯。So it was very clear that they both。And so what these previous results showed was while these models already encode some degree of clinical knowledge to be really used in actual real settings。

 you need to align these models better to the safety critical requirements of the medical domain。

But a big challenge is we did not have any kind of supervised our feedback data。

 and so we really need the alignment technique to be data efficient。But thankfully。

 we had instruction from tuning， which was introduced by Brian Lester and a few others at Google a couple of years back。

 and how this method works is it essentially freezes the big LLM model and only learns an additional small set of prompt vectors which can then be used to condition the model that inference when doing the generation。

And the nice thing about this is it allows very easy use of the model across tasks and domains。

 and you only need to like carry around these additional prompt parameters right and these tend to be much smaller than like the the billions of parameters that you have in the other。

And the other good thing is this is very computationally efficient as well。

 so like if you were to do end to end fine tuning often in our compute infrastructure。

 even with like a few thousand examples that would take like a few days whereas with instruction prompt tuning given the data set size is also reduced the number of examples that you need is quite small and just updating the prompt token vectors it meant that we were able to get model updates in like a few hours and so that was like really fast and enabled like really quick iterations for us。

So this was how we put together the final metPm model we so this was how we put together the final metPm model so we used instructions and exemplars from a panel of expert clinicians and these are in the order of hundreds not like thousands of tens of thousands and you see a few examples over there there's an instruction followed by model answer followed by an explanation and we use that to learn the prompt vectors and so the final metPm model is basically of plan pump plus these additional softpro vector parameters which are used to align the model to the requirements of the medical domain and why this works well is because as we have seen before the model already has medical knowledge encoded in it all we need is to like least the model how to use it properly in the given application setting and that's what these prompt parameters to for us。

So the question I wanted to ask is， nowadays you've probably seen a lot about like borrow in chat。

 and given the fact that you have all of these human preferences expressed by your elevatorers。

 and you guys and you guys try playing that a reward or preference model。

Yeah I think you can think about like difference changes of model development right so this is pre deploymentloyment and release in the real world so you can't put a crappy model out there in the real world so even before doing that if you can like get like maybe 10 examples from whatever experts that you can get heard of and use that to prompt your new model that's better that's a much better starting point before you expose the model to the real world and collect references from real users at scale and so I think RLH is also much less sample efficient compared to instruction prompt tuning again because you're probably trying to update your entire model as well so I think this is a very good starting point and so they can be combined depending on how depending on the lifecycl of the model。

The data set is public， I'll talk about the results in a bit within。

You mean the model responses and what the humans are that's a good point we are so far not considering releasing them。

 but maybe we can Okay do you see a use case for that。Do have a bunch of data in the prens。

 so frame the model to express those pregnancies and use that model R chat in medical。

So if I wanted to explainYeah that's a good point， I think the evaluation data set is I'll talk about this a bit here。

 it's still small but I think if we scale it up and we are doing it right now I think we can release that and that'll be I think a good resource of what you're trying。

可。So we have the metPm model as I said， and now we took the long form answers from it and compared that to the F palm model as well as to answers generated by expert clinicians and as I said we have two parts to the human evaluation one is by expert clinicians and the other one is by LA users and so what do these results look like on the one and 40 y questions that we got these evaluation results on what we observed typically。

Across the board was when we looked at like different axes while the fl palm model would be quite terrible honestly the metT pump model would do much better and typically close the gap to expert clinicians so on this axis you see that the fl palm model has probably 60% or accuracy in terms of like scientific consensus the metP model improves on that quite a bit and closes the gap to clinicians over here。

Similar story on other axes as well over here you see the like clinicians rating on the axes of how well the model can retrieve medical knowledge。

 how well it can reason about it and again we see the same trend as in the previous slide。

so the column correct so it's evidence of correct comprehension and the rights incorrect minus no so both can be present at the same time so you can have evidence of correct comprehension also evidence of incorrect comprehension sometimes so exactly so so that's why they're not one minus over here but the trends are the same or also that's why skipable but that's a detail。

Yeah again so there's a type over here， but this one pertains to incorrect our missing content but this was an interesting one because what when we were doing this from tuning thing was we were teaching the metP model to produce longer and more complete answerss and so you'd see a few qualitative examples later but what ended up happening the process was sometimes the model was maybe producing more incorrect information so that's why you see that maybe in this particular axis the fl palm model was slightly better but again this was much worse compared to transmissions。

喂，我也在这买。It's a good question， it is more like it's something something completely out of context。

 so it may be irrelevant to the question。So that's why I would say it。Okay。

We also looked at like possible and extent and likelihood of harm。

 and again we see that with the instruction prompt tuning we're able to close the gap to expert teenagers over here。

Same on the bias access as well。嗯不能确。Can you interpret the talk？诶其该。

So like I basically see like something de and then the clinicians at like6% see would you talk more like how to clarify exactly what that？

😊，Yeah so it's basically so there might be certain conditions or pathologies or diagnosis right like cancer and if for example the clinician has not caught that or has maybe given a response that does not appropriately convey the severity of the condition then that could potentially lead to severe harm or death and so that's what we were trying to capture a here so that's a very high level overview this is。

I think a very nuanced topic and there's a framework for it called the AHRQ framework and so we've linked that in the paper as well and so I think that gives you a very detailed notion of harm and bias that I would refer to that but at a high level this is what I'm talking about how that helps。

All right， so when later I class and I say the clinician had 5。

7 a possible which means like what would I say does that mean that they recommend something or maybe they fail to recommend something yeah。

 so it's basically a misdiagnosis or maybe failing to capture the severity of a diagnosis this is typical in life threatening conditions right so。

So it's more often than not mistakes， but rather just missing out on details。Yeah。

 so I talked about bias as well and then as I said the other axes of human evaluation was with layer users and so we asked them how well does the model answer address the intent of the question and again we saw with instruction prompt during MetPm closing the gap to clinicians and then we asked them like how helpful the responses were and what we see is that while fl palm responses were considered to be helpful like 60% of the time the number improved to 80% for MetPM。

 but was still fairly lower compared clinicians at 90%。嗯。

So here are a few qualitative examples and so what you see is that physicians and this is typically because they work in time constrained settings they their answers tend to be precise and succinct but sometimes it's very hard as layer users or patients to like decipher and decode the answer and get all the full set of details right and so what I think language models like MePm can help with is actually converting the physicians speak to something that's more easily digestible by layer users and so this is where I think how these models will likely fit in clinical settings in the neural term where they are going to augment physicians in terms of like interacting with patients and other physicians and researchers as well。

Sa good。Im just wondering because we visit。It that water is down and it's a very。

If I take as a patient， you do it or not authority。

That's right I think it's subjective and so that's why I think we're still seeing like lay users rate plan performance answerss to be helpful 80% well that's much higher for physicians so it's not perfect by any means but I think this is where its there there is a complementarity element we feel over here。

And we've asked that and like when we ask people like how easy is it to like interpret doctor notes or recommendations and they often say。

 oh it's very hard I need to go back to Google search for what these terms mean。

 what these abbreviations mean and so I think this is where a language model can come and take that note and convert that into something that's more easily digestible like so I think that's the opportunity over here I feel。

So that was all on our paper， but I also want to maybe very quickly point out a very recent work which came out last week with this rather provocative title do we still need clinical language models and by clinical language models they meant smaller models which are trained in domain with clinical data such as medical notes and records and so on and so forth and what this paper basically suggests is that smaller fine-ted in domain LMs are likely better than general purpose LLMs in this paper I think the evaluate on GT3 with in context learning so I think that's a pretty interesting any observation I think there's a lot of value for smaller in domain LMs such as P GPT and a few other lib variances but I think one thing that this paper does not do is consider in context learning sorry prompt tuning and I think that's where some of the benefits of these larger general purpose LLMs shine and again we haven't done any in domain LM pretraining。

These large general purpose。Models， but that's again an option for us as well to do dling。

So you can take these5 and 40 billion parameters and then still turn it on medical nodes or whatever domain specific data that you can get hold of and hopefully that'll probably further improve the performance。

So key takeaways so far。What I want to convey is general purpose LMs。

 it looks like they do encode medical knowledge and performance on medical reasoning tasks seem to improve with scale。

 however these models I don't think can be directly used out of the box in clinical settings and they need to be aligned with the safety critical requirements of the medical domain。

And I think instruction proing is an extremely efficient technique both on the data side and also on the compute side and we should probably use it more often。

 depending on and hopefully the API starts supporting it as well。

 and these models appear to be closing the gap to expert clinicians at least on this medical question answerscing tasks and while this is hugely exciting and has profound implications you can all probably dream up and imagine the application scenarios over here。

 I think comprehensive benchmarks and evaluation frameworks are necessary in order to further assess and improve these models for real use cases。

So I'll trouble over here any questions？完成。In medicineine， there survived。哎，是。对。你啊。

A lot of it is because these data sets tend to get locked in silos with privacy and other kinds of regulations which prevent them from being put out there in the real world。

 so you have to have like hip compliant systems for storage and so on and so forth so it's very difficult to get data out of these silos and put together an open benchmark so honestly I feel like that's probably not going to improve the scale of these data sets at least the open version of these dataset sets are going to remain。

Quite small compared to the big L training data sets or the computer division data sets on natural images and on and so forth。

 but what may happen in the future is we may have like more distributed fedated evaluation settings where you take the model into these private silos and get them evaluated on so they are never exposed and put out there in the public but rather we can have these fed rate evaluation settings so I think that there's some work on that already there's a system called MeF and probably see more of them。

就等个嘅发过先。Sure， so the question over here was why medical data sets are smaller compared to natural image data sets in computer division or LMP training data sets and so on and so forth。

What do you think are some of the earliest applications of medical LOms like deployed in industry？

I think the first set of use cases are probably going to be not diagnostic in it sorry the question was what do you think are the use cases of medical algorithms in medical industry settings and so。

The answer is I think the first set of use cases that we are going to see are probably going to be non-diagnostic in nature。

 but more around like if a patient comes in and interacts with a doctor。

 can you like generate summary notes and can you do like workflow tasks such as generating letters for insurance for medications for referrals and so on and so but I think these tasks are right up the alley of large language models and I think if not already in the next six months to a year we'll see a lot of these use cases coming up and I think that's going to make doctors like care providers life much easier because right now they're spending a lot of time doing these things and not actually providing care and attending to the patient diagnostic use cases I think will take a lot more time。

 we need a lot more evaluation the data sets as we can see are probably not there evaluation frameworks are not there but I think in the long run and that is the dream setting right。

And then maybe a follow up is M， I'm assuming meed Palm is not open source。

 what do you think the best open source model is for medical？诶。

Yeah I think it depends on the so the question is what is the best open source model for medical data I think depends on the evaluation setting so I think the PM GPT model from the Stanford Foundation models group is quite strong I think GPT3 or 3。

5 or whatever variant if you can bring in some domain specific medical data and do some in domain tuning adding that model can also improve quite a bit so I think those two would be my favorite chart points over here。

So feels like part of the important itself problem after section。

It's you can just think them as vectors corresponding to a few additional tokens so it's not really human legible so the question was what do the soft prospectors look like and are they human legible and yeah the answer is no they're not。

Just a you said mentioned by very know for larger quality policy third of route and usually on sites have hospitalized moral quality infrastructure online medicine we really believe that should be learning what we have。

On site hardware， on site and machines that contain these models of the board。Sure。

 so the question was given a lot of the hospital systems and providers networks are quite low tech and don't have good enough hardware。

 do you really think fed learning could be used for distributed training of large scale LLMs？

I think we are increasingly seeing a trend towards cloud and so a lot of these hospital systems are moving their storage and data and compute to standard chart providers like AWS or as your Google cloud and so I think that helps because these systems on the back and side do have the compute to be able to like train these kind of models I think it's going to be a very gradual process so systems that have high quality infrastructure probably we're going to start with that first and then gradually work our way into the long tail but it also feels like something that will inevitably exist in the world so 10 years down the line of 15 years down the line and we have these distributed La scale relevant training systems we always think why did I even doubt that this will not exist it's so obvious it's something that has to exist because that's where all the patient data is all the interesting data is right because I think that will just happen it's just not clear where that's going to be done by one company whether that's going to be done by consortium of academic or industry groups or。

s are going to be involved on and so forth。That's right。

 so the question over here is we seeing cloud computing but we are pretty much uploading the data to the same warehouse the answer is true but again I think these are all going to be separate buckets with their own access controls and so on and so forth so that is how you can differentiate between them。

对。好。It's been a lot of。这 really的。I。it doesn't seem like that thing in sense that we're that we're used to carry。

Sure so the question was has there been any studies in MePm looking at private information in these data sets and the short answer is no one of the criteria for selecting the data sets that we used in the study was to not include any kind of personally identifiable data or clinical data of that sort and that helped like you know get this paper out on time but I think that's an important point。

It's unlikely that we're going to have a lot of PI data in public well in the public data that we are training on but。

Even when you're training on say one private corpus and then you're using it in another application setting。

 you want to ensure that the model does not leak out any kind of PHI information during a generation。

 so I think those sort of studies are necessary， we haven't got into them yet。

So does think fairly on withs and exploring good models and that people be able having groups over the same experience towards the world trying。

So the question is what are the next steps in terms of improving these models further yeah retrieval is a very important one being able to cite sources and especially taking authoritative sources and use that in generating the answers and' also communicating that to the users is very important I think how you communicating uncertainty is very important so we gotten to some extent using instruction from tu but I think that can be much much better so I think that's another big bucket again I would stress on the evaluation side like you know looking at more data sets which for example may do Q&A on health records or or other kinds of medical data I think that will be important and also extending the evaluation both in terms of scale having a diverse panel of clinician sport and also in terms of the data that you're using maybe adverarily modifying the questions to include like demographic confounders or something like that I think those are all could be interesting directions I think on the modeling side the interesting question for me is again this interplay between。

Smaller。Domain specific alarms versus large general purpose alarms and how that's going to play out there seems to be some evidence of emergence over here especially with medical reasoning and so as you can see at lower scales sometimes the performance is not good enough I mean 50% I mean that's a good number but that's just not viable and but when you get to like 80 90% products really become useful right and so that we are seeing at like you know bigger parameter sizes of these models。

But I don't know， I think it's still an open question over here。yah诶。

The question was is hallucination an issue I think it still is。

 but I believe that you can control that fairly well with instruction prompt tuning but like any kind of feedback data I think it's not terribly difficult to do and so the。

I think it might have been overblown generally， so especially when you are doing it in a particular domain。

 I think it's easier to control。I the extent to which you have been leader or like like one it looks like I just。

Recently， there's been the product above and their。一本です。Okay。Yeah。

So I'm just curious because this particular app really very。

 very relevant and talking about high bar and quality so I'm just curious if speak。Yeah。

 so the question was there is a lot of talk and noise around hallucinations and general purple cell limbs and this in this particular application domain it seems particularly relevant and so can you expand on that a little bit further sure。

What we are seeing is even with like an order of a few hundred examples from expert clinicians teaching the model how to communicate medical information。

That is good enough to get the model to maybe stop。

Hucinating or at least communicate its uncertainty in a better way so at least in this particular domain or this setting it feels more tractable to us and the reason I'm saying this is we've looked at the answers qualitatively and we are seeing that the model does not tend to like generate like super long answers or you know or like you know make like very confident predictions but rather the tone itself becomes like very reserved and it starts using terms like you know maybe this needs to be done further or something like that which communicates uncertainty so how well is that actually correlatedated with the representation underlying uncertainty that we have is still I think an area of research but I think this is already promising for us that it feels controllable in limited application settings like medicine。

But if you have a general purpose Lmm trying to answer pretty much everything about the world。

 I think that's a much harder problem。Do you think that would be a feature of like the domain。

 David said？In medical situations， doctors are more reserved perhapss and don't。

Absolute have absolutely English for。On on certain things or do you think it's more that like you have like just specialized right like it could be。

Something else entirely also I'm just curious yeah， so the question is。

 do you think this the way how the model is performing in this domain is that a feature of the data sets in the medical domain and。

Typically based on how doctors communicate and I think that's true and I think that's something we need to build on and use over here and I think that's extremely helpful and hopefully this kind of behavior is general enough and can be transmitted to the model even when it's used in non-medical settings to be like you know more reserved when it's communicating hallucinateless and so on and so forth so I believe that that's one of the opportunities over here to like use these benchmarks come up with methods that reduce hallucination。

 communicate uncertainty better and then use that as a bidirectional learning opportunity to improve the general purpose cell this。

So if you have any further questions， I'll come back again at the end of the talk。

 but I want to cover the rest of the applications as well。

So the next domain I want to talk about is proteins and the papers from now I'm going to like zip through them a little bit given time。

 but the first one I want to talk is this paper from a few folks at Google Research back in 2020 called mass language modeling for proteins by linearly scalable long Con transformformers。

So the problem here is that modeling long range biological sequences requires efficient transformer architectures。

 and so in this particular paper what they introduced was this performer architecture。

 which approximates the softmax attention kernel via low rank decomposition。

And so this does not incorporate any spaity price， say like other methods like the reformer or there are many others。

 and this is good because spaity price may not be appropriate for biological data such as protein。

 which required global interactions to be modeled。And then the other thing is this model。

 the performance scales linearly rather than quadratically with the sequence link de。

 and the number of random features that you need to approximate this Somax attention kernel is completely independent of the input sequence link。

So just to very quickly visualize the speedups and the space complexity improvements。

 what you're having with this low angle decomposition is instead of having like fat matrices in your softmax attention kernel。

 you now have thinner mattressrices which are determined by the size of the random features and that basically reduces your quadratic complexity to something that is more linear in nature and also leads to space improvements。

So I would yeah there are more theoretical analysis and details in a paper and I would refer you all back to it。

 but what we see in terms of results when doing protein language modeling is that the accuracy of of this model is on par with transformers while reducing computational cost quite a bit and so what this suggests is that the approximation of the softmax attention kernel is a tight approximation so that is good and then when you compare that with other methods such as the reformer or the Lyformer。

 the accuracy is much higher at least on this task so it seems that compared to other methods that like try to build more efficient transformist this one is much better for biological sequence data at least in this setting。

And finally， if you look at the attention of the amino acid similarity matrix。

 you can see that the performma model recognizes highly similar amino acid pairs such as DE and FNY over here。

That suggests that the model is learning the right set of information that we really want over here。

So that was a two minute overview of the paper， but I wonder yeah。

 talk about another one which also I think is really really cool。

So this one is called protein Llum again by a few of folks at Global Research。

 and what this tells us is model based natural language protein annotation。

And why this problem is important is because the protein information is in very high demand。

So over 50% of all known protein sal in sequence we don't actually know what they do so it's important that we able to like decier that to some degree at least and then the second thing is we may want to for example find protein sequences with given functions and this is particularly important in the CRISPR domain and so if you can train bidirectional models that can do this I think thatll be incredibly helpful and。

And the reason I say this， again， is that the Unipro database that has over。

There is I think millions of researchers worldwide using it today。

 and so getting this information populated in that database would be incredibly useful and accelerate a lot of research in this space。

And so the European Bioinformatics Institute， they have curated this prett data about proteins and so basically you can use this protein in record to like train these models and so what you want to do is you want to maybe learn to directly map from amino acid sequences to natural language descriptions of them。

And this problem is not too different from an image captioning problem where instead of having a sequence of pixels。

 I don't know if sequence is right， but again if you have pixels instead you have a sequence of aminoamine acids and they can range a number from two to 40K and then what you want to generate out is a protein description of the protein。

And in this paper the way they do this is they train a T5 model on protein sequence sanitation tasks。

 so the tasks are set up in a bunch of different ways and the supervised data comes from a bunch of different sources in the protein record that they have and this model is an encode E T5 model so it's a very cool application and the results are that out of the 56 million proteins in that unprod database that were previously uncharacterized 49 million of them now have associated textual descriptions so we now have a handle on what they do。

嗯。And so that's really cool and then the other one I think which is probably even more interesting is now you can run queries like find me a smaller version of this CRISP Cas9 protein so that it can target certain tissue spectra and now the model can know come back with sequences and so I think this is again going to be incredibly useful and going to accelerate a lot of research in this space already there's a lot of momentum I think these models are going to further help。

诶。So that was on proteins， the last class of applications that I want to cover is on the genomics site。

Again， the first paper over here was somewhat last year from our Ge team at HealthI at Google。

 which is building gap our sequence transformers for sequence correction。

So this model is called deep consensus and so what role does this model play and why does it matter？

So if you look at the sequencing data lifecycl， what you do is you go from basically atoms to bits and so you have this physical specimen which hopefully has some DNA in it and you。

P it through a sequencing machine such as Spg bio and that comes out with the raw data and that raw data gets mapped to a reference genome and then sometimes there might be diffs between an individual and the reference genome and that can be corrected through this model called deep variant that was introduced by your team a few years back and that's open source and then once you have this sequence you can then use it for a bunch of different analysis such as ancestry or like just basic biomedical research。

So where deep variant fits in is it actually makes the raw DNA reads that comes out from the PA biosequenceer it tries to make it more accurate and so how the PA biosqueenceer actually works is it uses this circular consensus sequencing algorithm where the DNA molecule is like you know read several times and it produces multiple different subres and these subreads are。

They do contain some errors and so they're finally like assembled together and so what Deep W tries to do is it tries to improve on the errors over here basically that comes out from just this circularence sensor sequencing algorithm。

And so how does this model work so as that the basic task for deep consensus is to use the CSCCS data and the subreeds associated with them to generate a corrected sequence and so in this example when we run through the model what we see is that while the CCS identity was at like 95。

7% the deep consensus prediction identity was at 100% so it's a fairly simple task where you're trying to like reduce errors that come out from the fact bio with the CCS algorithm。

And so the very natural question is where do these labels come from？

So each CCS sequence that you have that is aligned to a high quality assembly and this high qualityality assembly is created by having many CCS reads stitch together and so that ends up having fewer errors and so you can then try to use that high- qualityality stitch assembly and map that back to the CCA trade for a given block and use that as that label so that results in more you know like stronger ground truth and you can use that to train the model to improve the accuracy further and so this is what the model is strained on and so the model looks like this it's a transformer architecture it takes these subres and this CCS read as well and it has a bunch of additional context features that come in from this the sequence itself the instrument sequencing instrument as well and these are all fit into the transformer model it produces polish segment and these segments are then like stitch together to produce the final polish read over here。

One thing I will point out over here is that in order to train this model。

 you can't use a cross enpy loss and this is because。You know。

 if you insert you often have insertions in DNA sequences and so that can when you use cross entropy loss like really throw off the model。

 even like a single error as you can see over here can propagate throughout the sequence and make it really worse so what you need is a special kind of alignment loss based on a distance that can really capture this error much much better。

And so making this alignment loss work on you know PUs and making a different shable is I think the real meat of this paper and so again go back to the paper if you're interested in that kind of topic。

 I think that's really cool。But at a very high level how well does this model work so if you look at the final output you have the read name you have the base predictions and also the predicted quality which can be thought of as a confidence score and these base predictions are often quite long and so you can see that continuous off screen because it's like you 10 k to electronick basis long over here。

And when you look at the quality， it improved quite a bit over the vanilla CCS algorithm over here。

 the per read accuracy over here improved quite a bit。

And so you may ask like what is the real world impact of this kind of model right so the answer is this model is already being used in the real world so at Stanford in the genomic stream by Dr。

 Ashley and a few others there was this recent ultra rapid nanopogen sequencing paper where they set a world record for the fastest genome sequencing and this deep consensus transformer architecture was used in that SM sequence and so in this particular study they were able to very quickly diagnose that Matthew over here had a heart condition due to a genetic reasons and so they were very quickly able to like put Matthew on the patient' donors list over here so that's a kind of real world impact we can have with these biomedical transform models and AI systems in general。

And very quickly， the last paper that I want to talk about is this paper from Deep Mind on effective gene expression prediction from sequences by integrating long range interactions this was published in nature methods。

And the motivation for this work is again that since the human genome Project there have been thousands of genome by association style hits where the goal is to you know map genetic variants to different kind of disease phenotypes but a lot of this involves experimentation and experimentation like real experiment patient takes a lot of time and so if we can like do that with machine learning models that's really and so that's what they set out to do in this paper and。

So if you look at the gene itself， there are like， you know。

 10% of the gene are going to be like coding variants and these influence protein function。

And then the way they can cause diseases is by disrupting the structure of proteins that are generated or by affecting the protein protein interactions。

The good part about these coding variants are they tend to be closer to the gene and so they're easier to interpret on the other hand the 90% of the gene is like noncoding variants and the way they work is they influence protein expression so they are more like regulatory sequences and so the way they can lead to diseases if they have any variants is by disrupting the transcription of proteins。

And given that these noncoding variants can be very。

 very far away from the gene and the coding variants。

 it's very difficult to interpret them and so the question is can we train transform models that can predict the influence of these noncoding variants and so that is the task over here and so this is a visualization again so the paper again looks at it focuses on transcription which is the first step in terms of converting DNA into RNA and the way this is done is you have RNA polymerase which gets recruited at the beginning of the gene by these proteins called transcription factors and these transcription factors are a binding side which correspond to these promoters which are quite close to the gene but then you also have these enhances which can be like very very far away from these promoters in terms of the linear space also influencing this transcription and you may ask how can such。

How can these enhances influence the activity over here。

 this is because while they may be far away in the linear space when the sequence falls and in the 3D structure they will end up being quite close to each other and so that they can completely affect the transcription process over here。

So it's a very high level overview of what's happening over here and in terms of the biology and so the question is if there are any variants in these noncoding variants and in these enhances they may like disrupt the transcription factor binding and this can internal turn lead to like you know no proteins and then finally to diseases right so we want to be able to predict that based on the DNA sequences that have been generated。

So the problem is kind of quite straightforward， it's a supervised learning problem。

 the setup is predict experimental data from these DNA sequences and this can take many different forms。

 the primary one is gene expression over here， but then there are also other tasks such as DNA accessibility。

 histone modifications and transcription factor binding and so on and so forth。

So as you can imagine the baseline model for this task for many years was the CNN model and as you stack up a little different CNNL layers you can increase the receptive field but there's a limit to that so in this work what they showed was you can use transformers instead and do better modeling of these long interactions so the final model is called Ener which is a combination of this enhancer and transformer and so if you look at the model itself it has a few CNN layers at the beginning but then it has a bunch of transformer blocks that are stacked together。

And the input is 200 KB DNA sequences and there are approximately 30k examples that have been trained and the output is like genomic tracks of this RNA expression with。

 and they have organism specific heads， so one for humans and one for mouse。

And finally one key detail is that relative position encodings that were used in this model were actually very key and these relative position encoings were modeling this power law of interactions and as a result of you know using these relative position encodings with the transformer block architecture。

 they were now able to model like interactions over 100 k based byes away。

And so you see that in the results over here， so you have the experimental data in green and you can see the CNN baseline over here and you see that as soon as you go far away you see that the CNN model is no longer able to capture these gene expressions。

 but you can see that the enhancer model is now able to like pick them up so you can see that as the model goes far away。

 the enhancer model is able to capture this whereas CNN model is no longer able to capture this。

And finally， one。I think very interesting experiment that they had in the paper was they were also able to like you know predict promoter enhancer influences and that prediction was actually on par with experimented data so this suggests that using this machine learning model we can sidestep a lot of these wet lab experiments and get like key details。

 which could be super useful。So yeah， so very quickly。

 I'm sorry I had to like cram through proteins and genomics applications over here。

 but I think what you would see is that overall when you look at clinical proteins and genomic applications。

 we see that transformers have incredible potential in biomedicine。And with clinical applications。

 I think the challenges are perhaps more centered around data and evaluation。

 but on the proteins and genomics side， I think there are some extremely interesting opportunities to innovate on the architecture here。

And finally as I said， there are incredible bidirectional learning opportunities。

 I think the problem of you know modeling long range interactions that's useful beyond proteins beyond genomics。

 I think it's useful in genomics and so I think any architecture improvement over here can inspire wider progress in AI so I think that's a big reason to work on this。

嗯。Any questions so far？Sorry， I covered a lot of ground over here， apologies for that。

 but I think these are super cool papers and you should go back and read them。mSo quite finally。

 I want to maybe spend a couple of minutes touch upon how I see the future of biomedical AI evolving。

Overall， I believe it's not a question of。Like if AI will transform biomedicine。

 I think it's rather a question of when and how。And I think the very specific thesis I have over here is given the nature of biomedical data and how multimodal in nature and with all the progress in transformers self learning large language models。

 I think we have an incredibly powerful like framework to like leverage all this richness at scale and like truly build foundational medical AI models。

So I think that is incredibly exciting。And。So I'm not I think it's you've already been in Hawaiias for far too long。

 so I'm not going to ask you to recognize these people。

 but they're actually famous physician science centers some of them went on to win Nobel prizezes and so。

I think what I want to say over here is there's no reason for a scientist to be different from a physician they can be combined together and that's what also want to like convey with our AI systems as well we don't have to like separate clinical applications and biological applications。

 I think when we combine them together we are going to discover a lot of new insights and I think that's going to accelerate biomedical research and internally to new discoveries and which is going to like you know be used to eradicate diseases。

 advanced human health plan and generally drive human potential forward。

So question I actually know these three are。Sure， I think the right most one is Alexander Fleming and then Jonas Sak and then Paula Liick。

So fleming is penicillin， salt， Capoo， and Elich was a bunch of different stuff。And。

And so maybe I ask this question to all of you， which field of AI do you think will which field do you think AI will win the first Nobel Priing？

You're an not going。嗯。I mean think but like。Satan noble noble was like it's not a real field I these people abroad。

 I think economics is。it's associated with the Noble Foundation no I think was like this is a joke and not worry these people and then it's almost like wait wait own economic noble bias what was biology that like can have like oh gay render this joke that do cancer or something。

That's right so I also feel the same way and I hope the overwhelming majority of you also think that it's going to be in medicine and I'm going to end on that note huge thank you to all my collaborators and teammates for most importantly allowing me to like ST slides and then also thank you to all of you for like patiently listening over here hopefully this was helpful。



![](img/867d42af88a583402ff132cc4ae7c3fa_3.png)