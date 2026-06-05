# APW Check-in — Transcript

**Date:** May 29, 2026, 1:30 PM EDT (5:30 PM UTC)
**Duration:** ~65 minutes
**Organizer:** Laura Perry
**Attendees:** Laura Perry, Aksana Rahouski, Richard Sacco, Aaron Diefes, Stone Marballie (Orases); Devon D'Andrea, Adam Curcie, Jon (APW)
**Recording:** https://tldv.io/app/meetings/6a19cd10c4457d001305d4ca

---

[1s - 8s] **Richard Sacco:** No, you're, you're not losing your mind. It was updated at some point. Um, we're still-
[8s - 13s] **Devon D'Andrea:** I'm not, listen, I'm not... I mean, I, it was, it looks great. I love it. It was just the fact that that one little sentence-
[14s - 14s] **Richard Sacco:** Yeah
[14s - 14s] **Devon D'Andrea:** ... and then like, man.
[15s - 21s] **Richard Sacco:** The copy, the copy didn't match the expectations that you had. No, that, that's perfectly makes sense. Yeah.
[22s - 22s] **Devon D'Andrea:** Yeah.
[22s - 28s] **Richard Sacco:** I, I need to go look and see as well, sir. I didn't really... I just was concerned about doing the immediate issue.
[28s - 55s] **Devon D'Andrea:** No, no, no. The, and that's all I care about. And I'm, and it's fine. It's not like, you know, listen, it's not, it's, it's, I just wanted to get it fixed so that it would stop sending out emails that say something that's potentially, you know, not, not factual. But I'm just, Adam and I are like, we're like, I don't remember a ticket to even, to even update that. Maybe it have, have the, have, have the bots gone rogue, Oksana?
[57s - 59s] **Aksana Rahouski:** I mean, I, I feel like-
[59s - 60s] **Richard Sacco:** I feel like the agent-
[60s - 61s] **Aksana Rahouski:** I didn't have-
[61s - 62s] **Richard Sacco:** ... agents have developed a mind of their own.
[63s - 63s] **Devon D'Andrea:** Yeah.
[64s - 73s] **Aksana Rahouski:** I didn't have time to go back through and, and find the ticket, but I feel like in the last month or so there was something related to the topic-
[73s - 74s] **Devon D'Andrea:** Well, did we... Well, so-
[74s - 76s] **Aksana Rahouski:** ... where I'm assuming it was updated within-
[77s - 87s] **Devon D'Andrea:** Well, I could tell you this. We did update the company deactivation email template, so I don't know if it was lumped in with that.
[88s - 88s] **Richard Sacco:** Oh.
[88s - 89s] **Devon D'Andrea:** Did that go live?
[89s - 90s] **Richard Sacco:** Mm.
[92s - 92s] **Aksana Rahouski:** The com-
[92s - 95s] **Devon D'Andrea:** Yeah. No, that's not even pushed to prod yet.
[96s - 111s] **Aksana Rahouski:** That one's not. That one's on the newest one. I know, Rich, you were gonna dig in, um, i- any way for us to track it to com- to commits and just to see, just to like have a date in mind and then we could... Uh, I, I mean, I-
[111s - 111s] **Devon D'Andrea:** I can tell you that.
[112s - 112s] **Aksana Rahouski:** Yeah.
[112s - 118s] **Richard Sacco:** I think John saw the, like, the day in which it changed 'cause he just-
[118s - 118s] **Aksana Rahouski:** Okay
[118s - 122s] **Richard Sacco:** ... like scrolling back in our emails, we, uh, most of us use folders for those.
[122s - 125s] **Devon D'Andrea:** It was Tuesday. Tuesday midday.
[126s - 128s] **Aksana Rahouski:** Oh, uh, what Tuesday? This week Tuesday?
[129s - 130s] **Devon D'Andrea:** Yes. Yes.
[130s - 131s] **Aksana Rahouski:** Oh, so we just pushed then?
[132s - 133s] **Devon D'Andrea:** Yeah. Yeah.
[133s - 134s] **Aksana Rahouski:** Okay. Well, now, now I'm-
[135s - 135s] **Devon D'Andrea:** Yeah
[135s - 136s] **Aksana Rahouski:** ... confused, like what are we-
[137s - 138s] **Devon D'Andrea:** They've gone rogue.
[141s - 143s] **Richard Sacco:** Oh, and they just took Devon out.
[143s - 149s] **Aksana Rahouski:** I was gonna say they, they're like, "This guy's onto us. Hang on." Wow.
[149s - 151s] **Richard Sacco:** His Claude agent has betrayed him.
[152s - 163s] **Aksana Rahouski:** No, I would've... Uh, the work that I think I'm thinking of was from not this week, so I don't know what it was.
[163s - 171s] **Devon D'Andrea:** Sorry about that. You know you have too many tabs open when you click on your tab and it, uh, you, the only thing you're able to click is the X out of it.
[174s - 175s] **Richard Sacco:** Too many tabs.
[175s - 176s] **Devon D'Andrea:** It's too many tabs.
[177s - 177s] **Richard Sacco:** Jesus.
[177s - 178s] **Devon D'Andrea:** Mm-hmm.
[179s - 218s] **Aksana Rahouski:** Oh, what we can ask in the chat? Uh, 'cause I'm curious. Well, the fact that it went live this week, it's just, we actually pushed Cake 4.6 upgrade to li- to production last week, which could kind of drag on some silent tick- or, I mean, we shouldn't have anything silent setting. Like, everything that we're pushing through is reviewed by you guys unless, um, unless it's some, like, um, tech debt thing that we find and we just kinda need to, like, p- p- push it up, which these are rare. Um, but we can definitely dig in to see, uh, if there was-
[220s - 220s] **Devon D'Andrea:** Right
[220s - 225s] **Aksana Rahouski:** ... it wouldn't just upgrade it on its own, that's for sure. Somebody did upgrade it.
[226s - 244s] **Devon D'Andrea:** I, uh... Yeah, no, I... That's interesting. You guys even have white labeling on it. I'm looking at one that was sent to a Meeli sub, and the whole email itself-
[245s - 245s] **Aksana Rahouski:** Uh-huh
[245s - 256s] **Devon D'Andrea:** ... is white labeled. Somebody did, somebody was doing this. And listen, I'm not, I'm not against it. I just know, you know.
[257s - 257s] **Aksana Rahouski:** No, it's-
[258s - 259s] **Devon D'Andrea:** We, you know, I, I...
[260s - 263s] **Aksana Rahouski:** We wanna know what was being pushed to production. I mean, I-
[263s - 265s] **Devon D'Andrea:** Well, I wanna know what we're paying for too.
[266s - 266s] **Aksana Rahouski:** Yeah, yeah, yeah, yeah.
[268s - 276s] **Devon D'Andrea:** Um, yeah. So all right, cool. All right. Well, we can get off of that. I still think-
[276s - 276s] **Aksana Rahouski:** Is it just, is it-
[276s - 278s] **Richard Sacco:** Which one, which one's white labeled?
[278s - 280s] **Devon D'Andrea:** I still think they're self-aware.
[280s - 280s] **Richard Sacco:** Oh, it's the Meeli.
[280s - 280s] **Devon D'Andrea:** Um-
[280s - 282s] **Richard Sacco:** Yeah, I see the Meeli logo.
[282s - 286s] **Devon D'Andrea:** Yeah. You, uh, log all the way to, or scroll all the way to the bottom of it. It's even has, like, a footer.
[288s - 289s] **Aksana Rahouski:** Can you guys share what you're looking at? I'm just-
[289s - 295s] **Laura Perry:** That might have been when, uh, Stone was working on the other invoice template updates.
[297s - 301s] **Aksana Rahouski:** But these were inwo- invoice updates, not email updates or...
[302s - 302s] **Laura Perry:** Mm.
[303s - 304s] **Devon D'Andrea:** So this is a, this is a-
[304s - 308s] **Richard Sacco:** Is that the right phone number that should be there though, Dev? It has their URL, but it's-
[308s - 313s] **Devon D'Andrea:** Um, I, I think they changed that so that they would have our number for tech support. That's, that's fine.
[314s - 314s] **Richard Sacco:** Okay.
[314s - 334s] **Devon D'Andrea:** But yeah, no, look, like, even this one is a Meeli customer, um, or sub customer I should say. And, like, it's got... So this is a regular one where it just says, "Hello, Golden West," and then it has our name at the bottom. And this is a white labeled one where it's got Meeli's... That's pretty crazy.
[336s - 340s] **Richard Sacco:** Yeah. Then at the very bottom says Meeli, our phone number, and the link.
[340s - 340s] **Devon D'Andrea:** Mm-hmm.
[342s - 342s] **Richard Sacco:** Uh-huh.
[343s - 354s] **Devon D'Andrea:** Um, yeah, and like I said, Tuesday at 11:46 was the old style, and then the very next one was, uh, a couple hours later and adopted the new style. So it sounds-
[355s - 356s] **Aksana Rahouski:** Can I see the old one? Let me see the old one.
[357s - 358s] **Devon D'Andrea:** The old one's just text.
[362s - 366s] **Adam Curcie:** So boring Really needed some more-
[366s - 366s] **Aksana Rahouski:** Wilson
[367s - 367s] **Adam Curcie:** ... razzle-dazzle there
[368s - 376s] **Aksana Rahouski:** I do think, I do think our emails are pretty dry so I look forward to that. Okay. Let me, um-
[377s - 377s] **Adam Curcie:** So-
[377s - 381s] **Aksana Rahouski:** Let me, because I'm now curious, 'cause it ta- like if s-
[382s - 384s] **Adam Curcie:** No, I'm, I'm thinking-
[384s - 390s] **Aksana Rahouski:** If somebody did some work but didn't tell, uh, we didn't, I didn't know about this one, so I don't know. I'm just super curious myself right now.
[391s - 391s] **Adam Curcie:** Yeah.
[391s - 391s] **Aksana Rahouski:** Mm-hmm.
[392s - 403s] **Adam Curcie:** I'm gonna stop sharing because I never muted my Teams messages. And some people may get really offended if they see our Teams messages.
[413s - 414s] **Aksana Rahouski:** Okay.
[414s - 417s] **Adam Curcie:** I have a quick question.
[418s - 418s] **Aksana Rahouski:** Um-
[419s - 434s] **Adam Curcie:** I, um, and we, we'll ultimately put a card in for this. Um, we found s- some of our older devices don't properly report their signal strength. Um, and then-
[434s - 436s] **Richard Sacco:** You, you put, what do you think?
[437s - 438s] **Adam Curcie:** What?
[438s - 439s] **Richard Sacco:** Did you- you put a ticket in for that.
[440s - 440s] **Adam Curcie:** What's that?
[441s - 442s] **Richard Sacco:** You already put a ticket in for that.
[443s - 446s] **Aksana Rahouski:** Yeah, for the over 100 one or something. Is that the one?
[446s - 448s] **Adam Curcie:** No, it was something else. Um-
[450s - 450s] **Richard Sacco:** Oh
[450s - 454s] **Adam Curcie:** ... yeah. It's, um... Here, let me, I think I have the device open. I can share this real quick.
[454s - 458s] **Aksana Rahouski:** Yeah, let's look. Any immediate bugs or things we need to look at?
[458s - 471s] **Adam Curcie:** No, no, no. It's, it's kinda like a feature, I guess. Like, you gotta just maybe modify, um, the way the portal determines signal. So can you see to my screen?
[472s - 477s] **Richard Sacco:** Mm, well, y- you would have to scroll in for us to see what it is you're trying to show.
[478s - 479s] **Adam Curcie:** Okay. Is that better?
[480s - 480s] **Richard Sacco:** Uh-huh.
[480s - 520s] **Adam Curcie:** So some, some of these older ones, which are the 4100s, um, with the newer firmware they don't report the signal correct. It comes in empty. But we found we can use what InHand calls the ASU value. Um, so we're just gonna have to like, may have, may, may be stuck working with that if we make the determination that we have to put this firmware on all of our 4100s. Uh, that's not something that's gonna happen, you know, Monday. Um, so, but I, I don't know what the scale is for this. Obviously we'll get that for you, but-
[520s - 520s] **Aksana Rahouski:** Mm-hmm
[520s - 558s] **Adam Curcie:** ... it's been in the devices for a long time, but only, like, for us to kind of see. It's not really something that gets reported. Um, but I imagine this is, it's at four bars, so that's, 50 might be the top of the scale. I just don't know, like, I don't really know, but I guess you guys would just have to like, mm, you know, if you see this, maybe use that instead of this. I don't know. I don't know how we'll do, uh, or w- how we'll, like, have to do it, but I just wanted to put that on your radar, 'cause I am gonna have to put a ticket in for that at some point soon, so-
[559s - 559s] **Richard Sacco:** Yeah. Just-
[559s - 559s] **Adam Curcie:** That's all
[560s - 569s] **Richard Sacco:** ... just put a ticket in. Just say for when you're processing the check-ins and you're determining signal strength, use ASU instead, or whatever you want us to do. And tell us-
[570s - 570s] **Adam Curcie:** Yeah
[570s - 575s] **Richard Sacco:** ... the deal, and we- just be sure to include that when you do the ticket and we should be good.
[576s - 577s] **Adam Curcie:** Okie dokies.
[580s - 581s] **Richard Sacco:** All right.
[582s - 587s] **Aksana Rahouski:** Okay. All right, so-
[587s - 590s] **Adam Curcie:** Oh, and how's the, uh, configurator coming?
[592s - 671s] **Aksana Rahouski:** So configurator, it's kinda sitting right now because we're trying to get a CAKE upgrade out, because that feature is gonna be so big, we don't wanna kinda start it o- while we're under construction. With that being said, there is still kind of discussion for us to keep, to keep, keep going kinda down the areas that are still fuzzy. Um, so it's on my plate to schedule something and keep going. But for now, because, like, uh, the team is, and Laura will give us an update, uh, we, uh, earlier we did tell you guys that we kinda broke our CA- CAKE PHP upgrade because we were, like, uh, multiple minor versions behind and a major version behind. So we did, like, the, the, the max minor bump, and we actually, that's what we launched on Tuesday. Um, so with that being said, like, um, if you guys see any, like, unexpected behavior, which, I mean, we would... Y- y- we've tested and we, by testing other features we've been testing, it's been sitting in actually review and beta for a while, uh, but definitely let us know. But one, after we do the major one, then we can go back to the config feature and actually start, like, uh, like, building, uh, once that kind of plate is clear. Does that answer your question?
[675s - 676s] **Adam Curcie:** Yeah. Of course.
[676s - 698s] **Aksana Rahouski:** Yeah. And, and that's one right now you'll see as kinda like these, like, smaller enhancements, right? That are kinda, we can still build. We don't wanna fully kinda sit under construction and don't do any work, so that's why we've been working on things such, um, like, all these, like, smaller features, right? Config is gonna be such a big rehaul of multiple modules-
[698s - 698s] **Adam Curcie:** Mm-hmm
[698s - 704s] **Aksana Rahouski:** ... so that's why, like, it, it would be too risky to do, like, cross it with the upgrade.
[705s - 710s] **Adam Curcie:** No, no, I remember we, you would, kinda, we all agreed that that plan made sense. Um-
[710s - 711s] **Aksana Rahouski:** Mm-hmm
[711s - 738s] **Adam Curcie:** ... and it makes sense. You know, I didn't, I get, I apologize 'cause I know I, I missed time last week, and I also was the, you know, I was not available on Monday when we were supposed to have this. So I'm, I'm thoroughly behind on things. I, I completely forgot you guys were kinda doing that PHP upgrade, so-Like, at, at least in like the regards to doing it like this week. So I'm sure you told us that, but I just... It was the last thing-
[738s - 738s] **Aksana Rahouski:** No, that-
[738s - 739s] **Adam Curcie:** ... on my mind, so
[739s - 764s] **Aksana Rahouski:** Like always, always ask though because it, it is like sitting kind of at back of my mind, but I'm trying not to distract the team at the moment with that as well while everybody's plate is already full. And on top of CACHE, CA- CakePHP upgrade, we actually have like when it rains, it pours, we have a SQL upgrade that is due July 1st. So that's another thing that kind of as a tech debt came out of, um, kind of nowhere. Richard's been working on that.
[767s - 769s] **Laura Perry:** Yeah. And just to-
[769s - 769s] **Aksana Rahouski:** Okay
[769s - 847s] **Laura Perry:** ... yeah, so overall it's like we have the, uh... Oops. Um, the CakePHP, the, like Oksana said, the minor version, that is out and, and finished. Um, Richard's been working on this MySQL like database upgrade that, um, we were made aware of just with some end of life support. Not end of life, but end of support. Um, and so we're getting that done now so that we have plenty of lead time before that July 31st deadline. Um, there's also another piece of the upgrade which is to the server and the, the PHP and moving it to 8.2. Um, that's all set up and ready to go. So next week we have the MySQL upgrade that can go to production and the server upgrade, uh, that can go to production. Um, for the server upgrade, there's no downtime. Um, so we can do that whenever we're ready. Uh, for the MySQL there will be a, uh, some downtime, so we'll wanna coordinate that with you. And Richard, I should have asked before this meeting, but how long is that downtime expected to be?
[849s - 854s] **Richard Sacco:** A couple hours max. Hopefully within a half hour sort of.
[857s - 857s] **Laura Perry:** So is there a-
[857s - 868s] **Adam Curcie:** Can we, uh... Yeah, I was just gonna state like, uh, 6:00 AM, is that an available start window? If it... 7:00 or 8:00.
[868s - 871s] **Richard Sacco:** Sure. Sure. We could do... So I could get up. Let's go-
[871s - 874s] **Adam Curcie:** How early do you like getting up some days, Richard? You know, like-
[875s - 875s] **Richard Sacco:** Um-
[875s - 878s] **Adam Curcie:** Maybe, maybe we can get some coffee delivered like-
[878s - 879s] **Richard Sacco:** No, no
[879s - 882s] **Adam Curcie:** ... like bright and early. It'll just be on your front steps and...
[882s - 889s] **Richard Sacco:** No, I, I try to get up early because I try to get up at like 5:00 or 5:30 because I try to hit the gym with my friend, uh-
[890s - 890s] **Adam Curcie:** Stay hot
[890s - 892s] **Richard Sacco:** ... on, on Monday, Wednesday, Friday. So it's okay.
[892s - 893s] **Devon D'Andrea:** That's what you're doing.
[893s - 894s] **Richard Sacco:** You might just-
[895s - 898s] **Devon D'Andrea:** Yeah, you gotta keep that, that, that, uh, that-
[898s - 898s] **Adam Curcie:** Just-
[898s - 899s] **Devon D'Andrea:** ... brain of yours healthy
[899s - 904s] **Adam Curcie:** ... you, you wake up, you hit the button to start and then go to the gym, come back, it'll be done.
[907s - 910s] **Laura Perry:** That's how all development is done. They just hit a button- ... and it goes.
[910s - 910s] **Devon D'Andrea:** Hit a button.
[911s - 916s] **Adam Curcie:** It's like that, that's how it is when I have to do a Windows upgrade. It's like, "All right, upgrade's gonna start now."
[916s - 916s] **Richard Sacco:** Go to bed scared.
[916s - 918s] **Adam Curcie:** Your computer may reboot several times.
[918s - 918s] **Richard Sacco:** Yeah.
[919s - 922s] **Adam Curcie:** Don't let it turn off. You hit the button, you go about your business.
[922s - 922s] **Richard Sacco:** Yeah.
[922s - 925s] **Adam Curcie:** Go to the store, do some dishes. You know, whatever.
[927s - 927s] **Richard Sacco:** Yeah.
[928s - 928s] **Devon D'Andrea:** Yeah.
[928s - 928s] **Laura Perry:** Um-
[928s - 936s] **Devon D'Andrea:** I, I, I mean honestly I think S- you know, and Richard, I mean if it's not an inconvenience then yeah, early, as early as you, as you can.
[936s - 942s] **Adam Curcie:** Yeah, that's all. You know, the earliest that you can, you know, accommodate is the preferred. Um-
[942s - 946s] **Devon D'Andrea:** Yeah, like if it had to be a 7, that would, that wouldn't be the end of the world.
[947s - 947s] **Adam Curcie:** Yeah.
[947s - 947s] **Richard Sacco:** Okay.
[947s - 962s] **Adam Curcie:** I mean, and, and it's also like, you know, how confident are you that it's gonna be 30 minutes versus two or three hours? 'Cause like if you're like 90% confidence, I'd be fine with you starting at 8:00. I would just prefer it to be working by 9:00, so.
[964s - 979s] **Richard Sacco:** Yeah, no, honestly if I do, I don't know, for some reason I get up late, I miss the alarm or whatever, I'll probably just be like, "Hey, let's delay the day or something." Well, you actually wanna communicate with your customers, I'm assuming. So I'll, uh, we-
[979s - 982s] **Adam Curcie:** Yeah, I would say Tuesday or Wednesday would be ideal.
[983s - 983s] **Richard Sacco:** Yeah.
[983s - 984s] **Adam Curcie:** And we'll let them-
[984s - 984s] **Richard Sacco:** Honestly-
[985s - 986s] **Adam Curcie:** We'll let them know Monday-
[986s - 986s] **Richard Sacco:** ... I don't mind either way
[986s - 986s] **Adam Curcie:** ... that's the plan.
[988s - 988s] **Richard Sacco:** Yeah, that makes sense.
[989s - 1001s] **Devon D'Andrea:** We'll, we'll put a banner up and then you guys typically, I don't know if it's something that you al- always do, but I know in the past there's sometimes there's a way where you can like-
[1002s - 1004s] **Richard Sacco:** Yeah, we'll put the page up, the offline page.
[1004s - 1005s] **Devon D'Andrea:** The offline page. Yeah, cool.
[1008s - 1014s] **Laura Perry:** So Richard, do you want to pick Tuesday or Wednesday now? Let Dina know like Tuesday would be better.
[1014s - 1018s] **Richard Sacco:** It would be on Tuesday because yeah, Tuesday.
[1019s - 1019s] **Devon D'Andrea:** Block it in.
[1021s - 1040s] **Laura Perry:** All right. So then we'll have, uh, some downtime on Tuesday for the MySQL upgrade. Um, then there's, uh, uh, Richard, what do you think about Thursday then for the server so that we have a day or two in between?
[1043s - 1047s] **Richard Sacco:** Sure. Yeah. Um, like the servers are no downtime, so-
[1047s - 1048s] **Laura Perry:** Yeah. No downtime-
[1048s - 1048s] **Richard Sacco:** But yeah
[1048s - 1049s] **Laura Perry:** ... but it would-
[1050s - 1050s] **Richard Sacco:** That's fine
[1050s - 1055s] **Laura Perry:** ... just, uh, to give that time so we know if, if anything pops up, we know that it's definitely from one thing or the other.
[1057s - 1057s] **Richard Sacco:** Makes sense to me.
[1060s - 1061s] **Laura Perry:** Okay. Cool.
[1064s - 1069s] **Devon D'Andrea:** I have one thing that kind of pertains to infrastructure.
[1070s - 1070s] **Laura Perry:** Mm-hmm.
[1071s - 1094s] **Devon D'Andrea:** Um, so I talked to Chad, I don't know if it was last week or the week before. Oksana, I believe he may have taken some time with you just to discuss the rebranding stuff. Um, I don't know how much is really gonna be necessary on the portal, um-
[1094s - 1094s] **Laura Perry:** Mm-hmm
[1094s - 1103s] **Devon D'Andrea:** ... you know, as opposed to, uh, our website, which we're gonna have-Our, um, our marketing company and Chad will be working on that.
[1104s - 1104s] **Aksana Rahouski:** Mm-hmm.
[1104s - 1143s] **Devon D'Andrea:** But, um, for the portal, I, I really doubt there's a ton of places where we would need to change the name. Let me... I, uh, but the other thing I had was, um, with a new domain for the portal, um, if there's anything, I guess, Richard, that you feel would be affected in terms of, like, any of the tunneling or network infrastructure for any of that, that we would need to work with MineSite or whatever. Just I, I, I don't know. I'm just, you know, just putting it out there once we, once we end up pointing this thing at a new URL.
[1144s - 1153s] **Adam Curcie:** No. Uh, but where are your check-ins pointed at? I think they're... Are they pointed at the load balancer IP or, I think they're pointed at the URL. So-
[1153s - 1158s] **Devon D'Andrea:** Check-ins are pointed at the URL. We don't need to change that, right? Uh-
[1158s - 1165s] **Adam Curcie:** Yeah. Not, not with any urgency. That's not... Yeah, I mean, they, they all point to apcommand.com.
[1166s - 1167s] **Aksana Rahouski:** Mm-hmm.
[1167s - 1167s] **Adam Curcie:** And we don't-
[1167s - 1167s] **Aksana Rahouski:** Mm-hmm
[1167s - 1173s] **Adam Curcie:** ... there's, there's no reason that we have to like, you know, take that out of existence.
[1174s - 1177s] **Devon D'Andrea:** Oh, okay. So it's just Allpoint wireless, not AP Command? Sorry.
[1178s - 1185s] **Adam Curcie:** No, no. None of the portal, like, where the portal is hosted and URLs customers access needs to change.
[1185s - 1185s] **Devon D'Andrea:** Just so that we-
[1185s - 1186s] **Adam Curcie:** But I'm saying that-
[1186s - 1186s] **Devon D'Andrea:** Yeah
[1186s - 1194s] **Adam Curcie:** ... for the sake of leaving the, you know, the listener, if you will, up on AP Command, that, that's fine to stay for the time being.
[1196s - 1196s] **Devon D'Andrea:** Mm-hmm.
[1196s - 1201s] **Adam Curcie:** If that's, like, uh... Yeah.
[1201s - 1202s] **Devon D'Andrea:** Just whatever the-
[1202s - 1204s] **Adam Curcie:** Think about that one for a minute.
[1204s - 1204s] **Aksana Rahouski:** Yeah.
[1204s - 1208s] **Devon D'Andrea:** Yeah. Obviously, the path of, uh, least resistance, uh, for us-
[1208s - 1208s] **Adam Curcie:** Yeah
[1208s - 1213s] **Devon D'Andrea:** ... is to not have to configure every device to send their check-ins somewhere else.
[1214s - 1224s] **Adam Curcie:** Yeah. And, and, you know, in, in regards to us owning and operating that specific URL for that purpose, that's not going to be in any way a problem.
[1225s - 1226s] **Devon D'Andrea:** Yeah.
[1226s - 1226s] **Aksana Rahouski:** Yeah.
[1226s - 1236s] **Adam Curcie:** Um, you know, the customer... I mean, AP Command, obviously, you know, we know what, why we named it that, but, um, the, you know, the complaint really h- doesn't have anything to do-
[1236s - 1236s] **Aksana Rahouski:** Mm-hmm
[1236s - 1245s] **Adam Curcie:** ... with the use of that URL specifically. It's just, you know, the way that we identify our, our business more so.
[1245s - 1259s] **Devon D'Andrea:** It's just unfortunately the name w- you know, really the only place since it's a closed portal, the only, the only place that any lawyer would care, care about is the landing login page. Um-
[1259s - 1263s] **Adam Curcie:** Yeah. Well, I mean, we do use Allpoint Command fully spelled out too.
[1263s - 1264s] **Devon D'Andrea:** That's what I'm saying. Yeah. So-
[1265s - 1265s] **Adam Curcie:** Yeah
[1265s - 1280s] **Devon D'Andrea:** ... but obviously we'll, we'll change it everywhere ex- with the exception is if the check-ins come in to apcommand.com, can you do something to redirect that to wherever you need it to go, or can it stay the same? Whatever.
[1280s - 1288s] **Adam Curcie:** Well, yeah. I mean, we can, we can... I would guess, but you can just leave it and, right? Like that.
[1288s - 1295s] **Aksana Rahouski:** How about for, for, like, your customers, are we gonna be just redirecting them to the new URL, or are you gonna just-
[1295s - 1296s] **Devon D'Andrea:** Yeah
[1296s - 1298s] **Aksana Rahouski:** ... blast them, tell them that there's-
[1298s - 1300s] **Devon D'Andrea:** Yeah, yeah. Redirects. Redirects. Yeah.
[1300s - 1300s] **Aksana Rahouski:** Right.
[1300s - 1305s] **Devon D'Andrea:** We don't wanna, we don't want to... We wanna make it easy. We're allowed to maintain the domain.
[1306s - 1306s] **Aksana Rahouski:** Okay.
[1306s - 1312s] **Devon D'Andrea:** Um, we're allowed to own it. It's just, it's all marketing based, right? It's all just-
[1312s - 1312s] **Aksana Rahouski:** Okay
[1312s - 1315s] **Devon D'Andrea:** ... visibility of the name. So, uh-
[1315s - 1315s] **Aksana Rahouski:** Okay
[1315s - 1318s] **Devon D'Andrea:** ... people that have the portal, you know, saved in their browser-
[1319s - 1319s] **Aksana Rahouski:** Okay
[1319s - 1321s] **Devon D'Andrea:** ... we want them to just get right through to the new name.
[1323s - 1323s] **Aksana Rahouski:** Okay.
[1325s - 1325s] **Adam Curcie:** Sounds good.
[1325s - 1330s] **Aksana Rahouski:** And is that temporary or is that, like, a permanent thing? Do we want-
[1330s - 1330s] **Devon D'Andrea:** So-
[1330s - 1332s] **Aksana Rahouski:** ... to gradually sunset it or?
[1332s - 1334s] **Devon D'Andrea:** It'll probably be permanent. Um-
[1335s - 1335s] **Adam Curcie:** Yeah.
[1335s - 1335s] **Aksana Rahouski:** Yeah
[1335s - 1417s] **Devon D'Andrea:** ... you know, and again, I don't know, uh, how, uh, this will... I guess it'll also pertain to white labeled URLs as well. Um, maybe. I don't know. Um, we could discuss that. So just as an idea of kind of like where we're at right now, um, two weeks ago was this quote unquote starting timer of 60 days where we're supposed to, uh, you know, change everything. Um, the, the, the other side's legal team has been, um, very difficult to get answers from. So there's a couple things we don't currently know. Um, we don't currently know how much of this has to be done within that timeframe. We don't currently know if they're going to accept the extension that our li- that our legal team asked for of 180 days. Um, and we don't currently know, this is the biggest one, if we go ahead and change everything now, if that actually means that they legally have to, um, take their, uh, you know, basically just back off.
[1417s - 1418s] **Aksana Rahouski:** Sure.
[1418s - 1429s] **Devon D'Andrea:** There, there, there actually is an unknown right now, which is if we change our name and change all of our branding, can this company still-
[1431s - 1431s] **Aksana Rahouski:** Come after you?
[1432s - 1433s] **Devon D'Andrea:** ... still, still sue us-
[1434s - 1434s] **Aksana Rahouski:** Sure
[1434s - 1435s] **Devon D'Andrea:** ... for damages.
[1436s - 1436s] **Aksana Rahouski:** Okay.
[1437s - 1442s] **Devon D'Andrea:** Which sucks. Um, the last unknown is the name.
[1444s - 1447s] **Aksana Rahouski:** Oh, you guys still don't have the name? I thought you decided on something. No?
[1447s - 1453s] **Devon D'Andrea:** We have, we have a, um, we have one in the, in the lead, but, uh-
[1453s - 1453s] **Aksana Rahouski:** Yeah
[1454s - 1457s] **Devon D'Andrea:** ... probably, hopefully gonna have it nailed down by next week. Yeah.
[1458s - 1459s] **Aksana Rahouski:** Okay. Okay.
[1459s - 1460s] **Devon D'Andrea:** So-
[1460s - 1468s] **Aksana Rahouski:** But, but you said again, like, so we'll get a new domain and we'll just redirect our customers. We are not, probably just some, like, logos
[1468s - 1471s] **Laura Perry:** Through the portal, as far as portal goes.
[1471s - 1471s] **Adam Curcie:** Portal, yeah.
[1471s - 1478s] **Laura Perry:** And actually we'll have to see, like, where we use old point command in emails as a signature or something.
[1478s - 1479s] **Adam Curcie:** Yes.
[1480s - 1487s] **Laura Perry:** So maybe, uh... Okay. But otherwise, I think this, we're not changing any, like, styles, guidelines, nothing. Colors, right?
[1488s - 1488s] **Adam Curcie:** Nope.
[1488s - 1489s] **Laura Perry:** Just kinda stay the same. Okay.
[1491s - 1503s] **Adam Curcie:** Okay. And yeah, I'm just looking through right now. There's really not, um... So logo, my settings. Yeah, we'll go, I'll go through and look myself as well.
[1508s - 1592s] **Laura Perry:** Okay. Okay. Um, well, I'm just gonna touch briefly on a few highlights. You guys are involved, so, um, I know you mostly know what's going on, but just kind of recapping that the Verizon, um, second account work went live. We're obviously going back and forth with you guys with some testing on the T-Mobile. Um, we've had a few other smaller tickets go live since we had our last check-in on the 11th. Um, so it's the invoice template redesign, the drift between Nacha and invoices, um, and this deactivated device cleanup command. Um, and not just to read this whole thing to you, the company deac- deactivation email is ready to go. Um, we'll probably get a few more tickets through here and, um, and the T-Mobile work done, and then we'll do our next push, and we'll kind of, uh, strategize that as well with the MySQL and the server upgrades. Um, but I do wanna make sure that we have time to talk about a few things. Um, and Richard, I'm gonna hand this over to you. So one was this check-in bug, um, and then two other items. So I don't know if you have anything that you wanna share, Richard, or if you just wanna talk through some questions.
[1592s - 1593s] **Richard Sacco:** No, I-
[1593s - 1594s] **Laura Perry:** The floor is yours
[1594s - 1634s] **Richard Sacco:** ... talk through some questions. Uh, thank, thank you. Um, yeah, the first thing I wanted to talk about was the check-in bug. Um, basically one of your devices came through with sort of like the, as their timestamp, an inaccurate, uh, timestamp coming in. Do you want us to take action based on that? Because how it was before where we would rely on our server to say like, "This check-in came in at this point," and we would just sort of discard the timestamp on the actual check-in, if that makes sense.
[1635s - 1670s] **Adam Curcie:** Yes. Is this, is this a cr- Adam, is this a critical check-in bug that really actually needs to do- anything to be done on like... 'Cause we determined that, like, this was probably a very rare case where somehow the check-in got sent before the time updated, and then we saw in the check-in, in the check-in, uh, export that it was just at the bottom because that export spreadsheet is sorted by date time of the check-ins. Like, what are we actually fixing here? I mean, if it's simple enough just to use the server-
[1670s - 1672s] **Richard Sacco:** Server time
[1672s - 1701s] **Adam Curcie:** ... check-in and, and just dis- I have no reason to have an objection to that. Yeah, I mean, it, the, the time that the devices report, you know, is arbitrary. You know, if you, if you're able to use the server time, then yeah, I mean, that's gonna always be more accurate than the time, you know. 'Cause a- I mean, yeah, this is very, um, rare. We never s- recall seeing this. It may have happened and customers just didn't notice it. It's the first time anyone pointed it out.
[1701s - 1701s] **Richard Sacco:** Right.
[1701s - 1714s] **Adam Curcie:** But, you know, it's... And the, the, the reason I kind of feel like it should be addressed is because if we are able to start selling more devices, um, using like cellular backup, uh, deployments, it, it could become-
[1714s - 1714s] **Richard Sacco:** Yeah
[1714s - 1736s] **Adam Curcie:** ... much more common just because the, um, you know, device's ability to pass data won't be re- like contingent on them having a cellular connection if they're sending data down another WAN interface, so it could happen more and more. So yeah, just, just use the server time if that's, if that's simple and easy to do, please, yeah, do that.
[1736s - 1779s] **Richard Sacco:** Cool. Okay, great. And then, um, the next thing I wanted to go over is I did wanna also bring up again the devices on the device export showing the $0 price. Um, what I mainly wanted to talk about there was all the, like, there's, uh, 15 abnor- normalities really. Um, 10 of them are SmartVen devices you told me to ignore. And it sounds like, it sounded like based on your responses, uh, for that one, that you just like, it's fine, you want, you don't want them to be billed anything and so-
[1779s - 1779s] **Adam Curcie:** Correct
[1779s - 1789s] **Richard Sacco:** ... that was the intention. Um, and then there was two wireless ATM store devices, and I could get you those serial numbers if you'd like, but-
[1790s - 1791s] **Adam Curcie:** No, we have them.
[1791s - 1791s] **Richard Sacco:** Okay.
[1791s - 1806s] **Adam Curcie:** I mean, anything that's wireless ATM store or test warehouse or whatever, like we probably just obliterated stuff in there just between testing and whatnot, so that's why I kind of just ignored those. Um-
[1807s - 1807s] **Richard Sacco:** Okay
[1807s - 1814s] **Adam Curcie:** ... there was one under a, on a sign. Like that, I wasn't very urgent to respond to you on the other ones-
[1814s - 1814s] **Richard Sacco:** Mm-hmm
[1814s - 1826s] **Adam Curcie:** ... because it seemed like outside of the SmartVen thing, um, it was just not anything that I was really concerned about. Um, and again, the SmartVen s- thing-
[1826s - 1827s] **Richard Sacco:** Yeah
[1827s - 1827s] **Adam Curcie:** ... is, is intentional.
[1828s - 1840s] **Richard Sacco:** You, you kind of, you know, I, I would say remedied all of the immediate concerns once, you know, you pointed out that it was the, the, the high number of devices was just kind of a symptom of the bill cycle cut-
[1840s - 1840s] **Adam Curcie:** Yeah
[1840s - 1842s] **Richard Sacco:** ... which made sense to everybody, so
[1843s - 1848s] **Adam Curcie:** You know, at that point, uh, our, I think, urgency and demand are, uh, kinda disappeared.
[1848s - 1854s] **Devon D'Andrea:** Yeah, we moved on. We moved on. Just to be sure because sometimes-
[1854s - 1854s] **Adam Curcie:** Yeah
[1854s - 1855s] **Devon D'Andrea:** ... like, I'm like-
[1855s - 1855s] **Adam Curcie:** I know
[1855s - 1857s] **Devon D'Andrea:** ... what, what, what actually happened, but.
[1857s - 1881s] **Adam Curcie:** Yeah, I'm, trust me, I'm the first one, and Adam and John will tell you, I'm the first one that'll just, you know, make something so big a- and in one second, and then the next second just be onto something completely different. Um, I told everybody today that we were all gonna lose our jobs, uh- ... 'cause we lo- we're losing a big customer, but I was just messing around, you know? But-
[1881s - 1882s] **Devon D'Andrea:** Just messing.
[1882s - 1882s] **Adam Curcie:** They call me-
[1882s - 1884s] **Aksana Rahouski:** Wait, are you, are you losing the big customer or?
[1884s - 1886s] **Adam Curcie:** Yeah, they, so-
[1886s - 1886s] **Aksana Rahouski:** Oh, okay.
[1886s - 1896s] **Adam Curcie:** We, we do not know with 100% certainty, but there's, there's more to the story that, that, you know, originally i- it was, you know, foretold. So-
[1897s - 1897s] **Aksana Rahouski:** Oh
[1897s - 1916s] **Adam Curcie:** ... there, un- unfortunately for us, we, we're very confident that to some capacity this customer will not be a customer by the end of the year, but it, it, it's not all 100% set in stone yet. We're, you know, time will tell. We did lose, um, a large customer that went out of business, but-
[1917s - 1917s] **Aksana Rahouski:** Oh, wow
[1917s - 1919s] **Adam Curcie:** ... that, that had nothing to do with us really.
[1919s - 1928s] **Devon D'Andrea:** Yeah, that was just, um, yeah, that was just a product of just poor, uh, operations and-
[1928s - 1929s] **Aksana Rahouski:** Sure
[1929s - 1939s] **Devon D'Andrea:** ... accounting on, on their part. Publicly traded Bitcoin kiosk company filed Chapter 11 and shut down about 8,000 kiosks across the country.
[1941s - 1941s] **Aksana Rahouski:** Wow.
[1941s - 1942s] **Adam Curcie:** Yep.
[1942s - 1952s] **Devon D'Andrea:** Um, yeah, they, uh, you wanna hear a funny story? So their CEO told our CEO, Repu- this is Republic, the company that we're not sure about right now.
[1952s - 1953s] **Adam Curcie:** Mm-hmm.
[1953s - 1972s] **Devon D'Andrea:** Uh, told our CEO last week, "We couldn't be happier with you guys. I apologize that we didn't come to you first. We're gonna, we're keeping all of our business with you." Um, Rick was like, our Rick was like, "This is great. Everything's fine. We, we, you know, we gave them better pricing."
[1972s - 1972s] **Aksana Rahouski:** Mm-hmm.
[1972s - 1985s] **Devon D'Andrea:** And then a week later we find out that, um, something, that something we don't yet know, triggered them to, uh, reverse their decision yet again.
[1985s - 1987s] **Aksana Rahouski:** Whoa. Interesting.
[1987s - 1988s] **Adam Curcie:** Potentially. We, we-
[1988s - 1988s] **Devon D'Andrea:** Potentially
[1988s - 1991s] **Adam Curcie:** ... again, and we don't know any of this with certainty, but that's-
[1992s - 1992s] **Aksana Rahouski:** Hmm
[1992s - 1994s] **Adam Curcie:** ... that's kind of how it's seeming, so.
[1995s - 1996s] **Aksana Rahouski:** Um, okay.
[1997s - 1997s] **Devon D'Andrea:** Yep, yep, yep.
[1997s - 2031s] **Richard Sacco:** I did have one last thing I wanted to bug you guys about, and that is, so we talked about the exports. The last one was we do have that script that goes through, checks the deactivated devices, and sees if they were deactivated last cycle. There was one anomaly I found there last time, and it was device W62620 that was, we, I found was suspended and not deactivated. So is that something that was done... What I wanna know, is it something that was done outside of the APW portal?
[2032s - 2035s] **Devon D'Andrea:** Let's take a peek. W62620?
[2036s - 2037s] **Richard Sacco:** Yep. Yeah.
[2039s - 2040s] **Devon D'Andrea:** Is that a Verizon, do you know?
[2040s - 2043s] **Richard Sacco:** Yep, it's a Verizon SIM. I could even give you the SIM number, yeah.
[2044s - 2045s] **Devon D'Andrea:** Nah, I got it.
[2045s - 2045s] **Richard Sacco:** Okay.
[2046s - 2047s] **Devon D'Andrea:** Uh, did the-
[2047s - 2047s] **Richard Sacco:** I'm gonna draw.
[2048s - 2069s] **Devon D'Andrea:** Phew. Thought I had it. Mm, mm, mm. Oh, I know what the problem is here. W62620.
[2071s - 2072s] **Richard Sacco:** Oh.
[2073s - 2220s] **Devon D'Andrea:** Yeah. My Allpoint is trying to load. I'm gonna steal that from the chat there, thank you. My Allpoint is trying to load, like, 40 serial numbers without any other filters right now. So, um, let's see here. Take that SIM and we have suspend. Logs show nothing in the last seven days. Let's do 60 days. Nine results. Last thing was Orasis deactivation failed. Orasis resume failed on the 24th of April. Um, you changed... All right, here's what you did. On... Let me share my screen. Here's our audit trail. We've got a April 3rd activation. Skip these two. Change service addresses. April 3rd, Verizon suspended it. That's probably a data trigger. And then on that day, a deactivation failed on April 3rd. Okay, so now it's still in suspend. You tried to resume it on the 11th. That's because your... Hmm. Wondering why this was a deactivation by... 'Cause this right here was Verizon suspending for data, and then it looks like you were just a little bit behind that, um, which would've been for data usage on April 3rd. However, the order type was deactivation, so not sure why that wasn't suspend, but I also don't know why it failed. Um, and then you've got your a, 11th of the month resume-That takes anything in data suspension and puts it back online for the new billing cycle. And then I don't know what any of this shit is up here. Uh-
[2220s - 2226s] **Richard Sacco:** Yeah. So basically we're not able to mess with that device at all, it seems. Oh, we did get it once.
[2226s - 2231s] **Devon D'Andrea:** No, you, you got, you got, you got the original activation-
[2231s - 2231s] **Richard Sacco:** Yeah
[2232s - 2234s] **Devon D'Andrea:** ... and you got a device group change.
[2235s - 2289s] **Richard Sacco:** Got you. Because, okay, what's happening with this device is it's still... Uh, it, it's going to be charged, but... Or no, it's not going to be charged because the deactivated thing thought it's deactivated. So just a heads up on that one. Um, but I was wondering why. I wanted to look into the deeper issue on this one and why it's not matching what, what our system thinks it is really. And it, it could be 'cause of that drift maybe where you had the data suspension and then our... And then w- the deactivate failed probably because I think it can only go, like, it can't go from suspended to deactivated, right? It has to go from active to deactivated or something.
[2289s - 2294s] **Devon D'Andrea:** So I believe it can go from suspended to deactivated-
[2294s - 2294s] **Richard Sacco:** Okay
[2294s - 2298s] **Devon D'Andrea:** ... but it can't go from deactivated to suspended. Let me see.
[2298s - 2299s] **Richard Sacco:** Okay.
[2300s - 2307s] **Devon D'Andrea:** Um, yeah. It, you can go from deactivated to suspended, just not the other way around.
[2308s - 2308s] **Richard Sacco:** Mm.
[2308s - 2317s] **Devon D'Andrea:** I'm sorry, suspended to deactivated. Not, not, you can't go from deactivated to suspended.
[2318s - 2318s] **Richard Sacco:** Right.
[2318s - 2332s] **Devon D'Andrea:** So, um, yeah. So they upgraded the service plan on April 24th, which is why we're seeing this, uh, this right here, the change service group.
[2333s - 2333s] **Richard Sacco:** Yeah.
[2333s - 2338s] **Devon D'Andrea:** That was also accompanied with a resume which failed.
[2340s - 2341s] **Richard Sacco:** Hm.
[2342s - 2351s] **Devon D'Andrea:** But then they ultimately also, they ultimately also deactivated it that day.
[2351s - 2351s] **Richard Sacco:** Which failed.
[2353s - 2374s] **Devon D'Andrea:** So 11:04, how would it change from, how would it change from the same timestamp? They changed it from... Okay, so we go, so we went data suspension to active by changing it from tier two to three, 'cause it was suspended on tier two.
[2376s - 2376s] **Richard Sacco:** Mm.
[2377s - 2382s] **Devon D'Andrea:** And then that should have resumed it.
[2384s - 2385s] **Richard Sacco:** Yeah, but it failed.
[2385s - 2388s] **Devon D'Andrea:** But it failed. And then why did it... And then it looks like-
[2391s - 2404s] **Richard Sacco:** Because I, it could be that if we detect a failure, we try to change it to its... But I don't think we call it again. We just try to change it in our system to its true status, but maybe we're unable to find its true status.
[2404s - 2404s] **Devon D'Andrea:** Yeah.
[2405s - 2415s] **Richard Sacco:** You know what? I could test on this. I could do some further tests on this specific Verizon SIM to see if we're getting, like, the right status or I don't know.
[2416s - 2421s] **Devon D'Andrea:** I'm just curious if, I'm curious if this Amber-
[2422s - 2422s] **Richard Sacco:** Mm-hmm
[2423s - 2436s] **Devon D'Andrea:** ... may have changed the s- changed, went in and changed the tier, the service plan, but then also changed the status on that. I'm not sure. Yeah.
[2436s - 2436s] **Richard Sacco:** Mm.
[2437s - 2438s] **Devon D'Andrea:** That, that's a weird one.
[2439s - 2442s] **Richard Sacco:** Oh, yeah, because you're right, it's exactly 11:04. That's what you were saying.
[2442s - 2443s] **Devon D'Andrea:** It's the same timestamp.
[2443s - 2444s] **Richard Sacco:** Oh, crap.
[2445s - 2447s] **Devon D'Andrea:** Unless she just has a quick hand.
[2450s - 2451s] **Richard Sacco:** Interesting.
[2452s - 2452s] **Devon D'Andrea:** Yeah.
[2453s - 2464s] **Richard Sacco:** Well, yeah. The, so yeah, to update you on this device, it's not going to be charged, but if you do want it to be charged, we can manually change that. But that was the one anomala- anomaly I found.
[2464s - 2465s] **Devon D'Andrea:** Yeah.
[2465s - 2466s] **Richard Sacco:** If you want, I could like put in a request-
[2466s - 2474s] **Devon D'Andrea:** No, I think, I think it says here that she changed it to deactivated. This, by the way, is the company that's, we were just talking about.
[2475s - 2475s] **Richard Sacco:** Yeah.
[2475s - 2481s] **Devon D'Andrea:** Um, so, um, we're gonna just go with that. And so that happened April 24th, so there should be no further charges.
[2482s - 2488s] **Richard Sacco:** Okay. The device is actually suspended. But yeah, that makes sense.
[2488s - 2493s] **Devon D'Andrea:** The device is suspended even though here it's deactivated. So I, so-
[2493s - 2493s] **Richard Sacco:** Yeah
[2493s - 2496s] **Devon D'Andrea:** ... okay, so what I have to do, deactivate it.
[2497s - 2497s] **Richard Sacco:** Yeah.
[2499s - 2501s] **Devon D'Andrea:** So I don't, so I don't pay for it.
[2504s - 2504s] **Richard Sacco:** There you go.
[2504s - 2505s] **Devon D'Andrea:** Cool.
[2508s - 2509s] **Richard Sacco:** Okay.
[2515s - 2517s] **Laura Perry:** All right. Is that everything you had, Richard?
[2517s - 2519s] **Richard Sacco:** Yeah, that's pretty much everything I had.
[2521s - 2565s] **Laura Perry:** Okay. Then... Let me get back up here. All right. Then we have just a quick call-out that Oksana has this ticket ready for you all to review. Um, so once you get a chance to look at that, um, we can-
[2566s - 2566s] **Aksana Rahouski:** Yeah
[2566s - 2568s] **Laura Perry:** ... move forward with that development. Anything-
[2568s - 2568s] **Aksana Rahouski:** Yeah
[2568s - 2569s] **Laura Perry:** ... more there, Oksana?
[2570s - 2603s] **Aksana Rahouski:** Yeah, and I would say for this one, we don't have to do it like now, but just this is what... Remember when we, uh, you guys s- we talked about renaming inactive and a- a deactive SIM, um, terminology to enable the outlet? 'Cause it seems like it's not only confusing us, it's confusing customers too. Uh, so this ticket just kind of covers like, um, what if effects and such. So if you guys can just quickly, uh, go no-go, and so we know what to do with it. That's as far as this one. Uh, any questions on that or immediate kinda reaction?
[2606s - 2612s] **Devon D'Andrea:** No, I just... I have a possibly just slight alteration to the tool tip text.
[2613s - 2613s] **Aksana Rahouski:** Yep.
[2613s - 2617s] **Devon D'Andrea:** Other than that, other than that, it all reads well.
[2617s - 2621s] **Aksana Rahouski:** Perfect. Uh, sounds good. Then I'll wait for you to either-
[2621s - 2622s] **Devon D'Andrea:** Mm-hmm
[2622s - 2648s] **Aksana Rahouski:** ... make an edit or just tell us if it's looks good. Um, okay. And then I wanted to kinda, uh, show you some prototypes, a directional approach on a multi-outlet power relay feature that we wanna add for our I-52 routers with this new accessory that you guys plan on selling. Before we go there, is that still happening? Or is this still like s- a product we'll plan on selling and wanna, like, support in the portal?
[2648s - 2649s] **Devon D'Andrea:** Hundred percent.
[2649s - 2650s] **Adam Curcie:** Yes.
[2650s - 2650s] **Aksana Rahouski:** All right.
[2651s - 2652s] **Devon D'Andrea:** We need a new shiny object.
[2654s - 2661s] **Aksana Rahouski:** Okay. Let me just quickly, uh... And again, for now, just kinda wanna show you a few ideas that I had.
[2662s - 2663s] **Adam Curcie:** I'm so excited.
[2665s - 2667s] **Devon D'Andrea:** Are you? Are you, Adam?
[2667s - 2668s] **Adam Curcie:** You have no idea.
[2669s - 2672s] **Devon D'Andrea:** For a thing that turns stuff on, off, and turns it back on?
[2672s - 2681s] **Adam Curcie:** This is gonna be... Like, this is gonna be like when we look back on 2026, we'll be like, "Yeah, that was when we made the outlet."
[2682s - 2682s] **Aksana Rahouski:** Okay.
[2683s - 2683s] **Devon D'Andrea:** That's good.
[2683s - 2949s] **Aksana Rahouski:** So before we kinda do go there, I wanna quickly kinda reset our... A- as I was kinda going through this, uh, I had some other ideas, and I just kinda wanna show you. So this, um, company power sche- so like company can now, um, get this feature to basically build schedules, uh, for their devices, right? That are, um, that have p- that, that they do have, like, power relay outlets. And that feature not only allows them to build a schedule and put devices on a schedule, it also allows them to, um, power them on and off on demand. So, like, putting on a schedule will happen at on a schedule timeframe. Uh, emergency kind of powers on and off are also available. With that being said, while I was kinda working on that, okay, now we have device that has, um, an accessory that has four outlets, right? And they could be using up to four. And we talked about, like, labeling these and g- giving an individual capability to power these on and off and, or restart, right? What kinda crossed my mind that I was like, when you were like... Like in this case, right? So I'm on a company view. I see the all of my devices. Uh, if I go to this device, which is right here, it, it would be actually really nice to understand if it's currently, uh, like, if let's say I powered it off or powered it on, or if it was powered on or off with an emergency, right? Like, to understand the status. Um, and so, okay, so now to kinda dive in few options. So few options here, like, again, for devi- and I just kind of fraction of the page, just the f- the s- the fastest I could get there just to, like, have something for us to chew on. So in the device page somewhere, we talked that, first of all, a model will dictate whether or not a device could have that accessory. And then, um, if it does, then on a device, we will be able, uh, uh, we need to have something like toggle that will help us to understand whether it's a singular or m- or multiple. And as you can see, like I... Even here, I'm assuming this is a single outlet device, right? Um, this is where I kinda pull through. I do think it would be helpful to understand what the power cycler state is, and whether or not it, it is on any s- schedule currently. Um, how that kind of flips into, like, if let's say this device is multiple, right? We know up to four. The only thing I think that's missing, we did talk about doing adding some kind of visual that the user can kinda visually understand how these are, um, at- attached, right? But up to four could be, uh, um, utilized, right? And we wanna give a user an ability to label them, so each one could be labeled, um, whether it's a, a jukebox or a, or a game or ATM or whatever. This one, this example shows us that three are being used. This one is not. If all four are used, then you just en- enable basically all four of them. Um, and then, yeah, from there they still can, um, label them so they understand and when, uh, attempt to, uh, again, power them on or off or restart, which is equivalent to our, um, restart power cycler functionality. And since... Now, h- here, uh, I figured because there are m- many, right? Just to make sure that it's safely done, we wanna have some kind of confirmation. Like, are you sure you wanna restart this particular outlet? Yes, no. Uh, and probably looking for an ability to do it in a bulk. Like, if you wanna restart all of them, is this in fact what you're trying to do? Um, so really kinda what I want to either this option is a grid option where it's laid out as a table with four cells, or this option is identical, but as a table. So it has these listed as a... I know today we're, our portal is very, like, table-heavy. Like, everything is displayed in the table, so this is probably more native to how portal is built today. Um, but I really kinda just want to see kinda your thoughts as far as-Directionally, a, are we on the right track? Uh, kinda did we capture everything we talked about so far? And b, which one do you like?
[2951s - 2951s] **Adam Curcie:** I like the table.
[2952s - 2963s] **Devon D'Andrea:** I like the table, and maybe, and you might even be able to, uh, in- um, incorporate the, the image using the table layout.
[2964s - 2964s] **Adam Curcie:** Mm-hmm.
[2965s - 2965s] **Devon D'Andrea:** Um, you know-
[2966s - 2966s] **Adam Curcie:** Yeah
[2966s - 2975s] **Devon D'Andrea:** ... if we could come up with some very basic crude image that has just, like, some way to show them the orientation of the-
[2976s - 2976s] **Aksana Rahouski:** Okay
[2976s - 2983s] **Devon D'Andrea:** ... of the product itself and which, which com- which, uh, which outlet pertains to one, two, three, four.
[2983s - 2984s] **Aksana Rahouski:** Yeah.
[2984s - 2989s] **Devon D'Andrea:** I don't know how, but-
[2989s - 2989s] **Aksana Rahouski:** Do you guys have a good-
[2989s - 2989s] **Devon D'Andrea:** Yeah.
[2989s - 2993s] **Aksana Rahouski:** Do you have a good image? 'Cause if you can give me a good image, then I don't-
[2993s - 2994s] **Devon D'Andrea:** We don't, we don't, um-
[2994s - 2995s] **Aksana Rahouski:** Okay.
[2996s - 3000s] **Devon D'Andrea:** Let me, let me see. Let me see. Let me see. Let me see. What was this thing called, Adam, before?
[3003s - 3003s] **Adam Curcie:** I mean-
[3004s - 3004s] **Devon D'Andrea:** I got it. I got it.
[3004s - 3004s] **Adam Curcie:** Yeah.
[3005s - 3005s] **Devon D'Andrea:** I got it.
[3005s - 3006s] **Adam Curcie:** Yeah.
[3006s - 3007s] **Devon D'Andrea:** I got it. I got it.
[3007s - 3019s] **Adam Curcie:** I will also just, just thinking out loud, Devin can smack me if he doesn't like this. Um, so by default, right?
[3019s - 3020s] **Aksana Rahouski:** Mm-hmm.
[3020s - 3024s] **Adam Curcie:** I-22s use the single outlet relay that we have now.
[3024s - 3024s] **Aksana Rahouski:** Yeah.
[3024s - 3033s] **Adam Curcie:** And by default, the I-52 will use the four-outlet relay that we're do- you know, waiting on parts for-
[3034s - 3034s] **Aksana Rahouski:** Oh, okay
[3034s - 3038s] **Adam Curcie:** ... uh, before we really get, you know, out to our customers. So-
[3038s - 3038s] **Aksana Rahouski:** Okay
[3039s - 3048s] **Adam Curcie:** ... having said that, y- y- having the option, like, in this module where customers can set these up-
[3049s - 3049s] **Aksana Rahouski:** Mm-hmm
[3050s - 3055s] **Adam Curcie:** ... y- you could, you could. You, you could run a four... Like, if... Just hear me out.
[3056s - 3056s] **Aksana Rahouski:** Mm-hmm.
[3056s - 3075s] **Adam Curcie:** If, if you're installing an I-22 and you have a four-outlet relay, you can, you can make it work. It... You only c- you can still only get one of the outlets, but, like, if that's just what the techs have and that's what they put together in the field-
[3075s - 3076s] **Aksana Rahouski:** Mm-hmm
[3076s - 3090s] **Adam Curcie:** ... and they come back here, like, maybe they could toggle, like, between, all right, it's an I-22, so by default, in this power management thing, you associate that to the single outlet, but this is actually a four-outlet one.
[3090s - 3090s] **Aksana Rahouski:** Uh-huh.
[3091s - 3098s] **Adam Curcie:** And then they're gonna have to pick, like, okay, well, I only was able to hook it to outlet, outlet one.
[3098s - 3099s] **Aksana Rahouski:** Yeah, correct.
[3099s - 3099s] **Adam Curcie:** So-
[3100s - 3100s] **Aksana Rahouski:** And th-
[3100s - 3101s] **Adam Curcie:** Yeah.
[3101s - 3114s] **Aksana Rahouski:** And that's kinda... Th- it's designed with that assumption that, like, again, we, like, the devices that we have out there today, um, they're singular, right? So they only have a single outlet.
[3114s - 3114s] **Adam Curcie:** Yeah.
[3114s - 3116s] **Aksana Rahouski:** And they get buttoned to restart.
[3116s - 3116s] **Devon D'Andrea:** Mm-hmm.
[3116s - 3129s] **Aksana Rahouski:** So this is really what I just kinda took it a step further and I was like, well, so, like, in this case, right, you would still, like, if, if it's a not four outlets, so the answer is no-
[3129s - 3129s] **Devon D'Andrea:** Mm-hmm
[3129s - 3154s] **Aksana Rahouski:** ... you would really see the same thing you see today. I just... The... I added this just to kinda show you that, like, it would actually be really nice to bring device, like, power cycler state and w- if it's on the schedule, understand, like, how... Because, like, today, right, that's discrepancy. Like, again, if I'm looking at this device, I don't know if that device is in fact sitting on a schedule.
[3154s - 3154s] **Devon D'Andrea:** Yeah.
[3154s - 3159s] **Aksana Rahouski:** And sometimes, uh, sometimes it'll be off, but it is, because right here I see it, right?
[3160s - 3160s] **Devon D'Andrea:** Yes.
[3161s - 3174s] **Aksana Rahouski:** So that's... For, for that reason, I kinda just figured s- we might as well since we'll be touching it. Like, again, i- first question on a device level is, like, is it one or is it four? If it's one, you just get one and you-
[3175s - 3175s] **Devon D'Andrea:** Yeah
[3175s - 3185s] **Aksana Rahouski:** ... do, and restart how you can do today. These other things that you're seeing are add-ons really that I'm just saying do we want them or not? Or if you are a four-outlet-
[3185s - 3188s] **Adam Curcie:** Sorry, my, my phone just had-
[3189s - 3209s] **Devon D'Andrea:** Yeah, no, I, I, I, I agree, Oksana, and just to, just to, just to throw a whole other thing out there real quick, In Hand's actually gonna make us a two-outlet version that is competitive against what we currently sell as a single outlet because our single outlet one, uh-
[3209s - 3209s] **Aksana Rahouski:** Okay
[3209s - 3220s] **Devon D'Andrea:** ... can no longer be purchased for less than about $37 a piece, so we're sourcing a new one and In Hand's gonna help us find, In Hand's gonna help us find a one or two-outlet one that's gonna be cheaper, so.
[3220s - 3223s] **Adam Curcie:** A lo- electronic salon has got a two outlet.
[3223s - 3224s] **Aksana Rahouski:** Okay.
[3224s - 3232s] **Devon D'Andrea:** Uh, yeah, no, In Hand, I talked to Ken yesterday. They're gonna source. They're gonna, they're gonna go on a world sourcing expedition for us.
[3232s - 3233s] **Aksana Rahouski:** Yeah. I, I-
[3233s - 3233s] **Devon D'Andrea:** Um-
[3234s - 3241s] **Aksana Rahouski:** I still think that we might wanna kinda treat those as two separate products because these-
[3241s - 3241s] **Devon D'Andrea:** Yeah.
[3241s - 3241s] **Adam Curcie:** Yeah
[3241s - 3242s] **Aksana Rahouski:** ... are two different, right?
[3243s - 3243s] **Devon D'Andrea:** Okay.
[3243s - 3243s] **Adam Curcie:** Yeah.
[3243s - 3253s] **Aksana Rahouski:** And especially wanna bring image and kinda show customer how things are laid out. And if, yes, if a two outpo- comes in, we will just now have option A, B, or C.
[3253s - 3253s] **Devon D'Andrea:** Yes.
[3253s - 3264s] **Aksana Rahouski:** It's, like, capable at this point, right? Yes, right now it's binary, yes and no, but it doesn't have to be binary. If we have more than two option, it can be m- uh, like, um, just pick one, right?
[3264s - 3265s] **Devon D'Andrea:** Yes.
[3265s - 3265s] **Aksana Rahouski:** Um-
[3266s - 3274s] **Adam Curcie:** Yeah, and also, like, one of the main reasons we need to make sure we keep them separate is because the API calls are all-
[3275s - 3275s] **Aksana Rahouski:** Exactly
[3275s - 3276s] **Adam Curcie:** ... backwards-
[3276s - 3276s] **Devon D'Andrea:** Yeah
[3276s - 3277s] **Adam Curcie:** ... between the two. That's-
[3277s - 3278s] **Aksana Rahouski:** Very factual. Exactly.
[3278s - 3280s] **Adam Curcie:** Kinda throws a big monkey wrench in it, so.
[3281s - 3325s] **Aksana Rahouski:** Exactly, because when we're start sending these, like, restarts or process check-ins, right? 'Cause we have these, like, race and conditions when... I think check-in or, uh, I mean, I'll have to, like, re- re- refresh my memory, but when we s- like, update these last statuses, we in fact are kinda relying on the information that we're getting, I think. But, um, as far as I remember.Speaking of, so this and then so, like, next area to rework for us would be a... Okay, if we have, like, a schedule manager, right? Uh, now we're saying before a device could p- be put on a schedule, and now we have this new concept of the device with multiple outlets-
[3325s - 3325s] **Devon D'Andrea:** Mm-hmm
[3326s - 3429s] **Aksana Rahouski:** ... can be either all outlets. It- it's like we're getting more dimension, right? We're, like, now one to many. For each of these outlets could be put on a schedule, and I think this, this was, like, one of the ideas that I had of how we could kind of bring it into existing, um, co- concept, um, and kinda just, uh, build on top of it, right? So you still add a device. Um, well, you have a device, but then you set up it on a, like, um, an outlet level, like whether it's which sched- That way you could put it on a different schedules if you need to. You could put all of them on a schedule. And then your schedule, like w- today, schedule just says how many devices are on it. Um, if I go here, uh, it just says two devices, right? Now we need to start counting number of devices and number out- uh, and outlets. Uh, so it's, a- again, there are a little bit probably more chewing I need to do on it. But I thought this would kinda, without, like, major rehaul, this just kinda expands the concept of you can either put a device or a device that can have multiple outlets. You could put a device outlet on the schedule and also, uh, still apply your, like, emergency on off actions and et cetera. Um, so I know it was just kinda, like, a lot of gum, but again, I just wanted to kinda, A, align that, yes, this is still something we wanna, like, uh, work on. And I, um, I'm, like, wrapping up product requirements document for this, and just kind of, again, wanted to visually kinda align us on d- like, direction, if we're directionally aligned, and you guys are happy with this, and we should proceed.
[3433s - 3437s] **Devon D'Andrea:** Can you do me a favor? Can you go back, uh... Can you go right there?
[3438s - 3438s] **Aksana Rahouski:** Mm-hmm.
[3441s - 3443s] **Devon D'Andrea:** Hold, hold that one second.
[3444s - 3467s] **Aksana Rahouski:** Mm-hmm. These are also, I'll say they're just the HTMLs. I can just email them to you. You're free to, uh, open it and just to go if you wanna, like, sink in it for a second and see what else we're maybe missing or...
[3472s - 3478s] **Devon D'Andrea:** Yeah. Um, let me just see something real quick. Hold on.
[3478s - 3478s] **Aksana Rahouski:** Mm-hmm.
[3482s - 3499s] **Devon D'Andrea:** Where is that? Copy. So, like, this is... I, I can't, I cannot believe that I'm about to do this, Adam, but you're gonna be-
[3499s - 3499s] **Adam Curcie:** What?
[3499s - 3500s] **Devon D'Andrea:** You're gonna be proud of me.
[3502s - 3503s] **Aksana Rahouski:** Wanna share something?
[3505s - 3509s] **Adam Curcie:** Is it paint? Oh, it's paint. Look at this guy.
[3509s - 3514s] **Devon D'Andrea:** I don't know. Maybe... So I know this is, like, it doesn't have the greatest resolution, but, like, do you think we could do it something like that?
[3515s - 3515s] **Aksana Rahouski:** Well, yeah.
[3515s - 3517s] **Devon D'Andrea:** Where I sent this to you, Oksana-
[3517s - 3517s] **Aksana Rahouski:** Okay
[3517s - 3519s] **Devon D'Andrea:** ... so you can put it in, like, professionally.
[3520s - 3522s] **Aksana Rahouski:** Yeah. I think this is definitely-
[3522s - 3522s] **Devon D'Andrea:** I'm just-
[3522s - 3530s] **Aksana Rahouski:** ... uh, what I would do. You know, I would take this image and maybe, like, um, each outlet you kinda, like, s- make it stand out, whether it's borders-
[3531s - 3531s] **Devon D'Andrea:** Mm
[3531s - 3546s] **Aksana Rahouski:** ... or something. We can rework. And that way you kinda understand visually that this represents, um... We also probably, like, what would be even cooler, 'cause keep in mind too, like, is, is these the-
[3546s - 3547s] **Adam Curcie:** I envisioned it-
[3548s - 3548s] **Aksana Rahouski:** Right
[3548s - 3552s] **Adam Curcie:** ... like, Devin, I, I envisioned if you flip it 90 degrees, this would be like-
[3552s - 3556s] **Devon D'Andrea:** Well, then we... Then you'd have to do the reverse table.
[3557s - 3564s] **Adam Curcie:** No, you just... No, no, you don't reverse the table. You just label it. You just, like, if you flip it 90 degrees, like, move, move the table down a little.
[3571s - 3572s] **Devon D'Andrea:** I hear what you're saying.
[3573s - 3573s] **Aksana Rahouski:** Yeah. And it's-
[3574s - 3579s] **Adam Curcie:** And then it's, yeah, outlet one, outlet two, outlet three, outlet four. You could even, you know.
[3579s - 3580s] **Aksana Rahouski:** Yeah. And we can, again-
[3580s - 3583s] **Adam Curcie:** Like, I would put it on... I would put it above the table. I would-
[3583s - 3584s] **Aksana Rahouski:** Mm-hmm
[3584s - 3589s] **Adam Curcie:** ... you know, select the table, drag the table down a bit. You're gonna have to... Yeah. See, this is-
[3589s - 3590s] **Devon D'Andrea:** Or, like, somewhere in-
[3590s - 3591s] **Adam Curcie:** ... this is why I do, this is why I do the paint.
[3592s - 3592s] **Devon D'Andrea:** Yeah.
[3592s - 3593s] **Adam Curcie:** Dev doesn't know what he's doing.
[3593s - 3594s] **Devon D'Andrea:** Yeah, I don't do this shit.
[3596s - 3597s] **Aksana Rahouski:** I have some ideas-
[3597s - 3597s] **Adam Curcie:** So-
[3597s - 3608s] **Aksana Rahouski:** ... what we can... Again, we could kinda label these on the image to map it with the count on the grid, right, or on the table-
[3608s - 3608s] **Devon D'Andrea:** That would be-
[3608s - 3612s] **Aksana Rahouski:** ... so that way user knows exactly kinda which cell represent what.
[3613s - 3624s] **Devon D'Andrea:** Yeah. I don't know. I mean, it... If you... The image that I sent you, I don't even know what size it is. It's 57 kilobytes. I, you know, just for purposes of-
[3624s - 3625s] **Aksana Rahouski:** It's okay. Yeah
[3625s - 3625s] **Devon D'Andrea:** ... designing.
[3625s - 3627s] **Aksana Rahouski:** That's exactly what I need. Yep. Mm-hmm.
[3627s - 3631s] **Devon D'Andrea:** Yeah. I, you know, or I can try to find something better, but...
[3633s - 3683s] **Aksana Rahouski:** Well, let's do this then, that I can, um, I'll finish product requirements doc and get you guys to kind of read through, see if it's... Because it's actually, that, like, spells out all the details as far as functional requirements and et cetera. 'Cause we also talked, it talks about how we agreed to move, uh, power cycler enable, disable from a config group level, which totally makes no sense for, to keep it there into the model, especially now 'cause we have model management space for the admin, where admin can actually configure which models can have it, right? Uh, so that includes that and a few other things. Um, so, and then, yeah, if everything looks good, then we'll just size it and put it on the backlog.
[3685s - 3685s] **Devon D'Andrea:** Sweet.
[3686s - 3686s] **Adam Curcie:** Good.
[3688s - 3690s] **Aksana Rahouski:** I think that's, that's all that we had for today.
[3690s - 3704s] **Adam Curcie:** The other thing I wanted to just s- throw out, uh, right before my phone's internet connection decided to just poop out on me, um, uh, a I-52 could, in theory, connect to a single outlet relay that we have now. So yeah-
[3705s - 3705s] **Aksana Rahouski:** Mm-hmm
[3705s - 3705s] **Adam Curcie:** ... just wanted to-
[3706s - 3706s] **Aksana Rahouski:** Okay
[3706s - 3708s] **Adam Curcie:** ... just wanted to throw all that in there.
[3708s - 3709s] **Aksana Rahouski:** Okay.
[3709s - 3717s] **Adam Curcie:** If we're g- if customers n- just l- you know, if that's the circumstance and that's how they install stuff, just make sure we got all of our, you know, bases covered, so.
[3718s - 3718s] **Aksana Rahouski:** Mm-hmm. Mm-hmm.
[3718s - 3723s] **Adam Curcie:** Um, but that's it. I was trying to get that all out earlier and then, like... Yeah, but anyway, so.
[3724s - 3725s] **Aksana Rahouski:** Okay. Sounds good.
[3725s - 3730s] **Devon D'Andrea:** But that, but that would be like... But that would just work inherently though, right?
[3730s - 3731s] **Aksana Rahouski:** Yeah. Yes. Yeah.
[3731s - 3736s] **Adam Curcie:** No. No. No, it wouldn't. It wouldn't because it's not gonna be in the correct spot.
[3737s - 3747s] **Aksana Rahouski:** Well, but no, I think what we're saying, it's not, it's not that we treat one as your one of four. We, it's, it, we treat them as two separate products.
[3747s - 3747s] **Adam Curcie:** Yeah.
[3747s - 3750s] **Aksana Rahouski:** You either have one, or you can have four.
[3750s - 3750s] **Adam Curcie:** Yeah, but-
[3750s - 3752s] **Aksana Rahouski:** You can choose one with-
[3752s - 3755s] **Adam Curcie:** It would, it would be different API calls though-
[3755s - 3755s] **Aksana Rahouski:** Yeah
[3755s - 3758s] **Adam Curcie:** ... because in, you know-
[3758s - 3758s] **Devon D'Andrea:** Different
[3758s - 3770s] **Adam Curcie:** ... I-22 it's DIO one. Yeah, this would be DIO four that would have to on, on the 52 that would connect to the, uh, uh, uh, yeah. We'll, we'll go over it.
[3770s - 3770s] **Aksana Rahouski:** Yeah.
[3770s - 3772s] **Devon D'Andrea:** Well, that's for us to figure out.
[3772s - 3773s] **Aksana Rahouski:** Yeah, let's just, yeah.
[3773s - 3773s] **Adam Curcie:** Yeah.
[3773s - 3775s] **Aksana Rahouski:** I think we'll, we'll have to probably-
[3775s - 3781s] **Adam Curcie:** But it wouldn't inherently work as is. I, I mean, it, it'd be a lot closer, but it still wouldn't be identical.
[3781s - 3783s] **Devon D'Andrea:** Yeah, you're right. I think we did test that and we didn't-
[3784s - 3784s] **Adam Curcie:** Yeah
[3784s - 3786s] **Devon D'Andrea:** ... we couldn't get it to work just as is.
[3787s - 3797s] **Adam Curcie:** No, 'cause you gotta put it on... 'Cause the comm's on the left, so you gotta put it on comm and then four and three, DIO four and DIO three. You can't put it on DIO one-
[3798s - 3798s] **Devon D'Andrea:** Right
[3798s - 3799s] **Adam Curcie:** ... or two.
[3799s - 3799s] **Devon D'Andrea:** So the-
[3799s - 3800s] **Adam Curcie:** Yeah
[3800s - 3804s] **Devon D'Andrea:** ... right. So the portal today would send it, would send the change to IO one.
[3805s - 3805s] **Adam Curcie:** Yeah.
[3805s - 3806s] **Devon D'Andrea:** Okay. I gotcha.
[3806s - 3810s] **Adam Curcie:** And then, yeah. Moving... It would have to be on four, I, I believe.
[3810s - 3810s] **Aksana Rahouski:** Yeah.
[3810s - 3814s] **Adam Curcie:** But we'd have to double check it. So w- but again, not something we gotta worry about now.
[3815s - 3815s] **Aksana Rahouski:** Yeah.
[3815s - 3819s] **Devon D'Andrea:** We can a- we can add that requirement, and that could even be a separate ticket-
[3820s - 3820s] **Aksana Rahouski:** Wow
[3820s - 3820s] **Devon D'Andrea:** ... altogether.
[3821s - 3843s] **Aksana Rahouski:** Yeah, and I think when we s- when we start talking about actual API calls for these outlets, I mean, there's pro- there'll probably more discussions that we'll have. But for now, I think, uh, uh, all I just need to know that, like, directionally we're, like, on the right aligned and, uh, we're, this is still relevant, that nothing changed on your end, that it's not needed anymore.
[3846s - 3846s] **Devon D'Andrea:** Correct.
[3846s - 3849s] **Aksana Rahouski:** Um, well, sounds good then.
[3852s - 3852s] **Adam Curcie:** All right.
[3853s - 3853s] **Aksana Rahouski:** Yeah.
[3853s - 3853s] **Adam Curcie:** All right.
[3853s - 3853s] **Aksana Rahouski:** You guys-
[3854s - 3854s] **Laura Perry:** Thanks, everybody.
[3855s - 3856s] **Aksana Rahouski:** Thank you.
[3856s - 3856s] **Adam Curcie:** Thank you.
[3857s - 3857s] **Laura Perry:** Happy next week.
[3858s - 3858s] **Aksana Rahouski:** Bye.
[3858s - 3858s] **Laura Perry:** Bye.
[3858s - 3859s] **Devon D'Andrea:** Thank you. Bye.
