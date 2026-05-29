# APW Stand-up — Transcript

**Date:** May 19, 2026, 3:00 PM UTC
**Duration:** ~47 minutes
**Organizer:** Laura Perry
**Attendees:** Aksana Rahouski, Laura Perry, Noah Bratzel, Richard Sacco, Stone Marballie, Aaron Diefes

**Source:** tldv recording

---

**Aksana** [00:01]
Yes. Wonder why. What am I... What am I sharing? Am I sharing my email? Can everybody see my email?

**Noah** [00:07]
Yep.

**Aksana** [00:08]
Do you see this? Like-

**Noah** [00:09]
Oh, yeah. Yeah, that's because I had Claude generate, and I asked Claude to put it on my clipboard, and then I copy paste- I just paste it in.

**Aksana** [00:19]
Okay.

**Noah** [00:19]
And then I noticed, I, I noticed some of them and I cleaned them up, but I guess I didn't get all of them. Yeah.

**Aksana** [00:26]
Yeah. It's... And that's actually, uh, funny you brought up because I- It's nice 'cause Claude allows you to, uh, let's say I often draft messages from C- from terminal and use Claude to p- publish into the channel through, through MCP, right? Like, that's nice. I wish it did it for email, too, because copying and pasting, uh, uh, there is so much formatting, like weird, like line breaks and characters that, like, would be nice not to have that.

**Noah** [00:58]
Yeah, it could probably, if I was nerdy about it or you nerdy about it, you could just put to- together like a cleanup skill. Have it format it, right, and then whatever.

**Aksana** [01:08]
Yeah. Yeah, sometimes I'll, like, bring it into a Google Doc and do, like, strip out all, all, um, formatting. 'Cause they'll do, like, weird line breaks and, you know-

**Noah** [01:21]
Yeah, I don't, I don't think there's too much you can do about it, it because it's coming from the terminal.

**Aksana** [01:26]
Mm-hmm.

**Noah** [01:27]
In that tool itself, a lot of times what I do is I take the output from the Claude code and put it into regular Claude and say, "Hey, can you clean this up for me?" If it's too bad, and it does something like that. But I also-

**Aksana** [01:38]
Yeah

**Noah** [01:38]
... um, also was working on a, a tool to automatically do that, and maybe I'll share that with developers sometime soon. But basically like a, a cleanup, like there's a JSON pret- pretty JSON whatever something, the tool you use, you c- copy in one side, it automatically cleans it up on the other side, and you have different options. So I was, like, working on something like that, that it also might be helpful with.

**Aksana** [01:59]
Yeah, 'cause, 'cause even, like back to like, I'll... 'Cause I have all the, like, Slack channels hooked up and even people hooked up, right? So I can be like, "Hey, publish the drafts, send the messages in the channel," blah, blah, blah. And i- in that case, actually, it will clean it nicely. I'm like, why do you not clean it for me then when I'm just copying? 'Cause copy always brings so much garbage with it.

**Noah** [02:25]
Yeah. It's 'cause it can't... There's nothing it can do with, with the, the output in the terminal is beyond, you know, after it. It's like a layer inside, and then it goes to the terminal. So all, you have all the extra spaces at the end of the lines, you got extra carriage returns. It's, it's like it has... Claude Code can't do anything about it, basically. O- other than making its output shorter. It's just can't, you can't do anything about-

**Aksana** [02:49]
Yeah

**Noah** [02:49]
... it's built into the terminal output, if that makes sense.

**Aksana** [02:52]
Yeah. I'm curious. So let's just drop. That looks-

**Noah** [03:01]
Worth trying, and if we have issues, I can fix it up better. But that, that's one thing that I've been using, is I just, like, vibe coded my own little, little solution to, like-

**Aksana** [03:11]
Tidy up

**Noah** [03:11]
... clean up output. Mm-hmm.

**Aksana** [03:15]
Okay. Can we use it if, can I use it since I have it now?

**Noah** [03:18]
Yeah. It's open, it's, it's available to anyone in the world. It's on a public website, so yeah. And it's open source, so if you don't like it, you can either tell me and I can fix things, or you can for- fork it and fix it yourself and have your own version if you want.

**Aksana** [03:32]
Oh, and it even does, like, remove leading padding around... Oh, wow. Fancy, Noah. Okay, thanks. I'll use it.

**Noah** [03:43]
A coup- a, a few different hou- a couple hours of messing with things on my own time to make sure that, uh, I had a solution that made me happier. Because, yeah, that always annoyed me. I copy out of-

**Aksana** [03:53]
Yeah

**Noah** [03:53]
... Claude Code output all the time.

**Aksana** [03:56]
And especially when you're, like, now with, like, with AI Cla- Claude, and you, like, move so much faster, and then you're like, "Uh, and now I have to format it." Like, are you kidding me? Just, just, it's like-

**Noah** [04:08]
Yeah. But the other thing-

**Aksana** [04:09]
... 20, 20 steps back.

**Noah** [04:12]
The other thing you can do with the email is just, you just paste everything in, and then you can open Claude in your, in your browser and just say, "Hey Claude, can you clean up this, uh, funkiness for me? I have, like, formatting issues." And it can also do that for you. It's like, there's lots of different solutions you can do.

**Aksana** [04:27]
Yeah. Okay. Well, I'm glad we talked about it. Now I have a cleanup tool.

**Noah** [04:38]
Yeah, and that's not perfect, so don't expect it to work. It's like, I haven't spent too much time perfecting it, but it, it'll do some things pretty well.

**Aksana** [04:46]
Mm-hmm. Yes. Okay. Thanks for sharing.

**Laura** [04:55]
Sorry, I was not listening very well. Had a little Ayla fire, uh, right as this was starting. Um, okay. Have we talked about standups yet?

**Aksana** [05:16]
Not yet. No, not really. We're d- we got distracted. Typical.

**Laura** [05:20]
Okay. Okay. All right. Oh, let me get the board pulled up. So sorry. Oh. Okay. Um, Noah, can you start us off, please?

**Noah** [05:52]
Sure. Um, first, I'm very distracted by CP today 'cause I got some issues to, I still have to resolve apparently, and I'm working on that furiously.

**Laura** [06:03]
Yeah.

**Noah** [06:03]
Um, but yesterday I think that we had- The one issue for Verizon FWA stuff.

**Aksana** [06:15]
Mm-hmm.

**Noah** [06:17]
Um, and so hopefully we have a f- a fix for that. The main thing is I think that they weren't expecting, or they are expecting all the service plans that are existing to basically be PPU, but they didn't set them to that. They, so they were all set to null, which basically is no restriction, meaning you can assign devices that are either V- Verizon or PPU to them, and I think maybe that doesn't make sense. I think now that we've done the work and see how they're expecting it to work, I think that probably what we should have done, which I haven't done yet, is service plans should either be PPU or, or FWA and not allow either like we're currently doing. So basically, what, with that assumption, I set all their existing service plans on beta to be PPU so that it would act like they're expecting. And I also updated the message so that it's a little bit more clear. Because what was happening was they weren't getting the message they were expecting because they were trying to test assigning a p- PPU service plan to a FWA device, but the, the, the service plan they were testing wasn't set to PPU, it was set to null. But they were expecting it to still act like... So that's what... And so the error they were getting wasn't the error they were expecting because it was actually the error of the API trying to actually change that device, and then it won't, it didn't work because, I don't know, under the hood it's like that's the wrong account basically, and it doesn't, it, that's not the... You can't assign that service plan to that FWA device even. So, so for right now, what I did initially is, like I said, is I ch- changed all the service plans, and I was gonna check today in this standup follow-up and see should I go ahead and make that change? Is that a fair assumption that we set that up, the spec's wrong? Right now, basically what we did, let me recap what we did, is devices work that way where you can only be a PW- PPU. It defaults to PPU or FWA. It's just a Boolean check. Service plans we did not, I did not set up that way. It's, it defaults to null, meaning no restrictions of what devices can be assigned to it. I didn't realize that that-

**Aksana** [08:31]
Mm-hmm

**Noah** [08:31]
... probably doesn't make sense, that you probably can only have a PPU or an FWA service plan. So I guess the question is should I have a follow-up ticket and do that quickly where I change that so that it defaults to PPU and you only have FWA or PPU service plans? Or is temporary solution of just assigning them all to PPU and assuming that they won't set them up to null it's probably not a safe assumption. I just don't see how they would ever have a service plan where two different kinds of devices are on it. That doesn't make sense to me anymore now that I understand how it works.

**Aksana** [09:06]
So you're saying that... 'Cause like today, like you said, provider account type, right, we have three types basically. No restrictions, PPU, or FWA. You're saying it's actually, it's the, we think it's two, not three, right? They're just like all, like old, old plans or before it was just one type of plan. We treat them as PPU, and we're adding now new type, FWA, which they don't have any. Well, they'll be creating them, right?

**Noah** [09:33]
Yes. And I mean, all their service plans, we know they're PPU 'cause they didn't have any other type of devices. That we weren't supporting any other type of devices. So all the service plans were-

**Aksana** [09:41]
Um-

**Noah** [09:41]
... for sure PPU service plans, except for the ones-

**Aksana** [09:44]
Yeah

**Noah** [09:44]
... that they specifically set up now for the FWA Verizon devices, which is just the one service plan, I think.

**Aksana** [09:50]
Yeah. So with that being said, we need to remove that no restriction as an opt-... 'Cause I still, I see that in beta you patched all the, all data. Like, all plans-

**Noah** [09:59]
Yeah

**Aksana** [09:59]
... now say as PPU, but however, if I go to edit, we still have no restriction as an option, which is probably to remove it, right?

**Noah** [10:07]
Yep. That's what, that's what I'm asking about and suggesting-

**Aksana** [10:09]
Mm-hmm

**Noah** [10:09]
... is that we don't have that option at all. We just have PPU and, and, and-

**Aksana** [10:14]
Sure

**Noah** [10:14]
... F- FWA. If that makes sense, I'll add a ticket and do that, and we'll get that to beta quick. And I don't know if that should delay our roll or just be a follow-up. I don't think it really... Up to you. But they haven't approved it anyway, but hopefully they will.

**Aksana** [10:32]
Yeah. And that's, I just read the comment. I kinda read it that way too, as, um, they saying it's almost like it's one or two. It's not one, two, or three, right? And we, you know, error transition from PPU to FWA, but no, no restriction to any, uh, to FWA, right? That's why they're not seeing the error. Um, yeah.

**Noah** [10:52]
Yeah. They were assuming, they were assuming that there was, and it-

**Aksana** [10:55]
Yeah

**Noah** [10:55]
... it's like, but they didn't change that service plan that way. So I think they thought all of them were that way, so that's why I went ahead and changed them all.

**Aksana** [11:03]
Okay. Does that mean, um, if when we go to production, you would have to go through and change everything to-

**Noah** [11:18]
Yes, unless I do this, um, this migration and change quick first before we go to production, in which case that migration would handle it, uh, you know, uh, as well.

**Aksana** [11:28]
Mm-hmm.

**Noah** [11:28]
It would just set everything to PPU by default anyway.

**Aksana** [11:31]
And really, Noah, from kinda the, the, the service plan, no restriction or PPU, they are identical, right? Like when we-

**Noah** [11:44]
No.

**Aksana** [11:44]
They're not. Okay.

**Noah** [11:46]
Well, I mean, the whole point of that is to restrict what-

**Aksana** [11:50]
Yeah, yeah

**Noah** [11:50]
... you can assign to device. So-

**Aksana** [11:53]
Yeah

**Noah** [11:53]
... no restriction means you can assign it to any device, and I, I just don't think that makes sense now that we've seen how it works. I thought it made sense-

**Aksana** [12:03]
Okay

**Noah** [12:03]
... when we were defi- when I was defining the tickets and the T- the TRB.

**Aksana** [12:06]
No.

**Noah** [12:06]
And now that we've done it, it doesn't make sense to me anymore.

**Aksana** [12:09]
S- yeah, it's A or B. It's not A, B, or C.

**Noah** [12:13]
Yeah.

**Aksana** [12:15]
I think that, you know what? I think, I think you're right, but I think, and, and I don't think we need to like talk to them, like, like meeting. But I do think maybe we just need to like summarize kinda here is what we're hearing and the assumption we're making, and here's what we're gonna do, just to make sure, um, that they like, yes, on the same... Like, I just wanna like validate with them maybe before we do it.

**Noah** [12:41]
Yeah. Well, I mean, I put that long comment in. If somebody can take that and, and clientize it and put it in Trello to them or else in person to them. Basically, I, hopefully I put the, that made sense on the ticket that I, I, that I commented.

**Aksana** [12:54]
Which one did you put it on? The Trello one or some different one? Sorry, I'm not-

**Noah** [13:00]
I think it's 1978, but I'm having trouble finding anything today. Too much going on.

**Aksana** [13:07]
1978.

**Noah** [13:08]
1978, I think. Where is that?

**Aksana** [13:10]
Yeah.

**Noah** [13:10]
How come I can't find that ticket?

**Aksana** [13:11]
1978. It's linked in the project channel. Um...

**Noah** [13:24]
Oh, I have Epic selected. I'm bad at Jira. I'm like, "Why can't I get this?" It's like, I had Epic selected the whole time.

**Aksana** [13:31]
1978.

**Noah** [13:32]
Filters are hard.

**Aksana** [13:38]
We all have those things, uh, where you're like, "Damn it, I do much harder things. Why can't I do this basic thing?" Like, like Jira ticket creation.

**Laura** [13:50]
It's like share screen.

**Aksana** [13:51]
No. Yeah. I'm like, I know I can do it. I can do much harder things than this. Um, okay. Let's see. Okay, so I have 1970. Oh wait, what? 70... 78. 1970. I'm going by ID 'cause I don't know where to find it. There you go. No, um...

**Noah** [14:19]
Yeah. And then it's the reply I made to Laura's post or that main th- the comment that came in from them. So it should be 18 hours ago, whatever. That one.

**Aksana** [14:33]
All right. Okay, so we're just saying two types, PPU or FWA. All current ones are converting to PPU. FWA is a new, and now one or two need to be selected as part of the plan, and devices can be assigned only to a plan that's match on a device type.

**Noah** [14:55]
Yeah. I don't, I don't think they'll have any pushback to that, 'cause it seemed to me that that's what they were assuming. It was working already.

**Aksana** [15:01]
And we were honestly, when we were looking at this whole like no restriction, was like, what's the point of-

**Noah** [15:07]
Yeah

**Aksana** [15:07]
... at first we're adding restriction and then we're saying, "And also no restriction." It's just like-

**Noah** [15:13]
Yeah, the-

**Aksana** [15:13]
... it's one or the other. Why do we have both?

**Noah** [15:16]
The, the point was until I saw that they were setting up one new service plan and that's how they were expecting it to work, I was assuming that it was gonna work where the s- existing service plans had like an FDA FWA version. That's wh- that's why I had those no restrictions in there, is I didn't know that they were, they were going to only be setting up new s- service plan and expecting it to work there. I just didn't understand that.

**Aksana** [15:39]
Yeah. And we, and we're not gonna have issue with, 'cause Verizon has... Verizon, for, if you, if they build a service plan for Verizon, we could be one or the other. T-Mobile also, because we just added T-Mobile business. But like AT&T-

**Noah** [15:58]
Yeah

**Aksana** [15:59]
... assuming they port to AT&T, they need to be building only PPUs, right? Because there are no business devices on AT&T. And we-

**Noah** [16:08]
So-

**Aksana** [16:08]
... we're not having the logic even for AT&T to route it differently for that service plan, right?

**Noah** [16:14]
Correct.

**Aksana** [16:15]
Oh, okay.

**Noah** [16:16]
And, and for, I mean, for the stuff we did for T-Mobile, which, you know, will come later, they're also expecting to use the existing FWA service plan. So it's, it'd, it'd be good if it also works the same way. That doesn't make sense for T-Mobile PPU devices to be on anything but a PPU service plan either. So for everything, I think it makes sense. And-

**Aksana** [16:40]
Okay

**Noah** [16:40]
... probably also, even though it's not set, it probably also makes sense that you can't set any of the AT&T devices to any, to the FWA service plan. So they should all be PPU only as well, right? Yeah.

**Aksana** [16:53]
Well, all-

**Noah** [16:53]
So I think on all levels it, that's what makes sense.

**Aksana** [16:58]
But do we have that validation for AT&T devices? Do we validate-

**Noah** [17:02]
I mean-

**Aksana** [17:02]
... that device, AT&T device cannot be put on that A, FWA?

**Noah** [17:08]
Yes. I mean, it works.

**Aksana** [17:10]
Okay.

**Noah** [17:10]
There's no, the PPU and the FWA stuff is not actually, um, carrier specific. You, it's set on the device, it's set on the service plan, it doesn't care f- about the carrier. The only thing that's actually carrier specific is that we're doing the little hack in the interface to make it show up under the carrier just because the client seems to feel, feel like it's that way, but it's not. It's just on the device and the service plan.

**Aksana** [17:34]
Otherwise it's a carrier agnostic. Okay.

**Noah** [17:37]
Yes.

**Aksana** [17:38]
Um, sounds good. I can draft something to them. And yeah, I think you're on the right track. Let's just then rework this.

**Noah** [17:47]
Okay. I'll add a ticket and hopefully, I need to get this-

**Aksana** [17:52]
Mm-hmm

**Noah** [17:52]
... CP stuff done, but as soon, as soon as I know what that I should be able to get it done pretty quick today I would think.

**Aksana** [17:57]
Okay. Okay.

**Laura** [18:01]
So f- then for all of this, Aksana, you'll follow up with Devon and explain the change that w- that Noah's gonna make?

**Aksana** [18:11]
Yeah. I'll just, um, draft a comment, um, to the, to the Trello ticket so they- Just in case if they come back like, "No, that's not what we're thinking."

**Noah** [18:21]
Right.

**Aksana** [18:22]
We, we don't s- waste time.

**Noah** [18:24]
All right. Okay, cool. Thank you.

**Aksana** [18:27]
Mm-hmm.

**Noah** [18:29]
All right. Um, that sounds good. Um, Richard?

**Richard** [18:40]
Mm-hmm.

**Noah** [18:42]
What have you been doing? I see some-

**Richard** [18:45]
Well-

**Noah** [18:45]
... of your stuff has moved.

**Richard** [18:47]
Yeah. So I've been, obviously I worked, I was focusing on that critical, uh, yesterday. It was just the invoice stuff, and then I figured it out, sent them the email. And I, I do plan on just, honestly, if, and if I'm the only one available, I, I'm fine just taking it. I've been doing it as a hotfix, putting it in review, putting it in beta, putting clear test stuff on it, and moving forward with that. I don't mind taking that. And then I also really wanna get us moving on the MySQL upgrade. Um, I kind of wanna just upgrade, uh, review with the latest because it's not on an RDS so it's fairly simpler. And then remember that smoke test that you did, Laura, for, where you were just like ran Claude to click on everything, I think. Is that what you did?

**Aksana** [19:44]
Uh, basically. I mean, I-

**Richard** [19:48]
Yeah

**Aksana** [19:48]
... I gave it the thing that Noah put together and was like, "Do this. Use Chrome."

**Richard** [19:56]
Yeah, so if we could do that again with the MySQL, and then I'll feel confident about the upgrade on review and be- and then probably just do beta. Might as well go ahead and then like sort of prep production as much as I can, and then give them a date where we can, uh, cut things over. But I do kind of like, I still wanna think about the cut over and stuff a little bit more. Um-

**Aksana** [20:22]
When you say cut over, are you talking about now MySQL upgrade or PHP upgrade or server upgrade?

**Richard** [20:27]
Uh, yeah, I'm talking just about MySQL upgrade. I think that one should trump the other one just because we have... I mean, it's not that hard of a deadline, but it is a harder deadline of July, uh, 31st, so.

**Aksana** [20:42]
Yeah. I think it's, it's just like, again, like everything else, is piling up on our plate, so the sooner we can get it off our plate the better. But are you saying though for that one we need to like do testing obviously, make sure that all the queries are delivering what we expect them to deliver, right?

**Richard** [21:02]
Yeah. Honestly, I have high confidence as long as the site doesn't break with an upgrade. I mean, that's, but as, in terms of like it is good to just test everything, 'cause maybe we're using a reserved keyword somewhere. I will do due diligence there too, but I want like testing on top of the reserved keyword of like you're able to click on everything, is how I'm thinking about it.

**Aksana** [21:25]
Do you know just for like out of curiosity-

**Richard** [21:28]
Yeah

**Aksana** [21:29]
... are we affected on just like select or just updates, inserts, deletes, ev- uh, every s- all SQL queries are affected-

**Richard** [21:39]
Well-

**Aksana** [21:40]
... by the server?

**Richard** [21:40]
Well, basically if we're using rev- reserved keywords anywhere-

**Aksana** [21:44]
Right

**Richard** [21:44]
... I think that's the issue that we would run into it, but I, it also depends how we configure that. But I don't think, I think the way it is though, if we do use a reserved keyword, we're in trouble, but I believe I ran a report already that says we weren't. I have to double check what's, what's going on with that. Uh, uh-

**Aksana** [22:04]
Oh, you-

**Richard** [22:04]
Yeah.

**Aksana** [22:05]
So you're saying that you techn- you didn't identify any changes that need to happen in our code. So we'll just up- you update it on the server side, right?

**Richard** [22:18]
Mm-hmm.

**Aksana** [22:18]
Database side. But you're saying the c- the code sh- worked fine. There is no like...

**Richard** [22:25]
Well, I, I do wanna do just a more thorough, another scan of the code myself real fast.

**Aksana** [22:30]
Okay.

**Richard** [22:30]
Not like, not looking at every file, but like having Claude do another more thorough scan-

**Aksana** [22:35]
Yeah

**Richard** [22:35]
... just so I'm sure. And then I do wanna just go ahead and upgrade. The other risk was with the users, but I'm thinking we just recreate them anyway on the new databases because there's like a different password hashing strategy or whatever that's default. Um, so I think we just mitigate that risk by just recreating the users, 'cause we have their credentials anyway. Just create them the same.

**Aksana** [22:59]
Okay.

**Richard** [23:00]
Um, yeah.

**Aksana** [23:04]
Um, if you have a capacity, uh, obviously, uh, let's... So this invoice to NACHA discrepancy-

**Richard** [23:13]
Mm-hmm

**Aksana** [23:13]
... bug that you found, we need to fix this first.

**Richard** [23:17]
Okay.

**Aksana** [23:19]
Um, and then yeah, I would say I'm aligned with you. Do the MySQL and then get back to the server.

**Richard** [23:29]
Then Noah, do you have any thoughts on that? 'Cause I know you said we had like a lot of stuff on review or whatever. What would you say? Merge to review. Reminder not to merge anything in the review branch now unless it's been approved. I-

**Noah** [23:44]
The review branch is currently the CakePHP upgrade branch. So anything, if we merge to review, that's means it's gonna go with a CakePHP upgrade. So I'm saying don't do that. Do a integration, keep it in a feature branch for now until we get at least revie- review over to beta. Once we have it to beta then we know we can, beta can go to, could be go to production with CakePHP upgrade. I just don't want to be having stuff making the Cake upgrade launch bigger with other features accidentally, if that makes sense.

**Richard** [24:19]
Uh-huh.

**Noah** [24:20]
So that's why review is kind of like frozen right now. And if you need to get it in testing, do... I already have the review environment into, in an integration branch, so you just keep your... feature branch and merge that feature branch into that existing integration branch, and then you can ha- have your stuff tested, but don't put it directly into the review branch because we kind of reserving that for the KP as we upgrade, if that makes sense.

**Richard** [24:43]
Okay.

**Aksana** [24:44]
Because we wanna-

**Noah** [24:44]
Also-

**Aksana** [24:45]
... separate those so it's easier to track what's causing if we start having, seeing any, any issues.

**Noah** [24:51]
Yeah, so keep that for bugs that we have with actually the upgrade or whatever else. And also note for the MySQL upgrade, if-

**Richard** [25:01]
Yeah

**Noah** [25:01]
... assuming that we are gonna have to have downtime for that, we should also plan for and include some, like, compacting of tables. I know with our purging project, we had all those tables that just have enormous amounts of, like-

**Aksana** [25:12]
Mm-hmm

**Noah** [25:12]
... gaps in them, so if we're gonna have downtime anyway, we can maybe include some, some of the planning for that too, just getting those tables com- what do you, what do you call it? I think it's just compact. Whatever it is.

**Richard** [25:22]
Yeah. I, I think how we might, and I, I have to think about this still, but I think how I might go about it is, like, you, you know how you could do that, like, an RDS snapshot.

**Noah** [25:33]
Mm-hmm.

**Richard** [25:33]
So I will take, like, an RDS snapshot of production. Obviously, it'll be launched into the newer 8.4, and then from there, you can maybe do, like, a diff of some sort after you stop all the services during the downtime. Like, I'm thinking that you have to stop all the services, then you have the snapshot. Then you, like, feed everything that's not into this, into the snapshot in the snapshot, and then you make it, like, go live, was sort of, like, how I was thinking it would go about it because the snapshot would handle everything, I think. I mean, I know it would take a while, but you could do the snapshot even while the other one's, while it's live. So you could wait till the snapshot's done, and then you could, like, do the differential thing. I was, that, that was just something I kinda, like, came up with right now, but I, there's might be a better way of doing it, but.

**Noah** [26:24]
I have no idea. I'll let you and review that with Swiz. Sounds, sounds scary to me, but I'm not, I've never done a database upgrade.

**Richard** [26:34]
Okay. Yeah, I'll review with Swiz anyway. I know, I know he already put together, like, some key points of what you should do to do it, but yeah.

**Noah** [26:45]
Yeah, and I'm sure he's doing the, a lot and supervising a lot of projects doing them, so it probably makes sense just to coordinate and make sure the, whatever we're doing aligns with what he's thinking.

**Aksana** [26:57]
I do think though, like, Richard, when you have a plan that you feel solid about, let's just make sure we get us all up to speed, so just for, like, learning purposes and visibility.

**Richard** [27:12]
Okay.

**Aksana** [27:12]
So we know what's happening.

**Richard** [27:14]
Will do. I'll, I'll work on that after I do the, the critical. Okay.

**Aksana** [27:21]
Okay. All righty. Hey, Stone. Are you, have you done any APW work, or are you still deep in Etink?

**Stone** [27:38]
Yeah, I'm still deep in Etink. Um, hopefully something this afternoon. The one ticket that's gonna be my good stopping point in getting close to the completion, and then-

**Aksana** [27:53]
Okay. Cool. All righty. Um, Aksana, anything else to update from your end? Um, nothing really right now. I just don't wanna, like, add more to the pile. It's already out of control so let's just scale that off. So I'll, I'll send the message to them about Verizon. Um, yeah, and then let me know if you guys, any of you are blocked on anything else, um, as far as, like, ongoing work. Um, I do have a question. Do we have few blocked bugs? Like, what, what are they? Richard, looks like they all have your name on them.

**Richard** [28:46]
Yeah, so I was, I was hoping that we could talk with the client about some of them because I did email them about them. These were all the critical issues, honestly, that came in, and I did get back to them on those. Um, uh, I was thinking it might be good to just hold off on calling them done until we actually formally talk to them because I, I think that they sort of understood what I said, but I wanna make sure, uh, that, that everything's clear to them. And-

**Aksana** [29:14]
Mm-hmm

**Richard** [29:15]
... that's the reason they're on hold really is just, like, just, uh, to remind myself that we should probably have a chat with them on some of the stuff.

**Aksana** [29:24]
And when you say, uh, where, are you at a, like, evaluation step, or you at, like, we fixed it, but let's align on what has been fixed?

**Richard** [29:34]
Uh, so some of them it's like-

**Aksana** [29:37]
Sure

**Richard** [29:37]
... I, one of them I wasn't even clear if they wanted anything, and they didn't make it clear by the emails, but I did explain to them why it happened. Like, that, that was the device export showing $0 price.

**Aksana** [29:51]
Yeah.

**Richard** [29:51]
That one, I, basically was until they close the current cycle, then the new one gets plopped in with values, and then the pricing shows up. So that's, that was sort of, like, our normal process.

**Aksana** [30:03]
Mm-hmm.

**Richard** [30:04]
And they just weren't aware of that, but I made them aware of that. But I did find, like, outlier devices with weird stuff on them, and they didn't, like, I think they did some of them on purpose that way 'cause they did not want those charged. So that, that, that one I feel like requires more communication. And then verified devices affected by cleanup deactivated devices. Oh, that was one where I found we mark it as deactivated, but one of them was suspended. And that I was hoping to hear back on, on why, so we could drill into that and get it more accurate for them. Uh, because they do not wanna charge deactivated devices in the next cycle. And then the critical check-in bug, I forgot what that... I have to look at what the heck that is, 'cause it's been a while, but... Uh, this box was checking in until this morning. Oh, this is where w- their, um, their devices can actually send us a check-in time that's inaccurate, and I wanted to see if they wanted to actually... It seems like they don't wanna do anything about it based on they didn't respond. But that's also I wanted to follow up on. So those three things.

**Aksana** [31:20]
Okay.

**Laura** [31:21]
So do we just wanna hold on these until we meet with them next week?

**Aksana** [31:30]
I mean, might as well since we have a lot going on already. But I do think that we need to probably, like, a- address and whether... And also I feel like sometimes, you know, if they're not responding, it doesn't mean that we shouldn't do anything. Sometimes, like, we could just kind of recommend doing A or B, right? And we could just communicate that, um, clearly to them. Um... Okay, but we can also, like, yes, we can also wait. We have some time with them next week, and, like, those could be things we address in if we don't get to it this week. None of it is, like, critical, right? And that's why I-

**Richard** [32:15]
No. Otherwise I, I would've raised this, think about it, but no, it's, uh, eh, as far as I can tell, yeah.

**Aksana** [32:22]
Okay. Yeah, 'cause it's, if it's more of a, "Hey, like, these are kinda the things that we mention, or notice, sorry, that are discrepancies or gaps or blah, blah, blah," and get more clarity and, um, make a recommendation on next step. Which could be do nothing, right? And could be do something. Okay.

**Laura** [32:47]
Okay. Um, only other thing I wanted to mention is reminder that Monday is Memorial Day, so we are closed. Um, and so that means the holiday is our backlog grooming day. Um, based on where we are with, like, the upgrades and all that, I don't know if we need that time to go through and try to size anything. Um, but I could look for time for us to meet later in the week if we want to try to work through any tickets. But I feel like the next things that we would really be sizing would probably be, like, the CIG5 tickets, um, which I know Noah still needs time just to get to that work.

**Aksana** [33:46]
And we're really, like, I mean, just to, like, say it out loud as a team, we, we really don't s- size things collectively, right? So just grooming could be backlog. This meeting could be used more for just like, let's see what's next in the queue and make sure it's clear as far as what is being asked. Um, 'cause I feel like every dev size their own tickets and we just kinda let it be that. So I would s- I would say then, like, I wouldn't expect even Noah to... A- and again, that doesn't mean that that's how we should do it. So just looking at Noah and Marjorie. Like, if Noah creates, like, five tickets, does Noah wanna size them as a group or do you just wanna size it, them yourself?

**Richard** [34:37]
I mean, you keep doing the, "Do you want to" thing. It's, it's not really the question. The que- the question is, is do you want to have estimates that are the most accurate for any reason? If you do-

**Aksana** [34:51]
Well, I'm happy with our estimates, yes

**Richard** [34:52]
... then, then it should be a team estimation, and then it should be... But if w- like, to me, the reason why I don't really care about it on this project is I don't think the client gives a shit about how long the things take, and so why are we spending time? It takes more time to estimate-

**Aksana** [35:10]
Yeah, and that's-

**Richard** [35:11]
... than it does just to do it.

**Aksana** [35:12]
That's-

**Richard** [35:12]
So that, it's just a matter... It's like, do you have a number on there that matches the work closer? Do we, do we care about that? It just doesn't feel like anybody cares, so why are we doing it?

**Aksana** [35:21]
Well, I'll, I'll tell you that I, I do care from a perspective of because that tells me how much work we have in the queue and whether or not I need to bring in more work, right? As a kinda-

**Richard** [35:31]
Mm-hmm.

**Aksana** [35:31]
Because if you're telling me that the backlog is only two hours of work versus 20 hours of work, there's difference, right? And-

**Richard** [35:38]
But for that, it doesn't have to be accurate. It could be a rough number.

**Aksana** [35:43]
It, it, it r- it doesn't have to be e- the exact, yes. It, it is the-

**Richard** [35:48]
Then-

**Aksana** [35:48]
... kinda T-shirt size. Um, but is, but I do need these numbers, and that's why, like, again, to me it's like I, I don't wanna make that decision. Let's all get in the room and try to size them together. Um, I do wanna see a number a- at, at the end. And, and it's not just me. I mean, like, from, like, um, work planning, um, we should be doing that as a team.

**Richard** [36:19]
Okay.

**Aksana** [36:19]
Okay. Um-

**Richard** [36:20]
I don't think I didn't hear any answer about what we're gonna do, but okay.

**Laura** [36:25]
It sounds like we can, once the f- five-

**Richard** [36:30]
Yeah

**Laura** [36:30]
... version five tickets are created, we can review them together and put estimates on them that will allow us to know how much work we have and how quickly Aksana or Aaron, if we get him back for BA work, how quickly they need to move on new tickets. Um, however, I think... I don't know. Noah, are you able to say, like, when, when you get start, when you get started on the version five tickets, let us know, and then based on that, I think we can probably find a makeup time for the backlog grooming meeting. Um, I just, I don't wanna move it to someday when, like, you're not gonna have the version five tickets quite ready yet or something like that. It's like-

**Noah** [37:44]
Yeah

**Laura** [37:44]
... it's probably gonna be-

**Noah** [37:48]
I imagine

**Laura** [37:48]
... the first time next week that is really open for everyone is, like, noon on Friday.

**Aksana** [37:56]
Yeah. I don't think we need to, like, move that meeting just to have that meeting. If we have stuff to accomplish, but... A- and it's not, and it's not just about CAEC five, right? We have two tickets in the queue right now that are also not sized. So I guess i- ideally, yeah, we wanna be able to size them as a team, but we also know that Stone will take them, and I'm guessing that he'll size them when he takes them.

**Laura** [38:26]
Yeah, I personally, I don't have a problem with, with it just being whoever's gonna work on the ticket sizes it.

**Noah** [38:32]
Yeah. I prefer that approach-

**Laura** [38:34]
Right

**Noah** [38:34]
... as well.

**Aksana** [38:39]
I will say one reason I don't like this approach, because when one person size it, there is no counterpart. You can... I could say something's gonna be 20 hours, and nobody's gonna argue me if it's one hour. It w- and as long as, I'm not saying I think we all, like, trust each other, but it's also, like, for that reason, you know?

**Laura** [39:03]
Yeah.

**Aksana** [39:04]
Yeah. But it's like, again, let's just kinda... I don't know. I know we've been kinda circling around this grooming, how we do it as a team, and we still have, kinda don't have, and yeah, from a client perspective, this client is not holding us accountable to the numbers we're putting on these, right? And for that reason, we're not kinda working on that muscle as a team. We're lucky here that they're not, like, counting minutes. Other projects are not like this, right? So, yeah.

**Laura** [39:44]
I'm, I don't know. I mean, there's, I feel, I feel like there's a case to be made either way. I feel like usually, like, most of my, like, 95% or more of projects I've worked on, it's whatever dev is working on the ticket is the one estimating it, and yes, that does leave things open to where, like, somebody could say, "This is gonna take me 10 hours," when really it should be an hour. But-

**Noah** [40:11]
Well-

**Laura** [40:11]
... usually-

**Noah** [40:12]
And it's also-

**Laura** [40:13]
... it's not

**Noah** [40:14]
... it's often the other way, too-

**Laura** [40:16]
Yeah

**Noah** [40:16]
... and which is usually worse, is that a dev-

**Laura** [40:18]
Yeah

**Noah** [40:19]
... doesn't realize, oh, that there's so much more involved in that ticket than they think, and they say it's gonna be two hours.

**Laura** [40:24]
Right.

**Noah** [40:24]
Then they get into it and they realize, oh, that was a 20-hour ticket, and it takes them 20 hours. And it's not because they're not doing the right thing. They're doing the right thing, but when they estimated, they didn't really understand or think through all the different steps that were required. You know, somebody else might n- know and see more involved. Like Richard knows all about some things, and he might say, "Oh, no, no, that's a much bigger ticket." So when, that's the main reason that I think most of the time when you do it, everybody's gonna pull up with the same number. But every once in a while, somebody actually understands something that the other people don't understand and know about that complexity, and that's where you get the value and that, oh, that ticket was actually a much better estimated ticket because somebody knew something that the other people didn't know.

**Laura** [41:03]
Right. That makes sense.

**Noah** [41:09]
So it, I mean, it's just a matter of spending the time to do it together takes time and takes work and takes effort, and there's a reason why most projects don't do it. It's because developers don't like to do it. And so that's why I-

**Aksana** [41:21]
Oh

**Noah** [41:21]
... it's like I said, I, I sometimes push for it. I sometimes push for it 'cause I know that's the better way to do it, but it doesn't mean that developers like it or that even I like it. It's just more like that's, if you're gonna, if you're wanting for accuracy, that's probably the better way to do it.

**Aksana** [41:37]
I, I hear you. And that's like, again, like, and it's not just from a developer, from whoever wrote it, right? Properly, yes. We need to sit down, kinda digest what's being asked to do, right? And uncover all the gaps and details and et cetera, and it's time. And whether, like, we all need to, like, sit together and spend an hour chewing on a ticket that one of us, you will take, is it the right approach? Yeah. You'll probably have a lot better acc- estimate at the end, but it's also, like, a- an hour of everybody's time spent.

**Noah** [42:13]
Yeah. You're also spending-

**Aksana** [42:14]
Yeah

**Noah** [42:14]
... more time. It's, there's an argument for you might get more accomplished by not doing it. You're just not gonna have-

**Aksana** [42:19]
Right

**Noah** [42:20]
... those accomplishments match what the numbers you put on, and if the client doesn't care, it's like I can see exactly-

**Aksana** [42:26]
Yeah

**Noah** [42:26]
... the argument for not doing it. It's like you're probably getting more done not doing it.

**Aksana** [42:30]
F- for that reason, I will say that's why, again, I think the whole framework that we've been doing, we're writing product requirements document for a ticket or a feature, right? Dev takes it and expect to come back with "Hey, there are gaps. I have questions. There are certain things that are contradicting or not clear," right? Uh, it, it, but it's one person spending time to put a plan together on how to do this, not all five of us billing clients to shape work for one of us. So I would say we, we still... I, I would say, like, Monday meetings, I s- it's just not grooming, but we do need to have that time to talk about upcoming work.

**Laura** [43:16]
Okay. Um, well then why don't I- I'll move the meeting to, from Monday the 25th, I'll say to Friday at noon.

**Aksana** [43:33]
Yeah. Set, let's, let's, let's have it, 'cause again, it kind of serves for us as a, like, soft sprint planning or climb and whatever. Other, 'cause otherwise we don't have any other than working sessions where they're not really planning sessions. Well-

**Laura** [43:45]
Yeah

**Aksana** [43:46]
... um.

**Laura** [43:47]
Okay. I'll move it now. And then, well, tomorrow we talk a little bit about... unless there's a very quick unanimous decision right now, on the Thursday working session, do we want to try to find a replacement time for that time slot, or do we just wanna go without a second planning session for, or working session for a while, since Thursdays are not a good day for the time being?

**Aksana** [44:26]
Yeah. For now my Thursdays are swallowed by Duke that, so it overlaps.

**Laura** [44:30]
Yeah. Um, I mean, I kind of wonder if we cancel it, and then if we find that we're, you know, our standups are con- are, you know, going really long or there's a lot more for us to work through, then we can find another day. But otherwise it's, uh, you know, moving it to a Friday or we're having a working session on Tuesday and Wednesday, um, which I feel like might not be effective.

**Aksana** [45:09]
Yeah. Let's just cancel for now. Let's keep Tuesday. We have our standups and let's, I mean, still have that, like, one hour every two weeks to kind of talk about upcoming work alignment. Um, as a, like, soft sprint planning, um, not grooming. I mean, it could be if somebody brings work forward and, again, maybe not as, uh, as far as like, let's put numbers on the tickets, but just like, let's see if the work is clear.

**Laura** [45:44]
I'll just change it from backlog grooming. I'll just rename it to pro- what? Project, project planning. Backlog planning.

**Aksana** [45:50]
Yeah. Okay.

**Laura** [45:53]
Mm-hmm. Okay. All righty. Thank you everybody. We'll talk to-

**Aksana** [46:01]
Oh, you, I know we kind of, uh, uh, totally ignored Aaron and he left. Does anybody know, uh, if he is working on APW at all this week?

**Noah** [46:11]
I don't.

**Laura** [46:12]
Okay.

**Aksana** [46:13]
'Cause we have tickets to test and I, uh, uh, and, uh, we could, maybe we could ping him. La- ping him in the channel. Let's see what he says. 'Cause if he's still planning to put some time in APW, like he should test, test T-Mobile, then that should be like two hours doing that. It's a very like, specific task.

**Noah** [46:32]
Yesterday in his post, he basically said mostly on ABC this week, and so I did comment on that post that T-Mobile FWA was ready for him, but when he has time, but it didn't sound like he's gonna have much time this week.

**Laura** [46:48]
Yeah.

**Aksana** [46:51]
Um, okay. So-

**Laura** [46:58]
All righty.

**Aksana** [46:58]
Okay. Talk to y'all.

**Laura** [47:03]
Bye.

**Aksana** [47:04]
Bye.
