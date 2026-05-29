00:00 Aksana Rahouski: start with anything from the push or- I would say let's just start with a quick, um, Stone, where are you at? Um, anything, you're still just kind of digging and-

00:15 Stone Marballie: Yeah, I'm still digging. Um, the only thing I've found is that, um, it seems that we can reset the last approved plan to the ATM unlimited plan version before you added the origin and the variations, and that should restore all the prices. Um-

00:34 Aksana Rahouski: We can do it on prod?

00:36 Stone Marballie: Yeah, it would be just a, a database update, um, update records.

00:43 Aksana Rahouski: Um, Devin, do you think there is any risks that any other customer going into custom service plan and start messing with this broken views? That's my worry. Um, which that it would restore, that it would basically flush out everything you, you guys did earlier for, um, basic- creating new version of ITM, but, um-

01:07 Devon D'Andrea: I, I would say that it's probably very low-

01:10 Aksana Rahouski: Okay

01:10 Devon D'Andrea: ... percentage chance that anybody did. Um, I can tell you that we, as WATM admins, have not done anything. Um, the only thing I don't know, um, and that perhaps I don't know if there's logging in the UI for, or if you would need to just do a quick query, is to see if any of my distributors went in and added anything today, but probably a low percentage.

01:41 Aksana Rahouski: Unless, Stone, are you saying this is kind of part of the solution that we need to s- take step back to step forward? 'Cause I know, um, and again, just kind of based on w- the tests that you ran on your sandbox that you told me about.

01:59 Stone Marballie: Um, yeah, so we've got a few options. Um, we could roll back the service plan to what it was before they added the origin. I think it, it looks like they added I-22 T-Mobile origin, AT&T-

02:15 Aksana Rahouski: Yep

02:16 Stone Marballie: ... and a bunch of other stuff, right? I assume that's the only thing you guys did, right? You didn't-

02:19 Devon D'Andrea: That's all we did.

02:20 Stone Marballie: Yeah. So if you roll it back to that, then we're right where we were before.

02:25 Aksana Rahouski: Uh-huh.

02:25 Stone Marballie: Everything should still work the same.

02:28 Aksana Rahouski: Okay.

02:28 Stone Marballie: Um-

02:29 Devon D'Andrea: I'm fine with that.

02:31 Stone Marballie: Um, so that's the other option, and then we could... But here's the, here's where I'm at, though. I tried that 'cause I have a copy of prod, you know, like an old version, and I did the same thing. I reset it and then added all the variations that you guys did to the ATM plan, approved it, and then went back to those companies you were saying, and, and their custom pricing transferred correctly. So I don't know where the delta is, 'cause when I, you know, did it, everything works the same way of what you guys were seeing when you tested it in beta and so-

03:06 Devon D'Andrea: Huh!

03:06 Stone Marballie: So, so that part's kind of interesting. Now, the only thing I did different, the only thing I saw on the server that you guys did, it looks like you guys made a bunch of changes to the service plan before you approved it, whereas I approved it in one go, 'cause I saw, you know, the end game of what you guys were trying to approve. That's the only difference.

03:24 Devon D'Andrea: We did. So what we did was, we did... We ma- we did one, we did one change and approval that j- we did... Okay, sorry. I'm sorry, to restate that. We've added the I-22 T-Mobile-

03:38 Stone Marballie: Mm-hmm

03:38 Devon D'Andrea: ... Origin AT&T, Origin T-Mobile, Origin Verizon, so four sub items.

03:44 Stone Marballie: Mm-hmm.

03:44 Devon D'Andrea: Added them all, hit save, and approved it. We made one other change after that, and subsequently approved it.

03:52 Stone Marballie: Okay.

03:53 Devon D'Andrea: We changed the Origin Verizon price on that same service plan, saved it, and approved it.

04:03 Stone Marballie: Okay.

04:04 Devon D'Andrea: It was just in an attempt to change that number, just to see if that would do anything, 'cause just in our-

04:10 Stone Marballie: Gotcha

04:10 Devon D'Andrea: ... in our simple troubleshooting. So we did-

04:13 Stone Marballie: Okay

04:13 Devon D'Andrea: ... we did one save with those four sub items, followed by an approval, and then one other save, just altering the Origin Verizon sub item, and with an approval after that.

04:28 Stone Marballie: Okay.

04:28 Devon D'Andrea: So are you saying right now, if I go into company service plans, that it's gonna look right?

04:36 Stone Marballie: No, I haven't made the change yet. I would have to go, I would have to go into the database-

04:40 Devon D'Andrea: Got it

04:40 Stone Marballie: ... and say, "Okay, the last approved one was the one before you guys added Origin."

04:44 Devon D'Andrea: Got it.

04:44 Stone Marballie: And then you see the prices. So that's basically what I've gathered from my-

04:48 Devon D'Andrea: Okay

04:49 Stone Marballie: ... test environment. Now, I don't know, my recommendation at this point would be, we could try that, right? See what happens, see if the prices are correct, and, um, go from there. And if we think things are screwed up, we could go the alt... You know, like, the very worst-case scenario is, you know, we back up the database every day. We could go back to a snapshot from before we did the migration. It's just that we would lose... You know, like, if your clients had come in and made anything between yesterday and today, we would kind of lose that information.

05:23 Devon D'Andrea: Sure. Well, we could, we... It doesn't necessarily, I don't think, have to be that way. We can judiciously bring back certain things. We don't have to do, like, a complete wipe, if that makes sense, but-

05:36 Stone Marballie: That's true.

05:37 Devon D'Andrea: Yeah.

05:37 Stone Marballie: Oh, okay, you're saying, like, we could just copy the company service plan table over and leave everything or just-

05:42 Devon D'Andrea: Exactly, yeah.

05:44 Stone Marballie: Yeah, yeah. You're right, you're right. That's, that's a very good point, because then you, uh, 'cause those things don't really change that much, and then you would still kind of... You know, all the-

05:53 Devon D'Andrea: Yeah

05:53 Stone Marballie: ... device usages, check-ins, et cetera, that's being coming in, those would have been preserved. So yeah, that's a good, that's a good contingency plan, in my opinion.

06:01 Devon D'Andrea: Okay.

06:02 Stone Marballie: But, um, to be honest with you, I was hoping to find a bug, if there was one-... between now and when we last spoke, but I was just hit Friday, and I didn't find anything yet.

06:13 Devon D'Andrea: That's all right.

06:14 Aksana Rahouski: But that's, that, that's concerning, right? Because based on your experiment-

06:18 Stone Marballie: Mm-hmm.

06:19 Aksana Rahouski: Not even sure there was a bug, right?

06:21 Stone Marballie: At this point, yes.

06:22 Aksana Rahouski: But yet we're seeing something that looks like the bug, because what you did, you kinda, you reverted back to the previous service plan-

06:30 Stone Marballie: Mm-hmm

06:31 Aksana Rahouski: ... and then you did that again, right?

06:32 Stone Marballie: Right.

06:32 Aksana Rahouski: Created a version.

06:33 Stone Marballie: Yeah, and then I just, yeah, then I just created a version with the origin, and I clicked save-

06:37 Aksana Rahouski: Right

06:37 Stone Marballie: ... and then I was like, "Oh, what up?" You know?

06:40 Aksana Rahouski: Which technically, I mean, like, let's, let's assume if we're on this on prod, we'll roll back ATM to the previous version yesterday, right? And then, and then that will kind of clean up, attach all the children to that, to the service plan, and then Devin and Adam do it again. They add new whatever T-Mobile pricing, right? And this should tell us immediately whether the same bug... And I would say at that point, if that happens again-

07:08 Stone Marballie: Mm-hmm

07:08 Aksana Rahouski: ... it's a, it's a bug, no-brainer, right?

07:11 Stone Marballie: Yeah.

07:11 Aksana Rahouski: It's not a fluke because we're, we're, we're thinking that perhaps something didn't clear cache or something like... I don't know, you, you had some ideas around, like, what, what could cause this behavior.

07:24 Stone Marballie: Right. That's, I mean, that's the only thing I could think of, but, um, it's, uh, you know, everything is done in an after save, right? We approve the historic- the service plan, and then while that save is going, it should affect all the children. You know what I mean? Which is the company-

07:40 Aksana Rahouski: Yeah

07:40 Stone Marballie: ... service plans that, you know, and it should all- it's all part of the same transaction. So I don't know what could have caused it to fail.

07:49 Aksana Rahouski: So let's just... Okay, because this is, like, our top priority, right?

07:52 Stone Marballie: Mm-hmm.

07:52 Aksana Rahouski: This is sitting in prod right now. We wanna resolve it ASAP. We don't wanna sit on it for too long, definitely not over the weekend.

07:59 Stone Marballie: Absolutely.

07:59 Aksana Rahouski: So, like, what are the steps we wanna take next? Do we want-

08:03 Stone Marballie: So, uh, do you guys want-

08:05 Aksana Rahouski: That-

08:05 Stone Marballie: Do you guys want me to reset it to the last one and see what happens?

08:11 Devon D'Andrea: I would probably say yes, and if we do that, we can, uh, then verify that the custom service plans do or do not go back to what they should be.

08:25 Aksana Rahouski: Yes.

08:27 Stone Marballie: I-

08:28 Aksana Rahouski: I am in favor of that because at least, at least in this case, you know that your data is, like, truly represented on the website, which is-

08:40 Stone Marballie: Mm-hmm

08:41 Aksana Rahouski: ... not the case right now. And then also, like, we can, again, Devin or Adam, one of you could, like, go back to ATM, do the same thing. You add just... And maybe don't add all of them, just one, one, like, Oregon Price, right?

08:55 Devon D'Andrea: Yeah.

08:55 Aksana Rahouski: Or just T-Mobile. Boom, two dollar or whatever. And if, if, if this repeats this behavior again, this clearly tells us it's a bug that we need to keep chasing, and not just a fluke that accidentally happened.

09:11 Devon D'Andrea: Okay. Listen, I'm down to share my screen and click some buttons whenever you guys want.

09:18 Stone Marballie: All right. All right, so you want me to do it right now?

09:21 Aksana Rahouski: Yes.

09:22 Stone Marballie: Oh, boy, no pressure.

09:24 Aksana Rahouski: Richard, if you, if you jump in, if you like, I can totally navigate the traffic. You be my safety net. If you feel like we're doing something that's risky-

09:35 Stone Marballie: Mm

09:35 Aksana Rahouski: ... stop us.

09:37 Aksana Rahouski: I would say as long as you, um, are able to reverse whatever you're about to do, just in case. So just keep that in mind. As, as long as things are done that way, we, we're good.

10:00 Stone Marballie: Let's see. We gotta go to Bastion Server, then hop over to production, right?

10:06 Aksana Rahouski: Yeah.

10:10 Stone Marballie: Um, which, which, uh, what credentials has the, has, uh, the keys to the kingdom?

10:22 Aksana Rahouski: So you can, uh, find this by looking in the production config. Um-

10:27 Stone Marballie: Okay, I've got one password opens my screen.

10:30 Aksana Rahouski: This- maybe we should do it just to, like, where we just, uh, touch base, me and someone could hop off another meeting.

10:37 Aksana Rahouski: Yeah, because-

10:38 Aksana Rahouski: Probably, it's probably a lot of pressure.

10:43 Aksana Rahouski: I, I was gonna say-

10:44 Aksana Rahouski: We'll be sure to-

10:45 Aksana Rahouski: I think we-

10:45 Aksana Rahouski: ... not mess anything up.

10:45 Aksana Rahouski: Yeah. Why don't you guys kind of, Stone and Richard, work on that so there's no this, like, pressure of all of us staring at you?

10:53 Aksana Rahouski: Yeah.

10:54 Aksana Rahouski: And let's demo, um, daily usage chart, if we have a sample, because that's, like, something so, uh, topic number-

11:02 Devon D'Andrea: Yeah

11:02 Aksana Rahouski: ... Devin and Adam, like I told you earlier, that daily usage, um, the feature is technically, uh, ready and half launched to prod. What I mean by that, so the- it's kind of twofold solution here. So what we did is that we already, um, cal- the way we calculate data usage today, we kind of put an extra, like, layer above it, and we said, "Okay, additionally, every night, grab all these usages and calculate daily and store. Calculate, store." So every day we store these daily storages for each device, which this was launched in prod yesterday with everything else. So starting from yesterday, devices will start building these charts, day one, day two, day three. What I wanna ask you, like, at what point we wanna kind of release the front end? Because if we release the front end immediately, what we're gonna start seeing that devices that start collecting daily data usages, we'll start seeing it starting as of, um, February fifth.... I don't want your customer to start falsely reporting that there is no data usage prior to the 5th. There is- there is no just, like, we, we didn't have a mechanism to slice it in days yet. So either we kind of let it sit for 30 days until we build, build out this, like, backwards, right? Or we kind of rip the Band-Aid off right now, but then there is a chance that you will have customers reaching out and saying, "Oh, my device in May didn't have any data usage between January and February 5th," which is not true, and you will have to explain to every single one of them what's happening there. Does that kind of make sense what I'm saying?

12:47 Devon D'Andrea: I have no, uh, problem whatsoever with just starting on an arbitrary date, as long as we just go... You know, as long as we're just moving forward from there.

12:59 Aksana Rahouski: Mm-hmm.

13:00 Devon D'Andrea: You know, I mean, the question of whether or not we want to give them access to seeing daily usage go in arrears into a previous cycle, that's maybe what we may- maybe have to discuss. But-

13:12 Aksana Rahouski: Mm-hmm

13:12 Devon D'Andrea: ... as far as launching this feature, I... You know, Adam, you could agree or disagree with me, I don't know, but I, I don't think I'm gonna have a problem with worrying about customers saying, like, "Hey, how come this only started on-

13:25 Aksana Rahouski: Mm-hmm

13:26 Devon D'Andrea: ... you know, X date?"

13:28 Aksana Rahouski: Mm-hmm. Adam?

13:28 Adam Curcie: Yeah, I mean, I, I don't think it's a, it's a big deal. I, I don't have any problem also just waiting 30 days if you think that there's a, like, a benefit. I, I mean, I, I'd imagine, you know, if we wanna do it in two weeks, that'll probably have, like, enough-

13:47 Aksana Rahouski: Mm-hmm

13:47 Adam Curcie: ... daily usage, um, you know, to make it, like, helpful perhaps. But-

13:54 Devon D'Andrea: I-

13:54 Adam Curcie: ... I mean, I, I... I'll tell you this, as, as neat as it's gonna be, I don't think there's anybody, like, waiting for this. So if we wait 30 days-

14:03 Aksana Rahouski: Yeah

14:03 Adam Curcie: ... I, I, I don't have an objection. That's me.

14:08 Aksana Rahouski: Um, uh, Aaron, do you wanna pull out and just, just show them what it looks like?

14:14 Aaron Diefes: Uh, yeah, sure. Uh, let's... Share screen here. Yeah, this should pull up the correct- Yeah, so, uh, this is just, like, a device that we, that I have. Like, uh, this is... I'm on review. Um, but, you know, so we had- this was in place previously, right? So we have, like, a monthly view. Um, so I generated some data. We actually- we do have a test feed where you can generate, uh, daily data usage.

14:39 Devon D'Andrea: Mm-hmm.

14:39 Aaron Diefes: Um, but I generated some test data for basically the past 31 days, so this would be if it was, like, full use. And I also generated a data spike on a given day, but this is just, you know, kind of what it looks like, right? So it's just past 30 days, um, hovering over tells you exact amounts, and, it, the graph just scales with, you know, your top usage or whatever in the past 30 days. Um, but yeah, that's kind of what we have here.

15:06 Devon D'Andrea: I love it.

15:07 Aaron Diefes: Yeah.

15:08 Adam Curcie: No, I think it's gonna be super useful but, you know, I, I definitely see looking at it, like, having at least, like, a week or two-

15:17 Devon D'Andrea: Mm

15:17 Adam Curcie: ... when it goes live is gonna be fairly important. So I don't, I don't think we should launch it any earlier.

15:24 Aksana Rahouski: Mm.

15:24 Adam Curcie: If we wanna wait a full 30 days, I think that's fine. I don't know, Dev, what, what's your thoughts?

15:29 Devon D'Andrea: So that means what? Pushing something to prod that is starting calculations?

15:36 Adam Curcie: Oh, they said it already is calculated.

15:38 Aksana Rahouski: Yeah, the c- the, the engine, the engine-

15:40 Adam Curcie: It is-

15:41 Aksana Rahouski: ... and it's just the front end. So, like, that's-

15:45 Devon D'Andrea: Oh

15:45 Aksana Rahouski: ... really simple part.

15:47 Aaron Diefes: Yeah.

15:48 Aksana Rahouski: Because, because if we do it now-

15:50 Devon D'Andrea: I misunderstood, yeah

15:51 Aksana Rahouski: ... If you do right now, you go to the device. What I don't want to happen, and perhaps it's not gonna happen, you have customer who will, like, jump on it and then like, "Oh, this is great." Like, "I see >16:01 Devon D'Andrea: Right

16:01 Aksana Rahouski: ... And they're like: "How come it's only one day? Was d- my device dead prior to the 5th?"

16:06 Devon D'Andrea: Right. Right, right, right, right.

16:08 Aksana Rahouski: So-

16:08 Devon D'Andrea: I, I... Okay, I understand now.

16:10 Aksana Rahouski: Mm-hmm.

16:11 Devon D'Andrea: Um, I was thinking- Okay. Yeah, no, I'm good with just, yeah, letting it run until we're ready to show some, some, uh-

16:20 Adam Curcie: Yeah

16:21 Devon D'Andrea: ... some data, and then, you know, probably get a request at some point. We can put this in the backlog to, like, export it or some shit. I don't-

16:31 Adam Curcie: Yeah.

16:31 Devon D'Andrea: You know what I mean? Uh-

16:33 Aksana Rahouski: My next question, whether export, but like, um, remember when we talk about API?

16:39 Devon D'Andrea: Mm.

16:39 Aksana Rahouski: And that customer we talked to, they actually did ask for it.

16:42 Devon D'Andrea: Yeah.

16:42 Aksana Rahouski: But I don't know if you've heard any feedback from them whatsoever. I mean, they-

16:47 Devon D'Andrea: We had- They've had a really... They've kind of-

16:49 Adam Curcie: Yeah

16:49 Devon D'Andrea: ... they went into kind of hibernation as far as correspondence goes through the holidays a little bit. Um-

16:57 Aksana Rahouski: Okay.

16:57 Devon D'Andrea: Although-

16:58 Adam Curcie: We, we're meeting with them Monday. Um, I think it's, uh, like, a whole hour or two. We have a lot of stuff to go over with them.

17:04 Aksana Rahouski: Okay.

17:04 Adam Curcie: Excuse me. Um, hey, real quick, Joe, I just had a crazy thought on the concept of exporting.

17:11 Aksana Rahouski: Mm-hmm.

17:12 Adam Curcie: Could you... I don't know if it's possible, but if there would be a way just to throw it in the ek- the, like, the check-ins export-

17:22 Devon D'Andrea: Ah

17:22 Adam Curcie: ... maybe on a separate sheet in the same workbook.

17:26 Devon D'Andrea: That is getting crazy.

17:27 Aksana Rahouski: What ex- what exactly would you want to export in this case? Uh, like-

17:31 Adam Curcie: The dates and the totals for the daily usage.

17:34 Aksana Rahouski: Date, age, and totals? Okay.

17:36 Adam Curcie: Just 'cause, like, I don't know, we have- we do have customers who like, you know, to, to get nerdy with things like we do.

17:44 Aksana Rahouski: Mm-hmm.

17:44 Adam Curcie: So, like, if they're... If they wanna look at a bunch of this data, they might export like, you know, a bunch of these and, like, make pivot charts and other stuff-

17:52 Aksana Rahouski: Mm-hmm

17:53 Adam Curcie: ... that does to help people. So-

17:55 Aksana Rahouski: Yeah.

17:55 Adam Curcie: Um, you know, I don't know. I, again, it's a backburner thing. They-

17:59 Aksana Rahouski: Yeah. Yeah, yeah

17:59 Adam Curcie: ... There's definitely not, you know, a, a threat.

18:02 Devon D'Andrea: Well, it's not a bad idea, though, if we just consolidate.

18:05 Adam Curcie: Yes.

18:05 Devon D'Andrea: Like, if you're gonna export-

18:07 Adam Curcie: Yeah

18:07 Devon D'Andrea: ... You're gonna ex-

18:08 Adam Curcie: If you just want to export-

18:09 Devon D'Andrea: ... with all the check-in data and then the usage

18:12 Adam Curcie: And yeah, the usage on another sheet.

18:14 Devon D'Andrea: I mean, shit, you could make it all-- you could make all three of them, the aggregated usage, three-month look behind. Well, I don't know the usage.

18:21 Adam Curcie: Well, I mean, if you do, if you do like a, a, a data usage sheet, you, you could have... I mean, 'cause think about like how much space on that spreadsheet is one column of dates and one column of megabytes gonna consume, right?

18:35 Devon D'Andrea: Yeah.

18:35 Adam Curcie: So you, you could just have the, the, the last three total aggregates all a little bit, a couple columns over, 'cause it's only gonna take up five or six rows at most, you know, if you have it formatted and spaced and stuff, so.

18:47 Devon D'Andrea: And Adam will provide any color coding that you guys want.

18:50 Adam Curcie: Color coding, paint, all of that will be delivered- -prior to, you know, us submitting the, the ticket. But yeah, um, we're really off topic.

18:58 Devon D'Andrea: Yeah.

18:58 Adam Curcie: Looks great, though, but-

18:59 Devon D'Andrea: But for what it is right now, yeah, like it clearly paints the picture of like... We just had somebody say yesterday, "Hey, my software team thinks that I should be using this data as of the-"

19:12 Adam Curcie: Yes

19:12 Devon D'Andrea: "... sixteenth."

19:13 Adam Curcie: Yeah.

19:13 Devon D'Andrea: This would be fu- this would be perfect.

19:15 Adam Curcie: It really would.

19:16 Devon D'Andrea: I'd be like, "Hey, Matt, look for yourself." Not that I would talk to a customer that way, but it'd be cool if I could. Awesome, I love that. Thank you.

19:27 Aksana Rahouski: And that's what we want ultimately, right? To make sure we're just pointing customers to where they can answer their own questions without you doing the job for them.

19:34 Devon D'Andrea: Exactly.

19:37 Aksana Rahouski: Okay, so, um, so how are our-

19:41 Devon D'Andrea: You wanna talk, you wanna talk BI?

19:43 Aksana Rahouski: Yes, I wanna talk... Okay, okay, so let's-

19:46 Adam Curcie: I did read through-

19:47 Devon D'Andrea: Adam is in charge of this one. Adam can, uh-

19:50 Adam Curcie: Yeah

19:51 Aksana Rahouski: ... okay.

19:52 Devon D'Andrea: Take the wheels here. Take the wheel.

19:57 Aksana Rahouski: Okay, so this one just to kind of like set the stage, right? I know you guys-

20:01 Devon D'Andrea: Yeah

20:01 Aksana Rahouski: ... at a high level, um-

20:03 Devon D'Andrea: Yeah

20:04 Aksana Rahouski: ... y- we need to, we have, we wanna be able to d- like, now to have two Verizon accounts. With that being said, I know we kind of kicked this conversation as of, like, at what point device needs to understand, like, for, like, a-activation, deactivation, whatever. We're- every time we ping, like, Verizon account, right, or make an API call, which one do we go to, one or two, right?

20:29 Devon D'Andrea: Yeah.

20:30 Aksana Rahouski: Um, and based on your response, this kind of back and forth emails that we had, what I heard from you that a, a sufficient solution would be... And let me just kind of, um... Okay, we'll kind of skip through this. Hold on, let me just do, um, even like, we'll stay on the scope here. Um, so obviously, uh, we'll have to store two devices, so, sorry, two accounts, so we can like point to one or two. But what we agreed on is we need to basically be able, on a device level, to pick which device, uh, account, which Verizon account that device is gonna go to. But additionally, from what I heard, is that a, a service plan also need to be, uh, um, we need to, like, specify which service pl- which Verizon account it's compatible with. And once we kind of have the device on, let's say, account, um, business account, and once this device does, um, is being put or mapped to a service plan, it only needs to be matched against service plans that are compatible with the same Verizon account this device is on, right? Does that track so far, or...?

21:53 Devon D'Andrea: Let me ask this: um, what is the need, Adam, what would using, using service plan do for us? Why would we need to use a service-

22:14 Adam Curcie: What would it do for us?

22:15 Devon D'Andrea: Why would we need a service plan?

22:21 Adam Curcie: Because the way the portal works, you, you, the... What, what is it? There are certain things you have to have a service plan for certain reasons.

22:30 Devon D'Andrea: Well, that's what I guess I'm maybe asking. Maybe it's not a question for Adam. Like, why... If all I want to do is stipulate, this device needs to talk to the BI account, and this device is just gonna be billed at a flat rate, 'cause that's what they're gonna be billed at-

22:45 Aksana Rahouski: Yep

22:47 Devon D'Andrea: ... why do I need a service plan? And I'm not saying this to say, like-

22:51 Aksana Rahouski: Uh-

22:51 Devon D'Andrea: ... we don't need a service plan. I'm asking.

22:53 Adam Curcie: Well, updates, aren't updates re- the-

22:57 Aksana Rahouski: If, if-

22:58 Devon D'Andrea: Right?

22:58 Aksana Rahouski: From a device perspective, right, you're right. It does not matter because we already decided that someone here, right, on a device view-

23:09 Devon D'Andrea: Mm-hmm

23:09 Aksana Rahouski: ... uh, will be configuring, if it's a Verizon SIM number, I'm guessing additionally, we'll have to pick which Verizon, right?

23:18 Devon D'Andrea: Yes.

23:18 Aksana Rahouski: So now we have two, because if Verizon grade, we'll have to like find a space here that a user can pick, um, or, uh, or d- whatever. However it gets in, whether we are importing it or we're manually adding it, right, that with a Verizon SIM, it will always tell him which account to map it to, right? Um, doesn't matter which service plan, and this is why it's, it's the most flexible approach, because this way, device does- is not relying on anything else in order to understand which API to call to a- activate statuses and et cetera, right? My understanding- sorry, I'm getting this. You guys additionally had said where it does matter, and again, perhaps I, um, unless I completely, like, misread what you said, that you said that-... when this device that is on this very specific business Verizon plan gets matched to a service plan, the service plans they have access to is very different from devices who are on a basic plan. Is that true or not?

24:33 Adam Curcie: Yes.

24:34 Devon D'Andrea: Yes.

24:34 Aksana Rahouski: Okay, so that-

24:36 Adam Curcie: And, and the customers can't switch back and forth without-

24:39 Aksana Rahouski: Yeah

24:39 Adam Curcie: ... us actually doing it. Like, we can't, we, we can't even- like, we wouldn't want to give any customers the ability to switch that without us actually doing it manually.

24:51 Devon D'Andrea: Right.

24:51 Adam Curcie: Um, because there's, there's not just difficult, like, steps that have to get taken, there's also, like, contractual reasons.

24:59 Aksana Rahouski: Mm-hmm.

24:59 Adam Curcie: Like, the SIM can't... I, I, I forget the exact, like, you know, fine print, but I don't think the SIMs are allowed to just go back and forth between those two accounts, like, at our leisure, just on, like, as many times in the same, uh, month or year as we want. I, I- but I don't remember all of the plans.

25:17 Devon D'Andrea: So then, so then does Verizon Bri- Business Internet flag on this device mean there is no service plan, and customer, nobody should add... be able to add a service plan?

25:34 Aksana Rahouski: I- is that what I would say?

25:36 Adam Curcie: Like, I, I, I don't know. Dev, what's your objection to the service plan? Because we already have rules that say, like, if you're on tier three, you know, you can't go back to ATM with data. So it's like, I'm sure they could make a rule that if a box is assigned the business internet plan, the customer can't pick another service plan, right? So I... And-

25:59 Devon D'Andrea: That's fine.

26:00 Adam Curcie: Aren't service plans also required for the devices to get the correct updates?

26:06 Aksana Rahouski: Um-

26:07 Devon D'Andrea: Well, yeah, if we're gonna have configurations-

26:09 Adam Curcie: Okay, so, so again, I don't understand why we're trying to, like, i- invent a new wheel. Like, just make a n- new type of service plan.

26:18 Aksana Rahouski: I don't-

26:20 Adam Curcie: Or-

26:20 Aksana Rahouski: U- l- yeah, so ultimately, it... L- and let me first c- and I, I'm just worried we're talking about different things. The way that I'm seeing it, like I said, we just talked about device earlier. Yes, so like, let's say I created a device or imported a device with a, a, a ser- a Verizon SIM card, and I picked, and then it's gonna be a business account, which, which is gonna be this, um, option to... Or whatever. However we choose, we switch between the two, right? Business, I'm creating business now. So now, when I create a service plan, right, because I'm assuming that for these special devices, right, that sit on the business plan, service plans are also specifically designed, meaning I should be able to create a service plan, a new service plan, that's gonna be for that Verizon business plan. And when I pick Verizon, I also now, and, uh, additionally, need to specify if it's available for a regular or a business one or both. And what that means, that when this device gets assigned to a service plan, if it is, it, like, matches the service plan. Let's say you have two service plan. One is to, uh, basic and one is to a, a business, right? So then if device is get attached to a service plan, it can only get associated to the one that is on f- built for a business plan, not the other one. Is that, is that what we're trying to accomplish, or, or no? Or are we saying service plan doesn't matter, devices on either Verizon account could use the same service plans?

27:57 Adam Curcie: No, no, no.

27:59 Devon D'Andrea: Mm-hmm. Are we gonna need, are we gonna need conf- uh, and this again, it probably goes into a whole another discussion we can have about how this new config engine is gonna work. But like, you know, Adam, you said, like, "Don't we need the service plan to know which config is needed?" And like, yeah, like, right now, yes-

28:18 Aksana Rahouski: Yeah

28:18 Devon D'Andrea: ... we're gonna be maintaining configs for, which I guess we need to. I just don't know-

28:23 Adam Curcie: I-

28:23 Devon D'Andrea: I just, I thought, I just thought the path of least resistance would just be like, if this is a BI Verizon device, you know, instead of going into adding a new service plan, trying to figure out... I don't know. I don't, I, I, I mean-

28:37 Adam Curcie: Well, I mean, the- even with the way that the config engine in the future is going to be built, there will still be logic that dictates-

28:48 Devon D'Andrea: Yeah

28:48 Adam Curcie: ... okay, this box is on the ATM service plan, so it gets this config. This box is on tier one, it gets this config. And this box is on the business plan, so it gets this config with a completely separate APN, because it's a completely separate Verizon account.

29:02 Devon D'Andrea: Mm-hmm.

29:03 Adam Curcie: So, I mean, I just, I, I think a service plan, if you go back to your screen that had the, where we alter the service plans-

29:12 Aksana Rahouski: Yes

29:12 Adam Curcie: ... I mean, you could literally, for all of our existing service plans, if you wanted to add a field in for all the carriers that have got additional, you know, spots for the account numbers, you can do that because it'll make it... I mean, but I- for the other carriers, we're, we don't have additional accounts right now, and I don't think we ever will.

29:35 Aksana Rahouski: Mm-hmm. Mm-hmm.

29:36 Adam Curcie: Um, I will tell you, which I'm, like, hesitant to even say, but, like, the, the way we're gonna have to do the T-Mobile business internet solutions are not different accounts, but they're, they're different plan codes.

29:53 Aksana Rahouski: Mm-hmm.

29:53 Adam Curcie: So that also might be worth just either considering or planning for down the road. I, I don't know.

30:00 Aksana Rahouski: Okay.

30:01 Adam Curcie: Um, but-

30:02 Devon D'Andrea: I mean, it's just like everything on this page is almost, almost irrelevant.

30:08 Adam Curcie: Yeah. Well, e- exactly, right? We don't need much of this. We would need enable Wi-Fi, we would need enable firewall.

30:15 Devon D'Andrea: Mm-hmm. Mm-hmm.

30:16 Adam Curcie: Um, we don't want-... them to not see usage, so that would stay like that. Mm-hmm. But is custom is still required. Yeah, I mean, we, we don't need to use the- Uh, is custom is only- Right ... is custom is only if you're making it just for one company. Yeah, which we might do, right? Like, we might have a company for some reason at some point that we need to make a custom FWA plan for. We don't know.

30:39 Aksana Rahouski: Yes.

30:39 Adam Curcie: So we're just gonna leave that there. We won't need these device group names to be required. That- Or a usage limit. Yeah, we won't be adding a usage limit, so, um, yeah, that's... Well, actually, I mean, for T-Mobile, technically there is a usage limit, but, um, even on the unlimited plan, it's not actually unlimited. So, I mean, no, like, that's- I would stipulate we could leave usage limit. We're just gonna be setting it incredibly high, so-

31:15 Aksana Rahouski: S- and my assumption, and again, kind of this form, it stays as is, right? We're just saying for Verizon, we're adding account picker, right? Whether we're setting up a plan, now you're saying, "Is this compatible with a plan A, plan B, or both of these plans," right?

31:37 Adam Curcie: Yeah, I mean-

31:37 Aksana Rahouski: Is that-

31:40 Adam Curcie: ... I, uh, y- you- if you wanted to, like, the way... So, so the way, right, like, that you're describing it with having, like, the A, B, or both-

31:53 Aksana Rahouski: Mm-hmm.

31:55 Adam Curcie: Like-

31:55 Aksana Rahouski: Maybe just A or B. I don't know if both is in-

31:57 Adam Curcie: Yeah, if A... You, you can't... Like, both isn't really relevant-

32:01 Aksana Rahouski: Um

32:02 Adam Curcie: ... because it's, it, it's, it's just A or B.

32:04 Aksana Rahouski: Okay.

32:05 Adam Curcie: Like, we, we can-

32:06 Aksana Rahouski: Okay

32:06 Adam Curcie: ... move them to A or B.

32:08 Aksana Rahouski: Okay.

32:08 Adam Curcie: But what- may- maybe it makes sense for the devices-

32:12 Aksana Rahouski: Mm-hmm

32:12 Adam Curcie: ... on the import to have a checkbox, we could- for FWA, so that the system knows they're on the second account.

32:20 Aksana Rahouski: Yeah.

32:20 Adam Curcie: That-

32:21 Aksana Rahouski: And, and then, yes, for device, again-

32:24 Adam Curcie: Yeah

32:24 Aksana Rahouski: ... um, think of it as kind of two different things. Yes, like you said-

32:27 Adam Curcie: Yeah

32:27 Aksana Rahouski: ... there is, there is a device, right?

32:29 Adam Curcie: Yeah.

32:30 Aksana Rahouski: That we just already agreed that, uh, devices that are Verizon need to add. Uh, additionally, we need to be able to specify which plan they're on. W- w- that means we need to think through the create form, the update form, the import, right?

32:47 Adam Curcie: Yeah.

32:47 Aksana Rahouski: Because most of them will get imported, right?

32:50 Adam Curcie: Yeah.

32:50 Aksana Rahouski: So, like, that every door-

32:52 Adam Curcie: Mm-hmm

32:52 Aksana Rahouski: ... that leads us to a device gives us device that has this new information, right? So, and we can, and we can handle that. I think this one is pretty straightforward. Step number two, now what happens? Let's even, like, kind of follow your normal business, kind of how you guys run things.

33:08 Adam Curcie: Mm-hmm.

33:09 Aksana Rahouski: You import devices in. Let's say we already decided that whether you are imported manually, we add also additionally which Verizon plan it gets added, we know, right? Step number two is what you guys do, is that you do a device assignment, right?

33:26 Adam Curcie: Right.

33:27 Aksana Rahouski: So let's talk through that. Like, maybe that'll be easier to have that conversation. When we assign a device, let's go to our assignment page, um, we'll just pick something-

33:37 Adam Curcie: Mm-hmm

33:38 Aksana Rahouski: ... see what would- what we need to populate. As we can see, service plan is something that gets assigned to a device-

33:47 Adam Curcie: Yeah

33:47 Aksana Rahouski: ... when we, I do a device assignment.

33:49 Adam Curcie: Yeah.

33:50 Aksana Rahouski: In this case, if I'm assigning a ser- a, a, a, um, sorry, a device on a business Verizon plan, which-

34:00 Adam Curcie: Mm-hmm

34:00 Aksana Rahouski: ... now we're saying I should only be able to assign it to a service plan that is, has, let's imagine there is a... Um, I'm not on the service plan, sorry. Let's imagine that we have here-

34:14 Adam Curcie: Yeah

34:14 Aksana Rahouski: ... um, one and two businesses selected. I can only associate it with a, a, a, a service plan that is also ha- like, is, like, supporting the business account.

34:27 Adam Curcie: Correct. Yeah. Correct. Yeah. The business- On that, if you click that dropdown, you would have an option that says- We actually have one in- Yeah ... we just don't want to use it. Yeah, we do. It's just not set up correctly for, you know, long-term success. But, yeah, I mean, it's gonna... We'll, we'll click Business Internet. Mm-hmm. We, or, you know, and then that, the devices that are in the text box above will need to be flagged.

34:54 Aksana Rahouski: Correct.

34:54 Adam Curcie: Right?

34:55 Aksana Rahouski: Yes.

34:55 Adam Curcie: So, and if they're not, then it throws an error- Wait, wait, wait, wait, wait.

34:58 Aksana Rahouski: Correct. Well, not flagged. They need to be configured with a Verizon business account.

35:04 Adam Curcie: Well, yeah, but what, what- Yeah, but I'm talking about- Wait, wait ... checkbox as, like, a flag, right? What's the- Like- What's the steps here, though? I thought we determined that we already know that these serial numbers are flagged.

35:17 Aksana Rahouski: Yes.

35:17 Adam Curcie: Yeah, yeah, yeah. No, but, but it's like, so like if Vince is assigning one, like, if, if Dan sends a b- gives, fills an order, puts a serial number in, and it's not in our system as checked or flagged or VI- Yeah ... then, then it errors, because it doesn't-

35:32 Aksana Rahouski: Correct.

35:32 Adam Curcie: Yeah.

35:32 Aksana Rahouski: Correct. Yes, and we'll-

35:34 Adam Curcie: Hold on. Right

35:34 Aksana Rahouski: ... make sure a validation is in place. And now, uh, we also have to consider the fact that this is a bulk action. Meaning, if I have a number of devices-

35:44 Adam Curcie: Right

35:44 Aksana Rahouski: ... that meet criteria, half of them and have the- but we already, we, we improved this actually a lot, the last round of changes that we did.

35:52 Adam Curcie: Right.

35:53 Aksana Rahouski: If we try to assign a hundred of them and only 50 match, um, we have, like, a good er- validation in place, a good error response. You could actually see which succeeded, which have failed, right? But yes, um, ultimately, we just need to know how to match it to the... or validate that device-

36:11 Adam Curcie: Mm-hmm

36:11 Aksana Rahouski: ... service plan match is possible, right? Because if this device is business, but service plan is not, that match is impossible.

36:20 Adam Curcie: Right.

36:21 Aksana Rahouski: Okay.

36:21 Adam Curcie: Yes. Yeah, I mean-

36:23 Devon D'Andrea: ... So then we just-

36:23 Adam Curcie: I think for both these screens-

36:26 Devon D'Andrea: We just need to figure out what we wanna do with that Add Service Plan page.

36:31 Adam Curcie: Well, honestly, wait, I, I have such a crazy thought. I mean, if you- on the Add Service Plan, if you put it at the top, right?

36:38 Aksana Rahouski: Mm-hmm.

36:38 Adam Curcie: Like, literally just a checkbox, like, and in between the name, the top line where you enter the name, and underneath Service Plan, you just put the checkbox right there. If you check that, then all this other stuff-

36:50 Devon D'Andrea: Goes away

36:50 Adam Curcie: ... doesn't become required. It ca- it can just be unre- unequal, you know?

36:54 Devon D'Andrea: Yeah.

36:54 Adam Curcie: You can get rid of all the asterisks then.

36:56 Devon D'Andrea: Right.

36:56 Aksana Rahouski: Well, are you saying... Okay, so are you saying that this-

37:00 Adam Curcie: It doesn't go away, but-

37:01 Aksana Rahouski: -the service plans that you create for this business account-

37:05 Adam Curcie: Yeah

37:05 Aksana Rahouski: ... are really dramatically different from everything else, where you don't need limit, you don't need unit, you don't need all this other stuff, right?

37:12 Devon D'Andrea: We don't need limit-

37:13 Adam Curcie: Not, not really

37:13 Devon D'Andrea: ... and we don't need, we don't need limit, and we don't need group- device group names.

37:18 Adam Curcie: Yeah, we won't use any device group names, um, and we-

37:22 Devon D'Andrea: As far, as far as the required fields on this page-

37:25 Adam Curcie: Yeah

37:25 Devon D'Andrea: ... we only will need that checkbox and the name.

37:29 Adam Curcie: And the, the name. Yeah, everything else is optional and probably n- won't be used.

37:33 Aksana Rahouski: Yeah.

37:33 Adam Curcie: And then again-

37:34 Aksana Rahouski: Yeah, yeah

37:34 Adam Curcie: ... for T-Mobile, it's gonna be... Y- you can really implement an, a, you know, T- uh, we, we'll, we'll not worry about T-Mobile. I'm gonna stop bringing it up. I'm sorry.

37:44 Aksana Rahouski: Well, I, I wanna make sure... Again, I do wanna make sure that we build it as a, like, scalable, right?

37:50 Devon D'Andrea: Yeah.

37:51 Aksana Rahouski: Add it be- because we're- right now we're saying ultimately we have three different carriers, and it sounds like we're start forking where, uh, how do we tailor our service plans for carriers? And right now, we're solving for Verizon. So what I don't wanna do is if we do it as simple as checkbox, yeah, you guys will know, I'll know in my head that this is for Verizon. But we want it... And again, that's like, we'll take care of it. We'll, like, package it in a way that it makes sense that this is, like, a custom Verizon service plan. Because tomorrow, if we need to handle a custom T-Mobile service plan, we can handle it in the same fashion.

38:31 Devon D'Andrea: Yeah, I don't think it's gonna be any different. It's gonna be-

38:33 Adam Curcie: Well, it could actually be the same checkbox, it, 'cause like, you know, your portal, uh, for the way it handles stuff with, you know, this current service plans- ... if it's all on T-Mobile, it's obviously gonna, like if, if there's a T-Mobile SIM, it'll see that, and it'll- Like I said, and the only thing for the T-Mobile that's actually different is you're just changing the price plan, and I don't even think we have the price plan. Um, I don't think we even have it available to us, but I do know it's- we can- we have to re- like, we've had conversations with them. They will give it to us when we want. We just haven't needed it, so-

39:13 Aksana Rahouski: Okay.

39:13 Adam Curcie: But, but that would be for, like, the actual, like, activation component where-

39:18 Devon D'Andrea: Yeah, the part of it where the portal needs to talk to the device-

39:23 Adam Curcie: Yeah.

39:23 Aksana Rahouski: Yes

39:23 Devon D'Andrea: ... and it's gonna be on the device level. This is how you talk to the device.

39:27 Aksana Rahouski: Yeah.

39:27 Devon D'Andrea: Service plan-

39:28 Aksana Rahouski: Yeah

39:28 Devon D'Andrea: ... is still just gonna be, uh, stripped down to, to, you know, is really just-

39:33 Adam Curcie: Well, not talking to the device, you're talking about it's when the portal sends the-

39:37 Devon D'Andrea: That's what I meant. That's what I meant. Talking to the right, talking to the- talking to-

39:40 Aksana Rahouski: Yes

39:41 Devon D'Andrea: ... the right thing that it needs to talk to and what it needs to talk to it about.

39:45 Aksana Rahouski: And that is, and that is, like, again, we said, like, w- we are a- attaching it to the entity that it matters for, which is device-

39:54 Devon D'Andrea: Right

39:54 Aksana Rahouski: ... right? Device need to know who to paying, what A or B, he knows, right?

39:59 Devon D'Andrea: Right.

39:59 Aksana Rahouski: But what right now we're trying to solve is, because device needs to have a service plan, and we're assuming that this Verizon business account i- is gonna have this special service plan, how do we match now device to a service plan in a way that it, it, like, it doesn't allow to attach it to anything else other than the special service plan?

40:19 Devon D'Andrea: Right.

40:20 Aksana Rahouski: Okay. So I think a lot of it, like, let us kinda m- make sense. I think we're, like, on the same page, and like, again, not to c- like, go back, but we do need to rethink through, uh, well, how do we change the import? How do we change the, uh, device assignment page, right, that the proper, like, service plans are attached, um, from a single device page where you can manually change the service plan? Same validation will be in place, that you can only attach it to the business plan, service plan, and nothing else. Talk to me now about this, like... So from what I heard, what you guys said, um, because now your customers have two option, right? They have Verizon regular, or they have Verizon Business, which is this, like, super tier, right? Um, and what you want is you want your customers to al- to allow them to kind of step up, right? So if I'm on a regular, I can upgrade myself to go to the Verizon Business, but I cannot step down. You said this is something that they can't, they can't upgrade.

41:29 Devon D'Andrea: No, no, no, no.

41:30 Adam Curcie: They can't-

41:30 Aksana Rahouski: Right?

41:30 Adam Curcie: ... they can't upgrade it themselves-

41:32 Aksana Rahouski: Okay

41:32 Adam Curcie: ... not from the portal.

41:34 Devon D'Andrea: Yeah, I think-

41:34 Adam Curcie: No.

41:35 Aksana Rahouski: Okay.

41:35 Adam Curcie: I, I think that the closest we're gonna come in the foreseeable future to automating it would just be a way for them to request it.

41:43 Aksana Rahouski: Okay.

41:44 Adam Curcie: Like, click of a button, because we kind of ha- there, there's several steps, and it's-

41:50 Devon D'Andrea: Yeah

41:50 Adam Curcie: ... it, it's, and it's very complicated if the device is already in the field.

41:54 Devon D'Andrea: And we might, we might have that with T-Mobile at some point, but definitely not with Verizon.

42:00 Adam Curcie: Yeah. Yeah.

42:00 Devon D'Andrea: Um, so yeah-

42:01 Adam Curcie: It's possible

42:01 Devon D'Andrea: ... so customer cannot go to or from business internet.

42:06 Adam Curcie: No.

42:07 Aksana Rahouski: Okay. So however, you can upgrade them, right?

42:12 Devon D'Andrea: Yes.

42:13 Aksana Rahouski: And w- with that being said, let's say even for a customer, um, l- let's say they're here, right? And they're editing themselves, e- e- they have SIM, but the plan is just, uh-... it's, it's a static info- like, they just get to see it. You are on plan A, you are on plan B. They can come to you, though, and say, "Hey, Adam, Dan, can we move me to the business plan?" So you need a tool, like, you need an option i- in the portal to be able to upgrade them or downgrade them, right? Do you or no- or, like, is it as simple as, like, as an administrator, do you need to have, like... L- later, this is an existing customer, it's an existing device. They've asked me to move them to the business Verizon, right? Meaning, I need to come here, and I need to, uh, c- like, assuming there is here, um, account, regular or business account, and they're regular, and I need to, uh, change that regular to business. With that being said, I probably that means that I need to change their service plan, um, right? Because the upgrading service plan is gonna be different. And again, like, TBD, what else needs to be taken care of, but is that something you need as a capability in the portal?

43:36 Adam Curcie: When you say capability-

43:38 Aksana Rahouski: Mm-hmm

43:39 Adam Curcie: ... you're, like, when I, uh- I'm assuming what you're, you're asking is if we need the portal to do it.

43:51 Aksana Rahouski: To allow. To allow-

43:51 Adam Curcie: And it-

43:52 Aksana Rahouski: -to do it.

43:52 Adam Curcie: So I'll, I'll... Well, well, I mean, I would, I would state, like, what we... Like, the billing, the billing is, like, the biggest implication.

44:02 Aksana Rahouski: Mm-hmm.

44:03 Adam Curcie: Like, I- so, like, 'cause, like, when we, we assign it, right? So then, like, we need the portal to do its assigning process so that it gets billed properly.

44:17 Aksana Rahouski: Yeah. And that-

44:18 Adam Curcie: But we don't really need it to do, like, the actual work to put it onto the Verizon account. Like-

44:26 Devon D'Andrea: Yeah.

44:27 Adam Curcie: So, like, I, I, I-

44:28 Aksana Rahouski: Let me work you through, like, let me walk you through a scenario. Let's say you imported a Verizon de- account, right? And we- on a regular Verizon plan. Um, you imported the device, two months went by, things are running, th- they're getting billed, whatever, business is as usual, right? All of a sudden, there is a need to upgrade this device to a business plan. In your head, what does that look like?

44:57 Devon D'Andrea: Oh, it's, it's, that's something that is not going-

45:00 Adam Curcie: Oh, yeah, it's-

45:01 Devon D'Andrea: -to be automated easily.

45:02 Adam Curcie: It's... Okay, so you wanna know the steps? These are the steps. So I'm going to... First, I gotta- if the box is online, um, that's helpful, but also, um, spare... It just makes it riskier. So we have to log into the device-

45:20 Devon D'Andrea: And it's way too much risk, way too much risk to automate this. I'm just gonna say that.

45:24 Adam Curcie: We have to log into the device, okay?

45:26 Aksana Rahouski: Mm-hmm.

45:27 Adam Curcie: We have to change the APN simultaneously, and I'm literally, I'm not using that word lightly. Sim- all- like, as close to simultaneously as humans can really be.

45:40 Aksana Rahouski: Mm-hmm.

45:41 Adam Curcie: Like, we have to go into Verizon, we have to deactivate the device immediately after clicking the Apply button-

45:48 Aksana Rahouski: Mm-hmm

45:48 Adam Curcie: ... onto the device where the APN information is controlled, and then you have about a two- to three-minute window to get into the other Verizon account, which you should have already logged into on a separate browser, and then-

46:02 Aksana Rahouski: Mm

46:02 Adam Curcie: ... subsequently reactivate the SIM card on the new account.

46:06 Aksana Rahouski: Okay.

46:07 Adam Curcie: If you want this to be even remotely quick.

46:11 Aksana Rahouski: Okay.

46:11 Adam Curcie: Because otherwise, the box is gonna sit there and try to connect for, like, 30 to 45 minutes if you miss your one shot.

46:19 Aksana Rahouski: Okay.

46:19 Adam Curcie: So yeah. So it's-

46:22 Devon D'Andrea: Let's say, yeah, I know-

46:22 Adam Curcie: It's numerous steps-

46:24 Aksana Rahouski: Mm

46:24 Adam Curcie: ... and, and the timing of it is, like, there- y- you can't even, like, y- there's a lot of, as you know, with Verizon, sometimes that deactivation request might take a minute.

46:35 Devon D'Andrea: Yeah.

46:35 Adam Curcie: Sometimes it's not gonna be a minute.

46:38 Aksana Rahouski: Okay.

46:38 Adam Curcie: So i- i- it's, like I said, there's several steps, they have to be executed in a very specific order.

46:45 Aksana Rahouski: Okay.

46:45 Adam Curcie: And yeah.

46:46 Devon D'Andrea: You cannot-

46:47 Adam Curcie: And then the IP address-

46:48 Devon D'Andrea: You-

46:48 Adam Curcie: -will change.

46:49 Devon D'Andrea: You cannot-

46:51 Adam Curcie: Um-

46:51 Devon D'Andrea: ... you cannot activate... The, the b- the BI account will prevent you from activating a SIM card that has not yet been deactivated from the, from our main account.

47:03 Aksana Rahouski: Okay, so-

47:03 Adam Curcie: You also cannot use a SKU activation.

47:06 Devon D'Andrea: No.

47:07 Aksana Rahouski: Is- are we- n- nightmare, right? So with that being said, are we saying that... Well, what is your plan for 15 devices that you already have on this business account? Are they in the portal or no?

47:20 Adam Curcie: Yeah, they're in the portal-

47:21 Devon D'Andrea: Uh-

47:21 Adam Curcie: -with flat rate billing.

47:23 Devon D'Andrea: They're gonna be, they've been one-offs.

47:24 Aksana Rahouski: So how are we gonna handle them? Like, let's say, because these devices now, right-

47:31 Adam Curcie: Well, well-

47:31 Aksana Rahouski: ... will they sit on, uh, whatever, the default one, technically, right? On the-

47:36 Adam Curcie: Yeah.

47:36 Aksana Rahouski: Well, and we will need to upgrade them from default to business.

47:41 Adam Curcie: Mm-hmm.

47:41 Aksana Rahouski: You, uh... Look, how are we gonna do that? How are we gonna do that? Like-

47:47 Adam Curcie: Well, after you guys add the, the, the checkboxes and we build the service plan-

47:52 Aksana Rahouski: Mm-hmm

47:52 Adam Curcie: ... we will just go into these boxes, check that they're- check their box to let the portal know these should be on the FWA plan.

47:59 Devon D'Andrea: We will certainly make sure-

48:00 Adam Curcie: And then we, and then!

48:01 Devon D'Andrea: We'll certainly make sure that-

48:01 Adam Curcie: We'll put the actual identifiers in, because I'm pretty sure-

48:04 Devon D'Andrea: Yeah

48:05 Adam Curcie: ... we don't have real identifiers in there-

48:07 Devon D'Andrea: Mm

48:07 Adam Curcie: ... for the IMEI or the SIMs-

48:09 Devon D'Andrea: So-

48:09 Adam Curcie: -because they're just all the API failures.

48:11 Devon D'Andrea: We've got... Yeah, we've have to, we'd have to have fail-safes in place so that there's no chance that-

48:16 Stone Marballie: Hey, guys, I'm gonna jump off, okay? If you guys need me, just, uh, ping me. I got a doctor's appointment, and I need to meet with another team.

48:21 Aksana Rahouski: Uh-

48:21 Stone Marballie: I-

48:22 Aksana Rahouski: So how did it go, quickly?

48:24 Stone Marballie: Um, Devon posted that, uh, it was just the one command. Like I said, I updated it, and he confirmed that the prices were restored. So-

48:32 Adam Curcie: Yeah

48:33 Stone Marballie: ... um, I don't know if you guys wanna try it again and see?

48:36 Aksana Rahouski: Yeah, well-

48:36 Stone Marballie: But when we looked in the database, there wasn't any artifacts of, like, some reason the change didn't take. You know what I mean?

48:43 Devon D'Andrea: Yeah.

48:44 Stone Marballie: So I don't know why.

48:47 Devon D'Andrea: Okay.

48:48 Stone Marballie: So-

48:48 Devon D'Andrea: All right, thanks, Darren.

48:50 Stone Marballie: Um-

48:53 Aksana Rahouski: Okay, so-

48:53 Adam Curcie: Yeah, thank you.

48:54 Stone Marballie: You guys, you guys, you guys can ping me, um-

48:56 Aksana Rahouski: Yep. We'll, uh, I'll-

48:57 Stone Marballie: I've got a couple of people coming. If anything needs my attention, just let me know. I can always jump back on.

49:01 Aksana Rahouski: All right.

49:01 Devon D'Andrea: I will. Thanks, man.

49:03 Adam Curcie: Thank you. Um-

49:05 Devon D'Andrea: So, so again, we, we have to make sure that when you guys are pushing this work to production, there's no chance that these devices that we already have on the business internet plan are gonna somehow, for some reason, get a configuration. Aside from that, I mean, there's really not m- much to worry about, I don't think.

49:27 Adam Curcie: Yeah.

49:29 Aksana Rahouski: Yeah, it's-

49:29 Adam Curcie: I mean, that's, that's the only thing, uh, yeah, they can't get... can't get a config that would, uh, you know... And, and again, I think if I'm not mis- I, I forget how they're on- actually set up right now for check-ins, if they're checking in at all, 'cause they're not getting configs done-

49:48 Devon D'Andrea: They're trying to check in, but, like, I don't think we- I think we, like, intentionally just left out some identifiers and kept it in our notes just to make sure-

49:58 Adam Curcie: Yeah

49:58 Devon D'Andrea: ... that the portal doesn't push something that it shouldn't.

50:01 Adam Curcie: Yeah. So, um-

50:05 Aksana Rahouski: I know we are, like, almost at time, but I, I, I think we need more time on this, because I think we need to, like, once again, kind of zoom on... We, we need to, like-

50:17 Adam Curcie: I don't know

50:17 Aksana Rahouski: ... understand why we're doing this. We're doing this so you can actually get these 15 devices to this business plan, which wo- at some point when we build this, right?

50:27 Devon D'Andrea: Mm-hmm.

50:27 Aksana Rahouski: Like, what is that gonna look like, right? What does portal... And think through adding-

50:33 Devon D'Andrea: This-

50:33 Aksana Rahouski: ... more devices like that, or perhaps there'll be 15 more that when you need to roll out-

50:38 Adam Curcie: Yeah

50:38 Aksana Rahouski: ... to the tier, right?

50:39 Devon D'Andrea: Right.

50:40 Aksana Rahouski: And we don't wanna-

50:40 Adam Curcie: I'm, I'm probably-

50:41 Aksana Rahouski: Mm

50:42 Adam Curcie: ... I'm probably way out of line with this comment, but I really think one checkbox on the device, wh- and which would correspond to one column on all, all of the applicable spreadsheets, that literally just says FWA, and, and then one checkbox on, on the service plan sheet, and then all the magic behind it. But I, I really think... I think that would get it done. I, I, I don't know how... I, I, I think that would probably suffice. I'm, again, I might really be off base suggesting that, but-

51:12 Devon D'Andrea: Well, yeah, and we have... Well, again, and then there's, and then there's, you know, and then there's, okay, do, you know, pro- uh, permissions-wise, nobody can move anything to or from the business internet service plan. Um-

51:24 Adam Curcie: Yeah, but we have some logic. Like, like, I think that requirement's not difficult to accommodate.

51:29 Devon D'Andrea: I don't know. I'm just saying that, that we can't forget about that. We can't just take-

51:32 Aksana Rahouski: I-

51:32 Devon D'Andrea: ... two checkboxes and call it-

51:33 Aksana Rahouski: Yeah, and again, like-

51:35 Adam Curcie: I said all the magic.

51:37 Aksana Rahouski: You... Nothing you said here is out of line. Listen, like, UI is the cheapest, the cheapest part-

51:45 Adam Curcie: Yeah

51:45 Aksana Rahouski: ... of this. We're saying-

51:46 Devon D'Andrea: Well, I just-

51:47 Aksana Rahouski: The, the, the, is the magic, what's the most expensive thing. That-

51:51 Devon D'Andrea: Yeah

51:51 Aksana Rahouski: ... um, we need to figure out what that magic is, right? When you check that box, what actually-

51:57 Devon D'Andrea: Yeah

51:57 Aksana Rahouski: ... happens, right?

51:58 Devon D'Andrea: Right.

51:58 Aksana Rahouski: And also-

51:59 Devon D'Andrea: Yeah

51:59 Aksana Rahouski: ... the question, when you uncheck that box, what happens, right? Because-

52:04 Devon D'Andrea: Yeah

52:05 Aksana Rahouski: ... it's, there's a lot of magic that goes back and forth, and-

52:09 Devon D'Andrea: A lot of magic

52:09 Aksana Rahouski: ... um, and that's what we need to... And, and then again, like, UI left here is, we can do whatever, right? We can, yes, we can solve it with just checkbox, assuming default. No checkbox checked is the default, checked is business, right? Because we only have two.

52:26 Adam Curcie: Yeah.

52:26 Aksana Rahouski: I, I will say-

52:27 Adam Curcie: Yeah

52:27 Aksana Rahouski: ... that, that solution falls through if there is ever three. Because, right, checkbox is good for one or two, but is never good for one, two, or three. But that, that, that's like, that's the only kind of pro and cons versus checkbox versus no checkbox. It's the, um, it's the kind of what's next thing that makes me worried, and I still think that we're not quite ready to jump in the solution. We need probably another, like, half an hour, um-

52:54 Devon D'Andrea: Let's do another half an hour after the week after Adam, um, gets back, and, meaning the week after next. And if you guys are cool with it right now- ... I'm gonna, I'll go in and add an, a letter-

53:10 Aksana Rahouski: Yes

53:11 Devon D'Andrea: ... to that service plan.

53:13 Aksana Rahouski: Yes, please do it. Uh, do you wanna share, so we can-

53:15 Devon D'Andrea: Uh-huh.

53:16 Aksana Rahouski: I'll watch you, and you can be nervous.

53:20 Devon D'Andrea: All right, so we're back to... That's the wrong one, of course.

53:24 Aksana Rahouski: Yeah.

53:24 Devon D'Andrea: All right, so we've-

53:24 Adam Curcie: I-

53:25 Devon D'Andrea: ... uh, let's just start over here. Browse Service Plans, ATM. We've got just the $495 base, okay? So we're just gonna add one thing here. I'm gonna do Origin T-Mobile 350.

53:40 Aksana Rahouski: Mm-hmm.

53:41 Devon D'Andrea: Adam, approve that, or somebody.

53:45 Adam Curcie: It's not gonna be me, I'm sorry. I'm in the car, but I had to grab my daughter before-

53:48 Devon D'Andrea: He's in the car. You can approve it from the car. I've approved shit from the car.

53:53 Adam Curcie: No, I mean, I, I, I absolutely can.

53:55 Devon D'Andrea: I'm, yeah, I'm jo-

53:56 Adam Curcie: Yeah.

53:56 Devon D'Andrea: I'm joking.

53:57 Aksana Rahouski: We can, we can too. We are, uh... I just need to go to prod.

54:03 Devon D'Andrea: Have I ever told you the story about when we were in Vegas, many drinks in, having to push configs for a customer at a high rate?

54:10 Aksana Rahouski: That's, that's never a good idea.

54:13 Adam Curcie: No, it was like a-

54:14 Aksana Rahouski: I just-

54:14 Adam Curcie: ... it was a literal, like, absolute catastrophe-level emergency, that they needed us to literally stop everything we were doing, in like, at a party, and, like-

54:26 Devon D'Andrea: A rooftop bar

54:26 Adam Curcie: ... many, many drinks in. Yeah, and, and we had to just go to a quiet space and, and sit there and upload a whole bunch of configuration files for like-... customer has, like, 2,000 units, um-

54:39 Devon D'Andrea: Yeah.

54:40 Adam Curcie: And we did it. It was great, but, uh-

54:42 Devon D'Andrea: We did it

54:42 Adam Curcie: ... it was not-

54:43 Aksana Rahouski: Oh, my God. Do you guys want me to approve it? I'm on the page.

54:47 Devon D'Andrea: I hit it. Yeah, hit that.

54:48 Aksana Rahouski: Okay, A-Team Unlimited, approve. Okay, done.

54:54 Devon D'Andrea: Okay, here we go. Approved. So now we wanna test if it broke the custom service plans.

55:02 Aksana Rahouski: Yes.

55:06 Devon D'Andrea: And it did.

55:09 Aksana Rahouski: Okay.

55:10 Devon D'Andrea: This should stay three.

55:12 Aksana Rahouski: Okay.

55:12 Devon D'Andrea: This should stay three.

55:13 Aksana Rahouski: Oh, yeah, that's bad. Yeah, that's, that's the same bug. Okay, so that at least tells us that it- there is, in fact, a bug in the system. Um, when Stone is back, we'll probably... Let me regroup with him when he's back and just to see what are our next steps, whether we'll restore the previous one again, so we kind of keep it clean, and ask you not to touch it for the time being.

55:37 Devon D'Andrea: That's fine. I think you could prob- we could even probably do that if we just hit the little reset button, it prob- actually, no, that- I'm not sure what that would do. Never mind.

55:49 Aksana Rahouski: Okay.

55:49 Devon D'Andrea: I'll let, I'll let him- I' hadn't let you guys... I don't know. I'll- yeah, I'll let you guys-

55:54 Aksana Rahouski: Yeah.

55:54 Devon D'Andrea: I-

55:54 Aksana Rahouski: Well, I'll, I'll, I'll send you, uh, an update at the end of the day-

55:58 Devon D'Andrea: Okay

55:58 Aksana Rahouski: ... as to how far out on this.

56:01 Devon D'Andrea: Uh, Adam, you wanted to bring up one more thing if time permits. Does time permit?

56:07 Adam Curcie: Yeah.

56:08 Aksana Rahouski: I think-

56:08 Adam Curcie: I mean, it's ultimately gonna probably have to be a card that we create, but there was-

56:13 Devon D'Andrea: Oh, yeah, yeah, yeah

56:14 Adam Curcie: ... a, an issue we experienced earlier this... It, it was either the end of last week, or I think it was we were doing all of the updating in our system-

56:24 Aksana Rahouski: Mm-hmm

56:24 Adam Curcie: ... for stuff that was not actually set up as an origin to make it an origin, and we discovered that when we changed about a hundred of them, um, we did not-

56:34 Aksana Rahouski: Mm

56:34 Adam Curcie: ... put the IMEI into the update spreadsheet, and when we went to the devices thereafter in the portal, there was- none of them no longer had IMEIs.

56:45 Devon D'Andrea: I'd say-

56:45 Adam Curcie: So the-

56:46 Devon D'Andrea: ... if we could, I'm sharing my screen, if we could, we'd like to see, um... We'd like to maybe in the card make this kind of a twofold thing. Number one, if you guys could confirm which fields have to be included, um, and which fields can be just left blank, and then, and then ultimately, that would make them stay the same.

57:09 Aksana Rahouski: Mm-hmm.

57:10 Devon D'Andrea: Um, and then, and then after that, we'd have to- we need to decide, okay, well, sh- you know, which ones would we want to tell you, like, "Hey, don't change it if it's left blank?"

57:21 Aksana Rahouski: Oh, so you're saying when you are updating devices with the inputs-

57:26 Devon D'Andrea: Yes

57:26 Aksana Rahouski: ... it overrides that-

57:28 Devon D'Andrea: There's, there's some, some of these fields, some of these fields do not require- do not require entries, and if they're left blank, they are-

57:35 Aksana Rahouski: Yeah

57:35 Devon D'Andrea: ... modified.

57:36 Aksana Rahouski: Yep. Yep.

57:37 Devon D'Andrea: But there's some of them that are, if they're left blank, they are changed to null.

57:44 Adam Curcie: Yeah, like-

57:45 Aksana Rahouski: So-

57:45 Adam Curcie: ... we were anticipating if we, if we put a serial number in, basically, and we only wanna change, like, one thing-

57:54 Devon D'Andrea: Yeah

57:54 Adam Curcie: ... like, a- and we're not entering every other piece of information for that device-

57:58 Devon D'Andrea: And ultimately, ultimately-

57:59 Adam Curcie: The other, it shouldn't, it shouldn't get changed

58:01 Devon D'Andrea: ... Yeah, ultimately, if we can make it so that serial number is the only, um, uh, um-

58:07 Adam Curcie: Then-

58:07 Devon D'Andrea: ... what do you call it? Um, primary, a primary key, if you will. Um-

58:12 Adam Curcie: Yeah-

58:12 Devon D'Andrea: Nothing else should change

58:12 Adam Curcie: ... like a key identifier. Yeah.

58:14 Devon D'Andrea: Nothing else should change unless actually put in here.

58:21 Aksana Rahouski: And that... Okay, so just quickly-

58:23 Devon D'Andrea: Well, we gotta make a ticket for that.

58:24 Aksana Rahouski: Quick question. So we're talking about device update, right?

58:27 Devon D'Andrea: Update devices.

58:28 Aksana Rahouski: Okay.

58:28 Devon D'Andrea: Update devices. This page right here, update.

58:32 Aksana Rahouski: Uh, let me ask kind of this rule of thumb question, true or false: Is it- d- do you ever intend to actually set null values? Because every time you pass nothing, it's a null value, and I'm gonna guess right now, it's just like, if you gave me device one, two, three, and- ... 10 out of n- 11 fields are empty-

58:53 Devon D'Andrea: Yeah

58:53 Aksana Rahouski: ... I'm gonna-

58:54 Devon D'Andrea: Yeah

58:54 Aksana Rahouski: ... Yeah. Or do we always want a system to kind of, what's empty, keep the old value for the empty, but what, what has the value-

59:03 Devon D'Andrea: Well-

59:03 Aksana Rahouski: ... reset?

59:03 Devon D'Andrea: So-

59:03 Adam Curcie: See, that's why, see, that's why, yeah-

59:05 Devon D'Andrea: So, yeah

59:05 Adam Curcie: ... And then there's, and then there's the other flip of the coin.

59:09 Devon D'Andrea: I didn't think about that.

59:09 Adam Curcie: Well, no, no, no, Devin, we did, remember? And, a- and the other kind of half of the equation or the other school of thought would be on this screen-

59:18 Aksana Rahouski: Mm-hmm

59:18 Adam Curcie: ... where you give us the ability-

59:19 Devon D'Andrea: Right

59:19 Adam Curcie: ... to download the template-

59:22 Aksana Rahouski: Yep

59:22 Adam Curcie: ... could we have a text field? So if I put in 135 serial numbers-

59:28 Aksana Rahouski: Mm-hmm

59:28 Adam Curcie: ... could you give me the template for, with all of the pre-existing data for those 135?

59:35 Devon D'Andrea: Wait a minute.

59:35 Aksana Rahouski: You-

59:35 Devon D'Andrea: Wait a minute, wait a minute, wait a minute, wait.

59:36 Aksana Rahouski: Well, you mean you wanna download-

59:38 Devon D'Andrea: It's-

59:38 Aksana Rahouski: ... you wanna be like, "Give me this five devices, this set of data." That's what you want.

59:45 Devon D'Andrea: No.

59:45 Adam Curcie: Well, all of the-

59:46 Devon D'Andrea: That's a waste of money.

59:47 Adam Curcie: What do you mean? A waste of money?!

59:49 Devon D'Andrea: ... I thought we talked about, I thought we talked about we would choose the fields that we wanted to update. 'Cause then you could all- then you could choose to choose null. To answer Oksana's question, which I didn't think about, which is very true, is like-

01:00:05 Adam Curcie: Yeah.

01:00:05 Devon D'Andrea: ... yeah, you shouldn't make it so that, you know, blank is not touched.

01:00:10 Aksana Rahouski: But-

01:00:10 Devon D'Andrea: You should have the option to do-

01:00:12 Aksana Rahouski: Not always bad, right? Blank is sometimes is intentional.

01:00:16 Devon D'Andrea: Right, and that's certainly in the pa- if, if history teaches us anything, and Adam, with the shit we went through with Miles and whatnot, it's not a bad thought. So yeah, we'll have to put a card in and really think through this. Like-

01:00:31 Aksana Rahouski: Yeah. Why don't you guys first kind of, again, discuss it internally-

01:00:37 Devon D'Andrea: Right

01:00:37 Aksana Rahouski: ... create a call and just like-

01:00:38 Adam Curcie: But that's not gonna help. I can't even remember all of the conversations me and Devin had just about this so far!

01:00:45 Aksana Rahouski: We can-

01:00:46 Adam Curcie: ... He's, he's thinking of something completely different than what I remember.

01:00:50 Devon D'Andrea: Well, no, I just-

01:00:51 Adam Curcie: Oh.

01:00:51 Devon D'Andrea: Well, no, no, I, but, but the, you're putting like-

01:00:53 Adam Curcie: I thought we- I thought that was the... All right, never mind.

01:00:54 Devon D'Andrea: Well, but just, just being able to find what's already in the database, like, you don't need them to build something here. That's what a freshie's for.

01:01:04 Adam Curcie: Fine.

01:01:05 Aksana Rahouski: I mean, you can go to a device detail page and find all that information, right? It's like-

01:01:09 Devon D'Andrea: Well, I'm just saying if you've got, like, 100 and you wanna just-

01:01:12 Aksana Rahouski: Or, or-

01:01:12 Devon D'Andrea: ... like, put it all in

01:01:14 Aksana Rahouski: ... up to you. Yeah, we can do whatever. I just, like, we just need- -to make sure that we know what it is that we're doing, and yeah, we-

01:01:23 Devon D'Andrea: I tell that to, I tell that to Rick, like, at least once a month, our owner, Rick. He's like, "Do you think horses could do this?" I'm like, "Rick-

01:01:29 Aksana Rahouski: They can do it

01:01:30 Devon D'Andrea: ... do we have, do we have money in the bank?"

01:01:33 Adam Curcie: Well, well, hold on. So if we want- is there a way to code in a null value that's not a null value by default, like the word "null"? Like, if... Could we create-

01:01:44 Aksana Rahouski: Yeah

01:01:44 Adam Curcie: ... could we modify the, the way that it pulls the data in so that if it actually... Like, if we want to change a whole bunch of stuff to not have an ICC ID that's in there now-

01:01:53 Aksana Rahouski: Mm-hmm

01:01:54 Adam Curcie: ... we would, that we would just literally type "null" for that whole column?

01:01:58 Aksana Rahouski: I would say don't, don't, don't think about it from a perspective of, how do I, how do I kinda push this thing in if it, if it doesn't fit? Like, tell me, w- what is your desired behavior? Like, do you-

01:02:15 Adam Curcie: Well, yeah-

01:02:16 Aksana Rahouski: So my-

01:02:16 Adam Curcie: ... my desired behavior is to only modify the stuff I put in there- ... 'cause it's a, it's a, it's a, it's a, you know, it's a-

01:02:23 Aksana Rahouski: And that's-

01:02:23 Adam Curcie: In my mind, it's a modification.

01:02:25 Devon D'Andrea: Well, yeah, yeah, that, I think that-

01:02:26 Adam Curcie: It's a value

01:02:26 Devon D'Andrea: ... I think that would be the best. Like-

01:02:28 Aksana Rahouski: Yes

01:02:28 Devon D'Andrea: ... only serial number, and then anything else, if there's a value, then if there's a value, then only change that, unless it all... unless it says null, and that means make it blank.

01:02:41 Aksana Rahouski: And, and again, we can, yeah, we can come up with a rule where, you know, if you are importing something in, let's say, 12 fields, only two fields have values. Whatever doesn't have a value mean we retain the old value. We just update the values that you said.

01:02:59 Devon D'Andrea: Yeah.

01:02:59 Aksana Rahouski: We can come up with a registered word, let's say, "set empty" or-

01:03:05 Devon D'Andrea: Right

01:03:05 Aksana Rahouski: ... "null," whatever, right? That tells us if you actually pass that in that column, that means we reset that value to nothing.

01:03:13 Devon D'Andrea: Right. Right, right, right, right, right.

01:03:15 Aksana Rahouski: That we can help.

01:03:15 Devon D'Andrea: And I gotta say, yeah, that, that... Yeah, Adam, I think they can accomplish that. Listen, Adam-

01:03:20 Aksana Rahouski: Yes

01:03:20 Devon D'Andrea: ... thought you guys were gonna take probably, like, two weeks to get us that export failed check-in, uh, spreadsheet, and I'm pretty sure-

01:03:28 Adam Curcie: I did not

01:03:29 Devon D'Andrea: ... I'm pretty sure we thought we g- we gave you the final requirements on a Friday afternoon and got it Monday morning. So I think-

01:03:38 Adam Curcie: I didn't say two weeks. You're putting words in my mouth, man.

01:03:43 Aksana Rahouski: Yeah.

01:03:43 Devon D'Andrea: Uh, that was remarkable how you guys were able to-

01:03:46 Adam Curcie: It was really fast, though-

01:03:47 Devon D'Andrea: Yeah

01:03:47 Adam Curcie: ... I have to say. We w- we were both com- like, when we saw the, we're like, "What?"

01:03:52 Devon D'Andrea: Yeah.

01:03:54 Aksana Rahouski: That's it.

01:03:54 Devon D'Andrea: You gotta be, uh-

01:03:55 Aksana Rahouski: That's the exports. So Noah and Aaron worked on that one.

01:03:58 Devon D'Andrea: Good job, Aaron.

01:04:00 Adam Curcie: Yes.

01:04:00 Devon D'Andrea: Noah is not here. Noah's not here, so I'll give you all the credit.

01:04:04 Aksana Rahouski: Mm-hmm. Okay, so okay, I st- like, still create a card, but I'm-

01:04:09 Devon D'Andrea: We're gonna create a card, and we're gonna-

01:04:11 Aksana Rahouski: Okay

01:04:11 Devon D'Andrea: ... we're gonna confirm dates for our, for our virtual meeting with In Hand, and we're gonna try to pick a time that following week as well, where we can get at least 30 minutes to finalize the BI conversation.

01:04:22 Aksana Rahouski: Yes, perfect.

01:04:23 Devon D'Andrea: And then we'll figure out what we're doing with this service plan, uh, current service plan part.

01:04:31 Aksana Rahouski: Okay, that's our top priority, yes. Okay, sounds good.

01:04:35 Adam Curcie: All right. Only 15 minutes passed.

01:04:38 Devon D'Andrea: Get yourself a cup of, get yourself a cup of hot tea or something, Oksana.

01:04:40 Aksana Rahouski: I know. I'm like-

01:04:41 Adam Curcie: Yeah

01:04:42 Aksana Rahouski: ... I-

01:04:42 Adam Curcie: Get, get some good-

01:04:43 Aksana Rahouski: I will count my thinking last much-

01:04:44 Adam Curcie: ... rest

01:04:44 Aksana Rahouski: ... I don't have to talk much.

01:04:46 Devon D'Andrea: Yeah, sorry.

01:04:47 Adam Curcie: Get good rest this weekend.

01:04:48 Devon D'Andrea: Yeah. All right, thanks, everybody.

01:04:50 Aksana Rahouski: Bye.

01:04:51 Adam Curcie: Thank you.