00:00 Aksana Rahouski: Okay.

00:00 Adam Curcie: Yeah.

00:01 Aksana Rahouski: Okay.

00:02 Adam Curcie: And, yeah, well, I mean, and there's so, like, and I'll just put this out there, right? We'd never told you to support because and like, so, first of all,

00:12 Aksana Rahouski: Yep.

00:13 Adam Curcie: 18, more expensive. So it's like for us as people operating a business for profit, it makes more sense to be pushing Verizon. so, when people want to use AT&T because there is no Verizon, we either do dual carrier or AT&T only So it's it's been very rare where we've had to take a dual carrier device and turn off the the Verizon service.

00:42 Aksana Rahouski: Gosh. Okay.

00:44 Adam Curcie: Localized network issues. But yeah, I mean like and again we so when I tell you like we don't support it. Now it's it's like we never asked you to have an equality to support that.

01:07 Aksana Rahouski: Okay. Gotcha.

01:10 Adam Curcie: Yeah.

01:10 Aksana Rahouski: okay, so with that being said, Perhaps we then don't need this quite yet. What, what I think though? Like when we need to define from our current, let's just go to like a device. technically, what I'm hearing you guys will never be Will you will never be importing this. Right. Okay. So you

01:39 Devon D'Andrea: Correct.

01:41 Adam Curcie: well, yeah, as it is right now I mean I can't necessarily say never But yeah, I mean most likely that they're really isn't a reason that we need to worry about creating or accounting for that scenario today.

01:57 Aksana Rahouski: okay, which is again like that is Like let's say like an ideal world, right? All your AT&T sim devices will be imported like this. You will have number and you will have that active, right? And that's what tells us that this sits in, um, Sim one. And we have a config to match.

02:23 Adam Curcie: Yeah.

02:24 Aksana Rahouski: Right, if So, if

02:28 Devon D'Andrea: We find out that, you know, people are just not having a good experience with it or like whatever whatever. Along the same. You know, amount of time. We also let's say happen to finally, like sit down with AT&T and get a better deal. now, we go to in hand and we buy another Thousand origins that we want to have for AT&T inventory. When we get them. I know Adams doesn't like what I'm saying but

03:32 Adam Curcie: Now, I don't have a problem with what you're saying and I know what you're going

03:35 Devon D'Andrea: When we get them, when we get them, they're gonna have, they're also going to

03:36 Adam Curcie: with this.

03:39 Devon D'Andrea: have a Verizon SIM number.

03:41 Adam Curcie: Yeah, I mean, what I want to kind of like I was about to interrupt you even before you interrupted her, I was gonna interrupt everyone because we when we finish the configuration overhaul on the way that we create configs, None of

03:57 Aksana Rahouski: this is gonna matter to the same nearly as much as it is because you won't really have to map Configs. You're just gonna be changing parameters and I think me and Devon did enough testing in the last 24 hours to know that like if we are going to have devices whether it's the second sim or the the physical or the esim like we can support it on a configuration level with the correct parameters modified correctly. It's just gonna be so much easier. Once we're actually working with the parameters on that independent level, then trying to map it to a config file that we have to pre-generate and host. So like I almost just want to tell you to like not even worry about it, like just like the, I kind of want to just say, like for this like next phase, like, Let's just talk about like, Building, what we need to build to support the T-Mobile. Uh-huh.

04:55 Adam Curcie: Only and the origin on like really yeah I mean for T-Mobile I feel like I feel like we should make as few changes as possible, right now in that have to do with the way Configs. Are are handled so like I don't want to get hung up on what to do about an AT&T only box that like I mean if things go well we'll never actually support. So it's like I mean it's not like it's a waste of time because like Devon's saying there's definitely other reasons we're like in the future. We could definitely lean on the same concept but like It's not nearly as pressing as getting the T-Mobile stuff done. So like I just like, I don't know if we're like overly fixated on it. I don't know. Devon, what

05:40 Devon D'Andrea: Well no I don't what I understand. You know I just think that for the purposes

05:41 Adam Curcie: do you think?

05:45 Devon D'Andrea: accidentally let's say Have, you know, put something at the AT&T active that has other SIM cards in it. Something is gonna matter to a config because it has to because it has to. We just have to determine if we're gonna decide we want them to either a to prevent us from actually being able to do that in the first place. Or we just tell them, Hey, we're just never gonna do that.

06:24 Aksana Rahouski: I think. Yeah. So basically for saying that yes, like your data input is always going to be spot-on. Super accurate, right? And like for example Scenario. Like I just said, if you're saying that you would, you're either when

06:39 Devon D'Andrea: Yeah.

06:41 Aksana Rahouski: Is the only SIM card in someone slot.

06:55 Devon D'Andrea: Right.

06:56 Aksana Rahouski: We in this is it like right? This is just kind of hanging there it doesn't

06:56 Adam Curcie: Yeah.

06:59 Aksana Rahouski: matter. We do not make any assumption around as long as this is not dual. This is not like that, would not using it. It's not active. When we're just regarding the position, you have the sim.

07:12 Adam Curcie: Yeah. But so I'm gonna tell you that if if you have it working in the following way, okay, I'll explain to you is this is gonna be what I'd say is best case.

07:19 Aksana Rahouski: Mmm. Okay.

07:24 Adam Curcie: So, this is if everything is live, right? For the word, the way and, and these checkboxes are only going to affect carries right now, right? So, this is technically, it's a dual carrier device because it's got two sims technically, not necessarily the way the portal looks at it, okay? But so the configuration says Dual carrier, and if you come in here and uncheck that today before you add these boxes it turns it into an AT&T only basically, like, Okay here, let's do

07:53 Devon D'Andrea: Yeah.

07:54 Aksana Rahouski: Yep.

07:54 Adam Curcie: this, Let's do this. So instead of the so check both boxes right and this is so

08:00 Aksana Rahouski: This is what we have. Let's say,

08:01 Adam Curcie: Yeah, but okay. So in order for the portal to determine it's AT&T only, Don't worry about the checkbox. Just delete the numbers out, right? Because now it only sees an AT&T sim for Verizon. I'm talking about sorry, delete the numbers for Verizon, right? So on the contrary, if you have this On the contrary of what we would do today which is just to literally delete out the Verizon SIM to make it in AT&T only. if you uncheck that box, And the and only thing that happens is the Verizon service is deactivated or just yet deactivated? That box is fine. It's gonna stay on the dual carrier. Config, it's going to try

08:37 Aksana Rahouski: Yeah. Yeah.

08:40 Adam Curcie: there's two Sims That's fine.

08:59 Devon D'Andrea: So is That's the question then. So can we have it so that if that's the if but yeah, right. So then so then, okay, so yeah, that's the solution then. So then if, if Verizon and AT&T Sims populated, but only AT&T active, we just keep that same dual carrier, config mapped to that, to that scenario. So, both whether or

09:23 Adam Curcie: Yep. Yeah. but,

09:26 Devon D'Andrea: not you have

09:26 Adam Curcie: What we thought charging for dual carrier here and we turn off the Verizon SIM but it's a dual carrier device configuration and we bill it AT&T only and the device will work fine on AT&T. And it doesn't remap.

09:41 Aksana Rahouski: So the the SIM value presence kind of tells us that this is a dual carrier.

09:48 Adam Curcie: Configured device, just not being built in a dual with both active.

09:51 Aksana Rahouski: Right. Yes. Okay. And I believe, Richard question to you that I don't think

09:54 Adam Curcie: Yes.

09:58 Aksana Rahouski: that's where we need to probably change our mapping engine, like, when we map because right now, right? It just only looks at that active flag.

10:08 Richard Sacco: Yeah. Is active via we change this to active has a lot of weight, but yeah.

10:14 Aksana Rahouski: Well we shifted yes, the SIM value weight into the active weight. But now we're saying it needs to kind of like if do both values are present in case of Verizon in AT&T but only AT&T is active. You see it?

10:31 Richard Sacco: Yeah. Yeah. We

10:34 Aksana Rahouski: Like a dual carrier box with a single sim.

10:38 Devon D'Andrea: Here. Yeah and for the purposes and for the purposes of what you guys need to

10:39 Adam Curcie: Here. Wait, I'm sorry.

10:41 Devon D'Andrea: know, we could call it config You know XYZ, you don't even know. It need to know that you don't even need to know that it's a, you know what I mean? It's just another config

10:50 Aksana Rahouski: We don't know, we don't need to know exactly, we just need to. We just

10:51 Adam Curcie: Well, no, it's not another thing. What do you mean? It's not

10:55 Devon D'Andrea: I'm just saying like It's you Yes it is a dual carrier. Config in the sense that all the parameters are gonna stay the same as a dual carrier, but it's not as it doesn't mean that. It's like you said being built now for the the and the more important thing is this this scenario is gonna happen, very rarely. But the scenario we talked about before where it's Verizon and T-Mobile both populated that that will that's gonna happen every day.

11:21 Aksana Rahouski: Talk about that one. What how do we treat that?

11:25 Devon D'Andrea: Same way.

11:26 Adam Curcie: Well yeah, it would actually be the exact same thing that we just said. except for, like, Well, maybe, I mean, but

11:36 Devon D'Andrea: we have to, to the, to your point, Aksana, we have to be able to go to a page and say,

12:00 Aksana Rahouski: Uh-huh.

12:01 Devon D'Andrea: if Verizon and Teamo SIM populated Only Verizon active.

12:09 Aksana Rahouski: Right.

12:10 Devon D'Andrea: Config equals this.

12:11 Aksana Rahouski: So we check, we asked two questions. Is it a dual box? And we answer that by

12:14 Devon D'Andrea: Yes.

12:18 Aksana Rahouski: value. One is their value too, is there? Yes, which one is active and if and

12:20 Devon D'Andrea: Yeah.

12:24 Aksana Rahouski: that's where if this Then. We map it to a config for a single carrier. Or sorry, I guess.

12:36 Devon D'Andrea: I don't know why I brought that up, but yeah, no, it would it you have to you

12:55 Adam Curcie: Yeah.

12:57 Devon D'Andrea: have to be able to I just lost my train of thought.

13:09 Adam Curcie: I'm, I'm also so.

13:12 Devon D'Andrea: Sorry, I just

13:13 Adam Curcie: right now, with

13:15 Devon D'Andrea: Like my eyes just rolled into that.

13:16 Adam Curcie: I don't know if this will help, but I kind of want to share for a second.

13:19 Aksana Rahouski: Well, let's see.

13:20 Adam Curcie: Okay. I think this it could help. I know no promises. Might make it worse. So when we look at the way, this is a test box. By the way, Devon don't. Don't worry. So Right now. So we don't have the ability to turn the Sims off individually obviously. Right? We we would do well we're talking about with the AT&T and Verizon and it's going to be similarly applicable for T-Mobile on the origin. Like if we uncheck this and both it defaults to the Verizon

13:51 Aksana Rahouski: Yeah. Okay.

13:56 Adam Curcie: So which and then that does change the config. um, I'm wondering if we may need to reconsider the way we almost Do the carrier indicator. Perhaps. we need a, because

14:15 Aksana Rahouski: Yes.

14:17 Adam Curcie: If the care got this thing's in the way here at a configuration. Like, do we need a second dual carrier? Like when we add T-Mobile, Do we need a computer

14:27 Richard Sacco: Yeah. Yeah. We

14:28 Adam Curcie: here That says, You know, variety of AT&T, and then another dual carrier, That's

14:29 Richard Sacco: Yeah.

14:33 Adam Curcie: gonna say Verizon T-Mobile because they are gonna be totally different configs

14:35 Aksana Rahouski: Yes.

14:39 Adam Curcie: that we that we're gonna need to like, if, if we're gonna say that checking or

14:40 Aksana Rahouski: Correct.

14:46 Adam Curcie: unchecking, the active box in the way, like, right now, I uncheck this.

14:49 Aksana Rahouski: Yeah.

14:50 Adam Curcie: I know it's not the same, but then it's a Verizon only box, right?

14:53 Aksana Rahouski: Mm-hmm.

14:54 Adam Curcie: So we it's gonna have to find a different Verizon only config, though on the origin. If if it's actually got like and, you know, so like

15:04 Devon D'Andrea: If it's an origin. Yeah now you've got even not yet so it's device. It's model

15:07 Adam Curcie: Yeah.

15:10 Devon D'Andrea: based. it's

15:12 Adam Curcie: And and divide and carrier. Yeah.

15:14 Devon D'Andrea: Yeah.

15:14 Adam Curcie: No. Yeah. So I said, We we may need to. But again, I don't know necessarily how much additional work it is to add this and I also don't even know if it's going to become relevant in the future.

15:30 Richard Sacco: Well.

15:31 Adam Curcie: When we change the configuration engine, but I feel like if we're

15:35 Richard Sacco: Yeah.

15:36 Adam Curcie: Like what's gonna come first? Is it gonna be us wanting to put T-Mobile in

15:53 Richard Sacco: Yeah.

15:57 Adam Curcie: i-22's? Or is figuration engine gonna be done? So, because it's like, I could see how in either case we may or may not need this necessarily, but You know, just something I wanted to kind of throw out there. So alright, perfect Richard, what do you got?

16:13 Richard Sacco: Yeah, so if you can see on Beta, they're actually already is a differentiation. We rename dual carrier to I believe like of the AT&T or something.

16:25 Aksana Rahouski: We not see my drop-down, hold on, I need to share the whole screen.

16:29 Richard Sacco: Okay. And so yeah we can add what is it vz teemo or whatever it is.

16:36 Aksana Rahouski: Okay. So for now

16:38 Richard Sacco: Yeah. As far as test ready, I think the best way to handle that is when you do an import we just assume they're all tests ready when they start and then we

16:39 Aksana Rahouski: This is what it is. And I think maybe it's a simple is we just kind of because again we change the device. Config will change that in fake builder, right? Config builder can

16:47 Adam Curcie: Okay. Yeah.

16:51 Aksana Rahouski: combination of dual so with that being said, I think that what we're trying to solve now is like if I create a CONFIGS that is AT&T carrier only and now we look at the device, it's a It's device. That's going to look like this. Matches it right? This is gonna be the device on a single carrier AT&T only. But need sounds like not not necessarily, right? Also this

17:23 Devon D'Andrea: Right.

17:29 Aksana Rahouski: is also needs to fall into It and, or which that's the question, which, which config if like, AT&T SIM is

17:33 Adam Curcie: Yeah. Yeah.

17:40 Aksana Rahouski: active, Verizon is inserted, but not active, are we mapping it to a Verizon AT&T or are we mapping it to an AT&T only?

17:51 Adam Curcie: No, you wouldn't. You would have to map it to the Verizon AT&T if that's the

17:56 Aksana Rahouski: Okay. Okay. So

17:56 Adam Curcie: case. Yeah. And it and it almost makes me think like the dual SIM check box. Really is only gonna pertain to if we're charging the customer for

18:07 Aksana Rahouski: Yes.

18:08 Adam Curcie: So, it's like that's like the billing, the Sims existing is the config and then

18:12 Aksana Rahouski: Yeah.

18:14 Adam Curcie: the carrier checkboxes are only pertaining to the carriers.

18:18 Aksana Rahouski: Correct. Because now almost

18:19 Adam Curcie: And they're like, separates all three of them.

18:21 Aksana Rahouski: There are two definitions. There are two kind of me to do. All right, is it dual

18:23 Adam Curcie: Yeah.

18:24 Aksana Rahouski: because charging for two and truly to our active, the pain for tool or is it

18:25 Adam Curcie: Yeah.

18:31 Aksana Rahouski: dual because it has to Sims, right?

18:33 Adam Curcie: Yeah.

18:34 Aksana Rahouski: because in this case, and which

18:36 Adam Curcie: What really only affect the configuration? Yeah.

18:40 Aksana Rahouski: Improve. We can like in, He owns our configuration engine or mapping, our mapping engine to if you have a device that has two numbers provided but only one is active. You still map it to a Verizon AT&T?

18:57 Adam Curcie: yeah, I mean and Dev think about like for everything we do for Bella, they would just be running a dual carrier config that's going to try Verizon first and never fail over to AT&T because all the Sims are Inactive.

19:11 Devon D'Andrea: Right.

19:12 Adam Curcie: so, Nope, that's not true. That's not true because they're not inactive they're in they're in inventory or test. Ready. So

19:26 Devon D'Andrea: Yeah.

19:29 Adam Curcie: so that's actually not the case for the time being

19:31 Aksana Rahouski: For. Can you can you repeat again one that use case? That you are.

19:38 Adam Curcie: So the way that I was just think. Okay, so like what I had said about, if we turn off Verizon and and that'll that'll actually allow dual carrier can

19:45 Aksana Rahouski: Mm-hmm.

19:48 Adam Curcie: in the backseat sound asleep, until we want to wake it up. so, Yeah.

21:03 Richard Sacco: Yeah. As far as test ready, I think the best way to handle that is when you do an import we just assume they're all tests ready when they start and then we

21:08 Adam Curcie: Yeah. Yeah.

21:12 Richard Sacco: know they are we don't have to query it for if it's test ready, and if it is ready, have special like

21:18 Adam Curcie: What?

21:19 Richard Sacco: Things in place to not mess with it. And I think that

21:21 Adam Curcie: Yeah. What the thing that will mess with it is, if it stays on a dual carrier, config and Verizon go down and because it will try the AT&T it will work.

21:29 Richard Sacco: Yeah, because you so that's why you want this one to be Verizon, only got you.

21:34 Adam Curcie: Yeah. So like that would yeah, so I guess you have to you'd have to consider

21:37 Aksana Rahouski: Is it?

21:39 Adam Curcie: of identifying it as an AT&T only it keeps the dual carrier config but not not vice versa.

22:06 Aksana Rahouski: Is this? Well yeah it makes sense. I'm just trying to figure like for this case

22:07 Adam Curcie: Does that make sense?

22:12 Aksana Rahouski: scenario, is it still considered dual carrier box.

22:17 Adam Curcie: Well, no, not in any way other than we know there's two Sims in there. We wouldn't want it to use the dual carrier. Config is what I've just realized.

22:26 Aksana Rahouski: Yes.

22:26 Adam Curcie: I'm talking myself into circles.

22:28 Aksana Rahouski: Yes. So I'm thinking

22:29 Adam Curcie: so,

22:32 Aksana Rahouski: Because if if like Verizon and ATC with active Verizon considered a single carrier like we need to be using single carrier, config but vice versa need to be using a dual carrier config and we can definitely the bake it in the system that it's like cemented that way, right? But if we'll if you ever need to like Now, I have a scenario where a Verizon and AT&T stems with Verizon active, but I do actually want to use a dual carrier. Config for this, you can't do that. Now.

23:31 Adam Curcie: yeah, I know, and the whole problem with even trying to think through this for the future is the fact that If we can make the configuration engine work, we won't actually have to worry about any of that. so,

23:44 Aksana Rahouski: But although like think about it, like the configuration engine will, it will

23:49 Adam Curcie: Yeah.

23:50 Aksana Rahouski: in order for you to like properly, like, create map these configs, you have to first build these configs, which is time consuming right now, right? But like mapping engine though, how do we map device to proper config right? That, is it really at the end of the day? It's a set of rules that kind of like Make a decision right at the end like which config file needs to be selected for this combination.

24:24 Adam Curcie: I know it but so okay. So let me let me kind of put it to you in different sense. Where is, if we weren't relying on configuration files per se and we were

24:36 Aksana Rahouski: Yes.

24:36 Adam Curcie: just talking about these parameters individually.

24:39 Aksana Rahouski: Yes.

24:40 Adam Curcie: the way that the portal would have to make changes would be Would be entirely different than mapping a configuration file. So like ideally right. Like if I wanted to which I and I could kind of give you all of this in an example if it would be helpful. But so for every single possible scenario of Verizon in SIM 1, another sim and sim 2 vice versa and all the different models and all the different combinations. Which at this point is a quite a bit more complicated than it was when we ever talked about this in the past. But We would have set values. that are like,

25:21 Aksana Rahouski: Populate your bags.

25:22 Adam Curcie: Yeah, so like if we yeah it would be and it would be a lot less. To just you like three parameters for 20 different combinations compared to 20

25:27 Aksana Rahouski: Yeah. Yeah.

25:32 Adam Curcie: different CONFIGS. So, you know, because again it's like if if it's a dual carrier, if it's a Verizon AT&T box and we want to turn off the dual carrier on the box. That's that's literally one parameter. I mean, I literally just uncheck the box that says Use dual SIM and it's done like and that's the only

25:53 Aksana Rahouski: And yeah.

25:54 Adam Curcie: difference. But you know, that's a completely different file inside of our system even if it's only one parameter it. It's a very important parameter. So we have a ton of files, a whole separate suite of files for dual carrier. It's like, you know, and but that would work if if we if we could have the portal recognize that like all right, it's a Verizon sim. The Verizon Sim is in you know, it on I-22s. We know it's always someone. So just, you know, in this instance, you would turn off. Sim 2 or SIM check box on the device.

26:31 Aksana Rahouski: But that's I totally hear what you're saying. So you're saying now let's say there is a like a blueprint config right that but like that it's almost like this file has the parameters like values. It's like they're not done yet. This is what drives them right. so, this values will be populated based on how an individual entity level such a device, has it configured And indicate this is like let's just to kind of make progress for what we're trying to do today. Configs like yeah, we'll be like talking about it separately. As far as like how these These final configs are built, right? But do we want to then for now just to kind of bake it in the system is a hard coded mapping.

27:30 Adam Curcie: I think if that's easier, we basically should do as as little work as possible. Knowing that it's all going to be redone. Anyway, Like it's a temporary solution. Does that make sense?

27:47 Aksana Rahouski: Yeah it. Yeah and we can do that as long as it can just like agree on that

27:47 Adam Curcie: like,

27:53 Aksana Rahouski: define like final set of rules which is In and then I think like the single ones is easy, right? If it's one, one sim card. You just gets mapped to config file with a single here. If it's too. And let's see. So Verizon in the AT&T, what are the two that we do support? We support this option? And we're saying, if Verizon and AT&T values

28:16 Adam Curcie: Yeah. Yeah.

28:21 Aksana Rahouski: are provided, but only one is active. We're gonna map it to a single Verizon Sim configs.

28:29 Adam Curcie: On on I-22s and I 52's. So like the the rules are that for i-22s and I 52's if there are two sims at all the Verizon Sims, always going to be in that sim one position, which you don't have to necessarily keep in the portal anywhere. But just for like the sake of the way we map configs, I just want to say it that way. Um yeah. Because on the origin it's it's not that way. So like you can hard

28:53 Aksana Rahouski: Yep. but,

28:59 Adam Curcie: code it specifically for. Yeah I 22s and I 52's.

29:03 Aksana Rahouski: so,

29:03 Adam Curcie: Yeah. Okay. I like this. I didn't see this before.

29:06 Aksana Rahouski: I 20. Wait, just somebody sharing something.

29:09 Devon D'Andrea: Sorry, I stole it.

29:11 Adam Curcie: Oh my God, Devon you scared me.

29:12 Aksana Rahouski: Yeah.

29:16 Devon D'Andrea: this probably, I 52's is similar to I-22, right?

29:22 Adam Curcie: Identical. Identical, you could fly.

29:24 Devon D'Andrea: But this would be, this would be what I would think are all the possibilities. Correct me from wrong.

29:34 Aksana Rahouski: oh,

29:38 Devon D'Andrea: This, this is supposed to be. This is supposed to be. No. Sorry.

29:47 Aksana Rahouski: okay, so Verizon supervisor

29:50 Devon D'Andrea: The. There could even be another one that's Verizon active. No, yes. Easywomely.

30:03 Aksana Rahouski: So, you're saying like second record, two Sims, oh, wait Verizon active it in to mobile single, then Yes, No, yes. And active okay, and then

30:20 Devon D'Andrea: So there's literally three. Combinations for Aaron. Only I-22.

30:25 Aksana Rahouski: Yeah. M-hmm.

30:29 Devon D'Andrea: and this config for an i-22, these would all be The same. Config, but it's just three ways of getting mapping to that. Config

30:41 Adam Curcie: Look at you Devon. I think we got some progress.

30:45 Devon D'Andrea: and then you have, For AT&T. You've got just an At&t's active SIM and that's our standard AT&T only. And then you've got a possibility where it's a, that's the rare case where you've got a Verizon and AT&T Sim,

31:06 Aksana Rahouski: um,

31:06 Devon D'Andrea: And so now we're inactive.

31:08 Adam Curcie: So why would you call that an AT&T only? Config, it's a dual carrier. Config

31:12 Devon D'Andrea: It's an AT&T only. I'm it's an AT&T only. It's an eight, it's the per it's for the purposes of it being AT&T only and that's why I have asterisks stool config with Verizon inactive.

31:28 Adam Curcie: All right. I mean, I would just call it a deep little carrier. Config

31:31 Aksana Rahouski: Because it's, I mean, can if we're if config is kind of the combination of carriers on a config, then it is a dual one, right? That we're having

31:40 Adam Curcie: Yeah, I would write, I would I would flip it the way you have it and write Dual

31:41 Devon D'Andrea: Yeah.

31:45 Adam Curcie: Carrier and then Asterix and AT&T with Verizon.

31:46 Aksana Rahouski: Yes. Yes.

31:49 Adam Curcie: Because it's gonna be what we refer to as configs as a dual carrier config. It's gonna start with DC in the in the host name.

31:58 Devon D'Andrea: I mean if we wanted to call it, that we could call it.

32:01 Adam Curcie: Well, that's what we call it right now.

32:03 Devon D'Andrea: I know. So you're saying dual but dual but it's really you.

32:07 Adam Curcie: Our I would write all carrier and then your asterix, and right AT&T only with Verizon and active. Inactive.

32:20 Devon D'Andrea: Tomatoes tomatoes.

32:22 Adam Curcie: Inactive is not tomatoes to model. That's like tomato and a baseball.

32:26 Devon D'Andrea: Now what I said and what now, it says, All right, that's fine. I-22 Um, for Teemo, only this is would be the standard, we get them just loaded with Timo Sims. that straightforward and then here you've got dual carrier Teemo only Verizon inactive and then you get to Verizon active AT&T active, No, teemo SIM populated you've got this? And then same thing you've got Verizon populated and active team of populated and active. No AT&T populated then you get that. and then you get to the origin which is every single one of these is gonna have I'm sorry is gonna have at least A esim plus a physical sim so Verizon only active esim inactive physical Timo. And you've got dual carrier Verizon only teemo, inactive. and then if we ever got them with AT&T Act with AT&T, um, That would be for the AT&T only. and then you would technically have Another one.

33:57 Adam Curcie: yeah, you would have one more potentially

34:01 Devon D'Andrea: Which is.

34:03 Adam Curcie: A Verizon AT&T.

34:05 Devon D'Andrea: Right. Yet, there would be one more, there would be.

34:10 Adam Curcie: No. No. You have it Verizon in 18.

34:14 Devon D'Andrea: Now that this, it would be the Origin. Yes, active e sim and then you have, yes. Inactive physical. No. And then you've got dual carrier. AT&T Sorry, Verizon only AT&T inactive.

34:42 Aksana Rahouski: This is gonna be our Bible.

34:46 Devon D'Andrea: So then you at AT&T only, it's gonna always have the inactive esim.

34:46 Aksana Rahouski: I will.

34:52 Devon D'Andrea: physical AT&T sim, which we can configure those just regular Um am I regular? I mean like without doing one of these dual carrier kind of modification things team. Only same thing. It's gonna have the inactivity sim

35:10 Aksana Rahouski: Why do we have Verizon? OH where I, okay, right AT&T dual forward or to order

35:10 Devon D'Andrea: a physical chemo and then this Activity, Sim activ physical, No Verizon AT&T activism no active, physical Verizon team though. I think that's every combination.

35:36 Aksana Rahouski: different model. Okay, question. Line 13. Verizon T-Mobile Because you guys don't have any CONFIGS. And I think when we like and I know like Guy on the road map and Phasing changed for this initiative times, we did not, do you want to add it in the config builder? So you will you be building these configs?

36:04 Devon D'Andrea: At 1000%.

36:06 Aksana Rahouski: Okay, so we'll load that. Okay?

36:08 Devon D'Andrea: Adding that here, right?

36:10 Aksana Rahouski: Yeah, yes. So we'll just add that here. Okay, so let's go back to that list. um, So if you can look at each one, right, how do we? So we know device model and then we need to answer questions like Which Sam and is it if it's active or not that.

36:28 Devon D'Andrea: Yeah.

36:29 Adam Curcie: What? So it's his column represents presence.

36:34 Aksana Rahouski: Yes.

36:34 Adam Curcie: Comma active.

36:35 Aksana Rahouski: Yes.

36:36 Devon D'Andrea: Right.

36:36 Adam Curcie: Like it does it exist and is it active?

36:40 Aksana Rahouski: Correct. So in a room for the this is how because like ultimately right when our

36:41 Adam Curcie: Yeah.

36:45 Aksana Rahouski: software reads like We need to build this now in the code right. It's gonna check the model, check a Verizon SIM value. Is Is there? Yes. Is it active? Yes, No and then derived to which then config to map for the origin devices right since Since there is like this, now Easton versus physical from what I'm seeing are. Verizon always reserves is him. so,

37:14 Adam Curcie: Yeah, and we can't, we have no control over that. When they come to us, they

37:15 Devon D'Andrea: Correct.

37:18 Adam Curcie: have pre-installed Verizon embedded Sims, that we have? No. Yeah. We can't do

37:22 Aksana Rahouski: so, we're just always gonna assume that

37:23 Adam Curcie: anything about that. So

37:28 Aksana Rahouski: Because we're not collecting whether it's is sim or physical sin, but we're assuming Verizon is there. It's it's him period.

37:31 Adam Curcie: No.

37:35 Devon D'Andrea: The difference is just the parameter of the name, that's in the config file.

37:36 Adam Curcie: Okay. Yeah, because it's an organ. It's gonna have to be referred to as a Nissan when you actually build out the config in the engine or in our configs. Not now that

37:48 Devon D'Andrea: Right.

37:49 Adam Curcie: I'm looking at this. I'm actually tempted to tell you that we can probably use this scheme as the foundation for the way. Carry your parameters are generated when you build the configuration engine, this may have a lot of applicability when we have that conversation, so make

38:06 Aksana Rahouski: Well, a lot.

38:08 Adam Curcie: sure you save this and email this to

38:09 Devon D'Andrea: Should I save this?

38:10 Aksana Rahouski: Yeah.

38:10 Adam Curcie: Yeah no. Please save don't lose this.

38:13 Aksana Rahouski: Please.

38:13 Devon D'Andrea: You get your aksana your your intern can't scrape this out of the screen share.

38:19 Aksana Rahouski: I mean I do think. How do you think I do it? I make a screenshot. I send it to my intern and it doesn't a table. Five seconds.

38:27 Devon D'Andrea: That's no funny. You do have Adam's? Thing he shared the other day but he didn't actually send that to you.

38:32 Aksana Rahouski: Work. No, I because I took a screenshot and then I build a table and two seconds. Okay, so okay. So we will find it since recorded, whether you share it or not. Okay, I

38:49 Devon D'Andrea: So yeah.

38:51 Aksana Rahouski: Think Richard, how do you feel about building this? How far are we off right now? Because like, this is not what we have right now in our mapping layer, right?

39:02 Richard Sacco: You know, because we either have done it to where we didn't have active before, right? And then everything would be based on present and then we've had now.

39:10 Aksana Rahouski: Yes.

39:12 Richard Sacco: It's nothing but active so presence doesn't matter. So now we need to

39:14 Aksana Rahouski: Yes.

39:18 Richard Sacco: combination of both. Well, the

39:19 Aksana Rahouski: Correct.

39:21 Devon D'Andrea: I'm also going under the assumption when I say no, that that means if there's nothing present, it can't be active.

39:30 Richard Sacco: Yes, really but that in this case, it doesn't matter. Even if it doesn't right,

39:34 Devon D'Andrea: Right.

39:35 Richard Sacco: it doesn't matter.

39:36 Devon D'Andrea: I'm just saying like, if it, if I were to go to if I were to go to this,

39:45 Aksana Rahouski: No, yes, no. You're referring to an actual SIM value, right?

39:50 Devon D'Andrea: I'm saying. Like if this is, if this is blank.

39:54 Aksana Rahouski: It's a no.

39:55 Devon D'Andrea: I, what if I do? What if that? What if I did that?

39:58 Aksana Rahouski: No, we should assist by the way, that's a good test case. Which we should not allow you to save.

40:05 Devon D'Andrea: Yeah.

40:05 Aksana Rahouski: Active sim that there is no sim there to activate.

40:08 Devon D'Andrea: Okay.

40:11 Richard Sacco: Yeah, we should.

40:12 Aksana Rahouski: system should prevent you from creating like, False scenarios like what you looked at but then as long as your scenarios which

40:17 Devon D'Andrea: Right.

40:21 Aksana Rahouski: is all these combinations of Simon active setups between the three Sims, right? That you should derive to a proper. Config file on a carrier matching.

40:35 Devon D'Andrea: Yeah, and it's like there's probably a better way to make this table where like you're not including like I don't know, might be easier, just visually to make this table like habits, so that there's like, I don't know how to make a table that has like, because I'm putting all, I'm putting presents active in active and whether it's Easter,

40:55 Aksana Rahouski: Now, that's

40:58 Devon D'Andrea: physical, sim all in the same cell. I don't know if there's a like

41:00 Aksana Rahouski: Yeah, well, I don't break it. I think this makes this makes sense. At least like a nurture, doesn't make sense to you. It makes sense to me.

41:09 Richard Sacco: Yeah, it makes sense. Yeah, I think the mapping between origin and i-22 is the

41:10 Aksana Rahouski: Yeah.

41:15 Richard Sacco: same anyway, except there's a special rule like, on AT&T really, if, like, you've broken down to, it's like most based parts. Like the, the only thing is like, if it's AT&T and and Verizon, it always has to be dual carrier, right? Unless only verizon's active. And then it that

41:37 Devon D'Andrea: Right.

41:38 Richard Sacco: Will be visa only but otherwise it's always dual carrier and then with Timo, you have it to where if it's Teemo and Verizon's. Inactive, it's always just teemo. So,

41:51 Devon D'Andrea: Yeah, and yes, like the difference between the models. really just like, Honestly like you're mapping the way you're gonna build the mapping. I don't think it is. I don't think it matters. What the model is the model pertains to

42:08 Richard Sacco: Yeah.

42:09 Devon D'Andrea: What we call. The sin. In in the config.

42:14 Richard Sacco: one in terms of mapping that we're just talking about,

42:16 Devon D'Andrea: Right.

42:17 Adam Curcie: Or the model does. Kind of matter.

42:20 Devon D'Andrea: No, I know like it matters. I know it matters. In terms of mapping. Yeah, cuz like you're still gonna have a you're still gonna be looking at it.

42:30 Richard Sacco: Mmm.

42:30 Devon D'Andrea: Like you're still have to look at it. Like is it an i-22 or an origin? but, and from that point on, like, You're mapping should all be kind of similar. I mean, unless you're saying you're gonna hard code every single one of these rules in

42:45 Adam Curcie: Well, no, but it's like you have to just look at the. You have to look at the model first before the care, because they're they map differently is all. So,

42:51 Devon D'Andrea: Yeah. Yeah. um,

42:55 Adam Curcie: Right.

42:58 Richard Sacco: Yeah, that has to got that has to get determined first.

42:59 Adam Curcie: Okay.

43:01 Richard Sacco: For this of the carrier.

43:07 Devon D'Andrea: Yeah, I mean the mapping if you think about well anyway I think you get the picture.

43:07 Adam Curcie: Yeah.

43:08 Aksana Rahouski: Yeah.

43:12 Devon D'Andrea: What about though? Like all the models.

43:14 Adam Curcie: so,

43:35 Aksana Rahouski: Yeah, what I was saying originally is Devon could add literally the slash character after I-22 and make these also pertain to 52's, because they're, they work interchangeably in that, in regards to the way you map, the configurations based on carriers for those models.

43:41 Adam Curcie: Okay, so do you so all models? Because like 10 We should.

43:42 Aksana Rahouski: Oh, yeah, and fives. I guess if you want to unfortunately include them

43:48 Devon D'Andrea: Uh, wait a minute.

43:51 Aksana Rahouski: What are all the models we have?

43:52 Devon D'Andrea: I mean if you mean if you want to get, if you want to really get into it here, we can do this.

43:59 Aksana Rahouski: Well, not in like when we build this code, right? The code.

44:03 Adam Curcie: Well, yeah, I mean like the 41s and 45 hundreds Nm5s actually are all single

44:04 Aksana Rahouski: Ask multiple question. What model are you? What sim you have or Do you have a B? Is it active? Yes. No right. So in models They're more models than the two that would just looked at, right? So how do we derive to an it? Um, a config is there like another like default south of rules for others?

44:32 Adam Curcie: sim.

44:32 Aksana Rahouski: Okay.

44:33 Adam Curcie: Oh, so you know.

44:34 Aksana Rahouski: Okay.

44:35 Adam Curcie: And then, Yeah. He just for the sake. Yeah. Just you can call that 41. /, 45 /, m5. There you go. Now, I think covered all of our bases.

44:43 Richard Sacco: Yeah.

44:46 Devon D'Andrea: well, I mean there are

44:50 Adam Curcie: But he got. oh, Yeah, other stuff. The 202s. We

44:55 Devon D'Andrea: Yeah.

44:57 Adam Curcie: That we, you know, you want to throw them with the there. You

44:59 Devon D'Andrea: This this this doesn't matter. This doesn't matter. We've got this kind of

45:06 Adam Curcie: With the origin.

45:08 Devon D'Andrea: The 202 would go be the same thing as the origin, you're right.

45:12 Adam Curcie: Yep. so,

45:14 Devon D'Andrea: Do they are they all have a Verizon Eason?

45:17 Adam Curcie: Yep.

45:18 Devon D'Andrea: All right, origin. Cr202 done.

45:24 Richard Sacco: Yeah. So basically, none of your models are changed the rules of mapping.

45:33 Aksana Rahouski: I think we have our Bible here and yes, you're right. It's gonna help us. Not only for now to kind of bake these rules in but also once we talk through how config gets build based on kind of some kind of like, Bowler plate. Custom plots how devised?

45:54 Adam Curcie: Yeah. It's a hierarchy. It, you know, you start at the top with Global and then you

45:56 Aksana Rahouski: Yeah. Yeah.

46:00 Adam Curcie: you make adjustments, or are you add parameters from top down? And you would, you would hit Model and then you would go to carrier? Yeah. In that hier.

46:13 Aksana Rahouski: Which we should start talking about that too maybe next week.

46:17 Adam Curcie: What about tomorrow? Are you free tomorrow?

46:18 Aksana Rahouski: like, I would like actually I do think like we need to start talking kinda about like the the levels and then which configs need to be available at which level and how they affect a final config file, as it gets built.

46:38 Adam Curcie: I don't know if we told you but we stole the main engineer in hand for over two hours and made him. Look at all 700 plus values in one can print by a line by line and tell us exactly what it meant. And I documented all, I'm not kidding, I went through every single one, I made

46:55 Richard Sacco: but,

46:58 Adam Curcie: him and he I gave him the option to opt out. I really did. I said something we

47:01 Aksana Rahouski: Right Hood.

47:04 Adam Curcie: don't have to do this right now, but you someone in hand is gonna do this for me. So he did it two hours, straight 700 plus config values. Parameters.

47:15 Aksana Rahouski: Yeah.

47:19 Richard Sacco: Yeah. Are you working on that? We need that now?

47:21 Devon D'Andrea: What where's that Bible at? Is that share on the chair drive?

47:21 Adam Curcie: all right, now I I Have it all. I have it. All right here. So but there was just to give you there, This is funny to say, I'm gonna say, I find this amusing. There was 36 parameters. He didn't know what they were, so we're still waiting for him to find that out for us. So,

47:40 Devon D'Andrea: Yeah. yeah, that was

47:48 Adam Curcie: We went through and identified labeled and described all of them except for 36,

47:48 Devon D'Andrea: Right.

47:53 Adam Curcie: He couldn't. So I said You got to go back to your team in headquarters and get me answers on those first. So, once I get that, then I will give you that that chapter of the

48:04 Aksana Rahouski: Because I I so far how far I've gone kind of did the same thing I took all you config files, kind of define the distinct list of all parameters and across all the, the basic on the costume, I break it into the kind of most used rarely used like 20%. As and never used, right? Because you kind of need to know that especially for building some kind of like configuration builder. Like, you don't want to like over over engineer, it especially for things that you do not need. Like if half of your like a configs, perhaps you're never used, right? You don't need to build any whizzy wig for those perhaps but just some analysis that I have to share. So I do think maybe like maybe sometime next week maybe like second half the week, I would think we should definitely start talking about this one and just put our heads together and keep it moving. but for this, then we have I think we have what we need, then changes that we talked about yesterday, which I summarized in the email that I send yesterday and then adjusting our configuration mapping based on this matrix that were just looked at

49:17 Devon D'Andrea: Dive one thing to add that. I don't know if Adam brought up we discussed it briefly yesterday. So A device. Let me just go back to sharing again. so, this scenario, Right. Race here. Nt only, right? So Adam Do we have to count? Do we have to account for?

49:47 Adam Curcie: Yeah.

50:06 Devon D'Andrea: if this box is on dual carrier, And it's changed to AT&T only.

50:20 Adam Curcie: The only way we can do it is by just having the portal turn off the Verizon Sim and keep the Verizon symbol.

50:29 Devon D'Andrea: Yeah.

50:31 Adam Curcie: Because then the config won't change. That was one of the things that through a wrench into what we were talking about yesterday was

50:38 Devon D'Andrea: if you were saying like that, it would

50:40 Adam Curcie: It's up. Yeah, if it's a if it's a dual carrier, config And and you change it.

50:46 Devon D'Andrea: I'm saying I'm sorry, let's say okay now that it's even an edgier case Edger

50:50 Adam Curcie: Okay. Okay, edgier, I like it.

50:51 Devon D'Andrea: edge case. It's dual sim. It's dual sim populated both are populated but it's up on

50:59 Aksana Rahouski: But isn't that what line 9 is?

51:01 Devon D'Andrea: Verizon.

51:02 Adam Curcie: Yeah.

51:04 Devon D'Andrea: It's up on Verizon Debbie the same thing, right? Because if it's up,

51:06 Adam Curcie: Uh-huh.

51:13 Aksana Rahouski: You have both do two Sims Verizon active.

51:15 Devon D'Andrea: Yes. Yeah. Yes, so it's this.

51:18 Aksana Rahouski: Yes.

51:18 Devon D'Andrea: Going to this. that's not a problem, right, because, right, because if it's got to get a

51:21 Aksana Rahouski: Yes.

51:24 Devon D'Andrea: config, it doesn't It's if it's The only way it's gonna work is. Like do you see what I'm getting at? Adam like you can't force it to to go to AT&T only, If it's not online, right? Like you, like like in these situation, I guess it doesn't matter because it's so obscure, right? Because it's so it's such an edge case. like,

52:00 Aksana Rahouski: are you worried about that actual transition like

52:03 Devon D'Andrea: I'm just like if this, if their Verizon is just like not working,

52:07 Aksana Rahouski: Okay.

52:07 Devon D'Andrea: but you if it's set to only try Verizon via this year,

52:12 Aksana Rahouski: Yep, and those switch to AT&T.

52:12 Devon D'Andrea: yeah, it's not even gonna there's nothing that you we can do to actually get it to, to come up on AT&T because the device It's yeah, I guess it just doesn't matter if it just down Verizon and they're

52:26 Aksana Rahouski: Not.

52:29 Devon D'Andrea: like, Oh can you put it on AT&T way? And we're like, if it can't touch Verizon, there's nothing we can do. Right.

52:39 Aksana Rahouski: There has yeah, unless we need some kind of

52:42 Devon D'Andrea: Right? I'm calling myself. Did he drop?

52:45 Richard Sacco: Yeah, he dropped.

52:47 Aksana Rahouski: Yeah, you left us.

52:47 Devon D'Andrea: That.

52:51 Aksana Rahouski: Thank you, Trista.

52:54 Devon D'Andrea: Alright, well that's that's just something that we have to handle operationally if somebody like because like there's nothing. I like this isn't flip going from this to. This isn't just gonna magically make this device navigate itself to AT&T.

53:14 Aksana Rahouski: And if that needs to be solved, we can solve it too.

53:17 Devon D'Andrea: I don't know. I don't, I don't know. I don't think there's anything we would be because if it's because if it's dual carrier, Where's my dual carrier? If it's this then we know already that It Verizon goes down or vice versa. It's still, it's still should be online, right? It's just I'm just thinking of an education of like verizon's just down and somebody wants this. That's just yeah. Forget it.

53:43 Aksana Rahouski: Okay, so we're gonna table that for now.

53:53 Devon D'Andrea: That table it, forget it.

53:55 Aksana Rahouski: Forget about it for now. Sounds good, I think return. No, you may have sometime later. Let's regroup on this and just to see how much work I have left. Um, and I'll send you guys an update later as far as like, the ETA for these changes and summary of them.

54:17 Devon D'Andrea: Wonderful.

54:19 Aksana Rahouski: Okay, sounds good. Okay, well, thanks so much.

54:21 Devon D'Andrea: All right. Thanks guys.

54:23 Aksana Rahouski: Bye.

54:23 Devon D'Andrea: I,

54:24 Garrett Hood: Thank you.