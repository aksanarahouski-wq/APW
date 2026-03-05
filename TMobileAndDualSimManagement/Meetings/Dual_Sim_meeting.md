00:00 Trista Smith: About adding a discussion point for today. If there's time Or yeah, I was Devin about the origin.

00:02 Aksana Rahouski: Yeah, yeah. We probably

00:05 Trista Smith: Product. May not have time.

00:13 Aksana Rahouski: I think we would need to book something.

00:16 Trista Smith: Okay. We can do that.

00:23 Aksana Rahouski: Let's see, like how what it wouldn't means if it's like that questions or work needs to be done or

00:30 Trista Smith: Yeah. Aaron.

00:34 Richard Sacco: Good day, right?

00:36 Devon D'Andrea: Hello.

00:37 Trista Smith: Julie.

00:38 Aksana Rahouski: Even Richard that you, what's happening? That's not

00:42 Richard Sacco: Yeah, I just like wearing it.

00:46 Devon D'Andrea: My house. Absolutely freezing. I I just been I've been hesitant to put my heat on and not working out.

00:57 Aksana Rahouski: it's so hoping it's gonna get better Devin

01:00 Devon D'Andrea: Yeah. Yeah. it also doesn't help that, I'm just like, you know, Type of guy that wears flip-flops, you know, nine months out of the year.

01:14 Aksana Rahouski: You're one of those, huh?

01:20 Devon D'Andrea: Like being at the beach. All right, John's here.

01:27 Trista Smith: Looks like we don't have Adam yet.

01:29 Devon D'Andrea: Yeah, he's coming.

01:30 Trista Smith: Okay. devon, we saw your note about adding, Bullet point about origin, the origin product.

01:40 Devon D'Andrea: Yeah.

01:41 Trista Smith: So we may need to schedule additional time to meet about that, but

01:46 Devon D'Andrea: So we've got we've got some time.

01:59 Trista Smith: Okay.

02:00 Devon D'Andrea: I did go. I did have a meeting with Chad this morning. Going over a couple of the A couple of the service plan offerings that we're gonna have with that project as far as the website goes and how that's gonna look like for ordering. But we're also going to have to make some tweaks. I guess to the way we assign devices. Which Adam and I were just looking through before this call, and yeah, we'll have to talk about that. So we can get started with all the other stuff and then see if we have time or skill another

02:37 Trista Smith: review some of the feedback. So Aksana, do you want to

03:01 Aksana Rahouski: Um yeah there's an outstanding question regarding the data usage and we'll jump to that in a second. I just wanted to kind of see overall How do you guys feel about testing? How far do you think you are with testing just overall like feedback and status update?

03:21 Adam Curcie: Yeah, I tested outbound from the portal so we can now ping. Thanks to Richard

03:22 Devon D'Andrea: Yeah.

03:28 Adam Curcie: and Larry get in the traffic flowing over the site to site. We saw check-ins

03:33 Aksana Rahouski: Hmm.

03:34 Adam Curcie: from From like a functionality standpoint seems to be okay. As, you know, we kind of mentioned The dual SIM and assignment. I think is still something that we're not a hundred

04:07 Devon D'Andrea: With.

04:09 Adam Curcie: percent.

04:11 Devon D'Andrea: yeah, we haven't done like a lot of testing yet on the dual Sim as far as like, How you assign stuff, you know, if you were to change the active sins on the On the Device page. Yeah, we're kind of just we we're kind of just banging our heads against the wall with that one because It's just there's just like so many things.

04:43 Adam Curcie: here we have got Definitely. A man bunion, my head against the walls a good way to to kind of describe this?

04:52 Aksana Rahouski: Hmm.

04:53 Adam Curcie: We were, and I don't know if we really even have time to go through it right now to any detail but It you, it's, I mean, we're stuck because there there's a lot of liability in the way. It's designed at the moment, I'll just put it that way, like I can share. Give me a second.

05:15 Aksana Rahouski: Let's see, because I was gonna ask you about because it is a change, right? Because before you just have to provide Sim number in order to kind of activate that sim right now. You your SIM number

05:30 Adam Curcie: Yeah.

05:31 Aksana Rahouski: the moment of importing Simon because again, kind of same idea, right before dual had only one meaning, now, they're two duals, right? So during time and not

06:06 Adam Curcie: Yeah.

06:07 Aksana Rahouski: only, you're expected to specify, which SIM is active, if Duel, you have to pick which dual it is, right? So it's it's like it was enhanced with this. So, because assignment today, I don't know if I'd like helps or not when you do device to a company assignment, you really do in kind of two things, you updating device metadata. So you update and device actually a SIM status, which one is active and if it's dual or not and you also adding a company to that device. It just kind of hidden with this like one form. We call a sign but really you

06:44 Adam Curcie: Yeah.

06:45 Aksana Rahouski: are updating device, that's what you're doing here.

06:48 Adam Curcie: Yeah.

06:48 Devon D'Andrea: Yes, so Adam just do me a favor. Just as an example, go to

06:52 Adam Curcie: Sure.

06:54 Devon D'Andrea: Go to W66189. So, this is something that will have to. Try to figure out how we're gonna avoid from happening, right? So this is a device that has populated Verizon and AT&T Sims. We? assigned it as AT&T only and that could just be from Something that was improperly.

07:25 Adam Curcie: Yeah.

07:26 Devon D'Andrea: Documented on an order or just a mistake or whatever. But now, if today, if this were live, that device would be a brick.

07:34 Adam Curcie: office, most likely And the reason for that is the configuration today, the way everything works today. It requires this sim to be in

07:59 Devon D'Andrea: Today.

08:00 Aksana Rahouski: Yeah.

08:05 Adam Curcie: The sim one position which it isn't on this device. and like a lot of these complexities or like you know, these there's a lot of idiosyncrink detail on this very specific to things that, you know, we could account for if we built out enough safeguards to do so like, you know, if the portal sees this, right, like again this hypothetically but like any time there's two sims in whether it's this and this or this and and this

08:36 Aksana Rahouski: Okay.

08:38 Adam Curcie: think of a reason. Right now, we were kind of tossing this back and forth amongst ourselves for a while today. But like For any i-22, I don't think there's a reason you would ever have an instance where the Verizon Sim is anywhere except someone. And then, either of these would be in Sim 2. So, but again, there's not anything in the portal to verify that or or enforce that right now,

09:17 Aksana Rahouski: But how?

09:18 Adam Curcie: So like if yeah.

09:20 Aksana Rahouski: So question.

09:21 Adam Curcie: Yeah.

09:22 Aksana Rahouski: From a portal perspective, right? How did it work? Why do you feel like it used to work? And now it's not going to work because before Portal would have two Sims but Non-dual it would automatically Assume it's Verizon, right? Like you were guys for saying,

09:38 Devon D'Andrea: It's using a problem that's not a problem because the Verizon or a dual carrier config it'll come online.

09:45 Adam Curcie: Because if it has both Sims and we don't select dual Sim it goes out, Verizon

09:48 Aksana Rahouski: Yeah.

09:51 Adam Curcie: only and that config supports the Verizon being in the sim one spot.

09:56 Aksana Rahouski: Right. Which

09:57 Adam Curcie: We, we can't like today I'm gonna go over here. We don't have a way to assign a box.

10:06 Aksana Rahouski: Uh-huh.

10:06 Adam Curcie: That's got. like, we don't have a way to assign a box with both Sims On an AT&T only plan. Do you know what I'm saying?

10:16 Devon D'Andrea: Yeah. It's gonna just the only way, it's gonna get an AT&T. Config only AT&T Only config is if the portal recognizes that it only has an AT&T sit.

10:27 Adam Curcie: Yeah.

10:28 Aksana Rahouski: Which which now it not. It has it that it's, it's an active sim. That's that's

10:28 Devon D'Andrea: Probably.

10:35 Aksana Rahouski: how kind of the shift out that was made.

10:37 Devon D'Andrea: Right.

10:39 Aksana Rahouski: In. And actually, you know, like we because part of this release is gonna be

10:43 Adam Curcie: Yeah.

10:45 Aksana Rahouski: value present, and it's it's activated with active flag, right? So that's why Richard has a script that's gonna go through all your devices. It kind of remapped them this way.

11:09 Richard Sacco: yeah, I think that they're saying with the issue is, is that for this specific case, Or it has both Sims. It can't ever just get the AT&T configuration.

11:22 Devon D'Andrea: Not ever not necessarily ever. We could figure out a way with when we when we

11:27 Richard Sacco: Yeah.

11:28 Devon D'Andrea: read, when we overhaul the configuration engine, we could certainly figure out a way to define, which SIM profile should be SIM profile and APN should be used based on is the same in the second slot or the first slot kind of thing.

11:44 Adam Curcie: Yeah. Yeah, because

11:48 Devon D'Andrea: like, yeah, but then but yeah but today if you get to your point today, the way it works, you take

11:56 Adam Curcie: and then the other thing, I was kind of and I hadn't really gotten enough time to thoroughly, you know, produce a suggestion, but what I was thinking of or like, where I was trying to get to Which Devon doesn't necessarily love the idea of having more service plans. But I mean we don't have a ton of service plans over here. So, I mean, I know on this screen, it looks a lot worse because there's a bunch that aren't actual real ones. But like I'm wondering if we don't have all of this necessarily on this screen,

12:34 Aksana Rahouski: Huh.

12:35 Adam Curcie: what's which device would pick. um, now for T-Mobile, it's gonna be a different price which is already complicated enough and it obviously will be a different config So like having that as a separate service plan, to me, makes sense. Literally, like having like, you know, this over here that literally says T-Mobile, that makes sense to me. Now, I'm not the person who works on the screen. So my opinion is definitely not the end, I'll be all but I feel like

13:21 Richard Sacco: Them.

13:27 Adam Curcie: That's kind of irrelevant. It will become relevant when we talk about configuration, but for the sake of this, right? Like we could have a, you know, this would be considered a new service, man. We could have a second new service plan. that would be, you know, for T-Mobile and Verizon Dual Carrier. Potentially. on ATMs, it just gets You know, it just

14:23 Devon D'Andrea: See that season. But then you got, then you again you run into the issue of like. Now you got to have now you gotta have like brackets, right? Because you gotta have if you have a T-Mobile ATM plan, then if somebody goes to upgrade

14:32 Adam Curcie: Yeah.

14:35 Devon D'Andrea: their service plan, they can only upgrade it to T-Mobile Tier one or T-Mobile. Tier Two or T-Mobile Tier Three.

14:43 Adam Curcie: alright, well here I will, I will, I will

14:55 Aksana Rahouski: so, I'm still kind of a little bit lost on The problem.

15:02 Devon D'Andrea: Okay.

15:03 Aksana Rahouski: Yeah. Sorry.

15:06 Devon D'Andrea: open up that box if you can but I can't because brick

15:09 Aksana Rahouski: Okay. So, we have a device this device has too soon.

15:12 Adam Curcie: Let me, let me unassign this real quick. I think I got that might help if I just

15:13 Aksana Rahouski: Right. Mmm.

15:19 Adam Curcie: Hit this out of here on a sign. This. So now this box is unassigned and it's still And I will come in here, okay? And I will put these as both active.

15:31 Aksana Rahouski: I have to pick duel.

15:32 Adam Curcie: Device can't be saved.

15:33 Devon D'Andrea: Yeah, you gotta hit the dual sim play. Yeah.

15:36 Adam Curcie: Great. See, that's awesome. I love that that makes so much sense though it. So,

15:38 Devon D'Andrea: Great, that's good.

15:43 Adam Curcie: so now, if I'm gonna assign this to Devon, so real quick right now, This box if this see right here. Perfect, perfect is exactly.

15:49 Aksana Rahouski: I found the right big, okay.

15:54 Adam Curcie: This is and and let me, let me see if I can open this where you can see it in a

15:59 Devon D'Andrea: Yeah, that's what I was gonna suggested open, any dual carrier box.

16:02 Adam Curcie: Well, I have to read after redo my share for give me one second. Sorry. Come back here. Present my whole screen. All right so for here, this is the config file and there's a Dual SIM main equals zero. Okay so which is actually like the first SIM so dual SIM main equals zero which you know is the first SIM

16:32 Aksana Rahouski: Uh-huh.

16:33 Adam Curcie: That zero represents SIM 1 on this. Um, so it's saying that for this box, the main sim is this, which it is? It's in sim 1, so that'll work. Now, The problem is that if I come in here and whether it happens in here or on the assigned page, they work in, you know, the same And we do this. It's got an AT&T config. You see that? So if I assign this to Devon and I and I accidentally Like it. Like, you know, again, we're we're speaking in terms of Moving through an operation where like Vince doesn't do anything like this. Now, Vince just comes in and clicks on the service plan. So if this was a box that was

17:31 Richard Sacco: One.

17:32 Adam Curcie: Handed to like, put in our system like this. And the order comes in for an AT&T, and one of the guys grabs this, which they hopefully shouldn't. I mean, there there are like, this is accounting for human error, but if, if this is goes out like this,

17:53 Aksana Rahouski: Uh-huh.

17:54 Adam Curcie: And this field is required.

17:56 Devon D'Andrea: You got to pick a different company. That's not my test account.

18:05 Adam Curcie: Seven test.

18:06 Devon D'Andrea: Not that one either.

18:08 Adam Curcie: Yeah, you know what? It's Devon test with a capital T is what it is.

18:13 Richard Sacco: Yeah.

18:17 Adam Curcie: Oh my gosh, I'm so bad at this. Why don't I know how to use my portal? Flat rate is required if there's no service plan. Okay, just gonna work. Nope. Well, it's not gonna work this way because there's not a configuration

18:33 Devon D'Andrea: Because you know, because you put pick Timo ATM, which that doesn't have a config tie to it.

18:39 Adam Curcie: Sorry.

18:42 Devon D'Andrea: oh,

18:48 Aksana Rahouski: well, you can see like Y in that you I mean, you

18:51 Adam Curcie: I know it's because of the configuration, there's not one on file.

18:55 Aksana Rahouski: Some matching one.

18:56 Adam Curcie: Cannot save.

18:57 Devon D'Andrea: That's you.

18:58 Richard Sacco: What?

18:59 Devon D'Andrea: Not associated. That's a, that's a new one.

19:01 Adam Curcie: company device you

19:02 Richard Sacco: Oh, I gotta look into that specific thing.

19:04 Adam Curcie: Oh wait, is it because I Click that.

19:07 Devon D'Andrea: Maybe.

19:09 Aksana Rahouski: Mmm.

19:11 Richard Sacco: No, it's I got to look into why this is happening.

19:11 Adam Curcie: Nope. It's probably my fault.

19:18 Devon D'Andrea: Me broken.

19:20 Richard Sacco: I don't think it's anyway, but basically I think I understand the issue, basically, if it's a dual SIM and they're immigration, they have the verizon's always going to be sim one, so you send the AT&T configuration,

19:33 Devon D'Andrea: Yes.

19:35 Adam Curcie: But we didn't do assign this way early. I just don't know what we did. Do that.

19:41 Devon D'Andrea: It's probably has something to do with you. Just pushing a lot of buttons in a short period of time.

19:45 Richard Sacco: But yeah, let me we can open a bug for this. I think this is this definitely

19:46 Adam Curcie: But that's gonna be the case.

19:52 Richard Sacco: buggy because it's giving you a nonsensical error message.

19:53 Adam Curcie: It's not associated. The company device.

19:57 Devon D'Andrea: Yeah, yeah. But yes that is that is the problem. The only way an AT&T are our

19:57 Aksana Rahouski: yeah, because we have

20:04 Devon D'Andrea: standard AT&T config will work on a device. Is if the AT&T SIM is in the first seat,

20:12 Richard Sacco: I see so yeah, let me ask you this. That is there a way to do, like a custom? Remember we have custom configuration values. It's very quick. Custom

20:24 Devon D'Andrea: Like technically, yeah, you could have an AT&T config, the portal would have to

20:26 Richard Sacco: configuration value here. That'll just fix this. In terms of that, Do you know what I'm talking about? like you, you

20:33 Adam Curcie: Well. Yeah.

20:42 Devon D'Andrea: recognize that the AT&T SIM is in that. There's a Verizon and an AT&T sim. And then yes, you could send it in AT&T config that has. That's telling the box to use SIM 2

20:55 Richard Sacco: Yes, that's what I was thinking. Because there, if we can like, With this special scenario.

21:02 Adam Curcie: What this? But this kind of comes back to like the actual like

21:06 Richard Sacco: Yeah.

21:07 Adam Curcie: If this was the situation right and the portal was actually creating the configuration that would be an instance where it would create the All here. It would look like this, though. It would.

21:23 Richard Sacco: But kind of out of scope in terms of like what the current because we want to get mobile launched, right? So

21:30 Adam Curcie: Store them, main equals one. so,

21:34 Richard Sacco: Yeah yeah that's what I'm like. What if we just did to a dual sim main equals

21:36 Adam Curcie: On the to.

21:40 Richard Sacco: two? Is that the fix to this?

21:43 Adam Curcie: Mm. Hold on, let me see.

21:50 Aksana Rahouski: This is the problem. So ultimately that we allow these like wrong assignments like this device sim should not be tied to this config because it's not properly represent some positioning.

22:05 Devon D'Andrea: Well yeah. So like I would say that that would be like a hard rule but that won't work. And the reason for that is because I could say like Hey if we're trying to, if we're trying to take a device that has Verizon and AT&T Sims and

22:18 Aksana Rahouski: Uh-huh.

22:18 Devon D'Andrea: we're trying to make it AT&T only don't allow it. Right. Where we could. We can we could do that. However, we can't do that

22:25 Aksana Rahouski: What?

22:26 Devon D'Andrea: because The new product that we're getting is. Gonna have in every single one of them, a Verizon SIM and a T-Mobile Sim. so, It'd be the same problem with T-Mobile.

22:43 Adam Curcie: Yeah. So and on the device, if if we were going to have this device actually in

22:50 Aksana Rahouski: Yep.

22:52 Adam Curcie: the field like this as AT&T only with this inactive, The configuration would be dual SIM enabled. Even though SIM one is inactive, we still have to have dual SIM enabled to even access them too. So it would just be the primary sin and then all these other parameters, we

23:07 Aksana Rahouski: oh,

23:12 Adam Curcie: would have to account for so that it never even tries to fail over. um which is basically just to set all of these to zero otherwise disabled

23:23 Devon D'Andrea: There's no, I don't know. How did I not know that there's no way to configure

23:24 Adam Curcie: What?

23:27 Devon D'Andrea: device to use SIM 2 only,

23:30 Adam Curcie: Too only. Now you have to have, you have to enable this set this and then you

23:31 Aksana Rahouski: so,

23:36 Adam Curcie: may even be able to set this to like disabled or something or but yeah. You. Yeah, you have to yeah.

23:41 Devon D'Andrea: Huh.

23:45 Aksana Rahouski: Just I mean, the dual SIM definition is not only when two Sims are active, it also, when the second SIM is active, that's also considered or needs to be dual.

23:57 Adam Curcie: Well no not for where we call dual sim. We're talking about the same terminology used in two different applications.

24:03 Aksana Rahouski: but like, for, for this, for the for this word, what you're showing,

24:06 Adam Curcie: On the yeah, on on the device label, the device level. That's their, that's the parameter that enables the ability to use the second SIM is dual SIM. So when what we we charge it under the assumption, both Sims are being used. Like we can figure it with both. Like we don't ever really do this with like when we have a box that goes out with two Sims but only one's active. It's always the first sim. so, And and we don't really have to worry about the second Sim because the box isn't using it and it's and it's inactive. Anyway. So, But yeah.

24:49 Aksana Rahouski: So, so boxes with two Sims positioning of the same matter for configurations.

24:50 Adam Curcie: Hmm. Yes, for the configurations, very much. Yeah, yeah.

24:59 Devon D'Andrea: You very much.

25:04 Aksana Rahouski: Okay.

25:05 Adam Curcie: unfortunately, it'd be great if it did in, but but I mean, at the same time, like I said, before, for every single i-22 except for one, I know that there's like one box out there that we have that belongs to J and D solutions that we are not allowed to update. But

25:22 Devon D'Andrea: Yeah. Well

25:22 Adam Curcie: Whatever.

25:24 Aksana Rahouski: All right.

25:25 Devon D'Andrea: That's good. That'll get we got to make sure we get that. Make sure we know which one that is because when they run these scripts,

25:32 Adam Curcie: Yeah. But for it, for every single that I 22, that could use two sims. If there are two Sims present. I know that for 99.9, there may be one there may, I might not even be that but at most there is one box that doesn't have. The Verizon is the first sim for every single other thousands and thousands Verizon is in position one. That I know. at most there was one that doesn't match, but there's definitely not

26:00 Aksana Rahouski: so, does that does that mean ultimately at a high level that when we are saying the device could have many Sims It the minute we activate that sim. It's also important for us to know which position this SIM is in in order to map the right configs properly.

26:21 Devon D'Andrea: Yes.

26:21 Adam Curcie: Well yeah, and I'm saying it in the way I'm saying it, so that you understand if there is a Verizon Sim that will always be someone like that, there's no guesswork. We're never gonna have A T-Mobile and Verizon or or AT&T Verizon, dual carrier device where the Verizon

26:34 Aksana Rahouski: Okay.

26:41 Adam Curcie: isn't in Sim one. We just

26:46 Aksana Rahouski: but you would you showing to us

26:47 Adam Curcie: so,

26:53 Aksana Rahouski: so, this one is Okay, I think you know what? I think we should book a separate meeting to like To dive on because I'm sure we can find a solution, but I think the way it is right now, it's obviously there's a loophole that we're all seeing, right? And you guys don't need to like, tip it toe around it. We could figure out how to solve it. I think we'll just need to yet define it a little bit better. Like what is the desired behavior that we want from the system and then we can model it and build it.

27:29 Devon D'Andrea: Yes.

27:30 Aksana Rahouski: Okay, so let's yeah. Go ahead.

27:31 Adam Curcie: I mean, oh, I'm just gonna perhaps answer that exact question with just like, Getting that configuration engine to just make the configs because I think I think that'll give us a lot more flexibility. like, I mean, you know,

27:52 Aksana Rahouski: yes, but it's still no like Like, when you look at the portal, right? Right now is just, hey, there are all this. Like three Sims could be technically, three. Even, I know there will be no boxes with all three, but there could be like all three sims provided, right?

28:05 Adam Curcie: Yeah.

28:09 Aksana Rahouski: They could be activated and they could be comboed, right? There's like, there's this dual concept but also it sounds like to me that there's also a positioning of these things Sims that matters for Appropriate configs to that device, right? Which right now it's like completely missing. For you to configure that this is slot 1 versus a slot two and right and how that that place. So even I feel like even when you do have your configuration engine and you can build your configuration much faster, right, it still doesn't solve the problem of. How do you configure device that it knows exactly how to match configs? Because the positioning is like a big key in this decision is

28:55 Devon D'Andrea: Yes.

29:01 Aksana Rahouski: nowhere to be found right now. Like baked in. And like I know before was the easier because they were just two, right? So you're only default behavior was assume. Now, there are three. Right? So, with three in different combinations, it's much harder to kind of bake it in the system.

29:18 Devon D'Andrea: Yes. Yeah, it's special. When like most of the devices that we've gotten from, in hand over the past, you know, seven years, the way they come to us, as far as Sim configuration is the way that they get assigned. And what I mean by that is like if we get 5,000 unit PO and we get 2,000 of them dual carrier, they load the Sims in, for us, ship them to us. We put them on our shelves and we sell them as dual carrier. They sent us. 3000 that a Verizon only, and they only have a Verizon SIM card. We put them on our shelves and we sell them as Verizon only That's all gonna change with the new product, they're gonna sell send us. And a lot of that has to do with the fact that the new product has a has an embedded sim. So every single one of them already has an embedded Verizon sim and then a physical T-Mobile Sim. So every single one of those devices that comes to us is gonna be populated into the portal with the Verizon SIM and a T-Mobile SIM and that is not going to um, Dictate how we sell them. That doesn't mean we're going to sell them. All is dual carrier, we're gonna be selling them as some will carry or some Verizon some T-Mobile. So, yes, all of this very important,

30:31 Aksana Rahouski: Yeah. Um okay, so I suggest, let's try to squeeze another meeting this week for like an hour and and this as soon as possible in which tomorrow and just resume this

30:42 Devon D'Andrea: Yeah.

30:48 Aksana Rahouski: and like one more time kind of go over to see what's missing. Because again, like configs or not like whether you have a builder or not, that allows you to quick access to this configs and create them at a faster speed, right. I think, right now, we're still talking about defining the device to a degree that allows that matching like easy,

31:12 Devon D'Andrea: Yes.

31:14 Aksana Rahouski: Okay, and we can also like during that meeting, talk about the data usage.

31:22 Devon D'Andrea: Okay.

31:23 Aksana Rahouski: How you guys, quick question, that would you send today that we need? Also talk about this origin product, Is this, the product that you just described with the T-Mobile. Okay, so it's all related. So, I think it makes sense to kinda go over

31:35 Devon D'Andrea: Yes.

31:39 Aksana Rahouski: all these things like where today, the new product brings to the table, right?

31:41 Devon D'Andrea: Yes.

31:45 Aksana Rahouski: And like, what CONFIGS on the device other than SIM value active SIM,

31:45 Devon D'Andrea: They're yes.

31:52 Aksana Rahouski: Positioning maybe something else matters like for device, configuration.

31:57 Devon D'Andrea: Right. Yeah, the other big thing we need to talk about with the origin is another program that we're offering the It's gonna have to I think we talked about it before but we need to leverage potentially leverage, the The Financing option on the assigned page in order to kind of just make it so that we can do it as an offset rather than an additional charge per month for a set period of time. We can talk about that.

32:27 Aksana Rahouski: Okay. Um, so we will table that until Trista will help us to find some time in Buckhead. I have a quick question regarding that. Email that I sent you guys yesterday and now if you had a chance to look regarding that, The the Distributor Upcharge um shift the we're doing so basically moving 795 minimum charge for sub companies and applying different subcharge model to distributors with just one device and one payment method. So they're some use cases, I've rushed out. I just wanted to make sure you guys saw it and

33:09 Devon D'Andrea: I see it now. And yeah, I'll have to

33:11 Aksana Rahouski: Okay.

33:15 Devon D'Andrea: Fully to fully die. I, I listen, I wish we didn't have to do this. It's just seems like work that we Shouldn't have to do. But the same time it's like I don't know, it's just annoying because we have this one distributor that has all of these one-off accounts and a lot of them bounce, their AC H's. It's pain in the ass. And then, you know, we're dealing with that, we're putting in additional work with email correspondence for Balance Dch and at the end of the day we're looking at it and we're like, Wait, we're only netting. 375 out of this guy, you know? And that's not including our ACh fees and carrier, you know. So it's like

33:56 Aksana Rahouski: You don't have. And listen, you don't have to apologize for you. If work needs

33:57 Devon D'Andrea: We kind of have to just have that. Floor. For this one-off situation.

34:07 Aksana Rahouski: to be done, it needs to be done with our goal is just to make sure we could

34:08 Devon D'Andrea: Yeah.

34:11 Aksana Rahouski: define karma's properly. So we don't build something that we don't want to. So

34:15 Devon D'Andrea: Sure.

34:16 Aksana Rahouski: that's why I flashed out. There's some like, negative use and again, I know there is only one, but keep like, if there will be more, all these use cases could exist, right? So I just want to we're thinking through all the stuff and

34:27 Devon D'Andrea: Right.

34:30 Aksana Rahouski: then I I don't think it's gonna be like, a heavy you left. So, but like, for now we're just trying to figure out that require nail the requirements. So we are building the right thing.

34:40 Devon D'Andrea: Okay.

34:40 Aksana Rahouski: So, yeah, maybe just three three. I just wasn't sure if like because I know you guys usually respond right away. So I wasn't sure if you just like, were confused by it or had a question or just didn't get to it yet.

34:52 Devon D'Andrea: Now, it was one of those were you know, it's 10 of five on a Monday, I think we were pushing out of s*** ton of firmware's for something and Just one of those, okay.

35:02 Adam Curcie: I'm gonna I'm gonna I'm gonna re hijack. This screen share for like literally

35:02 Aksana Rahouski: Okay.

35:07 Adam Curcie: hopefully less than 30 seconds.

35:08 Aksana Rahouski: Huh.

35:10 Adam Curcie: And I can I put this together really quick.

35:13 Devon D'Andrea: You know what you mean?

35:16 Adam Curcie: Really it's it's that helpful. So like this is basically every combination of

35:19 Devon D'Andrea: Yeah.

35:23 Adam Curcie: the way that the devices exist right now. In terms of how their SIM cards are installed,

35:29 Aksana Rahouski: Okay.

35:30 Adam Curcie: These would be active Sims. So for Verizon, only there would not be something here, T-Mobile and AT&T, and then in the instances, if we ever do have ones,

35:37 Aksana Rahouski: Yeah.

35:41 Adam Curcie: that would be require the AT&T or T-Mobile to be the primary and in the SIM two position and this would be Deactive, just trying to give you like a visual, maybe. I don't know if this helps at all, but I can send this over just to try

35:57 Aksana Rahouski: No, that's a lot.

36:00 Adam Curcie: to you know, because again we're not

36:02 Devon D'Andrea: And also you should also add. You should also add that Verizon only could exist with both populated.

36:11 Aksana Rahouski: because you're by,

36:12 Adam Curcie: You both.

36:13 Aksana Rahouski: Sing by Sim one and two, you're dictating the positions, right? Okay.

36:19 Devon D'Andrea: Yes.

36:19 Adam Curcie: Yeah, this position on the it's a physical tray that has to hold Sims.

36:25 Aksana Rahouski: Okay, and then your columns, that's what you're like. It's Verizon only. I'm

36:26 Adam Curcie: so,

36:31 Aksana Rahouski: assuming it's a, it's a, it's a box with Verizon Sim active, right?

36:36 Devon D'Andrea: Yes.

36:37 Adam Curcie: Well, you like our like the way we're billing it to customers like dual carrier.

36:37 Aksana Rahouski: And it's a

36:41 Adam Curcie: Like this is what we have today. This is like today, don't carrier next door. Carrier would be like that. So,

36:45 Aksana Rahouski: It's got. Okay, okay.

36:53 Adam Curcie: yeah, but like for anything with more than one same, which, you know, be these and these verizon's always gonna be in the sim one position, so,

37:02 Aksana Rahouski: And when you were saying earlier, when we started with your said, this particular device would be a dead if we did that like what was combination that

37:10 Adam Curcie: Yeah. So that would be that would be this combination verizon's in the front and

37:12 Aksana Rahouski: would happen.

37:19 Adam Curcie: At&t's in the second but you would have sent this config which accounts for AT&T is someone

37:26 Aksana Rahouski: I say.

37:27 Adam Curcie: That's our AT&T only today. So like this, this could exist. So again this would like could exist should probably be right here, but yeah, but they don't exist right now not in the portal. Like the devices can support this, we don't have configurations in the portal

37:43 Aksana Rahouski: but also like, okay, I'm but also

37:47 Adam Curcie: that support this.

37:50 Devon D'Andrea: Right. It just comes down to like it comes down to what it, what the device, what what the what the configuration is telling the device like yeah.

38:01 Adam Curcie: Yeah, it cut. Yeah, exactly. It comes down to how the portal is going to send the information to the device.

38:09 Devon D'Andrea: So, today we don't have the portal sending anything to support this. We could but we don't, that's just how it is, and that's been fine that way. but like I said, if we change it, yeah, if we think but problem is the problem is, is that today win beta, you can make it so that that does happen, but it not doesn't, it doesn't work, it breaks it

38:33 Adam Curcie: And they're really not a way that, you know, we'd I can't think of a way in here in service plan. Sorry in Here, I can't think of a way that we could differentiate, you know, like when you go to attic configuration, we have a few options but like, you know again and we don't want to have to build out 30 more, config files either. That's kind of the whole reason. We put the ticket in to make the engine is because we've already got, you know, hunt over a hundred, config files that we're maintaining in here.

39:11 Devon D'Andrea: yeah, I mean think about if you had all the different config files for like a custom company and all the sudden they needed like all of these extra

39:18 Aksana Rahouski: Yeah. Yeah, you have to update all your files now.

39:24 Devon D'Andrea: Yeah.

39:24 Adam Curcie: All these files, all these fun calls.

39:27 Aksana Rahouski: I actually downloaded, well, not all these files, but a lot of these files since I started kind of chewing on this and

39:34 Devon D'Andrea: Yeah.

39:35 Adam Curcie: So and like you know, we could certainly, like I said it's not a lengthy you know it's I I think the number of rules to kind of you know close all of those loopholes and gaps is far less than we'd have to deal with. In terms of the number of configuration files. We'd have to make Um, so like, you know, we could definitely have a call to talk about like, you know, This is where things are today. This is where it's going and these are definitely possible. Um, but it would just, again, come back to the portal. Having to literally see this and know. Okay. Well, if I have these two, but I'm only using this one that I sent, you know, this file, which is right here, which AT&T in SIM 2, and that's the one we're using this one deactivated on the carrier level. So we will make sure that the configuration doesn't allow for the device to use it because it'll fail. So which again, like it's all possible. So just more parameters and more more coding for Richard and Stone and Aaron.

40:38 Aksana Rahouski: Okay. and just this, this does not include yet the whole like new device with that, you just described earlier

40:52 Adam Curcie: Know not really because technically on the new device you don't have this, you have He said, Yeah. Someone and you have ease him and I forget which order

40:57 Aksana Rahouski: Okay, another box. Okay. Okay.

41:04 Adam Curcie: they're in because it actually works in the same manner that I kind of was showing you here. So one of the, I think it's the esim

41:12 Aksana Rahouski: Okay.

41:15 Adam Curcie: Is only available if dual sims enabled or vice versa. But yeah, like you can't up top. I think it's Sim one. And you can't make like you can't have the second sim whichever sim is resort one of them's reserved basically to be dual sim enable. So But yeah, so it does work in the same manner where but it's just it's a different. It's a different parameter on the devices actual config because it's not sim 1 sim 2. It's esim and sim one and I can get all of that for you. I don't have it in front of me but we have other devices that operate that way too. The Cr202 operates in the exact same manner for having any embedded sim

41:58 Aksana Rahouski: All right. but, But ultimately, there are only two slots right in a box.

42:07 Adam Curcie: Well, I'm yeah on. So there's there's two The right now. For the devices that we support within and they can only support a maximum of two SIM cards.

42:18 Aksana Rahouski: Okay.

42:19 Adam Curcie: So, on the I-22, you do have two physical Sims and you know, slots if you will.

42:23 Aksana Rahouski: Uh-huh. Okay.

42:25 Adam Curcie: This one, technically, I mean, just in terms of, you know, specifically, the word slot isn't applicable. This only has one physical sim slot and it only takes one physical thing. This sim is an embedded sim, so with the device,

42:37 Aksana Rahouski: That John.

42:38 Adam Curcie: whether we use it or not,

42:40 Aksana Rahouski: Okay.

42:40 Adam Curcie: So and the more I think about it, I believe it's like that. I believe it's the sim one and then that you in order to take advantage of the embedded sim, you actually have to enable dual SIM and set main SIM to Esim

42:53 Aksana Rahouski: So you can. So you really as far as like Sims Device could have

42:59 Adam Curcie: The max is too.

43:01 Aksana Rahouski: Two, but they could be called on some models. It's someone some two on all the models, it's someone in Esim.

43:08 Adam Curcie: I've I believe so I'll double check that and update you on the ticket or an email. Um but it honestly it may show. I know it shows differently on the

43:13 Aksana Rahouski: Okay.

43:18 Adam Curcie: interface but in terms of how the parameter itself actually is reference in the big file, it might be the same. I need to double check

43:25 Aksana Rahouski: Okay.

43:27 Adam Curcie: But, and this one, you know, yeah. That's all we got. That's all I got right now.

43:34 Aksana Rahouski: Okay, no, this is helpful. If you

43:36 Adam Curcie: We we're way over time and we, I guess we need another. We're gonna need more time.

43:40 Aksana Rahouski: Now we do well we'll book another meeting and let's talk about that about the new device, it's because this kid get out of hand really fast daily. We want to find

43:51 Adam Curcie: It already is out of hand. I'm not

43:51 Aksana Rahouski: Leaners solution because it's one of those things you could just keep snowballing and then it's just you just gonna it's not gonna be good. I'm hoping we can find some kind of like Lean Solution to it that but also like,

44:01 Devon D'Andrea: Yep.

44:07 Aksana Rahouski: Because my problem right now I feel like that it portal doesn't like truly represent. What device is it but it's trying to match your CONFIGS based in the information is missing. So that's going to figure out what's missing.

44:22 Adam Curcie: Well, yeah, it's that. We designed this to match configs based on far, fewer per or like, requirements that we have that we're going to have. Yeah.

44:28 Aksana Rahouski: Yeah. Yeah.

44:31 Adam Curcie: So we have to adjust that and we have to do it in a way that doesn't make Vince's life, a living hell, because I would I would be fine with putting all of this on Vince because it's not my job. So he gets it there and figure out how to match all of these Sims to service plans but he'll probably, you know, murder me after eight finds out, I did it. So,

44:49 Devon D'Andrea: He was murderous.

44:51 Adam Curcie: Yeah.

44:53 Aksana Rahouski: If you're not. Okay. So let's Just have you can find an hour and

45:00 Trista Smith: Yeah, tomorrow at 1:00 or 1:30 works for the oris's team. Does that work Adam and Devon?

45:06 Adam Curcie: Wow, we're all gonna be in the office if we can do one, what would it be one to two?

45:11 Trista Smith: Yep. or yeah, one or two or

45:14 Adam Curcie: I don't know. I think we can.

45:18 Devon D'Andrea: They're already do we I already I already see some of my calendar for that with you guys. Is that old? It's canceled. It says okay.

45:24 Trista Smith: Yeah, that was over here.

45:26 Adam Curcie: He?

45:27 Devon D'Andrea: Yeah, one's fine.

45:29 Aksana Rahouski: Okay.

45:30 Trista Smith: Okay. All right. We'll do one to two and then Devon and Adam. I'm gonna email you because we had on the agenda, to talk about a site visit down here, in Frederick

45:41 Devon D'Andrea: Yeah.

45:42 Trista Smith: I'll email, you get that conversation started there and then we can always touch base about that. So

45:50 Devon D'Andrea: That's great.

45:51 Aksana Rahouski: Right. Sounds good. Well, we'll resume tomorrow, bring your ideas.

45:55 Devon D'Andrea: All right. Yeah, we'll be in the office of what plenty of devices to

45:59 Adam Curcie: yeah, we're all gonna there tomorrow, so be

45:59 Aksana Rahouski: Perfect.

46:00 Devon D'Andrea: Show you. Yeah.

46:03 Aksana Rahouski: Awesome. Alright.

46:04 Devon D'Andrea: See you guys.

46:04 Adam Curcie: All right, talk to you then.

46:06 Aksana Rahouski: Bye.

46:07 Trista Smith: Right.