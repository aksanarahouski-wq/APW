00:00 Aksana Rahouski: Hey guys. Anding never ending list of things to solve.

00:06 Trista Smith: Thank you.

00:08 Devon D'Andrea: Mmm, I'd like to introduce you all to Garrett. He's been with us for years but just, you know, he won't get a little more involved. See what's going on with

00:15 Aksana Rahouski: Uh-huh.

00:18 Devon D'Andrea: stuff. So don't

00:19 Aksana Rahouski: Yeah.

00:19 Garrett Hood: Hello closer to me now, everyone.

00:20 Aksana Rahouski: Hi Garrett.

00:22 Devon D'Andrea: So, Garrett Garrett and John, just just to be fully transparent if they're on

00:23 Aaron Diefes: Actually.

00:30 Devon D'Andrea: these meetings with me and Adam, or me, or Adam. They're also multitasking with other stuff. So if you see them kind of Dip in and out, just that's why.

00:41 Aksana Rahouski: that we identified yesterday. How do we properly with Three Sims dual combinations position of the stems. And how do we kind of manage the device properly to the portal? And at the same time, can guarantee a proper configuration assignment. But I think maybe before we go there, if you guys want to tell us more about this new device, because I think this new device adds even more flavor into the landscape of Simpson positions and etc.

01:24 Devon D'Andrea: Yes.

01:26 Aksana Rahouski: And then we kind of maybe start there and see how much complexity and scope This adds. I do I did kind of we synced a little bit on the computation we had yesterday. Um do I want to like we want to maybe walk through some options ideas etc.

01:44 Devon D'Andrea: There's really only like two things that are vastly different. Number one, there's no IO digital. This is not compatible. It does not have an IO port as far as physically. So you will not see the, the IO settings in the configuration file. So there is not need to be, I think on, I can't remember, but I think we already have the ability to not show the restart ATM or restart whatever button. For some devices. So that's that's number one.

02:35 Aksana Rahouski: And I think that is tied into the model. Which are you guys going to be creating a new model for this devices or reusing? When it

02:43 Devon D'Andrea: Yeah. Yes, it's a brand new model and it's a brand new manufacturer.

02:48 Aksana Rahouski: Okay.

02:49 Devon D'Andrea: Config groups. So yes. Okay. So we do already have. See, this is where I know like this kind of I know like this is can be built out build upon as far as, you know, we're doing I know it's like, says browse

03:07 Aksana Rahouski: Funny that you said that because we literally have like a card sitting at the

03:10 Devon D'Andrea: configuration groups but but yeah, we would add it in here, but as far as the manufacturer goes, I think we have to give you. I think we have to tell you that.

03:27 Aksana Rahouski: top of the queue for you before to add that like administrative layered

03:28 Devon D'Andrea: Yeah. Right.

03:34 Aksana Rahouski: manufacturer model management. So you get build your own values and not come to

03:36 Devon D'Andrea: Right.

03:39 Aksana Rahouski: us. Every time I need add a new one.

03:41 Devon D'Andrea: Hold on. Just want to show you. Some.

05:07 Adam Curcie: Hey.

05:08 Devon D'Andrea: goes, it's either gonna be Sim or Esim. I'm not sure what that looks like in the config though, but we can look at that.

06:02 Adam Curcie: Yeah, we can.

06:04 Aksana Rahouski: Quick question on this while we're on it. Duels them from perspective of In-hand network editor and dual SIM from a perspective of Apw portal. There are not an exact match like it's not the same definition of a dual SIM, right? Or are they, sorry?

06:25 Devon D'Andrea: Oh yes. Well the yes because us. This being enabled is the same as us. Checking the box.

06:34 Aksana Rahouski: Okay, okay, so that's good.

06:36 Adam Curcie: will be on T-Mobile only but in the actual device is config, that dual SIM check box is still gonna have to be I'm sorry, Verizon only. Yeah, Verizon not T-Mobile. So with the origin, if we're building a customer for Verizon, only that check that dual carrier. Checkbox on the device will be checked but not on the portal.

07:14 Aksana Rahouski: so, like do

07:15 Adam Curcie: because the only way we can have these devices register on the tower with the Esim is if that box is checked and the Esim is set to the primary sim and then the other

07:29 Devon D'Andrea: No, that's right.

07:29 Adam Curcie: the ones that are dictating to the device to not check and switch. So like if we set the max or the csq signal at cellular signal quality threshold, if we set that to five then the device will know if the cellular signal is too low to switch but if it's zero it's disabled so it's not going to switch. So it's going to continue to use the Easton but we can't. Actually have it, use the esim, unless it's configured as you're looking at right there. So, so right now, Yes, dual sim is dual sim, but in the future that checkbox and the way we build customers for both Sims is not going to be something that are like mutually in tandem

08:16 Devon D'Andrea: Yeah, that's true. Yeah. Like, but that's just gonna be dictated by the configuration parameters. Like, If it's gonna be T-Mobile, I'm sorry. Verizon only then. I'm sorry. Which one is it? Adam? Is it Verizon only?

08:34 Aksana Rahouski: One.

08:36 Devon D'Andrea: Or is a T-Mobile only has to be.

08:38 Adam Curcie: Yeah. Horizon only yeah because the Sim is Verizon which is you know funny since

08:40 Devon D'Andrea: Yeah. Right.

08:43 Adam Curcie: it's not actually certified with Verizon but

08:45 Devon D'Andrea: And then we just the variety, the T-Mobile will be inactive. So these, these settings will be set to the minimum threshold possible without disabling them and so that it fails over a pretty much just immediately to Verizon to the to the Eastern.

09:15 Aksana Rahouski: okay, so Verizon Sim is

09:19 Devon D'Andrea: And that's just a limitation of the firmware. We asked them to fix that, but we don't know when we're gonna when or if we're gonna get that.

09:26 Aksana Rahouski: Mmm.

09:27 Devon D'Andrea: Ideally, you would just have this disabled. And then up here, there would be a setting that says, you know, what's

09:35 Adam Curcie: It really all they have to do is move the main sim, drop down. That is visible with the dual sim enable checkbox that main SIM just needs to go above

09:41 Devon D'Andrea: Yeah.

09:43 Adam Curcie: everything else. But

09:44 Devon D'Andrea: if I wasn't here like we wouldn't have a problem so it is kind of annoying but

09:46 Adam Curcie: Yeah.

09:48 Devon D'Andrea: it is unfortunately the what we have to do

09:51 Adam Curcie: Yeah, but we yeah they have not gotten back to us with a definitive. like date that they'll be able to do that if they can even do it at all but it has been requested at least two or three months ago but

10:03 Devon D'Andrea: Yeah, so like this Max number of dials would be one. This would be you know what? One

10:11 Adam Curcie: now, I think it's got to be zero for

10:13 Devon D'Andrea: This one, this one can be zero.

10:14 Richard Sacco: so, so what the one of the issues that we're talking about is that, because this has special properties, where you have to make it, that this box is checked in a certain values, have to be a certain way because it's an esim

10:30 Devon D'Andrea: Yes, it's yes. Yes.

10:31 Richard Sacco: the configurations wouldn't apply as usual, is that

10:36 Adam Curcie: Well, I mean for this model it is going to be different in the way that the

10:41 Richard Sacco: Mm-hmm.

10:41 Adam Curcie: configure handled. Yeah.

10:43 Richard Sacco: Oh, because you could do it by model anyway. Okay, got you? So

10:46 Adam Curcie: Yeah, because this like I said the i-22s, they don't have e sims.

10:47 Richard Sacco: Okay.

10:51 Adam Curcie: so,

10:51 Richard Sacco: Mmm.

10:52 Adam Curcie: yeah, and we don't have any we still like as I was kind of referencing for other

10:53 Aksana Rahouski: so,

10:58 Adam Curcie: reasons yesterday, we don't have any i-22's out there where, Sim Two is the main sim Like for every single I 22 on, if it's dual carrier the sim one is still the

11:08 Richard Sacco: Yeah.

11:12 Adam Curcie: main sim. So

11:13 Richard Sacco: Okay.

11:14 Devon D'Andrea: Yeah, just for reference on the I-22s. Under the cellular parameters. it would you would have Sim one network provider would be corresponds to the the APN listed here. So, like you would just choose like we don't If we wanted to, if we wanted to activate an a SIM card that was in, I get what we have to do with the same way. Adam. If we were to

11:49 Adam Curcie: What?

11:50 Devon D'Andrea: For an i-22. If we were to have it be if it had AT&T and Verizon in it,

11:59 Adam Curcie: And we wanted only AT&T in SIM 2.

12:02 Devon D'Andrea: Yeah.

12:03 Adam Curcie: Yeah, you'd have to enable dual SIM and then you'd have to select main SIM to

12:06 Devon D'Andrea: Right.

12:08 Adam Curcie: SIM 2.

12:09 Devon D'Andrea: right, you would deactivate Verizon

12:10 Adam Curcie: and then, Change. Yeah, that would allow it but we don't have that set on any devices. Except for that one JND box. Wait.

12:24 Devon D'Andrea: Right.

12:26 Adam Curcie: So for that one single box.

12:28 Devon D'Andrea: Yeah. Yeah. So there's that. So now we're getting into a can of worms that has nothing to do with why we even scheduled this call.

12:37 Adam Curcie: but,

12:38 Aksana Rahouski: Yeah.

12:39 Adam Curcie: so,

12:39 Devon D'Andrea: That that's cool.

12:41 Aksana Rahouski: into the the new model that's coming in right? And on top of it per device, you need to define which of this Sims is a main sim.

13:07 Devon D'Andrea: Mm-hmm.

13:08 Aksana Rahouski: Or is it? So let me let me share something I build something on. Just to kind of help this conversation.

13:15 Devon D'Andrea: Yeah, we need help.

13:15 Aksana Rahouski: so,

13:17 Adam Curcie: Oh yeah.

13:18 Aksana Rahouski: Okay, so This is a mock-up of our current other device page, right? This is whereas we're setting this like three Sims. And then this is the table that we looked at yesterday.

13:31 Adam Curcie: Yeah.

13:32 Aksana Rahouski: whether it's active or not. It's also important in which position the SIM is sitting which we know someone too. Is simmers coming now too. So there's three positions, right? So other than saying, like Verizon sim number sitting in someone active, Do we what's? I'm trying to figure out If this kind of combination of these three properties. We can solve for all these different scenarios of devices to at least like when we look at the device we understand like what that device is or do we need that? Additionally To this poor device. We also need to identify which one of these themes is a main sim or are we saying that main is the same as active? Or.

14:42 Devon D'Andrea: that that's just not sure.

14:55 Aksana Rahouski: Oh, and maybe like, let's look through these different. These, these are the options that we looked at yesterday, right? Verizon only. So, when I look at

15:01 Devon D'Andrea: Yeah.

15:05 Aksana Rahouski: this first column, are we saying that? It's a literally, it's a device with just one sim, right? And no, it's only Verizon Sim is, is gonna be active? So if we lean on this, my Verizon is going to be sim one and it's going to be active. None of these other things better, right? So, look, technically, we will not have. I mean, it doesn't have undefined that's me. Reload this.

15:27 Devon D'Andrea: Right.

15:31 Aksana Rahouski: and is sitting in the position one. So I'm treating as a single slaw device with one SIM active. It's Verizon.

16:00 Devon D'Andrea: Yes, very straightforward. Yes.

16:03 Aksana Rahouski: Okay. So, dual Verizon AT&T, so we will do. Some two active dual. So now we have two Sims one, two, both are active,

16:12 Devon D'Andrea: Yeah.

16:16 Aksana Rahouski: so now T-Mobile same idea you

16:20 Devon D'Andrea: and in both of those that you already went through, only apply to I-22s. Both of the scenarios. You just went through in the way that you're showing it on this screen is only about I-22s.

16:36 Aksana Rahouski: Yeah. Is that? Somebody had a question, Adam.

16:42 Devon D'Andrea: Adam. Jump in.

16:45 Aksana Rahouski: Yeah.

16:45 Adam Curcie: Yeah. So and How much want to apologize because like we're gonna have to just pump the brakes here for a second.

16:53 Aksana Rahouski: Hmm.

16:53 Adam Curcie: No, I I was talking to Devon about all of this and other things that like we didn't account for that have actually nothing to do with anything. We talked about so far on this call and I actually had. So I just want to show you something that I think might make this easier.

17:08 Aksana Rahouski: Yeah. Yeah.

17:12 Adam Curcie: um, So I gotta find my paint here. Where did it go?

17:18 Devon D'Andrea: We're all gonna share. We're all gonna share our screens by the end of this

17:18 Adam Curcie: With my p. We're all good.

17:21 Devon D'Andrea: call. We're not, we're not.

17:23 Adam Curcie: Why there it is. Okay, so here

17:26 Aksana Rahouski: There isn't like any ideas, a good idea. Let's just brainstorm together.

17:30 Devon D'Andrea: Oh yes, thank you. This is a great call. We got paint. Open. Here we go.

17:36 Adam Curcie: so we were I was thinking for devices where we know we're gonna have Sims in Both like we're gonna have two sims and devices.

17:48 Aksana Rahouski: Uh-huh.

17:48 Adam Curcie: At assignment, which ones should be active. We should probably do that before. So, a 9 1 4, 8, whatever it is. And we and and so for like a do, this would be a

18:00 Aksana Rahouski: Okay.

18:08 Adam Curcie: dual carrier box. Both of these Sims are here.

18:11 Aksana Rahouski: Uh-huh.

18:11 Adam Curcie: So, we don't lose it. And it doesn't matter which carrier it is because it's not active. But it's really just like almost like and then, if we ever want to re we ever decide like, okay, we know what, we actually need these. 100 boxes that we pre-designated to be T-Mobile only, we can re-import them or do a bulk update so that it looks

19:14 Devon D'Andrea: or or have or had something or have horses, build us a way to like

19:18 Adam Curcie: We already have bulk updates or an important. I mean it

19:20 Devon D'Andrea: but yeah, if it's messy though but I I hear what you say,

19:25 Adam Curcie: so,

19:27 Devon D'Andrea: What fun?

19:28 Adam Curcie: I don't know, man. Don't get off my font.

19:32 Devon D'Andrea: So yeah, so that

19:33 Adam Curcie: But that that was I think might make this just way more easy because we're never gonna have three Sims, right? So it's only ever gonna be one or two. And then, you know, and if it's too but we only need one, the carrier doesn't

19:42 Aksana Rahouski: Uh-huh.

19:43 Devon D'Andrea: Yeah.

19:47 Adam Curcie: So it's like we can just set that to inactive because that might make it easier and less messy than having to. Have all of the I'm gonna say liability of knowing which Sims are active and inactive. like, at the time because it

20:13 Devon D'Andrea: Right? Because then because then it's more so it's just like a it's more this makes it more of like a like a you don't have to think you.

20:20 Adam Curcie: Well, just make more in line with the way we assign now, which is that over no

20:24 Devon D'Andrea: That's I'm saying.

20:26 Adam Curcie: consideration for the carrier.

20:27 Devon D'Andrea: So so if you, so right now, a lot of this predicated on like if there is something populated in these sim fields, right? So if you take, if you take the T-Mobile sim and put it back to the way, Adam hit control Z a few times. If you take, if you move it back to the way he had it to, where the T-Mobile Sim was was not, there was in the bottom part.

20:50 Aksana Rahouski: Former.

20:50 Devon D'Andrea: And it's like, okay, it's there, we have it just for like record.

20:55 Adam Curcie: Yeah.

20:55 Aksana Rahouski: Uh-huh.

20:56 Devon D'Andrea: But in the terms of like what the portal needs to know is now, you're just looking at it the same way that you're looking at it today which is like, Oh, this is a Verizon only box, there's nothing else in any of the other fields.

21:10 Adam Curcie: I'll tell you how, I'll tell you how this is that.

21:10 Aksana Rahouski: But that's what I'm like. How is that different though? From what we have right

21:11 Devon D'Andrea: Well, that's where we needed.

21:16 Aksana Rahouski: now because it's, it's really The tricky part about this. Let's say you guys hire me today and I come to this portal. I'm not no idea of this used to be AT&T or to Mobile. Number, right? That's all I know. But like, where that number you come from. I have no idea.

21:36 Devon D'Andrea: Pre-designated for Verizon only, we still want to know what that T-Mobile sim is. But maybe it make life easier. If we just Store it somewhere, but not in, but not in that field.

22:08 Adam Curcie: Yeah. Yeah, so it it like, okay so we can talk through a hundred different

22:13 Devon D'Andrea: I don't know.

22:19 Adam Curcie: hypotheticals. So, but you know, if if we get boxes and they're going to well, this redo it, Let me get one of these fancy guys. Here there is no AT&T like, you just take it out of the picture, it doesn't matter. We will never put an AT&T sim in an origin. So it's, it's

23:04 Devon D'Andrea: Never say, never.

23:06 Aksana Rahouski: For this for this new device, right? We're talking about the new device.

23:06 Adam Curcie: Never true.

23:08 Devon D'Andrea: Yes.

23:08 Adam Curcie: Yeah. So, and but for

23:10 Aksana Rahouski: Yeah.

23:13 Adam Curcie: Is this thing?

23:15 Devon D'Andrea: You wouldn't remove the field because it's still, you know, but

23:17 Adam Curcie: No, no. I know. I'm just trying to help simplify in terms of like we. There is

23:20 Devon D'Andrea: Yeah.

23:23 Adam Curcie: gonna be three possible ways that these devices V are warehouse after they come in. And that's gonna be this is active, this is active or both of these are active.

23:33 Aksana Rahouski: Yeah.

23:34 Adam Curcie: so,

23:35 Aksana Rahouski: But that's also like you have to consider not the new but all devices who also could come in as Verizon plus AT&T combo, right? Because we're not building it for a new device only we're building, we're trying

23:44 Devon D'Andrea: Yeah.

23:47 Aksana Rahouski: to scale. What we have to this new device that's gonna come in, Ultimately what

23:51 Adam Curcie: Yeah.

23:53 Aksana Rahouski: we're saying is just these devices basically come in with two Sims and you need to be able to specify, which one of these Sims is actually if it's one of them that's active or if it's both that are active.

24:08 Adam Curcie: Yeah. And I so right now just to give you a little bit Our. Workflow in the warehouse, when the

24:15 Devon D'Andrea: For three years for years, we haven't, we haven't, we haven't actually

24:17 Adam Curcie: For years, when the light is show up.

24:18 Aksana Rahouski: Huh.

24:23 Devon D'Andrea: redesignated a device.

24:25 Adam Curcie: Yeah.

24:25 Devon D'Andrea: I don't remember the last time.

24:27 Adam Curcie: We we keep active inventory on shelves and if you're ever fortunate enough to visit the glorious town in North Wales Pa we go. Give you a tour of our beautiful facility but we have got shelves and shelves, hundreds of devices of every model and carrier combination. Just sitting there waiting to be ordered.

24:47 Devon D'Andrea: Thousands. Yeah.

24:49 Adam Curcie: It doesn't sound all that efficient. When you say it like that, but that's how

24:51 Richard Sacco: Okay.

24:53 Adam Curcie: we've done it. So, like, at minimum there's two to three hundred of every device in every model and carry your combination. Ready to ship.

25:03 Aksana Rahouski: Yeah.

25:03 Devon D'Andrea: Ready to go here.

25:04 Aksana Rahouski: So if, if somebody comes in and orders a hundred of these that are Verizon only, we'll just pull them off, this shelf, scan them, chip them out, and then we're So, are you?

25:14 Adam Curcie: like, Oh no, we need another hundred Verizon only, but we don't have any all we have or ones where they're either, they look like this, or they look like this. We'll just have to scan them, manipulate them, and then send them out. Before they get assigned. so because the, the orders are filled before the devices are assigned to with the customer, So, when the order, when the order comes in Dan, says, All right. I got a

25:41 Aksana Rahouski: Yeah.

25:44 Adam Curcie: hundred boxes, going to this guy here. The serial numbers then Vince, looks at those hero numbers and assigns them to the customers.

25:52 Aksana Rahouski: Okay, so I we I think I search for saying, so you're reacting to the fact that with the new workflow you're importing this devices and not identifying which sim is active like how it's pre-configured to work, right? It's like, it's it's on the assignment to actually. Now do that to activate the SIM

26:12 Devon D'Andrea: No, no.

26:13 Adam Curcie: well no, I mean we basically there there are some but we activate the Verizon

26:20 Aksana Rahouski: And not portal perspective. Let's talk. So you sold a bunch of these, right?

26:21 Adam Curcie: Sims. When they get put onto the shelf to go out, not when they're ordered and not when they're assigned. so, Before.

26:36 Aksana Rahouski: And then you guys import them in the portal. is that or just like walk us through, because

26:42 Devon D'Andrea: time that we activate them. That's why Adam. That said, we only keep a couple

26:54 Aksana Rahouski: Yeah.

26:57 Devon D'Andrea: hundred at any given time of each. So we activate them at the time that we import them into the portal. And for, for all intention purposes for a long time, we are putting them into the portal with the exact way that they are going to be sold. So they were

27:16 Adam Curcie: I'm going to get real childish here. I'm so sorry.

27:18 Devon D'Andrea: needs is they're going into the portal with all the Verizon onlys are going into the portal with one Verizon Sim card, because they only have

27:27 Aksana Rahouski: Yes.

27:28 Devon D'Andrea: One SIM card, all anything that's getting imported as dual carrier is getting

27:32 Aksana Rahouski: Yeah.

27:33 Devon D'Andrea: imported with both and anything that's getting imported. As AT&T only is getting imported with just AT&T Sim. So like and and Adam's, what Adam had mentioned is

27:42 Aksana Rahouski: Yes.

27:47 Devon D'Andrea: if we ever got into a jam, We could redesignate other inventory because all all it takes is just moving sim

27:50 Aksana Rahouski: Uh-huh.

27:56 Devon D'Andrea: cards around if you have to. But but that is so rare that we ever actually do that.

28:05 Adam Curcie: Yeah.

28:08 Devon D'Andrea: so,

28:09 Adam Curcie: yeah, I mean

28:09 Richard Sacco: So basically you're trying to take the you're taking the decision-making out.

28:14 Devon D'Andrea: Yes.

28:15 Richard Sacco: Of that of that. Yeah. And you're

28:17 Devon D'Andrea: Yes.

28:17 Adam Curcie: Yeah, because that will reduce any error like if there aren't decisions that need to be made.

28:24 Devon D'Andrea: We're trying. Yeah on it. You taking the decision away from the assigned part of

28:25 Adam Curcie: Yeah. But we don't want.

28:31 Devon D'Andrea: this?

28:31 Adam Curcie: You know, and there's there's we don't want errors number one and we also don't want to make Vince's job take longer or be any way more like much more difficult or cumbersome you know. So if we just know that because the customer ordered

28:44 Richard Sacco: Mmm.

28:48 Adam Curcie: them. Verizon only Then that's how things have been working for a while. So yeah, I mean because the assign actually like, for like from So it would kind of be like this is where the orders are filled.

29:11 Richard Sacco: Yeah, I mean let me ask you something. Like, I I got you.

29:12 Adam Curcie: And then fine actually. so,

29:18 Richard Sacco: know I I get all the the thing I think we originally did this like mostly because it was like Verizon T-Mobile and sometimes you wanted it to just be T-Mobile but I'm guessing it sounds like you're like we can decide that on import like we don't have to

29:35 Devon D'Andrea: Well you that's if that's if we do something like what Adam is showing similar

29:37 Richard Sacco: Person.

29:41 Devon D'Andrea: to what Adam is showing like we want to be able to store that that Verizon Sim

29:43 Richard Sacco: if but

29:46 Adam Curcie: Yeah, because

29:47 Devon D'Andrea: but just not happy in that in that spot where the portal is like, can you?

29:47 Aksana Rahouski: so,

29:51 Adam Curcie: Looking you lose it. Basically, because if you calls us up three months down the

29:53 Devon D'Andrea: Yeah. Yeah.

29:56 Adam Curcie: road and says, Hey, I moved this box from, you know, location a location b. It's not working. Well, here, can we turn that T-Mobile on? We want to be able to say, of course.

30:09 Aksana Rahouski: Yeah and and yeah. What what? I'm so gonna again like this solution.

30:11 Adam Curcie: And vice versa.

30:17 Aksana Rahouski: the only kind of problem that's missing because before T-Mobile came in the picture, right knowing that ever have Max of two With Max active is too, so you automatically know that inactive is AT&T. No, no, brain are there right now. You kind of don't know where it go based, unless you do, right? It's like baked in. You guys know, because you're experts, You know, these devices like my memory right to are you. Like I like,

30:45 Adam Curcie: Yeah, but it's

30:47 Aksana Rahouski: which is why we kind of introduce this concept of storing him, kind of dedicated to a carrier, additionally, to active status. What we could do though in

31:06 Devon D'Andrea: I just I just I just I just had a really good thought I'm sorry Aksana but the way you the way you have it mocked up can definitely work with what we're trying to do. And I think the way it is all works together is just so that that we can. Make that determination when we're importing.

31:29 Aksana Rahouski: Yeah, and that's what I think. Like, if we, if we're just saying now that let's import if we're just want to put down the rule that only ever maxims, can be

31:38 Devon D'Andrea: Yes.

31:41 Aksana Rahouski: imported, right? And the ones, because what you're saying is when we import this Sims are our active Sims. In fact, right?

31:50 Devon D'Andrea: correct.

31:54 Aksana Rahouski: Yeah, it means if you're importing a device with two Sims, both of these go in, mark them active, right? But yeah, we will never allow importing three, because

32:02 Devon D'Andrea: If we want, yeah.

32:06 Aksana Rahouski: we know only, like, certain combinations are allowed Max of 2 is hard for dual

32:10 Devon D'Andrea: Right.

32:12 Aksana Rahouski: and the system when it imports it, it automatically puts them, Here's your Verizon sim. Here's your T-Mobile Both are active. So whatever the reason, unless you're saying that import should

32:19 Devon D'Andrea: Right.

32:24 Aksana Rahouski: be smart enough to also import it as Verizon as active. But T-Mobile is not.

32:31 Devon D'Andrea: That's exactly what I'm saying, what?

32:33 Aksana Rahouski: so, it needs a whole

32:33 Adam Curcie: Yeah, and that's why on the import I was going to suggest. We literally have a column that says Inactive and I mean please don't take this with any disrespect,

32:41 Aksana Rahouski: Yeah. No, no.

32:45 Adam Curcie: okay? You had said a minute ago with regards to If you do not know if it's AT&T

32:46 Devon D'Andrea: Yeah but yeah but it doesn't matter though. It doesn't matter though because the

32:48 Aksana Rahouski: I'm I'm not.

32:52 Adam Curcie: or T-Mobile to be honest. That's the portal. Doesn't need to know if it's in an active state. has an absolutely no relevance in there other than merely record, keeping, because in the future, there's not

33:09 Devon D'Andrea: way that aksana showed how it's mocked up and the way that it's currently in beta like you it doesn't have to be like this just agnostic field for inactive sin it can it can literally just be exactly how they have it. It just comes down to if we import it. When we imported, we're telling it what combination and for it to be in so that when Vince goes to a sign, he can use that new thing. You guys built that says, What does it say? It says, Keep keep sending configuration.

33:39 Aksana Rahouski: Now. As as yes.

33:42 Devon D'Andrea: Is. And then we're getting to the same exact solution here. Adam, you know I'm saying?

33:49 Aksana Rahouski: I hear you and you said, like it it's not important for an active, what it is. The moment it becomes important that customer does call you and say, Hey, for this device. Can you actually activate a second sim and I have no idea what that sim is. Unless like somebody tells me that, hey, go put it in a T-Mobile

34:12 Devon D'Andrea: I'm still, I'm stealing the the shared and

34:16 Adam Curcie: Good, good. I'm tired looking at my pain.

34:17 Richard Sacco: if but

34:19 Aksana Rahouski: But yeah, and we can and listen, we can go either way. I'm just trying to like

34:23 Richard Sacco: Yeah.

34:24 Aksana Rahouski: I want to make sure we find a solution that is like. When you look at it, it

34:25 Devon D'Andrea: Yeah. Yeah.

34:30 Aksana Rahouski: clear, you don't need a PhD, you need to go in the code. You don't need to call Richard, you need to call me to ask. What's going on, right?

34:36 Devon D'Andrea: It might not looking at the right device. I thought I just assigned unassigned it. I might have typed in the wrong number.

35:00 Richard Sacco: Okay.

35:00 Devon D'Andrea: That unassign. Okay unassigned. All right. So It's not a sign, right? So this is what it would look like. When we import it, obviously some of this stuff wouldn't be here, like the Check in time and whatnot, but we would import it like this because we will put this on the shelf as a T-Mobile only.

35:20 Aksana Rahouski: Okay. But two Sims are inverted but only one is active.

35:23 Devon D'Andrea: Yes. Yes, so we want to put this on the shelf as T-Mobile only. So Dan takes this off

35:27 Aksana Rahouski: Okay.

35:30 Devon D'Andrea: the shelf when we sell it and puts it in the back end of the website. Vince, Vince goes to assign it.

35:42 Richard Sacco: Yeah.

35:42 Devon D'Andrea: And he's gonna click as long as we all trust each other. He's going to click this. And that's it.

35:49 Richard Sacco: Yeah, now that you're saying all this though does it even make sense to have

35:50 Aksana Rahouski: Yes.

35:54 Richard Sacco: that whole some section there?

35:56 Adam Curcie: Now. Yeah, I would I would just get rid of that because I really feel like if

35:57 Aksana Rahouski: Yeah.

35:57 Richard Sacco: On the same page.

36:02 Adam Curcie: we're gonna have to do that there's and that you can correct me if I'm wrong. I I really doubt that we're ever gonna be in a position where we're doing the that, you know, the carrier change to determine, you know, or redesignate Pre-designated inventory on a one by one basis, right? Like if we get low we're

36:22 Devon D'Andrea: Right. Right.

36:24 Adam Curcie: gonna do a whole bunch of and just restock shelf.

36:25 Devon D'Andrea: We're gonna do it. Yeah, a ball.

36:27 Aksana Rahouski: What?

36:28 Devon D'Andrea: so,

36:29 Adam Curcie: So like having that there probably doesn't, yeah, we don't need the maintain existing SIM and we don't need the select carriers.

36:36 Devon D'Andrea: I also want to make sure we quickly talk about.

36:39 Adam Curcie: so, now we have the exact same device assigned screen as we've had

36:44 Devon D'Andrea: basically and and I would just and honestly, like

36:44 Aksana Rahouski: and this actually,

36:48 Devon D'Andrea: This can stay here, I guess and just have a defaulted as check. I don't even

36:53 Adam Curcie: Now.

36:54 Aksana Rahouski: To be there if we're saying that growing our company assignment, we never change

36:56 Adam Curcie: Yeah.

36:56 Devon D'Andrea: Right.

36:59 Aksana Rahouski: active sims. It, it doesn't need to be there because this is just misleading because when you

37:06 Devon D'Andrea: Well, what if what? If what if, what if what if what if

37:10 Aksana Rahouski: Well.

37:12 Devon D'Andrea: What if? Hear me out. We have a group of devices and this does happen. All right, let me think about this, Adam. All the Bella requests, let's say Bella requests a hundred single Sim Of boxes that are already marked as dual sim. Bella is a customer, sorry.

37:37 Aksana Rahouski: Yeah, I good. okay, so can I

37:43 Adam Curcie: yeah, right now, when we, when we have those requests,

37:43 Devon D'Andrea: I don't want to. Yeah.

37:48 Adam Curcie: The process is just to uncheck dual carrier or dual SIM.

37:53 Devon D'Andrea: Oh, that's I That's not how I do it when she sends me a big ass list. I go into this. I read. So I reassign them.

38:01 Adam Curcie: Okay. So this would be two steps you would have to, you'd have to do a bulk update or

38:05 Aksana Rahouski: Yeah.

38:09 Adam Curcie: maybe just one step. If it's they're already assigned, you don't have to reassign them. You can literally just bulk update. All of them.

38:18 Devon D'Andrea: If yeah if we can right because if with that yeah we can add that as a as a as a new column on the

38:25 Adam Curcie: But it will have to be a new column on the import. If

38:28 Devon D'Andrea: On the it's already here. But then there's this other button that they have which is Where is it?

38:38 Adam Curcie: Well, bulk actions is one.

38:40 Devon D'Andrea: No, not bulk actions. Although it could be a bulk action.

38:45 Richard Sacco: Are you talking about update devices or any words? Yeah.

38:45 Adam Curcie: Of you.

38:47 Devon D'Andrea: Update device. Yeah, I'm sorry. Yeah this

38:50 Adam Curcie: Yeah.

38:50 Richard Sacco: Yeah, where it's like maybe Verizon send Verizon SIM active agency SIM and then

38:51 Adam Curcie: It could be on that too. It could be all three.

38:58 Devon D'Andrea: Yeah.

39:00 Richard Sacco: active for that or whatever. Yeah.

39:01 Devon D'Andrea: Right now. Now here's the other thing that I wanted to mention just so we go make sure we're on the same page here. When we say, we want to import something. And designate that it's active, right? So let's say, I want to import We're let's say it. Let's say it's an i-22 and I want to import it as active Verizon and active AT&T, or let's say it's the new origin device, and I want to import it as active Verizon and active T-Mobile. That does not mean, That I want you. To activate it. The Verizon is different. But for AT&T and T-Mobile, I do not. I cannot, we

39:43 Adam Curcie: Yeah.

39:45 Devon D'Andrea: cannot have an API call occur that change.

39:47 Richard Sacco: Yeah, Unimport. Yeah, you wanted to always occur on a sign as it has been

39:52 Devon D'Andrea: No.

39:53 Adam Curcie: Well, no. So

39:53 Richard Sacco: oh,

39:54 Devon D'Andrea: No.

39:55 Adam Curcie: really, we don't

39:56 Devon D'Andrea: So let me I mean, so, yes. So let me let me do this, let me do this.

40:04 Richard Sacco: Man. Okay.

40:04 Devon D'Andrea: Gonna learn. Learn you. All right, let's see here.

40:08 Richard Sacco: Give us some learnings.

40:09 Adam Curcie: This is gonna be a teaching moment for all to you by.

40:12 Devon D'Andrea: So this here, this this port, this is our simple portal, and what you have here is general, status inventory. Let me just hit save search, that's not what I wanted to search. I don't know how to do that. I'm gonna go back. So, let me just show you, right? So, Frustrating.

40:38 Adam Curcie: You don't know how you don't know how to search it.

40:41 Devon D'Andrea: I I was going. But anyway, every time everything we buy sim cards and this this

40:43 Adam Curcie: I think you.

40:47 Devon D'Andrea: is very similar to AT&T, but Slightly different first for it, for T-Mobile, every time we buy the SIM cards, they come to us and you're not gonna see it here, because you can't go back from activated to inventory, but they come to us with a status called inventory. Inventory, Status. The box will not come online. However, I got to find, hold on. General Status. Inventory. Isn't there? Just a button? There we go. So everything that comes to us is inventory, we just like we have to activate it on Verizon. We have to also activate our T-Mobile stuff and this is something that we do. We don't want you to do because you don't want to, we don't want you to start a clock. That doesn't need to be started. The reason I say that is because we can put it into this, what's called test

41:39 Adam Curcie: Yeah.

41:44 Devon D'Andrea: ready?

41:44 Richard Sacco: Mm-hmm.

41:45 Devon D'Andrea: And when I do that, For T-Mobile, at least, I get six months. before I have to pay, if that SIM card doesn't actually come online, but the difference between inventory and test ready That in test ready. It can come online. So if you put this as activated, if you had an ape, if you had an API call that put the the T-Mobile SIM to active. Same thing for an AT&T SIM, if you had an

42:19 Adam Curcie: Yeah.

42:19 Devon D'Andrea: API called API, Call that activated that on import or Assign. I'm gonna start paying for it when I don't necessarily have to

42:28 Adam Curcie: I mean we have, we have AT&T Sims that have been out in the field on standby for years that we've never paid about

42:36 Richard Sacco: I believe you're current logic. Now I have to look into what's going on, but I believe that if it's just an AT&T sim we do activate it on a sign or if it's but if it's a dual sim but it's not marked as dual sim, then you're AT&T Sim isn't touched, it's left the test ready. Otherwise only your Verizon Sim is activated. I think that's how it's currently working. just, but, But I get it if it's in test ready, you don't want us to touch it because you get a free service for a while.

43:19 Devon D'Andrea: well, I do know, I do know that if I if a dual SIM device is deactivated

43:20 Adam Curcie: Well, and if also inventory.

43:25 Richard Sacco: Yeah.

43:26 Devon D'Andrea: I do know is, is that if we go and reactivate it, yes you are telling the AT&T SIM card to go to active status and that's fine because if we deactivated it I mean, we would have to add in all this crazy crazy logic for you to, like, let's say, for example, let's just say, for example, not to get off track. Let's

43:49 Aksana Rahouski: say, for example, this device has been online for, however, long. And the AT&T SIM has never been used, and the status of it is test ready.

44:00 Devon D'Andrea: If this customer deactivates it. Now, the AT&T status is deactivated because that's what you're telling it to do. Now that device can never go back to test ready or that's that's him.

44:13 Aksana Rahouski: Correct. We in fact, Richard, we just tested all this stuff every time. You manipulate SIM status. Only flip two, statuses active and inactive.

44:28 Devon D'Andrea: Right.

44:28 Aksana Rahouski: So whenever like consider any other statuses or said them, or yeah. So now it's

44:35 Devon D'Andrea: Right.

44:36 Aksana Rahouski: just on or off.

44:39 Devon D'Andrea: I just, I want you to leave all the first activation and when I say activation, I mean, on Verizon, we activate on T-Mobile, we change it to test, ready AT&T, They come to us as test ready, so we don't have to do anything. I'm telling you that all of that initial Activation the steps and just explain that needs to be left up to us, internally, not the portal.

45:08 Richard Sacco: Yeah. So what what's happening right now in the portal is that, let's say a device, you import it, right? It doesn't have a status on it, it's not set the

45:16 Devon D'Andrea: Right.

45:17 Richard Sacco: active in such, I believe that when you do assign it, though, it does get set to active and the status is change and that's currently. And I'm not talking about the code changes. We made clear.

45:29 Devon D'Andrea: I don't believe that's true. I don't believe that's true because we assign stuff all the time and that is deactivated and intentionally assign it and it remains

45:35 Richard Sacco: Yeah.

45:41 Devon D'Andrea: deactivated, it intentionally. I don't believe that you're correct me if I'm

45:43 Richard Sacco: No.

45:47 Devon D'Andrea: wrong but I'm like 90% sure that assigning does not

45:50 Richard Sacco: Okay.

45:52 Devon D'Andrea: Send any?

45:53 Richard Sacco: Okay, may I I might be wrong. I would have to double check that but okay

45:56 Adam Curcie: Hey, I'm gonna I'm gonna share the screen for one second.

45:57 Richard Sacco: interesting.

46:00 Devon D'Andrea: Excuse.

46:01 Richard Sacco: Okay.

46:03 Adam Curcie: Just this will be really quick. I

46:05 Devon D'Andrea: This. But with that what? Yeah we're done with what I was saying anyway. I think

46:08 Richard Sacco: Okay.

46:10 Adam Curcie: So, like, these are ones that have been assigned. Since 2021 that are still in test, ready on AT&T?

46:20 Devon D'Andrea: Yeah.

46:20 Adam Curcie: So it's yeah, which is great. Yeah, we haven't paid

46:21 Richard Sacco: Oh awesome. Okay.

46:22 Devon D'Andrea: We've never paid for them.

46:26 Adam Curcie: Anything. And like so, these ones, January February, I filter the test ready and I sorted by data. So these have been in the portal for years and these ones are clearly assigned to devices because we put the vice IDs in with them. But we've never

46:42 Richard Sacco: I get that but those are I don't any of those are AT&T single Sims though and I could

46:48 Devon D'Andrea: They're not.

46:48 Adam Curcie: No no no. These are all dual carrier but the good come online. It any moment if

46:49 Devon D'Andrea: They're not.

46:52 Richard Sacco: Oh, okay. Yeah.

46:55 Adam Curcie: customer ever like if the Verizon ever fails, these will populate instantly

46:55 Richard Sacco: Oh, of course, yeah.

46:55 Devon D'Andrea: Yeah.

46:59 Richard Sacco: Mm-hmm.

47:02 Devon D'Andrea: The same. This the same logic applies though for Single SIM team mobile or AT&T

47:02 Adam Curcie: though.

47:03 Richard Sacco: Yes.

47:08 Devon D'Andrea: assignment because that device that they buy from us, may sit on a shelf for six

47:08 Richard Sacco: Mm-hmm.

47:14 Devon D'Andrea: months.

47:14 Adam Curcie: Well, no, it's not the same on simple simple, doesn't give us unlimited test ready? They give us inventory.

47:20 Devon D'Andrea: Well, they give us. They give us six, they give us like six months.

47:23 Adam Curcie: Yeah, I know, but that's a lot less than unlimited, which we know. This is literally unlimited. We pay nothing.

47:29 Devon D'Andrea: I know but my point is is that somebody that buys inventory from buy stuff and for purposes of inventory, they're going to end up using it. Some of them might not but most people are gonna end up using it. I just don't want to start paying T-Mobile

47:42 Adam Curcie: Oh yeah, no. I get

47:44 Devon D'Andrea: In the three, four, five months that they may not even have used it. All right.

47:51 Aksana Rahouski: Okay.

47:52 Devon D'Andrea: We're.

47:54 Aksana Rahouski: We are.

47:55 Devon D'Andrea: I think, what I think we've actually made a lot of progress, surprisingly.

47:59 Adam Curcie: I think Big Circle like we're, we're back.

47:59 Aksana Rahouski: Not good. I think you need. We made a circle around. Yeah, I have. I have some

48:06 Adam Curcie: questions a lot, what? Yeah. Okay. So let's start with We covered a lot of ground.

48:12 Aksana Rahouski: Because there is kind of like milestones that portal supports, right? And does certain jobs from activating some, in the portal, actually, pinging a seam on the provider and activating etc. We just, let's kind of start with like importing, right? So we just agreed that part of the import is also going to be carry along these like, status of a sim, right? That when I'm imported this device, I actually can like, right away without doing anything. Sim is active in this one is not assuming I imported two sins, right? So means this is our template file today, so we'll never allow three so we'll validate that only two marks could be imported. Um, and with these I'm guessing we need some probably new columns to like kind of carry the starters with the SIM so Verizon SIM value. This is it active true false or somehow looked into if this needs to land as active or not AT&T sim this active or not? Is is it just to want to validate that? This is what we kind of talked about that now.

49:22 Adam Curcie: Yeah, I would envision literally a column right in between right to the right of each scarier thing that would say active. Why?

49:29 Devon D'Andrea: Yeah.

49:30 Aksana Rahouski: Yeah. Yeah well yeah. Well you like Verizon active one or zero. Whatever is

49:30 Devon D'Andrea: Yeah.

49:37 Aksana Rahouski: easier world will make it as you probably like one of zero so that we

49:40 Adam Curcie: Yes. No, Yes. No.

49:42 Aksana Rahouski: Yeah. Yes no true. False whatever is easier is pick. Yes No it's great.

49:46 Devon D'Andrea: you would you would you would make Adam so happy if you made it so that we could

49:47 Richard Sacco: Me.

49:51 Devon D'Andrea: cut if we could color code them, which you're not gonna do and I don't want you to but I just

49:56 Aksana Rahouski: So a lot.

49:56 Adam Curcie: Why would that make me happy? Why do I want to color code them?

50:00 Devon D'Andrea: You love your colors?

50:02 Aksana Rahouski: So like this, right? So Verizon said awesome. Active.

50:04 Adam Curcie: He's home here. Now yes, that is yeah that's

50:06 Devon D'Andrea: Yes.

50:07 Aksana Rahouski: Right.

50:08 Devon D'Andrea: Perfect.

50:08 Aksana Rahouski: Okay, and we will add. So each one is gonna need a buddy like that. So it needs some active. We will also add um, same idea.

50:14 Devon D'Andrea: Yes.

50:21 Aksana Rahouski: And it's also going to be. Yes, No. And then the same for T-Mobile so that way you can import any combinations of your Sims. Um, with the active flag right away, so you don't need to do it on the assignment. Right.

50:42 Devon D'Andrea: Yes.

50:43 Aksana Rahouski: Okay. So let's talk though for a second to what you just guys described. So when I do that, This device was created.

50:54 Devon D'Andrea: Well, wouldn't you be doing that by active at? It's setting both of to the Sims

50:54 Aksana Rahouski: Oh, first question is important to support dual or is dual something that is required if you want these two to be active, you actually need to go to this device and do it from the portal or should I be able to import dual devices?

51:17 Devon D'Andrea: to active anyway?

51:20 Aksana Rahouski: You could and that's where so like if we import devices, let's say, yes. And yes, right. We automatically made a decision that these two are active and we mark dual flag as true. So then you will see active active dual. Yes,

51:35 Devon D'Andrea: Yes.

51:36 Aksana Rahouski: Okay, so then next question, um, when this happens, right technically? What we we take this data, we create a device. We save it with these flags and we also we ping horrors active. Verizon thing activate T-Mobile activate. But you're saying don't do that, right? On imports. So on what we actually don't want to ping a precarrier to manipulate

51:59 Devon D'Andrea: Yes.

52:05 Aksana Rahouski: that sim status, meaning

52:06 Devon D'Andrea: Correct.

52:09 Aksana Rahouski: Um, we're kind of gonna rely on the fact that you guys did the work in. This seems already in the status that allows you to get free six months, freebie or whatever, right? But you're gonna hear that.

52:19 Devon D'Andrea: Yes. Yes.

52:21 Aksana Rahouski: Oh on an import will not contact like calling a carrier API to manipulate that some status. Okay, what happens if I now get imported grade? Now, I go and I change something about this device.

52:37 Richard Sacco: Mmm.

52:38 Aksana Rahouski: Either I activate deactivate or I play with the combination of active and act, Maybe I change my mind. It's not going to be dual is going to be single, Verizon only I hit save because that is another moment of when you change device through the portal gooey. It also things Verizon API and change status to match this

53:00 Devon D'Andrea: Worth.

53:01 Aksana Rahouski: flag

53:02 Devon D'Andrea: Right. So that's a great question, right? So, Let's say we have one in there. That is currently the way you have it and then we want to change that to be. AT&T active, nothing else, active. So what is it gonna do? Number one. Number

53:27 Aksana Rahouski: So right.

53:28 Devon D'Andrea: one, We need to make sure without a doubt that we have a configuration that is even going to support that.

53:36 Aksana Rahouski: Okay.

53:37 Devon D'Andrea: Number two. In a perfect world. I would have you Read in the status of the AT&T SIM. And set it to test ready if you can but that's a perfect world.

53:56 Aksana Rahouski: And that's that's actually possible, right? If you're let's say, let's say, in this case, we are just saying that kill Verizon

54:06 Devon D'Andrea: Overriding.

54:07 Aksana Rahouski: Activated TNT, right? And we also treat a kill as like, no matter what status it

54:08 Devon D'Andrea: Yeah.

54:12 Aksana Rahouski: is in deactivated.

54:14 Devon D'Andrea: Yeah.

54:15 Aksana Rahouski: Or whatever you're choosing to activate. And that's where like it's it's a little bit more priority complex made trucks because probably different carrier has did this like, temporary statuses, but we could build it in as like, if you're 18t. Check the status first. And if you're Like, because if you're coming from this, I guess we can figure out to, like, not even like Track the status. But if I'm trying to make this active right, I'm gonna first try to make it. Ready for that. Whatever that like status was right, my guess is they probably

54:52 Adam Curcie: Yeah.

54:52 Aksana Rahouski: have something built in that it gives you less expiration time frame. That after six months, you cannot set it to that status anymore. So we could

55:00 Adam Curcie: Well, so

55:01 Devon D'Andrea: You in this instance, in this particular example, that's not true. It's it's

55:02 Adam Curcie: Yeah. It's, it's

55:05 Aksana Rahouski: that's,

55:06 Devon D'Andrea: forever for me, but yeah.

55:07 Adam Curcie: Yeah. It's weird because all of them will different like AT&T if every single

55:09 Aksana Rahouski: Okay.

55:13 Adam Curcie: sim shows up his test ready, once it's no longer in test, ready, for whatever reason, you can never put it back to test, ready?

55:20 Devon D'Andrea: But it can stay in test ready forever.

55:21 Aksana Rahouski: oh,

55:23 Adam Curcie: Forever. So one.

55:24 Aksana Rahouski: Is that what? No? Is that what we want?

55:26 Adam Curcie: That's what you want until. Yeah. I mean, like, yes. I mean you because if but if they're actually using it right, it's the only sim Like you, you could put it in test ready but it's not gonna stay forever if it gets online. So it's really weird. I guess to try to like find the the correct terminology to like like describe it. Like

55:51 Devon D'Andrea: You know, once it come once it actually provisions on the AT&T network, it is no longer test ready. It's active it hits it hits a certain threat of kilobytes

55:58 Aksana Rahouski: Okay, so All right.

56:01 Devon D'Andrea: that. They grant you for just testing purposes and they're just changes to

56:04 Aksana Rahouski: Yeah.

56:05 Devon D'Andrea: active and it can never go back to test readings. So,

56:08 Aksana Rahouski: Hey, so do we want then?

56:08 Devon D'Andrea: Yeah.

56:11 Aksana Rahouski: To back to our original option. If I'm trying to activate AT&T, I'm gonna first, get the status.

56:17 Adam Curcie: Yeah.

56:18 Aksana Rahouski: Let's let's say status comes back deactivated. Do you want to set it to test ready? Or do we want to attack?

56:25 Devon D'Andrea: Yeah, you can't.

56:26 Adam Curcie: You can you have to go to activate it. Ready activation ready or activated? I

56:29 Aksana Rahouski: Okay. so,

56:31 Adam Curcie: don't even know.

56:32 Devon D'Andrea: Which is good at you. Just go to activated at that point. You can't. Once once, it's once it's it comes to us, test ready? And if it's changed from test, ready

56:36 Aksana Rahouski: I see hood. Is that done true? That if the status is test ready, keep it test

56:39 Devon D'Andrea: to anything else. You cannot go to test, ready?

56:42 Adam Curcie: Yeah.

56:48 Aksana Rahouski: ready, otherwise make it active.

56:48 Adam Curcie: Yeah.

56:50 Devon D'Andrea: Correct. 100%.

56:51 Adam Curcie: That would be the correct logic if it plus ready and you want customers to

56:54 Devon D'Andrea: The.

56:57 Adam Curcie: consider it active and you can leave it. If it's deactivated, you have to make it activated.

57:02 Devon D'Andrea: the more important thing though is how we're gonna have a configuration file built in here to recognize this particular scenario, if we have to use this scenario,

57:12 Adam Curcie: Which is what?

57:12 Aksana Rahouski: Okay. Okay.

57:14 Adam Curcie: That they're burning authorizing.

57:16 Devon D'Andrea: A config file that has AT&T and SIM two.

57:20 Adam Curcie: Well, I mean, we don't, we don't have the, There is no ability for customers to

57:23 Aksana Rahouski: Well.

57:27 Adam Curcie: even ask for that in any way. Other than sending email, or over the phone, like there's no buttons. We give them that they can actually push. That would tell the portal to turn the Verizon off and leave the AT&T on.

57:40 Devon D'Andrea: yeah, we have

57:41 Adam Curcie: I think we're giving them that option anytime soon.

57:44 Devon D'Andrea: We're not. These buttons are just for us. So but if we push these buttons, what's gonna have

57:50 Aksana Rahouski: Let's for second. So let's let's kind of table. Config, Let's first kind of do

57:53 Devon D'Andrea: Okay. Yeah.

57:53 Aksana Rahouski: this active round of How do we manipulate statuses? So an import on update and

57:55 Devon D'Andrea: Sure.

58:01 Aksana Rahouski: then we just say Now on assignment, we want to be move. An option to basically reset up active because this is another way of like when I'm assigning today, right? And I'm resetting it to Let's say this. It also will basically update that device and make an API call Verizon in AT&T We can apply the same rules as far as what status we want to set. But also earlier we said Since now we're saying that import will take care of What's active, what's not? Let's take it off assignment. So assignment is really ties device to a company, but not more than that. Like in it doesn't manipulate like that.

58:49 Devon D'Andrea: Yeah. The only, the only reason I can think that it would be meaningful and and would be for, for that situation, we were just talking about, but if we needed to do it in bulk, but really, like I use the assigned page to do things in bulk, but I feel like I shouldn't use the assigned page to do things in bulk. You know what I'm saying?

59:08 Aksana Rahouski: Yeah, and we can kind of, we can go that way for now, let's say there's a scenario where you need to update 100 devices and their active sims, instead of doing it through assignment, you just do it through a bulk device.

59:21 Devon D'Andrea: That update the update devices thing. We are

59:23 Aksana Rahouski: Which will update it to kind of support the same structure so you can import like active sims and etc, right? So it like, reset it for your and this art of

59:30 Devon D'Andrea: Right.

59:32 Aksana Rahouski: assignment. So, during assignment really, this entire section is gone,

59:36 Devon D'Andrea: Yes, correct. And the reason I don't like doing bulk stuff in the assignment

59:37 Aksana Rahouski: Okay.

59:41 Devon D'Andrea: page is because of the Oh, there's always like because it's just the potential ramifications that it has on the billing because even though I'm just reassigning it to the same company, like I like I just it just doesn't feel like, you know what I'm saying? Like like if all I'm doing it for the purposes of like changing dual sim to single sim, like I'm I'm assigning it to the same company. I'm hitting transfer current Billings. Now you've got, you know, a portion the willing cycle and this will on this company and then you're transferring the rest of the billing to the same company. Just like, I feel like

01:00:16 Aksana Rahouski: Uh-huh.

01:00:18 Devon D'Andrea: I shouldn't be using that because it'll mess with that whole, you know I'm saying. So yeah. So just keep to keep it outside of this screen.

01:00:26 Aksana Rahouski: Yeah, and it does keep it simple. I'll tell you because this is like

01:00:28 Devon D'Andrea: Yeah. Yeah.

01:00:31 Aksana Rahouski: A big liability here because we have a lot of these like validation rules baked

01:00:33 Devon D'Andrea: Right. Yeah.

01:00:38 Aksana Rahouski: and now because if you have like 10 devices and nine match and one is off, right, you need to validate for all these things. So take taking it out, I think we'll make things easy like just less. Safer basically. For now. Okay, so configs I think we still haven't talked about, I unfortunately have another meeting, I'm already late for, but I do think we need to go back to this because I still would you guys are saying that can next step is like other than, you know, What is this device status and status? How do we ping API to set the proper status? How do we import them and etc? How do we match proper configs? And where we started is the position of this? Sims does matter for how we map config to device, right? So, I don't know if we need to like now and again as you could my brain is already going. Like if this becomes part of a config, does this need to be part of the import as well? So you need to import which sim this is sitting in. So I feel like we need more time to talk unless you guys feel like, but I feel like the problem of matching to the right? Config still exist. We haven't solved it yet.

01:01:54 Devon D'Andrea: Well no because I don't think you have to think worry about it on import because here's why we know with a hundred percent certainty. That the I-22 is going is going to have Verizon always in Sim 1 and then one of the other carriers in SIM

01:02:03 Aksana Rahouski: Huh.

01:02:08 Devon D'Andrea: 2 and the origin device is always gonna have Verizon as e-sim and you know T-Mobile as as the regular sim. So like I don't know that you need to like step

01:02:20 Aksana Rahouski: You don't need as you think, okay?

01:02:22 Adam Curcie: Yeah, there's not as there's really not that many combinations. So,

01:02:26 Aksana Rahouski: Okay. And that's if we, let's say we can also kind of proceed with the changes that we just discussed. and then kind of lube back on whether or not, you know, Configs are kind of like we're gonna just make these assumptions. It because it's a limited combination, right? As far as like Sims said.

01:02:47 Devon D'Andrea: Yeah.

01:02:50 Adam Curcie: The only.

01:02:50 Aksana Rahouski: As long as our mapping algorithm already supports that.

01:02:54 Devon D'Andrea: We just need, we need to talk. We need to talk about that, that scenario that we just went over. Where it's

01:02:59 Adam Curcie: Yeah, we are. We right now don't really have a good solution for if a customer has a dual carrier box. And they want to transition to AT&T only,

01:03:10 Aksana Rahouski: Yeah.

01:03:11 Adam Curcie: It has, it goes way deeper than just this conversation. We don't have a good way to do it and we found that out which is why I will forever reference. We have one box that we cannot touch ever again because we already made changes to it and it's on AT&T, but it's it's not good. It's like a ticking time on my feel like to put

01:03:31 Aksana Rahouski: Which one is it listed here?

01:03:34 Richard Sacco: You know, it was on beta a few. I mean Oh,

01:03:35 Adam Curcie: Yeah, no, no for the customer that's out in the field. But the thing is that, okay? So right now, right, when you look at the way the portal maps, How do I know it's an AT&T only, right? Well it's got an AT&T that's active in a Verizon that's inactive. So like is the portal going to be able to know based on that

01:03:52 Devon D'Andrea: Yeah.

01:03:56 Adam Curcie: that the SIM is in Sim 2, where is the ATM? I mean, the AT&T only config is in

01:03:59 Devon D'Andrea: Yes.

01:04:04 Adam Curcie: one. Right, so it's like that's that's the tricky part because like they would otherwise be, you know, the same config, but

01:04:13 Aksana Rahouski: Yeah.

01:04:13 Devon D'Andrea: If you if you're looking at, if you're looking at an i-22, can you go back to your mock-up screen? Aksana? If you're looking at an i-22 and I'd say I wanted an inactive, the Verizon and you keep the that active. Now. Yeah. Now what we

01:04:29 Aksana Rahouski: just,

01:04:31 Devon D'Andrea: also have to do is we have to and maybe maybe it's something we could just have automated, but we have to then have the same position changed. Well, now because you still need to account for the ability for the device to get the figuration right? And so it's I don't know. This is this is risky and I

01:04:47 Adam Curcie: Yeah. So yeah. I mean

01:04:53 Devon D'Andrea: I think we should possibly just like not even I don't know if we shouldn't allow it or just for us internally just like never do this.

01:05:02 Adam Curcie: Yeah. We try really hard just like if customers find that there's a location

01:05:03 Richard Sacco: Right.

01:05:05 Adam Curcie: where only the AT&T works but they have a dual carrier, I try really hard, just tell them to buy an AT&T and take the dual carrier somewhere else. Like maybe my

01:05:14 Aksana Rahouski: Platform.

01:05:15 Adam Curcie: life so much easier fell, just buy one more box.

01:05:17 Devon D'Andrea: It's like super, it's super risky. Like you may, you know, you may need them to

01:05:20 Adam Curcie: Yeah. if you don't real quick, if you just go to beta and not with the sim position I can it's like

01:05:27 Aksana Rahouski: Go to.

01:05:28 Adam Curcie: It. Yeah. Go to edit click at it. So it's like, Where you have right now in Verizon's active in At&t's inactive. So if you check AT&T and uncheck horizon, sorry. Yeah, that's fine. And then remove the Verizon

01:05:41 Aksana Rahouski: Right. And that's

01:05:45 Adam Curcie: Sim like now you're talking about a config that if sent will destroy the device and it'll never come back online. Like, if it doesn't account for the fact that there is an AT&T in SIM 2, like you had it on the other screen. My problem with the other screen is like, I just

01:06:00 Aksana Rahouski: This one.

01:06:03 Adam Curcie: feel like it's gonna become too much where it's like, you know, we just looked at the import where you have to add as you put it at the buddy to determine if the sims active or inactive. So now you're gonna have to add another body that

01:06:15 Aksana Rahouski: Yeah.

01:06:17 Adam Curcie: yeah, for the same position and it's like it's just gonna be, you know,

01:06:20 Devon D'Andrea: Well, no, I don't think you would though.

01:06:21 Aksana Rahouski: I, And we can also like keep in mind that we can bake these If you're saying that let's not put it on the import but kind of on the system to map it by these like default rules, right? A system could land like this but in but at the same time, you have visibility into kind of the snap of words today and ability to change it for the few in the future, right? What? I'm still kind of confused. If you were saying if this will Fry the box. How do we avoid doesn't happen?

01:06:54 Adam Curcie: No, no, that that's fine because it accounts for they're being a Verizon SIM if you remove the data. From Verizon Sim.

01:07:06 Devon D'Andrea: In this in one that you have to explain though because this is assuming that

01:07:07 Adam Curcie: and now, yeah, it

01:07:13 Devon D'Andrea: works it works the same it does today so where if you hit save the the portal will still think it's a dual carrier config and that's the only way it'll work.

01:07:24 Adam Curcie: Yeah.

01:07:24 Devon D'Andrea: It's not dual Sim checked.

01:07:26 Adam Curcie: Yeah.

01:07:26 Devon D'Andrea: It's dual carrier config because the only way to keep the AT&T SIM card in SIM 2 is by having it on a dual carrier config and for it to just fail over

01:07:34 Aksana Rahouski: Wow.

01:07:37 Devon D'Andrea: immediately.

01:07:39 Adam Curcie: which again, it does work that way and but if for whatever reason the portal,

01:07:40 Aksana Rahouski: um,

01:07:44 Adam Curcie: Determines that. It's an AT&T only are all of our AT&T only configs assume that the AT&T SIM is in the SIM one position.

01:07:52 Aksana Rahouski: Yeah.

01:07:54 Adam Curcie: Which is a problem.

01:07:54 Aksana Rahouski: You got, let's I think we should continue this lab, but let's me, I'm sorry, I'm being yelled at already for being like eating but I can we do you guys have capacity tomorrow because I think we should still get to the bottom of it be, especially since we're about to uncover this whole like config conversation, right? I think you might hope to kind of lead us in the right direction. So like, I'd like to understand a little bit better. How can they aligns to this like edge cases?

01:08:25 Devon D'Andrea: So we do, we did we did have a tentative, very important meeting but it looks like that is gonna not be at that.

01:08:34 Adam Curcie: It's that 2:30 now, 2:30 tomorrow.

01:08:37 Devon D'Andrea: Okay, so yeah, one works for us.

01:08:40 Aksana Rahouski: Okay so let's just book another meeting and then we will like focus on configs tomorrow and see whether we define these like hard-coded rules in the system. Pull it through into configurable layer, but I think we need to like get crystal clear on how these configs are gonna get. Mapped.

01:09:03 Devon D'Andrea: Yes. Can you can you please send me?

01:09:04 Aksana Rahouski: Okay.

01:09:08 Devon D'Andrea: Whatever the then, however, the notetaker is able to figure out everything we just talked about. Can you send me the summary?

01:09:16 Aksana Rahouski: Yes. Well, listen, yeah.

01:09:18 Trista Smith: Yeah, and

01:09:20 Devon D'Andrea: Please because I feel like we just were all over the place.

01:09:22 Trista Smith: Yeah.

01:09:23 Aksana Rahouski: Yeah, touch on a lot of things.

01:09:26 Trista Smith: I was gonna say, Devon, do you ever go into confluence? Because I also have the recording links there too, but I'll email those to you for yesterday and today, so you have them

01:09:34 Devon D'Andrea: You certainly go in there, does it give you that AI summary as well?

01:09:39 Trista Smith: You have to right now, go into the TLDV link that has the transcript in there.

01:09:45 Devon D'Andrea: Yeah. so,

01:09:47 Aksana Rahouski: I need to jump you guys, but well continue.

01:09:49 Adam Curcie: All right.

01:09:50 Devon D'Andrea: Good. Thank you.

01:09:50 Adam Curcie: Yep. Pick this back up tomorrow. Yeah.

01:09:51 Garrett Hood: But that's everybody.

01:09:52 Trista Smith: All right.

01:09:53 Devon D'Andrea: I will talk tomorrow. Thank you.

01:09:55 Trista Smith: Eggs. Right.

01:09:56 Richard Sacco: Thanks.