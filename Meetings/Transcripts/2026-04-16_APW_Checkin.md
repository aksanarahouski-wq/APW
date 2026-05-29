# APW Check-in — Meeting Transcript

**Date:** April 16, 2026  
**Time:** 10:00 AM ET (2:00 PM UTC)  
**Duration:** ~66 minutes  
**Organizer:** Aksana Rahouski

## Participants

- **Aksana Rahouski** — Senior Product Manager / BA (Orases)
- **Laura Perry** — Project Manager (Orases)
- **Noah Bratzel** — Developer (Orases)
- **Richard Sacco** — Developer (Orases)
- **Stone Marballie** — Developer (Orases)
- **Aaron Diefes** — Developer (Orases)
- **Adam Curcie** — Technical Lead (APW / Allpoint Wireless)
- **Devon D'Andrea** — Operations (APW / Allpoint Wireless)

---

## Transcript

**[00:00:00] Aksana Rahouski:** I'll share, and I should have shared it ahead of time with you, but we'll do better next time. Um, a few things before we kinda really just... And whatever else you guys have, uh, configuration, we'll talk about at the end or most of the spread. I brought Noah here in this meeting 'cause I know you guys, we mentioned to you many times, Noah obviously joined the project kind of a while ago now, I feel like, but we've never actually met him, so this is my ride that Noah's doing all the hard lifting.

**[00:00:33] Adam Curcie:** We, we've met.

**[00:00:34] Aksana Rahouski:** And-

**[00:00:35] Adam Curcie:** We, we've met at least once. I think.

**[00:00:36] Aksana Rahouski:** Oh, you have then?

**[00:00:37] Adam Curcie:** Yes.

**[00:00:37] Aksana Rahouski:** Gotcha. Okay. So yeah. So just like, again, yeah, kind of bring him in too so he could show his face maybe, or jump in and add whatever you wanna add so everybody knows who you are.

**[00:00:52] Noah Bratzel:** Hi, I'm Noah. Yeah, you... We, we've met or I've been introduced on this project before, uh, quite a few years ago when the first time I, I worked on it for a little bit and so.

**[00:01:01] Adam Curcie:** Yes.

**[00:01:03] Noah Bratzel:** That's been-

**[00:01:03] Aksana Rahouski:** Um-

**[00:01:03] Noah Bratzel:** That's been at least two or three years, so I don't remember how long that's been, but yeah.

**[00:01:09] Adam Curcie:** It's pretty crazy how, uh, how far we've come, huh?

**[00:01:13] Noah Bratzel:** It's a little different. It's a little different. A lot of dif... Lot of, lot more devices in production, that's for sure.

**[00:01:18] Adam Curcie:** Yeah. Yeah.

**[00:01:21] Aksana Rahouski:** Yeah. I know-

**[00:01:22] Adam Curcie:** All right.

**[00:01:22] Aksana Rahouski:** Noah's been working a lot.

**[00:01:24] Adam Curcie:** Six fifty.

**[00:01:28] Noah Bratzel:** I missed that.

**[00:01:28] Aksana Rahouski:** Yeah. So, um... Yeah. So Noah, I mean, you know Noah been probably for two months already, so he's been working a lot of things. Currently, he is actually just to kinda start... He's, he's leading our Cake update and he started, um, you guys did kinda last time in the sponsor update, we talked about how we wanna kinda break it into three releases. No-Noah's working on one of them currently with Richard together. Um, and actually Verizon second account feature is something that, uh, Noah developed as well. Um, it's right now with Aaron for testing. Okay. So that out of the way, quick update, kind of work. Um, I broke it into bugs and features. So kind of tell you where all the bugs are. Um, 2070, so that's the one that you guys submitted by email that kind of fall out of 1939 with, uh, SIM card activation failure. Um, I think Stone just jumped on it literally yesterday, so in progress. Um, okay, what are these ones? Okay, you guys feel free to... Aaron, I know you have a few that you are testing or about to test. I know this one that 1953, so that's the one that you guys reported on the... Let's go, 1953. This guy. Um, the command that runs every billing cycle and take devices that are inactive and turn off, remove them from the billing cycle. And you noticed discrepancy between devices were pulled where SIMs were actually not deactivated. So, um, Stone worked on this bug, um, um, and Stone you did like, we did like a query update, uh, that to make sure that the job actually validate not only device status but also SIM status. Um, I know Stone ran a report on production to make sure that there are no other devices like the one you guys kind of submitted as an ex- as a sample, and we didn't find any, so that's good news. Uh, so now this is something that needs to be tested. Stone, the only question I had for you for this one, because I know we have it sitting in QA that, um, when we do the check, right, uh, and we pull devi- it, whether or not a device is deactivated and SIM is deactivated before we pull it out of the billing cycle, what do we do if i- if there is a discrepancy? Did we ca- like are we-- how are we, um, routing there?

**[00:04:12] Stone Marballie:** Sorry, I was on mute. So I created a new report that'll come to, you know, the APW admins that show the list of what-

**[00:04:20] Aksana Rahouski:** Mm-hmm

**[00:04:20] Stone Marballie:** ... is being pulled out, what, um, what came up as flagged that couldn't be verified to look over. So it'll be a detailed thing of, um, actionable items.

**[00:04:32] Aksana Rahouski:** Okay. So these are... Okay. So we'll catch these and email them to-

**[00:04:36] Stone Marballie:** Correct. Yeah. It'll-

**[00:04:37] Aksana Rahouski:** ... admins so they know. Okay. And then, and then obviously fixing that, um, deactivation like or, um, cur- billing cycle will only pull devices that are deactivated, uh, for, um, SIMs are de-deactivated as well. Um, okay. So I wanted to, uh, to ask you guys 'cause we have a lot of stuff in QA right now and, and so one of the options, right, for f-for example, for something like this, so we could just give it to you to beta to test, um, because we're like really... Aaron is sitting on a pile of work testing right now, which means 1953, it's, it's gonna take some time for him to catch up with everything. It's a mess. Or we could give it to you to be in beta and you guys could test. Just an idea. What do you think?

**[00:05:30] Adam Curcie:** Uh, I don't have an objection.

**[00:05:37] Aksana Rahouski:** Okay.

**[00:05:39] Adam Curcie:** I don't-

**[00:05:39] Noah Bratzel:** Yeah, sure. I mean...

**[00:05:44] Adam Curcie:** Okay. Yeah. Uh, we'll, we'll do that.

**[00:05:46] Aksana Rahouski:** And also if it's... Yeah. And also again, just to like if it's like ur-urgent or I guess we're just starting a new billing cycle, so it's not probably as pressing, right?

**[00:05:59] Adam Curcie:** Correct.

**[00:06:01] Aksana Rahouski:** OkayUm, okay, so commissions, I know you guys on the go approved that one, ready to go. Richard, is that something that we could push today or no?

**[00:06:11] Richard Sacco:** Yeah.

**[00:06:11] Aksana Rahouski:** Or no?

**[00:06:12] Richard Sacco:** Yeah.

**[00:06:12] Aksana Rahouski:** I don't know if that's in hotfix.

**[00:06:15] Richard Sacco:** Yeah, it's a hotfix. It can be pushed.

**[00:06:17] Aksana Rahouski:** All right. All right. Okay, sounds good.

**[00:06:21] Adam Curcie:** Yeah, we're good. We're good with the, uh, the numbers that, that Richard pro-uh, provided. We'll just, we'll just give that... We'll just add that, tack that on the next month.

**[00:06:32] Aksana Rahouski:** Mm-hmm. Okay. Sounds good. And then, so then a quick update on the features. So like I said, Noah's working and Richard are... Started already CakePHP, that release one and two. Verizon second account is currently actually with Aaron for testing. I know he started yesterday. Um, Aaron, I know you pinged me, you said that there's, uh, probably, like, a few day of testing because of all the different, like, routing, two type of devices, two type-

**[00:07:00] Richard Sacco:** Yes

**[00:07:00] Aksana Rahouski:** ... of service plans and such. Okay.

**[00:07:03] Richard Sacco:** Yep.

**[00:07:04] Aksana Rahouski:** Okay. Um, company invitations, I believe there is some feedback that Aaron found that is, um, with Stone to jump on, um, after he is, um, he-he's gone through all the bugs. Um, okay, so one thing I wanted to get this, you'll see this, so d- our teams, so developers, while they're working on the code, and if they find kinda along the way some improvement opportunity, they will create, like... Or, like, bugs that are basically found by us, not by you. We'll throw them into this tag depth feature. So some... And, and if we have, like, something critical, we'll, like, pull in, like, a ticket, um, and work on it. Just to, like, heads up for, for your visibility.

**[00:07:50] Richard Sacco:** Okay.

**[00:07:50] Aksana Rahouski:** Any thoughts or comments on that?

**[00:07:58] Richard Sacco:** Yeah.

**[00:07:58] Aksana Rahouski:** Okay. Um, and I think... So I think that's as far as kinda update this is. Anything that you guys think have that we didn't cover or any questions you have regarding the work that is live right now?

**[00:08:11] Adam Curcie:** Yeah. I, I have a general question just about that SIM card activation ticket, because I kind of realized-

**[00:08:19] Aksana Rahouski:** Yep

**[00:08:20] Adam Curcie:** ... after I put it in that I probably didn't explain it properly. If you go to-

**[00:08:28] Aksana Rahouski:** Mm-hmm

**[00:08:28] Adam Curcie:** ... like, the logs in our portal and put that SIM card in, um-

**[00:08:34] Aksana Rahouski:** Mm

**[00:08:35] Adam Curcie:** ... I just wanna, like, get clarification on what I saw, unless you already know what I'm gonna mention. I can just give you the different-

**[00:08:45] Aksana Rahouski:** Um, I don't know.

**[00:08:47] Adam Curcie:** What's that?

**[00:08:50] Aksana Rahouski:** I would say it would be s-super helpful because I know Stone just jumped on it. So al-always kinda keep in mind, the more clarity you give us, the less time we spend digging through what does it mean, right?

**[00:09:02] Adam Curcie:** Yeah.

**[00:09:02] Aksana Rahouski:** So-

**[00:09:03] Adam Curcie:** No, no, and that's-

**[00:09:03] Aksana Rahouski:** Um, do you-

**[00:09:04] Adam Curcie:** I, I can just quickly show you. Just give me a second. I gotta-

**[00:09:10] Aksana Rahouski:** Do you wanna... Yeah, do you wanna share, Adam?

**[00:09:12] Adam Curcie:** Yeah. Um, sorry. Give me one sec. I got a million things to deal with. Uh, logs. All right, so company's gonna be in our quarter. Gonna be system logs. Yeah, okay. Okay, okay. So, uh, window, it's gonna be this guy. So all of these still today are just logging all these device API failures for every single SIM assigned to them, and for, like, the majority of these, I'm... I-I... We, we just recently assigned them 200 SIMs, um, but we assigned them as deactivated. So I don't know if this is one of them, but I'll change the way I'm searching in a second if it's not.

**[00:10:23] Aksana Rahouski:** Mm-hmm.

**[00:10:24] Richard Sacco:** They're deactivated-

**[00:10:25] Adam Curcie:** Yeah. So-

**[00:10:26] Richard Sacco:** They're all... They also don't exist-

**[00:10:28] Adam Curcie:** If they are-

**[00:10:28] Richard Sacco:** ... in Verizon.

**[00:10:30] Adam Curcie:** Yeah. And, and that's why you... I'm-why they're, the API, like, I don't know why Allpoint is sending API calls for a deactivated SIM card in general. So, but the, the activation did fail, and it was o-and it was one of those instances where I couldn't, for the ones that were active, uh, in your card. Get this out of here. Uh, uh, active. For any of these that we did try to activate, we tried to do it through the bulk, um, but there was no, there was no activation attempt in here. These, see, these ones just stopped failing after I went in and activated them b-but it, the system didn't actually activate them, or even what I, I couldn't even find if it tried to activate them. So, 'cause, like, right here, changed from deactivated to active, but there was no API to do that, that occurred. Does that make sense?

**[00:11:52] Richard Sacco:** When you said, I, I did not understand the very beginning-

**[00:11:56] Adam Curcie:** Sure

**[00:11:56] Richard Sacco:** ... when you said they started as deactivated.

**[00:11:58] Adam Curcie:** Yeah.

**[00:11:59] Richard Sacco:** What do you mean by that?

**[00:12:00] Adam Curcie:** Oh, the, I mean, they are... None of the SIMs that we sold, that when we sent and assigned these, they were not active on Verizon.So they're gonna be-

**[00:12:13] Richard Sacco:** I-

**[00:12:13] Adam Curcie:** ... they're gonna be coming in and turning them on as they need them. 'Cause SIM cards are like $3, so they just bought 200.

**[00:12:21] Richard Sacco:** Mm.

**[00:12:21] Adam Curcie:** Just 'cause it's like... And y- you gotta remember, it's a credit card size piece of plastic. So the, to ship s- two or to ship 2,000, it's the same amount of money. S- it's like $15 to ship 'em.

**[00:12:32] Richard Sacco:** Okay.

**[00:12:32] Adam Curcie:** And they're only three bucks, so they bought 200 SIM cards, and we sent them all out to them deactivated, and they're just gonna come in here and do the bulk action and drop the ones that they need in to activate as they need them activated. But it wasn't working as we thought it might yesterday, so that's why I put the ticket in. And again, I, I know I didn't really provide enough context. I was kind of busy with a bunch of stuff-

**[00:12:59] Richard Sacco:** Sure

**[00:12:59] Adam Curcie:** ... so just wanted to go over it.

**[00:13:01] Richard Sacco:** And maybe I just need to do a little refresher myself, but when you do assign them initially, aren't they... isn't an activation attempt supposed to be attempted on Verizon SIMs? Or am I not...

**[00:13:17] Adam Curcie:** Well-

**[00:13:17] Devon D'Andrea:** We, we, yeah, we're not, we don't expect that.

**[00:13:21] Richard Sacco:** Oh, okay.

**[00:13:22] Adam Curcie:** Well, I, I mean, for SIMs I guess it's different, right?

**[00:13:27] Richard Sacco:** Okay. For, because this is a SIM.

**[00:13:28] Adam Curcie:** I don't know.

**[00:13:29] Richard Sacco:** It's not, it's not a- another...

**[00:13:32] Adam Curcie:** Like when we-

**[00:13:34] Devon D'Andrea:** Well, it depends

**[00:13:34] Adam Curcie:** ... when we assign regular devices-

**[00:13:36] Richard Sacco:** I've been, I've been under the impression this whole time that assigning does not do anything to status.

**[00:13:42] Adam Curcie:** Yeah. Yeah, it, yeah. But I guess if it was checking, I... 'Cause I think when we originally assigned them, they, the portal did indicate that they were active or, or, or was under the assumption that they were active, right? So then we came in and then did we have to change them to deactivated? Devin, do you know?

**[00:14:12] Noah Bratzel:** But we're saying-

**[00:14:16] Devon D'Andrea:** I don't... Hold on.

**[00:14:20] Adam Curcie:** Let me grab one that's... Oh, this. See, 'cause it says it was modified from deactivated to active. So if we did the assign... Excuse me. If we did the assignment here, delayed billing, payment method, this doesn't say if it chose to be active or deactivated, so, but it was deactivated. I just don't know. I can't remember. I think we had to set it to deactivated. I just don't see a log of it.

**[00:14:56] Devon D'Andrea:** No, it is. Vince pre- set it to deactivated before he assigned it to Intercard. That's how we do it.

**[00:15:02] Adam Curcie:** Yeah, I know. But it's not in here as... It doesn't say that, does it?

**[00:15:08] Devon D'Andrea:** Yes, it does.

**[00:15:09] Adam Curcie:** It does. I'm not seeing it. I'm sorry.

**[00:15:12] Devon D'Andrea:** The device modification for-

**[00:15:14] Richard Sacco:** Mm

**[00:15:16] Devon D'Andrea:** ... uh, for April 10th, 2:13 PM.

**[00:15:21] Richard Sacco:** Okay, so-

**[00:15:22] Devon D'Andrea:** That part.

**[00:15:23] Adam Curcie:** Yeah, but it, where does it say it set it from active?

**[00:15:26] Devon D'Andrea:** What device are you looking at? 8366?

**[00:15:31] Adam Curcie:** Because up here it says it set it from deactivated to active, so it was deactivated. I just don't... Yeah. But I think you're right.

**[00:15:41] Devon D'Andrea:** Mm-hmm.

**[00:15:41] Adam Curcie:** Our process is we set them all to deactivated, then assign them.

**[00:15:46] Devon D'Andrea:** Are you looking at all logs? Yeah.

**[00:15:49] Adam Curcie:** Yeah.

**[00:15:50] Devon D'Andrea:** I... Hmm. Did Vince perhaps-

**[00:15:54] Adam Curcie:** I have logs

**[00:15:54] Devon D'Andrea:** ... did Vince perhaps not deactivate, set the, one of the, one of the hundred sets to deactivated?

**[00:16:01] Adam Curcie:** No, 'cause look, it was status changed from deactivated.

**[00:16:04] Devon D'Andrea:** Is the w- where's Vince's... I'm looking at another SIM.

**[00:16:07] Adam Curcie:** Actually, hold on just a second.

**[00:16:08] Devon D'Andrea:** I'm looking at another SIM. You take this SIM, um, uh, type in, um, 8-9

**[00:16:17] Adam Curcie:** Yeah, just gimme the last, uh, number

**[00:16:19] Devon D'Andrea:** ... uh, 8914800000, uh, 7137608358. What the hell?

**[00:16:38] Adam Curcie:** Am I not allowed to see it? Is that... Y- can you see it? All right. Well-

**[00:16:49] Devon D'Andrea:** Do you, any... Wait, scroll back up. Take off Intercard, that's your problem. You got Intercard in there. It'd be, it happens before it goes to-

**[00:17:01] Adam Curcie:** Ah. Oh, that's right. I see. Yes, yes. Okay. That makes sense. Oh. Yes. Uh, right there. Service plan set to null. Yeah. Okay. So that's the workflow. We set them to deactivated, then Vince assigns them. Status changed from active to deactivated. IP change null to null. So-

**[00:17:33] Devon D'Andrea:** So they, so all those, all of those, all of those API failures yesterday that we got notifications for were when you tried to activate them the first time?

**[00:17:42] Adam Curcie:** No.

**[00:17:42] Devon D'Andrea:** When, when-

**[00:17:43] Adam Curcie:** When-

**[00:17:43] Devon D'Andrea:** ... Riley tried to activate them at 12:09, nothing happened.

**[00:17:48] Adam Curcie:** Yeah. But these happened every, like, two hours basically-

**[00:17:50] Devon D'Andrea:** Well, those-

**[00:17:51] Adam Curcie:** ... from the moment-

**[00:17:51] Devon D'Andrea:** ... those, yeah.

**[00:17:52] Adam Curcie:** Yeah.

**[00:17:53] Devon D'Andrea:** I'm talking about yesterday when the customer-

**[00:17:55] Adam Curcie:** Yeah

**[00:17:55] Devon D'Andrea:** ... actually called to turn 10 on. He, he-

**[00:17:57] Adam Curcie:** Yeah, I don't think anything happened

**[00:17:59] Devon D'Andrea:** ... nothing happened in Verizon and nothing, we didn't get any API failure.

**[00:18:04] Adam Curcie:** Yeah. So that was the full extent of what we observed yesterday. And again, I apologize, I didn't take the time to put all of this into the ticket, but, um, I just wanted to make sure we brought it to your attention. So-

**[00:18:19] Devon D'Andrea:** And so just to be clear on exactly what happened. Again, sorry if I'm bothering you, but the- initially you set them to deactivated.

**[00:18:28] Adam Curcie:** Yeah.

**[00:18:28] Devon D'Andrea:** Then you assigned them, and then those, the company went into bulk actions and attempted to activate them via that.

**[00:18:35] Adam Curcie:** Yeah.

**[00:18:35] Devon D'Andrea:** That's where you did not get anything. Okay.

**[00:18:38] Adam Curcie:** Yeah. And I would, I would state that if stuff is in a deactivated state, unless you can tell me why-

**[00:18:48] Devon D'Andrea:** I see the problem.

**[00:18:50] Adam Curcie:** You see the problem.

**[00:18:51] Devon D'Andrea:** Mm-hmm.

**[00:18:52] Adam Curcie:** How do you see the problem?

**[00:18:55] Devon D'Andrea:** I see the problem. Give me one minute.

**[00:18:57] Adam Curcie:** All right.

**[00:18:59] Devon D'Andrea:** Give me 10 seconds.

**[00:19:01] Adam Curcie:** Nine, eight.

**[00:19:06] Devon D'Andrea:** Are you sharing still?

**[00:19:06] Aksana Rahouski:** It's like who's

**[00:19:07] Adam Curcie:** No, I'm not sharing

**[00:19:09] Aksana Rahouski:** ...

**[00:19:10] Devon D'Andrea:** Whoever imported, whoever imported these SIMs messed up.

**[00:19:14] Adam Curcie:** Oh, okay. Well that wasn't me, so I don't care.

**[00:19:18] Devon D'Andrea:** Then I'm gonna temp administer you.

**[00:19:19] Aksana Rahouski:** Not my problem.

**[00:19:20] Adam Curcie:** No, exactly. Oh, was there a space?

**[00:19:31] Devon D'Andrea:** No.

**[00:19:32] Adam Curcie:** Or just the fact that it's inactive?

**[00:19:34] Devon D'Andrea:** Yes. Should not say that.

**[00:19:37] Adam Curcie:** Ah, you're right. You are correct. That is probably the reason then.

**[00:19:42] Devon D'Andrea:** There was no yes to, there was no yes in the column for active SIM card.

**[00:19:47] Adam Curcie:** Ah, you know, I'm gonna have to smack somebody. All right.

**[00:19:51] Aksana Rahouski:** Did you mean to import them as active though?

**[00:19:54] Adam Curcie:** Well, the SIM flag has to be active so that the system knows that this device like has-

**[00:20:01] Devon D'Andrea:** Is using a Verizon SIM.

**[00:20:02] Adam Curcie:** Yeah.

**[00:20:03] Devon D'Andrea:** Yeah. And w- a- a- and like, again, it brings us back to the conversation where we're, we're gonna need to change the wording on this little thing here because it, it, it has confused our customers. It's-

**[00:20:14] Adam Curcie:** It needs to stay enabled

**[00:20:16] Devon D'Andrea:** Yeah. It's confused our, my coworkers, it's confused all of us.

**[00:20:20] Adam Curcie:** It confused us.

**[00:20:22] Devon D'Andrea:** This doesn't mean like this, this doesn't have anything to do with the status up here.

**[00:20:27] Adam Curcie:** Yeah. It needs to just-

**[00:20:27] Devon D'Andrea:** It just means-

**[00:20:28] Adam Curcie:** ... stay enabled or disabled because-

**[00:20:30] Devon D'Andrea:** Yeah

**[00:20:30] Adam Curcie:** ... the term active and inactive adds more of a weight when we're talking about carrier status than, than what-

**[00:20:38] Devon D'Andrea:** Yeah

**[00:20:38] Adam Curcie:** ... it's actually representing there. So it gets, yeah, it gets confusing. So we gotta just change-

**[00:20:44] Devon D'Andrea:** So-

**[00:20:44] Adam Curcie:** ... that to enable or disable.

**[00:20:46] Devon D'Andrea:** So what we need to do is we need to figure out, we need to s-

**[00:20:52] Adam Curcie:** Well, we gotta just do an update. We gotta do an update.

**[00:20:55] Devon D'Andrea:** We have to update these.

**[00:20:56] Adam Curcie:** A bulk update.

**[00:20:56] Devon D'Andrea:** And then, and then we need to, and then we need to do a bulk update on some test SIMs to make sure it works properly and then we can push this out.

**[00:21:03] Adam Curcie:** I have 65 more that we have to get ready for them, so we'll test it. Don't worry about that.

**[00:21:08] Devon D'Andrea:** There you go.

**[00:21:09] Adam Curcie:** Yeah.

**[00:21:10] Devon D'Andrea:** Now the other ticket, as far as like adding in the additional-

**[00:21:14] Adam Curcie:** I'm sorry, I gotta email him about that

**[00:21:15] Devon D'Andrea:** ... the different service plan, uh, activation, that's a whole nother ticket, right?

**[00:21:21] Adam Curcie:** You asking me?

**[00:21:23] Devon D'Andrea:** Did you put that in? I think I saw-

**[00:21:25] Adam Curcie:** Did I work it?

**[00:21:25] Devon D'Andrea:** ... that. The ticket, Adam?

**[00:21:30] Adam Curcie:** I don't think I put a ticket in about service plan stuff. I only put one ticket in

**[00:21:34] Devon D'Andrea:** The Verizon service plan. I could have sworn-

**[00:21:39] Adam Curcie:** I'm not-

**[00:21:40] Devon D'Andrea:** Never mind. I could've-

**[00:21:40] Adam Curcie:** Can you pull the ticket up? I don't know if I did.

**[00:21:42] Devon D'Andrea:** I thought, I thought you said you were putting in something that they had to like allow us to be able to activate on the other Verizon plan.

**[00:21:52] Adam Curcie:** Oh, oh, oh, oh yeah. Okay. Oh yeah, that's... You, you confused me when you said service plan. Um, you... Yeah, it's the other APN. Yeah. I don't know how much time we need to spend on that right now.

**[00:22:05] Devon D'Andrea:** Okay. I just did, I was just checking to see if we had a, had a ticket in. Okay.

**[00:22:09] Adam Curcie:** And, and it's not a correct ticket. I'll have to just put a correct one in later.

**[00:22:13] Devon D'Andrea:** Okay.

**[00:22:13] Adam Curcie:** I don't think all the... I, I, I mashed it into the same ticket-

**[00:22:17] Devon D'Andrea:** Yeah

**[00:22:18] Adam Curcie:** ... um, which wasn't the right thing to do at all. Um, but yeah, no, we'll, uh, the, we don't have to talk about that. I'll circle back to that later.

**[00:22:28] Devon D'Andrea:** All right. Well, so this ticket, I think for now you guys can just stop looking at.

**[00:22:35] Adam Curcie:** Yeah, for the time being.

**[00:22:36] Aksana Rahouski:** Ignore for now.

**[00:22:38] Devon D'Andrea:** Yep.

**[00:22:39] Aksana Rahouski:** Um, just go back though. We, we, like when we import, when you guys import devices, right? With SIMs and you can import like up to three SIMs and enable max two is dual, that's like the max, right?

**[00:22:56] Devon D'Andrea:** Mm-hmm.

**[00:22:57] Aksana Rahouski:** Um, are you say- But, but I thought, I'm pretty sure, like if I remember correct, like on import, we never actually enable any SIMs. I mean changing already active, from active to I- if SIM were saying enable, disable, not active, inactive. Right?

**[00:23:15] Devon D'Andrea:** So-

**[00:23:15] Aksana Rahouski:** Is that correct or no? Or do we, are we assuming that in some cases we might want to import devices and immediately SIM need to be, uh, enabled?

**[00:23:26] Devon D'Andrea:** Yeah, so the SIM, everything that we i- import is gonna have something that's enabled. Our guy, our guy who's on the, who's actually on this call right now, we can, uh, we can all, you know, berate him. Um, he, he misunderstood.

**[00:23:44] Adam Curcie:** Yeah.

**[00:23:44] Devon D'Andrea:** And instead we wanted them put in deactivated, he thought we meant to put no under active SIM.

**[00:23:52] Adam Curcie:** Yeah.

**[00:23:52] Devon D'Andrea:** And he did that and it's, you know, it's cool.

**[00:23:55] Adam Curcie:** And it, yeah, it's just the, it's just the wording. It, it's fine.

**[00:23:58] Devon D'Andrea:** It's cool. And we're gonna-

**[00:23:59] Adam Curcie:** Yeah

**[00:23:59] Devon D'Andrea:** ... we're gonna fine and admit to this call, so it's all good.

**[00:24:02] Adam Curcie:** Say goodbye to Garrett. We'll never see him again.

**[00:24:05] Devon D'Andrea:** Everybody say goodbye to Garrett.

**[00:24:06] Adam Curcie:** It was an innocent mistake.

**[00:24:09] Devon D'Andrea:** So no.

**[00:24:09] Adam Curcie:** Ruin it all.

**[00:24:10] Aksana Rahouski:** Okay.

**[00:24:11] Devon D'Andrea:** The intention, the intended, the intention was-Upload them where the Verizon SIM is enabled, meaning that active Verizon SIM... Again, this goes back to-

**[00:24:23] Aksana Rahouski:** Yeah

**[00:24:23] Devon D'Andrea:** ... that word active is just gonna plague us, right? So we want yes in that column-

**[00:24:28] Aksana Rahouski:** Yep

**[00:24:28] Devon D'Andrea:** ... that's enabled. But when we, but we wanna... We don't... When we s- when we assign them to this company, we don't want any billing to kick in until they choose to turn these SIMs on. So we have to put them in a deactivated state and then assign them to them. Or we could just a- assign them to them and then do a bulk deactivation and waive billing. Ei- either way. It's, you know... Uh, but in this case, we as- we assigned, we deactivated and, and then assigned them. And then unfortunately, just when they went to do a bulk action, the system didn't do anything because you know, we don't have, we didn't have any enabled SIMs whatsoever. So how the hell did it know what to do? If I'm like, you know, thinking as a system.

**[00:25:21] Adam Curcie:** Yeah. Yeah.

**[00:25:24] Aksana Rahouski:** 'Cause the flow is, like you said, so first you import them, right? And you can, I mean, yes, you can import it enabled or disabled, right?

**[00:25:31] Devon D'Andrea:** Yes.

**[00:25:32] Aksana Rahouski:** And then you assign. So you just, so you just, the order in what you did things was wrong or like the-

**[00:25:40] Devon D'Andrea:** No

**[00:25:40] Aksana Rahouski:** ... statuses were-

**[00:25:41] Devon D'Andrea:** The status that, where you've highlighted right now is what was done wrong.

**[00:25:47] Adam Curcie:** Yeah.

**[00:25:47] Aksana Rahouski:** Wow.

**[00:25:48] Devon D'Andrea:** That was put in as no.

**[00:25:49] Adam Curcie:** We just wanna change that to the word enable or billing.

**[00:25:51] Devon D'Andrea:** That was put in as no instead of yes. So effectively, even though there was a Verizon SIM present, there was no enabled SIMs-

**[00:25:59] Aksana Rahouski:** Mm-hmm

**[00:25:59] Devon D'Andrea:** ... whatsoever like in, in there at all.

**[00:26:03] Aksana Rahouski:** All right. So do you want us to create a small ticket to, to rework or change i- SIM wording for active, inactive to enable, disable? So then we have de- device status-

**[00:26:16] Devon D'Andrea:** Yeah

**[00:26:17] Aksana Rahouski:** ... as active and active-

**[00:26:17] Devon D'Andrea:** Yes

**[00:26:17] Aksana Rahouski:** ... but is enabled, disabled.

**[00:26:19] Devon D'Andrea:** That's it. Enable, disable is much better.

**[00:26:21] Aksana Rahouski:** Okay.

**[00:26:21] Adam Curcie:** Yes, please.

**[00:26:23] Aksana Rahouski:** But-

**[00:26:23] Devon D'Andrea:** And then if Garrett does it again, we really will fire him.

**[00:26:27] Adam Curcie:** At, at least, at least chop off some of his fingers.

**[00:26:32] Aksana Rahouski:** I like, I like how fast you-

**[00:26:34] Devon D'Andrea:** Some of his fingers

**[00:26:34] Aksana Rahouski:** ... you fired him.

**[00:26:37] Devon D'Andrea:** Yes.

**[00:26:40] Adam Curcie:** Well, we have so many people that work here. I mean, we could stand to lose a few. I mean...

**[00:26:45] Devon D'Andrea:** Well, you know what actually we're gonna do? We're, we have a trade show next week. We're just gonna go ahead and make Garrett do all of the booth set up and break down.

**[00:26:53] Adam Curcie:** Again?

**[00:26:54] Devon D'Andrea:** Again.

**[00:26:55] Adam Curcie:** Again.

**[00:26:56] Laura Perry:** I thought you're gonna say you're gonna leave him behind.

**[00:26:59] Devon D'Andrea:** Or leave him behind, yeah.

**[00:27:02] Adam Curcie:** That's even worse.

**[00:27:03] Aksana Rahouski:** Okay. And then, so here we wanna also update enabled then, I'm guessing.

**[00:27:10] Devon D'Andrea:** Yes.

**[00:27:11] Aksana Rahouski:** And, and so when you ch- ch- select the checkbox-

**[00:27:15] Devon D'Andrea:** No, I think there's three places

**[00:27:16] Aksana Rahouski:** ... you are enabling or disabling, right?

**[00:27:18] Devon D'Andrea:** I think there's three places, right? There's there, there's the-

**[00:27:21] Aksana Rahouski:** Yeah

**[00:27:21] Devon D'Andrea:** ... the import column header itself and the, and the-

**[00:27:25] Adam Curcie:** Well-

**[00:27:26] Devon D'Andrea:** ... device

**[00:27:26] Adam Curcie:** ... it could also go on the export and the updates. It just-

**[00:27:29] Devon D'Andrea:** It's not on the export.

**[00:27:32] Adam Curcie:** Oh, yeah. I think we meant-

**[00:27:33] Aksana Rahouski:** Well-

**[00:27:33] Adam Curcie:** ... to ask that you put that on there, but I don't know if we decided if we needed to ask that. We talked about it. We... Remember, 'cause we did an export and we were looking for it, but we couldn't find it.

**[00:27:42] Devon D'Andrea:** Yeah.

**[00:27:44] Aksana Rahouski:** Yeah. Um, just-

**[00:27:45] Devon D'Andrea:** We might as well add it in for-

**[00:27:48] Aksana Rahouski:** Yeah.

**[00:27:48] Adam Curcie:** Yeah. A couple more columns-

**[00:27:51] Aksana Rahouski:** On the export

**[00:27:51] Adam Curcie:** ... aren't gonna hurt y'all.

**[00:27:51] Devon D'Andrea:** On the export, just like same as the update devices, the im- same as the import I should say. Like, just next to each ICCID column, just put the is enabled or disabled column.

**[00:28:04] Aksana Rahouski:** Well, this update also has these. So-

**[00:28:07] Devon D'Andrea:** Yes. The device update has them

**[00:28:10] Aksana Rahouski:** ... import and bulk update.

**[00:28:10] Devon D'Andrea:** The import has them-

**[00:28:11] Aksana Rahouski:** Mm

**[00:28:11] Devon D'Andrea:** ... but the export does not.

**[00:28:14] Aksana Rahouski:** Okay. Okay, so we change wording on import, bulk update, uh, device, uh, view and edit page, um, and add these statuses to the device export as well.

**[00:28:28] Devon D'Andrea:** Yes, please.

**[00:28:29] Aksana Rahouski:** Okay.

**[00:28:29] Adam Curcie:** And if you find it anywhere else-

**[00:28:30] Aksana Rahouski:** Okay

**[00:28:30] Adam Curcie:** ... just change it there too.

**[00:28:34] Aksana Rahouski:** Okay. So then this ticket for now just hold it.

**[00:28:40] Devon D'Andrea:** Hold it until-

**[00:28:41] Aksana Rahouski:** Don't do anything.

**[00:28:42] Adam Curcie:** Yeah.

**[00:28:43] Aksana Rahouski:** See how-

**[00:28:43] Adam Curcie:** I have to give you guys a bunch more context because that's gonna be a, a, a pain. Uh, and I don't, I don't, I know we don't have time today, so-

**[00:28:51] Devon D'Andrea:** Not, so not, so not this ticket that we just talked about, not the APN. She's saying, uh, this one's on hold.

**[00:28:58] Aksana Rahouski:** Mm-hmm.

**[00:28:59] Adam Curcie:** Okay. All right. Never mind. So-

**[00:29:00] Aksana Rahouski:** All right. That's-

**[00:29:01] Devon D'Andrea:** I thought I, I thought I mashed them into one-

**[00:29:02] Aksana Rahouski:** So it's on s-

**[00:29:03] Devon D'Andrea:** Maybe I didn't. I think they might have... They, maybe they separated it out. It's in there.

**[00:29:09] Laura Perry:** So you created... Yeah, the ticket that was just up, you'd emailed about, and I created the ticket for it.

**[00:29:15] Adam Curcie:** That's right. I sent an email. That's right.

**[00:29:17] Laura Perry:** And then-

**[00:29:17] Adam Curcie:** I didn't even put a ticketing

**[00:29:19] Laura Perry:** ... right. Then you, but you did create a ticket about the APN, and that is in the backlog column right now. But I, I added a note to it that you needed to update it.

**[00:29:30] Adam Curcie:** Yeah. Yeah, I do. I did, yeah.

**[00:29:33] Aksana Rahouski:** Okay. Okay, then I think-

**[00:29:35] Devon D'Andrea:** I think we need to quickly jum- we, we need to quickly jump back to the comments that I'm just reading from Noah.

**[00:29:43] Aksana Rahouski:** Mm-hmm. Yeah. Noah said status.

**[00:29:50] Devon D'Andrea:** If there is no status, if there is no status, we believe that it will activate on assign.

**[00:29:55] Adam Curcie:** Yes.

**[00:29:56] Noah Bratzel:** The default, default status is active. If, if there's no status when you import it, it will set the status to active. So then as soon as it's saved, that will trigger the activation.

**[00:30:04] Devon D'Andrea:** Okay.

**[00:30:04] Noah Bratzel:** So as long as you're doing the deactivation like you said-

**[00:30:08] Devon D'Andrea:** Yes

**[00:30:08] Noah Bratzel:** ... then it should work fine.

**[00:30:08] Devon D'Andrea:** Before. Okay.

**[00:30:09] Noah Bratzel:** But if you miss-

**[00:30:09] Devon D'Andrea:** Okay. I'm good with that. I'm good with that.

**[00:30:12] Adam Curcie:** Agree.

**[00:30:16] Devon D'Andrea:** Unless-

**[00:30:17] Aksana Rahouski:** I feel like we need-

**[00:30:17] Devon D'Andrea:** It's one of those sit-situations where... Sorry. Unless it's one of those situations that we talked about before where we ended up making like logic for like looking for test ready for AT&T stuff.

**[00:30:31] Richard Sacco:** Yeah, yeah, we ha- we have that in there as well.

**[00:30:33] Devon D'Andrea:** Okay. Okay, okay. Okay. Cool.

**[00:30:34] Richard Sacco:** If it's test ready, we don't change it either.

**[00:30:36] Devon D'Andrea:** Got it. Got it, got it, got it. All right. Sounds good.

**[00:30:41] Aksana Rahouski:** We should have that-

**[00:30:42] Devon D'Andrea:** All right

**[00:30:43] Aksana Rahouski:** ... meet somewhere probably. We can dig it up. Might be helpful. Okay. Like, so I think we can move on or anything else regarding this? I feel like also we should do some, like, diagrams around status and what flips it and when. Okay. Maybe it's something to work on when you have some w- we have some breathing room. Okay. Uh, I think we can jump to configs now, unless is there anything else that you guys need to address while we're here as far as on... any work in flight, bugs, da, da, da?

**[00:31:25] Devon D'Andrea:** Uh.

**[00:31:30] Aksana Rahouski:** If trial is easier to-

**[00:31:32] Devon D'Andrea:** I don't think so.

**[00:31:34] Aksana Rahouski:** Okay. Okay. Sounds good. Well, I think then... I don't think we need everybody just to kinda keep, um... I feel like Noah Stone feel free to jump off for now. Um, Richard is our safety net for configs. Just a lot of it was just discovering still. Um, okay. Um, okay. And I know you guys... So I send you a bunch of documents whether you had, but like document kinda explains, like, what these pages are for. For the most part, I think we can just work off this, um, mock-up that I put together. And let me... Like, so I know last time, you and me kinda sync on it already a little bit, but it was, like, a little bit chaotic. We're like fifteen minutes, but then there's still... things were still, like, coming up. But so for the most part, do you- do we wanna kinda walk through the whole thing again, kind of from the schema to... Oops. Um, I just would... don't know how much, like, you wanna revisit, like, kind of start from scratch or, um, kinda jump in right into the... I think three-way rule and company overrides, that's where we're, like, discussing last time you and me met. Um-

**[00:33:05] Devon D'Andrea:** Yeah.

**[00:33:05] Aksana Rahouski:** Any opinions on that?

**[00:33:06] Devon D'Andrea:** I think we should jump, I think we should jump in and...

**[00:33:14] Aksana Rahouski:** Yeah. And to... just to, like, for, for everybody else, I put this resolution logic page here, right? That kinda explained that configuration engine ultimately. And that's where, like, again, in, in the documents, uh, when I... you guys, I gave it to you. I kinda broke it into, uh... So we will be bu-building like, um, a config and, um, a config builder, right? So a system that allows us to, like, set up configuration schema, set out which models can be enabled for configurations, which parameters come in for each model, build three-way rules where the, uh, carrier model, service plan, and c-create company overrides, right? Which ultimately all that cascades to the, uh, device, um, final config set the device is gonna get. Additionally, last time I know we kinda brought up, well, great, now what about how are we gonna push these things, right? And s-so I put together to like a separate document, um, um, so just like later if you didn't get to it. So that kinda goes through, okay, how it works today, um, uh, which gaps you guys kinda identified yourselves already, right? That today, like, when check-ins, common configuration get pushed, we don't know whether or not it actually was updated, right? Uh, so there's no, like, reporting around or, like, no really manual way, like, pushing it. Let's, let's say pick these five devices, push configs now instead of waiting for a check-in to, uh, kick it off. Um, so l- that stuff is flashed out in this kinda separate. Uh, but back to, uh... So this, I would say, as you kinda get familiar, s-start with this page. It literally kinda walk you through, uh, how resolution, like, for each parameter starts, right? So you have the grand schema that ultimately gives you an entire set of what we have almost seven hundred of them, right? Um, then certain values have defaults set on the schema, means these are gonna be applied for every, uh, device out there. Uh, certain values will be empty because you do have probably like good chunk of them, like half sitting at either like nulls or zero or empty string values. Um, then it's gonna check device model. If that model has any defaults, it will apply it, then it will move on to, to the combo of a three-way rule, like, for the... If a device is a specific model carrier service plan, is there any rule, um, uh, are there any values that are set for that combo? Um, it's gonna check that, override anything that needs to be overridden, and then it's gonna apply company overrides. Um, and then device overrides is kinda the, the, the last one. That one will, um, if, if it sets any values for a- any previously set parameters, it will override that as well, and that's how like ultimately each device gets like a final set of values. Um, I'm guessing that the final kinda check is gonna be parameters on the schema will have to be... Some of them are gonna be, let's say, required, right? Um, so make sure that if a value didn't, uh, cascade somehow to or, or parameter did, did not end up getting a value for whatever reason, it should kinda, like, highlight it and say, "Okay, this is invalid because it's a required field that has no value." And, um, if every, uh, this kind of the chain ofA logic, uh, executed into a valid payload or like key value pair, right? That's considered good. So I think it's, like, again, kind of helpful to always go as a reminder to understand how this configs the layers of decisions that it goes through. Um, and then kinda high level for whoever hasn't seen it, obviously schema, which, uh, ba- basically give us parameter name. Um, we did... We don't need to, for now, kinda like dive in on like... But some key, key value, some categories to make it a little bit easier to filter through them, uh, what data type that is, description if we need to, any default value that, as I was saying earlier, the schema needs to be set, any validation rule needs to, whether it's required, um, and layers available. So that's where I don't know if these two flags will be enough, but for the most part, this is something that we can use to control, um, like for example, for a company level override, right? And for now we'll just kinda leave it there. Like, this parameter will be available in the company override blocks that you can then override for any company if you need to. Um, and then, okay, so then schema, we have our models. Like we talked earlier, so for a model, and I, I, I'll still so- repeat it 'cause I, I realize our team actually, a- a- some haven't even seen some of it. So if I look quickly for a model, um, for... So every model is gonna be... Because like you said earlier, not every model, model is needs configurations. And by the way, feel free as I'm kinda like stating these things, if somewhere along the way is, is this is not correct, right? Like, stop me. But so for every model then you guys will have to, "Yep, this model is con- ne- needs configs." Uh, so that how we will kinda control for each models the schema, um, sorry, the, the parameters need to be set. So like in this case, for this one says, "Nope, we will not be setting any configs," and for the rest of them is yes. So model level, it just serves as a, like, true false flag. Yes, expose it to the config engine. No, do not expose it to the config engine. And then in a configuration management itself, then the models that are enabled for configuration are, are gonna be configured with, um... So this kinda shows you out of total list of parameters, this 16 are what this model needs to have available for configuring. Uh, then model defaults will be set for like that particular model. For the 16 keys, right, we can set up values. Um, okay, so the kinda... But for... So I'll, I'll stop here for a second just to see if there are any questions or comments so far.

**[00:39:52] Devon D'Andrea:** So before you get to the model on the page that you're at right now-

**[00:39:58] Aksana Rahouski:** Yes

**[00:39:58] Devon D'Andrea:** ... does, does all of the available... Like, when we, when yous have selected parameters, is that like gonna be-

**[00:40:10] Aksana Rahouski:** Mm-hmm

**[00:40:10] Devon D'Andrea:** ... just the parameters that we want, that we want to be set in the config or is it like that could be things that we want specific to that model and then everything else on the left just stays default, I guess?

**[00:40:32] Adam Curcie:** Yeah, I believe that's-

**[00:40:33] Aksana Rahouski:** Yeah. So think-

**[00:40:34] Adam Curcie:** I think-

**[00:40:35] Aksana Rahouski:** Yeah. Yeah. So think about it as a, for like I22, this is like-

**[00:40:40] Devon D'Andrea:** Mm-hmm

**[00:40:41] Aksana Rahouski:** ... your manual, right?

**[00:40:42] Devon D'Andrea:** Yes.

**[00:40:42] Aksana Rahouski:** Manual is your entire list of schema, right? That's your schema. And then for I22, um, okay, this, this one also I wanna push 17. This one I, I, I wanna push, um, 18. So you're saying for I22, this 18 is what makes this like sub-menu for this model, right?

**[00:41:06] Devon D'Andrea:** Mm-hmm.

**[00:41:07] Aksana Rahouski:** That you kinda-

**[00:41:07] Adam Curcie:** Yeah

**[00:41:07] Aksana Rahouski:** ... basically build this like narrow list of what is applicable to this specific model.

**[00:41:13] Devon D'Andrea:** Sure.

**[00:41:14] Aksana Rahouski:** Um, so that's step one. Step two, now I need to, okay, I preselected 18 that I wanna manipulate, um, and then you need to set values, right, for these 18. So that's where, and this screen is meant to, okay, for this 18 then you go in and you start setting what your model defaults.

**[00:41:33] Devon D'Andrea:** Yeah.

**[00:41:33] Aksana Rahouski:** So for I22, value of the field is gonna be foo, right? That way what, what... It, it gives you the power, uh, to custom control what list each model has and set default per model.

**[00:41:52] Devon D'Andrea:** Right. Got it.

**[00:41:52] Adam Curcie:** Yeah.

**[00:41:53] Aksana Rahouski:** So it's-

**[00:41:53] Adam Curcie:** So for like the cellular APN- Before you- I... Sorry. Can you go back to the I22?

**[00:41:59] Aksana Rahouski:** Okay.

**[00:41:59] Adam Curcie:** 'Cause I just wanna, uh, like talk through what I've... I'm thinking or interpreting, right? So you've got-

**[00:42:05] Aksana Rahouski:** Right

**[00:42:05] Adam Curcie:** ... cellular APN there, and then you go to the model defaults.

**[00:42:10] Aksana Rahouski:** Yes.

**[00:42:11] Adam Curcie:** So we don't set this here because it's going to come later on this, on the, the three-way rule.

**[00:42:18] Aksana Rahouski:** Mm-hmm.

**[00:42:18] Adam Curcie:** Right? But we are basically-

**[00:42:20] Aksana Rahouski:** Mm-hmm

**[00:42:20] Adam Curcie:** ... flagging-

**[00:42:21] Aksana Rahouski:** Yeah

**[00:42:21] Adam Curcie:** ... the engine to tell, like it needs to find a, like this is required. It has to find it somewhere down the road. Wherever it finds it, it finds it, and then for that parameter it will find it on the, you know, three-way rule because it's a s- it's gonna say Verizon.

**[00:42:41] Aksana Rahouski:** Cross off. Yes. Yes. So it is, it, it is exactly like what you said. It's actually, like building that like subset, right?

**[00:42:50] Adam Curcie:** Yeah.

**[00:42:50] Aksana Rahouski:** So this 16. They-

**[00:42:52] Adam Curcie:** Yeah

**[00:42:53] Aksana Rahouski:** Every time, th- these are also like play into a three-way rule every time you will be playing building a three-way rule when you say model is I-22-

**[00:43:05] Adam Curcie:** Yeah

**[00:43:05] Aksana Rahouski:** ... these 16 is what you're gonna receive, right?

**[00:43:09] Adam Curcie:** Yeah.

**[00:43:09] Aksana Rahouski:** As far as values, yes. So the way they will cascade, if you set a value here, then again back to... Let's just quickly I will open it as a separate, um-

**[00:43:20] Adam Curcie:** Yeah. Yeah. So yeah, exactly.

**[00:43:22] Aksana Rahouski:** Um, if, if-

**[00:43:24] Adam Curcie:** If, if we set a value-

**[00:43:24] Aksana Rahouski:** If you don't set it... Yeah, if you s- if, if you, um, resolution logic. Again, just to kind of follow it, right? If model has a default, right?

**[00:43:36] Adam Curcie:** Yeah.

**[00:43:36] Aksana Rahouski:** Um, it will use it. But if three-way rule for that and model also has a value, it will override it.

**[00:43:44] Adam Curcie:** Yeah.

**[00:43:44] Aksana Rahouski:** So three-way rule override model default. If model doesn't have default, it falls back into the schema default.

**[00:43:52] Adam Curcie:** Yes.

**[00:43:52] Aksana Rahouski:** So like if this value is not there-

**[00:43:54] Devon D'Andrea:** Yes, so you don't-

**[00:43:54] Aksana Rahouski:** ... what it means

**[00:43:55] Devon D'Andrea:** ... so you, so you wouldn't need to add it there. You wouldn't need to add it there then.

**[00:43:58] Adam Curcie:** We would leave it blank, correct? On th- right where we're looking at it right now.

**[00:44:03] Aksana Rahouski:** Yes, look. If-

**[00:44:03] Adam Curcie:** Yes.

**[00:44:04] Aksana Rahouski:** If you think about it, so if you, if you have it blank, that means-

**[00:44:09] Adam Curcie:** Yeah

**[00:44:09] Aksana Rahouski:** ... it will use schema value. And for any three-way rule where you state I want it to be for I-22 Verizon and ATM unlimited, it, it-

**[00:44:22] Adam Curcie:** Yeah

**[00:44:22] Aksana Rahouski:** ... will override default one.

**[00:44:24] Adam Curcie:** Yeah.

**[00:44:24] Aksana Rahouski:** If you do put it to something-

**[00:44:27] Adam Curcie:** Yeah

**[00:44:27] Aksana Rahouski:** ... then this will override default and three-way will override it. So it's like, again-

**[00:44:32] Adam Curcie:** Yeah

**[00:44:32] Aksana Rahouski:** ... this is like this kind of tree.

**[00:44:35] Adam Curcie:** Yep.

**[00:44:35] Aksana Rahouski:** It means that value is set.

**[00:44:37] Adam Curcie:** No.

**[00:44:37] Aksana Rahouski:** If there is override to the next level, replace it. Another value, replace it. So like, um, default... Like model wins over default. Three-way rule w- wins over model. Uh-

**[00:44:49] Adam Curcie:** Yep

**[00:44:50] Aksana Rahouski:** ... uh, so-

**[00:44:50] Devon D'Andrea:** Well, so in, in that example, if-

**[00:44:53] Aksana Rahouski:** That comes into play

**[00:44:53] Devon D'Andrea:** ... in that example, if we're not choosing to make any model modification to the cellular APN, we don't need to move it over to that right side.

**[00:45:06] Adam Curcie:** No, you do because it has... 'Cause the system needs to know that it's going to get it. Like, it can't make a config without it, you understand? So you have to basically... 'Cause you understand what I'm saying, Devin? Like, you're basically flagging this as something that needs to exist. Where you fill it in is gonna change depending on which parameter it is, right? Like we could, for DHCP, we could have a enabled DHCP true.

**[00:45:32] Devon D'Andrea:** Correct.

**[00:45:32] Adam Curcie:** That could exist there like that, and then if it doesn't show up anywhere else throughout the engine, then it's fine to use that. But this one would be blank because it's not gonna get set until the three, three, uh, three-way rule. I'm having a tough time saying that today. I'm sorry. And if it makes it all the way through the life cycle of the engine and there is no value, then it's gonna go at the, the error that she showed at the very bottom of the resolution logic.

**[00:46:00] Aksana Rahouski:** Mm-hmm. Yeah.

**[00:46:02] Adam Curcie:** So this-

**[00:46:02] Aksana Rahouski:** So like in this like-

**[00:46:03] Adam Curcie:** Go back to the resolution logic. I'm sorry. At the very bottom I, I, I... It has that error for, like config required parameter has no value. So you're stating that the cellular APN is a required parameter, so if it doesn't find it anywhere it can't make the config.

**[00:46:20] Aksana Rahouski:** Yeah. And, and that's why like, again-

**[00:46:21] Devon D'Andrea:** Where are, where are we stating, where are we stating that it's a required parameter? Only if it shows up here.

**[00:46:26] Adam Curcie:** Well, we put it on the left, from the left to the right.

**[00:46:28] Aksana Rahouski:** On the schema.

**[00:46:29] Adam Curcie:** That's why I'm saying you have to set it.

**[00:46:32] Aksana Rahouski:** So here on the schema. O- on the schema when you like take a-

**[00:46:35] Devon D'Andrea:** No, it's on the schema, isn't it?

**[00:46:36] Aksana Rahouski:** I don't know the exact name. Yes. The schema decides whether or not it's required, means-

**[00:46:42] Devon D'Andrea:** Yeah

**[00:46:42] Aksana Rahouski:** ... this va- this needs to have a value, and that's why I would say-

**[00:46:46] Devon D'Andrea:** Yeah

**[00:46:46] Aksana Rahouski:** ... a good rule of thumb to follow would be to make sure that every required field has a default value.

**[00:46:54] Devon D'Andrea:** Right.

**[00:46:54] Aksana Rahouski:** Because again, like kind of back to, back to our like model setting, right? This... Like if we look at an example. Cell APN, right? It tell, says that-

**[00:47:04] Adam Curcie:** Yes

**[00:47:04] Aksana Rahouski:** ... um, it, the schema did not set a default value, right?

**[00:47:10] Adam Curcie:** Yeah.

**[00:47:10] Aksana Rahouski:** Meaning if we don't set a value for a model, for a three-way rule, like there will be devices that will not have any value. And, and for some it's okay, because some values you in fact do now like set to nothing.

**[00:47:24] Adam Curcie:** Correct.

**[00:47:24] Aksana Rahouski:** Like, right? But ones that must have a value, these are gonna be required.

**[00:47:29] Adam Curcie:** Mm-hmm.

**[00:47:29] Aksana Rahouski:** So like, again, when you're saying no map-

**[00:47:31] Devon D'Andrea:** Yeah, but if you make it required, if, if you make it required, you don't have to actually, you don't have to actually tell it what it is until you get to the three-way rule.

**[00:47:39] Adam Curcie:** Yeah, but doesn't it need to be on the right side in order to configure it at the three-

**[00:47:45] Aksana Rahouski:** Yes

**[00:47:45] Adam Curcie:** ... three-way rule?

**[00:47:47] Aksana Rahouski:** And maybe let's-

**[00:47:47] Adam Curcie:** Or-

**[00:47:48] Aksana Rahouski:** I think we're like, let's come to three-way rule for, for like as a sample, right? So again, like for-

**[00:47:52] Devon D'Andrea:** So it has to be an I-22. Yeah, but I mean, I thought that that's where we were talking about last time, where like APN doesn't... The APN isn't, isn't really, isn't... Is kind of model agnostic, isn't it?

**[00:48:06] Adam Curcie:** Well, no, because the, the parameter you use for APN is different between 41s and 22s. So that's why when you make the 41, you say, "This is my APN," and you put it to the right side, and then like-

**[00:48:20] Aksana Rahouski:** Mm-hmm

**[00:48:20] Adam Curcie:** ... 'cause y- you can't use the same parameter. It's the same value for the parameters, but the, the parameter name is-

**[00:48:26] Devon D'Andrea:** The parameter name, yeah.

**[00:48:28] Adam Curcie:** Yeah, so then in here-

**[00:48:30] Aksana Rahouski:** Yeah

**[00:48:30] Adam Curcie:** ... you've got the, the I-22 specific ones that you can configure.

**[00:48:39] Devon D'Andrea:** So you... So when you get to the... You're on three-way rules right now. So if that-

**[00:48:43] Aksana Rahouski:** Yeah, so for-

**[00:48:45] Devon D'Andrea:** So you couldn't-

**[00:48:46] Aksana Rahouski:** Yeah

**[00:48:46] Devon D'Andrea:** ... you couldn't, you wouldn't be able to call... You wouldn't be able to say I-22 Verizon and then find the cellular APN unless it's in the two-way rule?

**[00:48:59] Aksana Rahouski:** Until-

**[00:49:00] Adam Curcie:** It's just a model that's defined

**[00:49:03] Aksana Rahouski:** ... unless it's configured, unless i-i-unless it's configured for this model. For example, like, like watch, watch me. Like remember how we looked at I22? Let's go back-

**[00:49:12] Adam Curcie:** Yeah

**[00:49:13] Aksana Rahouski:** ... to I22.

**[00:49:14] Adam Curcie:** Just like-

**[00:49:15] Aksana Rahouski:** Um

**[00:49:16] Adam Curcie:** ... seven. For a forty-one hundred you wouldn't be able to configure IO because you wouldn't put it on the right.

**[00:49:22] Devon D'Andrea:** Yeah.

**[00:49:23] Aksana Rahouski:** Right.

**[00:49:23] Adam Curcie:** Yeah.

**[00:49:23] Aksana Rahouski:** So like watch this.

**[00:49:24] Adam Curcie:** Because you-

**[00:49:24] Aksana Rahouski:** So like for I22 I have sixteen preselected, right? I... Whether or not I choose default, as you can see in this example, only s- ten are set with some default values, meaning-

**[00:49:36] Adam Curcie:** Mm-hmm

**[00:49:36] Aksana Rahouski:** ... every device of this model is as the default, right?

**[00:49:40] Adam Curcie:** Got it.

**[00:49:40] Aksana Rahouski:** But when I... Sixteen is what this model should... This sixteen is what I care about for when I talk about this mo- devices of this model.

**[00:49:50] Adam Curcie:** Sure.

**[00:49:50] Aksana Rahouski:** So but then when I build a new three-way rule, so let's go to three-way rule, and we say add, and we pick I22 or whatever Verizon carrier plans.

**[00:50:03] Adam Curcie:** Mm-hmm.

**[00:50:04] Aksana Rahouski:** Sixteen is what I get. So it-

**[00:50:07] Adam Curcie:** Gotcha

**[00:50:07] Aksana Rahouski:** ... it filters down to only what's is, what's configured for that model. If you want the sixteen to be ten or twenty, you go back to the model and you move more to the right or last from the right, and that's how way t- you control that these, this, this-

**[00:50:22] Adam Curcie:** Yeah

**[00:50:22] Aksana Rahouski:** ... subset would, uh, relevant to this model.

**[00:50:27] Devon D'Andrea:** Right.

**[00:50:27] Adam Curcie:** I got, I gotta just tell you, I am so happy with everything we've seen right today. Like, I feel like we're actually very close, like really close. This is, this is gonna be awesome. Very. This is, this is exactly what we needed. This will a- this is gonna like a- allow us to like triple how many different l- like customers we have and not have to-

**[00:50:55] Aksana Rahouski:** Mm-hmm

**[00:50:55] Adam Curcie:** ... get six Johns on our staff to manage all the configs. So.

**[00:51:00] Devon D'Andrea:** Can you go back to, can you go back to when you're creating the, um, this... Okay, I see it there. So you can, you can select multiple service plans. Okay.

**[00:51:10] Aksana Rahouski:** Yes. So that's... And that's the thing what we talked about last time, right? You said, "Hey, what if I ever want for a specific customer, I want any model, any carrier, any service plan," right? It's-

**[00:51:23] Devon D'Andrea:** Mm-hmm

**[00:51:23] Aksana Rahouski:** ... th- this is where things get complicated, 'cause up until model it's actually quite simple. So like you pick a model, you select subset, you pick one model, one carrier, many. So it must be one model, one carrier, but as many service plan as you want, and you build the, the three-way rules. So it basically tells for all devices of this model on this carrier, these are the rules we're gonna apply. So then now when we talk about companies, right, where we said, okay, so now there are these company overrides. So that's the thing that, the curve ball that you guys threw at me last time.

**[00:51:58] Devon D'Andrea:** Just-

**[00:51:58] Aksana Rahouski:** So you said, "I wanna build these like override sets," right? "Maybe I wanna like use a, a five parameters like cord, but I wanna apply to twenty-five companies, and I wanna build this like building blocks," right? "For, for... That I can then apply to many companies if, if I choose." And not only that, the, this, the, the parameters that I specify for model carrier or service plan could be loose as far as it, it could be like similar to a three-way rule telling me exactly what model carrier and service plan it is.

**[00:52:38] Adam Curcie:** Yeah.

**[00:52:38] Aksana Rahouski:** Like most to be like for like, like this one says that override set named cord firewall, um, is apply co- uh, now being applied to twelve different companies. Um, and it's agnostic of model carrier and service plan. So it's really you're just building this like building block that you are like ignoring other decisions that you made up until now, right?

**[00:53:05] Adam Curcie:** Mm-hmm.

**[00:53:05] Aksana Rahouski:** And you're saying these, uh, forty-five parameters, right? Um, this is what... And th- this is perhaps this is where like I wish we had a little bit maybe better sample. So if like give me five companies and how you would like build these b- like subset of params and like distribute, right? Um, uh, but, uh, um, so the... And then... Hold on, where's my company? Um, uh, let's pick something shorter. Um, this guy's gonna be probably shorter. Um, company assignments. Oh, it's because I didn't get it. Okay. So subset, right? Think about it as, um... So again, this is... Y- you kinda, you give it a name, you call... And, and that's where again, 'cause the assumption is these like blocks of key value pairs, right? Uh, that is, uh, built, built based on carrier model and service plan, but could be specified for specific or any, right? And then you also, you, you pick like, okay, what, what parameters make the list? You set the values, and then you choose which companies I wanna apply it to. Um, so last time when we guys talked, we kinda ran into this issue that I designed it with an assumption that a company can only w- have a single subset. Um, but y- but you kind... You said that really this is not realistic because what we're trying to do is like build these subsets that a, a company could have multiple be... An assumption that the keys will not overlap. Like there is no... Like if the same company, let's say Cord Financial, right?Has three different subsets overrides. Cellular APN can only be in one of them, cannot be in all three, because then you have-

**[00:55:12] Devon D'Andrea:** Right

**[00:55:12] Aksana Rahouski:** ... racing con- with three who wins, right? Uh, so as long as there is no overlap in parameters, a company could have like one too many. Um, and s-subset is where this flexibility comes in that you could specify a model, but you could also say any. For any model, any car- any carrier, any service plan, give me a block. Um, so I will say like, let's talk for a second, what makes the subset, right? 'Cause like forty-five parameters, where did they come from, right? Why is it forty-five? Um, so if you choose a model, a model will drive the list. So if I choose R22, I will only see sixteen. But if I choose any, where it comes from, if, if we go back to the schema, on a schema every... That's why this, um, available at a company level flag comes in. So if we're saying-

**[00:56:19] Devon D'Andrea:** Yeah

**[00:56:19] Aksana Rahouski:** ... if I ever build an overnight set for a company, that is... And like again, if, if, if, if, if model is specific, model list drives the list. But if not, then every key that has the available of company level flag will give you this forty-- this is where the forty-five came from. Assuming forty-five fields in the schema were flagged as available on a company level.

**[00:56:48] Devon D'Andrea:** Okay.

**[00:56:50] Aksana Rahouski:** So this one is a little bit, I will say the documents that I shared actually, uh, d-does like talk in depth about kind of this concept, right? Then because the tricky part here will be, okay, as I build an override, right? And then so, okay, great. I pick whether it's there's a model or there isn't a model, uh, with carrier or any service plan or any, right? I get my list, I set these values. Um, again, back to kinda our resolution tree, as you can see, company overrides is like it will ov-- like if there is a three-way rule k- model default, schema default, company override will override all of it. This is gonna win because this wins before device value wins. Um, and then, so once you kinda, and then you set your values, um, and then you work on your assignment. So like once you, if we go to any like existing... Oops, um, actually I just noticed time. Okay, so let's just figure out like next steps. I do have a meeting which I, I need to run to, uh, and-

**[00:58:04] Devon D'Andrea:** Okay

**[00:58:04] Aksana Rahouski:** ... we need like to schedule part two, um-

**[00:58:09] Devon D'Andrea:** Quick question

**[00:58:09] Aksana Rahouski:** ... and kinda keep working through it.

**[00:58:11] Devon D'Andrea:** Quick question. I mean, I, I, I, I, I honestly think this is like really, really, really far along. Did you, um, happen to consider any of the like validation stuff yet for another meeting that we could have at some point? Or we just start talking about that.

**[00:58:28] Adam Curcie:** You just, in regards to validating what boxes need configs when?

**[00:58:36] Aksana Rahouski:** When you say validation, like you, you-

**[00:58:37] Devon D'Andrea:** Yeah.

**[00:58:38] Aksana Rahouski:** Like-

**[00:58:38] Devon D'Andrea:** I was just wondering if that... I remember last time we kinda were like, "Let's just table that while we're working through this." I was just wondering-

**[00:58:44] Aksana Rahouski:** Yeah

**[00:58:44] Devon D'Andrea:** ... if, if we were, if we were-

**[00:58:45] Aksana Rahouski:** Gotchu

**[00:58:45] Devon D'Andrea:** ... wanting to start thinking about that stuff too.

**[00:58:51] Aksana Rahouski:** There is a little bit.

**[00:58:52] Devon D'Andrea:** There's gonna have to be.

**[00:58:53] Aksana Rahouski:** So there is like a def-

**[00:58:54] Devon D'Andrea:** Yeah.

**[00:58:55] Aksana Rahouski:** Um, yeah, validation and like that's where like the document, the second document I referred to as far as like the push mechanism, right? How do we then apply? Like do we wanna have that on demand, right? That I can then... In here I have, there is actually a screen for that too. If you go to, um-

**[00:59:17] Devon D'Andrea:** Yeah, I thought I saw something earlier. That's why I brought it up. It's just, I just was wondering.

**[00:59:22] Aksana Rahouski:** Okay. Yeah. So-

**[00:59:23] Adam Curcie:** I, yeah, and I, I know they have their own, um...

**[00:59:26] Devon D'Andrea:** Yeah.

**[00:59:27] Aksana Rahouski:** Yeah. Con- Uh, so let's just, again, so like, um-

**[00:59:31] Devon D'Andrea:** Okay. Okay

**[00:59:32] Aksana Rahouski:** ... part of this design that I wanna have is that also kind of what else, right? What else we're not seeing because yes, we did talk about how do I build config, right? Uh, with the way that it's like a lot of these components are reusable and, um, a lot more scalable from what it is today. How do we apply them, right? And am I guessing like that the check-in concept as device comes in and we identify that it needs new config, go, right? Stays.

**[01:00:03] Devon D'Andrea:** Yeah.

**[01:00:03] Aksana Rahouski:** But additionally, do we want something like, "Hey, pick these three and push immediately." Right? Um, so any kind of reporting and, um, and also like, yes, so validation, kinda how do you guarantee, right? That every time that you change something, every single device out there get a valid config.

**[01:00:24] Devon D'Andrea:** Right. Okay.

**[01:00:26] Aksana Rahouski:** Um, so we need... I would say like you guys, let's r-read these docs that I shared because I do think like a lot of it will help us to kinda like start from the same, same page. Um, and then I'll try to find, um, more time to keep going down this.

**[01:00:48] Adam Curcie:** Yeah.

**[01:00:48] Devon D'Andrea:** Sure.

**[01:00:49] Adam Curcie:** Um, I think, you know, whenever you have your next available time that we can get together, try to just let us know, so we'll move anything that we absolutely-

**[01:00:58] Aksana Rahouski:** Yeah

**[01:00:58] Adam Curcie:** ... need to, to be there for-

**[01:01:00] Devon D'Andrea:** Except for next week.

**[01:01:00] Aksana Rahouski:** Okay.

**[01:01:01] Devon D'Andrea:** But you're-

**[01:01:02] Adam Curcie:** No, yeah, that's, yeah.

**[01:01:03] Devon D'Andrea:** I think that-

**[01:01:03] Adam Curcie:** Well, Monday. We're good up till Monday.

**[01:01:06] Devon D'Andrea:** Mon-Monday's gonna be a little-

**[01:01:09] Aksana Rahouski:** Okay

**[01:01:09] Devon D'Andrea:** ... hectic, but yeah. I'd say-

**[01:01:14] Aksana Rahouski:** I do have like later today more time, but I don't know if that gives you time to like read through those documents, like sink in them.

**[01:01:26] Devon D'Andrea:** I don't-

**[01:01:26] Adam Curcie:** I'll read like-

**[01:01:27] Aksana Rahouski:** Or maybe we don't need to

**[01:01:28] Adam Curcie:** ... three thirty or 4:00 Eastern?

**[01:01:32] Aksana Rahouski:** Yeah. Yeah, I can do, uh, anything after thir- uh, 3:30 Eastern, yes. If you wanna like resume later and just kinda keep... 'Cause, 'cause what I want this, again, like to be like, just, yeah, ask away, throw in like what else are we not seeing, right? Let's talk through all these like overrides and everything and, um-

**[01:01:55] Devon D'Andrea:** Do you have anything available tomorrow?

**[01:02:00] Aksana Rahouski:** I'm actually tomorrow taking half day off, and then my morning is already-

**[01:02:05] Devon D'Andrea:** Okay

**[01:02:05] Aksana Rahouski:** ... zoo. Until... Unless it's like 6:00 in the morning, but I don't wanna do it at 6:00 in the morning.

**[01:02:14] Devon D'Andrea:** I mean, we could do something Monday.

**[01:02:14] Aksana Rahouski:** Um, and then you get in... Could do what?

**[01:02:20] Devon D'Andrea:** We could do something Monday.

**[01:02:23] Adam Curcie:** Do, do you have time Monday?

**[01:02:26] Aksana Rahouski:** Actually, like, so for me, uh, I'm traveling Monday, Tuesday for work.

**[01:02:30] Devon D'Andrea:** That's right.

**[01:02:30] Aksana Rahouski:** My other client.

**[01:02:32] Devon D'Andrea:** Yeah, we talked about that. We talked about that.

**[01:02:33] Aksana Rahouski:** Um, that's why I'd, uh-

**[01:02:33] Devon D'Andrea:** Listen, I mean, if, if you wanna just, if, if you wanna just put the 4:00 Eastern on today, you know, I got a few other things, um, you know, we got coming up, but, um, you know, I, I just don't know that I'll particularly-

**[01:02:50] Aksana Rahouski:** Yeah, like-

**[01:02:51] Devon D'Andrea:** ... the documentation, but...

**[01:02:53] Aksana Rahouski:** Yeah. So how... Like, and then next week is out, full, full week is out?

**[01:02:59] Adam Curcie:** Yeah.

**[01:03:00] Devon D'Andrea:** Yes.

**[01:03:00] Adam Curcie:** Other than like maybe Monday, uh, we're gonna, we're leaving Tuesday early, and we're not coming home till I think like Friday morning. Plane lands at the red eye.

**[01:03:11] Aksana Rahouski:** Okay.

**[01:03:12] Adam Curcie:** So-

**[01:03:13] Devon D'Andrea:** I mean, unless you, unless you wanna just like... I... You know what I think might be worthwhile is like if we do wanna do the 4:00 today is just kinda like talking out loud a little bit discovery-wise on the validation stuff. I, I don't know. 'Cause I feel like-

**[01:03:30] Adam Curcie:** Yeah.

**[01:03:31] Aksana Rahouski:** Yeah

**[01:03:32] Devon D'Andrea:** ... that would be a good use-

**[01:03:33] Adam Curcie:** I'll be able to read through all of the documentation and get Devin some, some cliff notes if he doesn't have time to finish reading it. Um, so I, I'd say let's do 4:00 today.

**[01:03:46] Aksana Rahouski:** Yeah, sure. Okay. Let me... I'll, I'll send something. Let's, let's, let's, let's do it. Um-

**[01:03:51] Adam Curcie:** Yeah.

**[01:03:51] Aksana Rahouski:** 'Cause again, even if we just try to-

**[01:03:52] Adam Curcie:** I've got very little to do for the next five hours that, that I could say is more important than this personally, so. I know Devin can't say that.

**[01:03:59] Aksana Rahouski:** No.

**[01:03:59] Adam Curcie:** But I can.

**[01:04:02] Aksana Rahouski:** And, yeah. And again, even if you didn't read, it's fine. Let's just, again, kinda use it as a... 'Cause a lot of it still need to be brainstorm, right? I'd like us to do it together, right? Like, like-

**[01:04:12] Adam Curcie:** Yeah

**[01:04:12] Aksana Rahouski:** ... push these assumption, challenge these things. What else are we not seeing, right? It's, it's, it's, I mean, it's, it's further along, but I don't think it's quite final yet, and that's where I need us like as a group collaborate on this together. And yeah, so informal I'll book us another hour, but I think while it's fresh-

**[01:04:33] Adam Curcie:** Yeah

**[01:04:34] Aksana Rahouski:** ... it's like it will, it will be better rather than waiting another like 10 days and-

**[01:04:38] Adam Curcie:** Oh, yeah. Yeah.

**[01:04:39] Aksana Rahouski:** We all forget about it.

**[01:04:40] Devon D'Andrea:** Yeah. Yeah. Let's ki- Yeah, let's keep this.

**[01:04:43] Adam Curcie:** All right.

**[01:04:44] Devon D'Andrea:** All right.

**[01:04:44] Adam Curcie:** We'll talk at 4:00. You gotta go.

**[01:04:46] Aksana Rahouski:** All right.

**[01:04:46] Adam Curcie:** See you guys later.

**[01:04:48] Aksana Rahouski:** Yeah.

**[01:04:49] Devon D'Andrea:** All right.

**[01:04:49] Aksana Rahouski:** All right, sounds good. Thank you. Bye.
