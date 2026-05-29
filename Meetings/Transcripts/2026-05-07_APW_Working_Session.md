# APW Working Session — Transcript

**Date:** May 7, 2026, 2:30 PM UTC
**Duration:** ~21 minutes
**Organizer:** Laura Perry
**Attendees:** Aksana Rahouski, Laura Perry, Noah Bratzel, Richard Sacco, Stone Marballie, Aaron Diefes

**Source:** tldv recording
**Meeting URL:** https://tldv.io/app/meetings/69fca1e0c1e4cf0013509dcd

---

**Aksana Rahouski** [0:00]
It... Well, that's interesting. That's- But anyway, so, like, let's say it was, it's 20, I wanna make it 35 and 150 dollars. Right? So, like, these are the price I'm setting. So I hit save, and if I go back to this plan, I do not see... I see the difference, but I don't see the value. Like, it's not showing me. It's showing me that, that it's plus 15 and plus 50, but it's not 150 or 115. And I, I have no idea at what point or why this is happening, but it is not how the system behaves in higher environments. Well, I guess to take it a step further, we can validate what beta has, because if beta is broken, it means it's not a CAKE upgrade. Just to narrow it down to what's causing it.

**Noah Bratzel** [1:10]
Yeah, I have no idea. I, I'm very confused about how these work, whatever. So if we are listening to this bug, hopefully be very clear about what's expected and what is happening, 'cause this is completely... I know that there's weird stuff that, that this does, but I don't understand what it's supposed to be doing.

**Aksana Rahouski** [1:27]
My... I mean, it, w- what it, it... What's not clear about what prod is doing? I guess that's what... As far as kinda clarity on what's expected. It's the... If I re- if I set price to 20, I see 20. I don't see, uh, 12 or 11.15. That's what I'm saying. Does that-

**Noah Bratzel** [1:47]
Are you sure there's nothing else involved?

**Aksana Rahouski** [1:50]
Yes.

**Noah Bratzel** [1:52]
I'm-

**Aksana Rahouski** [1:52]
Well, Stone is here. He-

**Noah Bratzel** [1:54]
I'm not, I'm not sure. I'm not sure that there's nothing else involved. That's what, that's what I'm saying, is, like, this is not... The, the change, a change in behavior like this is not going to be because of a, of a CAKE upgrade. It's because there might be some different setup. I don't know. I don't know. I'd have to dig in. But what I'm saying is I don't think that that's all-

**Aksana Rahouski** [2:10]
Yeah

**Noah Bratzel** [2:10]
... that's happening. There's some other rules that we're forgetting about.

**Stone Marballie** [2:13]
Oksana, could you show Stone again? 'Cause, like-

**Aksana Rahouski** [2:16]
Yeah. Hold on

**Stone Marballie** [2:16]
... Stone obviously worked on it. Yeah.

**Aksana Rahouski** [2:18]
Oh, sorry.

**Noah Bratzel** [2:19]
May- maybe there's-

**Aksana Rahouski** [2:20]
Let me do that

**Noah Bratzel** [2:21]
... inheritance from distributors, something else. I don't know.

**Aksana Rahouski** [2:25]
Could be. Um, but, but Stone is here. We can... He, he worked on it prior, so he might have a little bit more understanding into this layer. But I wanna see, though, how beta behaves because... Just to narrow it down to what's causing it. Okay. Um, 10, 9.

**Stone Marballie** [3:13]
This is only a persistent bug on review, right?

**Aksana Rahouski** [3:18]
Can I change that to 9 too? Hold on. That was a weird one, but let me see quickly. Yeah, so it is review only, which makes me believe that-

**Noah Bratzel** [3:31]
Is this the same company?

**Aksana Rahouski** [3:33]
It doesn't matter if it's the same company or not.

**Noah Bratzel** [3:36]
It, it matters, but the, the same company or not, for sure.

**Aksana Rahouski** [3:39]
I mean, it... Not, not really, because service plan has nothing to do with the company.

**Noah Bratzel** [3:47]
It, it has something to do with what company is inheriting from what other company. And so if you have a different setup of in- inheritance... Stone, is there something else involved-

**Aksana Rahouski** [3:57]
It's on, um-

**Noah Bratzel** [3:57]
... besides the base prices on here? Or do you, do we inherit from other companies when you have, like, on Oris' test company, for example? Or is there other stuff going on?

**Stone Marballie** [4:08]
Well, Oris' test company is a distributor, right? So it should only be-

**Aksana Rahouski** [4:12]
Oh, it's-

**Stone Marballie** [4:12]
... inheriting from base.

**Aksana Rahouski** [4:14]
Yes.

**Stone Marballie** [4:16]
Only sub-companies have the second layer inheritance.

**Aksana Rahouski** [4:19]
Okay.

**Noah Bratzel** [4:19]
The-

**Aksana Rahouski** [4:19]
So the bottom line, and also, like, uh, what I'm trying to do, I'm trying to help us to narrow it down to what could be causing it. Stone, I know you joined a little bit later, so here's what we're seeing. Custom service plans, uh, when you... And I kinda accidentally came across this. When I, um, create a custom service plan for a company, so let's say this is a custom service plan. Base price is 20, and I set, set this to, uh, 30 and 110. So I'm bumping $10, $10. I'm hitting save. When I go back to that plan in review, I see the difference, not the full price. So I see that it's, the price is 30. It's not 10. But it's just showing me the difference, 30 minus 20. It's showing me 10 for some reason, only difference in this case. So then, so then we start talking about kinda... Well, I was like, "Did it always behave like this?" So I went to prod. This is beta.

**Stone Marballie** [5:33]
Hold on.

**Aksana Rahouski** [5:33]
Prod shows-

**Stone Marballie** [5:34]
Hold on. Go back again. W- so what did you... What was the first thing you did there? You updated the prices on the main plan itself, the service plan?

**Aksana Rahouski** [5:43]
No. This, this is custom service plan.

**Stone Marballie** [5:47]
Okay. So this is the, the custom plan for the, for the, um, for the company?

**Aksana Rahouski** [5:52]
Yes.

**Stone Marballie** [5:53]
Okay.

**Aksana Rahouski** [5:54]
And we can even, like... Let's just forget about this one. Let's create a brand-new one, where we'll take any other company. Who else do we have who is a distributor? Let's say Bad Company. That company I wanna give a, build a custom for FWA. Continue. Here.

**Stone Marballie** [6:15]
Mm-hmm.

**Aksana Rahouski** [6:16]
What is that? This is-

**Stone Marballie** [6:19]
So when it shows it in red, it means that the parent plan has the, um, these custom changes, but it hasn't been applied to the child yet, right?

**Aksana Rahouski** [6:31]
Yeah. Well, it also, it also now is completely screwed up because look at this. Custom s- service plan. I go to service plan. Why is it not loading? Okay. Service plans. Let's find our service plan. FWA, that plan cost $20 and 100 and, and $100. Why am I seeing 30 and 110? Unless that company is a sub-company of Ora's test company. Is that what's happening? Okay, bad example. It's cool.

**Stone Marballie** [7:32]
This is true.

**Aksana Rahouski** [7:34]
Okay, so let's take another... Give me another that is not... Aaron, do you have anybody memorized in review that is-

**Stone Marballie** [7:47]
I mean, uh, like I could just-

**Aksana Rahouski** [7:48]
Yeah, I can do anybody, right? Like, I could do this.

**Stone Marballie** [7:50]
Yeah.

**Aksana Rahouski** [7:52]
Okay, so this guy. We'll take this guy, and we'll do FWA. $20 base, right? $20. And for this company, I'm gonna be charging them 30 and 110 on this plan. I wanna up the price for this company. So if I save, um, company ID, then go back.

**Stone Marballie** [8:23]
Mm-hmm.

**Aksana Rahouski** [8:23]
Here's what I see. I don't see 30, I don't see 110. I see the difference.

**Stone Marballie** [8:35]
Yeah, that should, that should have, uh... That's not the behavior that we, we had designed for, right? It should have the new price, the adjusted price.

**Aksana Rahouski** [8:44]
Right. Uh, so that's where we're trying to figure out kinda-

**Stone Marballie** [8:49]
Yeah

**Aksana Rahouski** [8:49]
... what it, what's causing it.

**Stone Marballie** [8:52]
Yeah, so it's probably a database thing where it's looking for the wrong table, so it could be an ORM mismatch because the way these tables were linked was not the greatest design, if I remember. This is one of those complicated ones where I have to go back-

**Aksana Rahouski** [9:10]
Yeah

**Stone Marballie** [9:10]
... and look at it every time 'cause it-

**Aksana Rahouski** [9:11]
Yeah

**Stone Marballie** [9:12]
... it's not clear.

**Aksana Rahouski** [9:13]
Yeah.

**Stone Marballie** [9:13]
But I have to admit-

**Aksana Rahouski** [9:14]
Yeah. 'Cause it was, like, saving... 'Cause I remember it was something instead of saving it absolute value, it saves- Status ... the gap.

**Stone Marballie** [9:24]
Yeah.

**Aksana Rahouski** [9:24]
Right? Or something like that. I remember it was, like, some- something so, like, strange about how it's designed that...

**Stone Marballie** [9:32]
Yeah, it's showing the adjusted price versus-

**Aksana Rahouski** [9:34]
Yeah

**Stone Marballie** [9:34]
... you know, mm, oh, yeah.

**Aksana Rahouski** [9:36]
The, the price. Okay.

**Stone Marballie** [9:38]
Oh. Yeah, if you give me a chance, I can look into this one. Is this related to the same bug that I was assigned?

**Aksana Rahouski** [9:47]
I don't know what bug you were assigned, I guess. Um, I, I don't, I d-, I, I'm, I came across it, and I think, again, just to kinda narrow it down, beta doesn't have it, and-

**Stone Marballie** [9:59]
Okay.

**Aksana Rahouski** [9:59]
... prod doesn't have it. So that's what makes me think it's a Cake upgrade that's caused-

**Stone Marballie** [10:02]
Yeah

**Aksana Rahouski** [10:02]
... so we need to figure out why, right? Um-

**Stone Marballie** [10:05]
Yeah

**Aksana Rahouski** [10:05]
... let's figure out who takes this because this is obviously something that we need to fix before we proceed with, um...

**Stone Marballie** [10:13]
Yeah, just create a ticket for it, and, um, I can take a look. But there is, um, I'm glad it's not in beta or prod, so it's not something we have to worry about right away, but it's something that we need to worry about before we make the upgrade, right? And-

**Aksana Rahouski** [10:33]
Before we start rolling it out, yeah.

**Stone Marballie** [10:34]
Yeah. But, I mean, I guess, uh, Noah, you've, uh, I guess in your, uh, deed of... I need to... Sorry, I need, I need to run. I have a child I have to go pick up apparently who's from school, so I'll have to pick up with you guys later. Hang on, re- re-

**Aksana Rahouski** [10:50]
I'm good.

**Stone Marballie** [10:51]
All right.

**Aksana Rahouski** [10:51]
All right, sounds good.

**Stone Marballie** [10:52]
See you guys.

**Aksana Rahouski** [10:53]
Hope everything's okay. Bye. Bye.

**Stone Marballie** [10:54]
Me too. See you.

**Aksana Rahouski** [10:57]
Um, okay, so I could create a ticket for it. Uh, Stone, are you on APW or...?

**Stone Marballie** [11:05]
Yeah, I've got the two things, um, the invoice thing, and there's a ticket in prod that, um, Richard wanted resolved before they ran their billing tomorrow, so I was gonna try to flush those out.

**Aksana Rahouski** [11:17]
Yeah, so, so Stone's been on APW yesterday afternoon through, like, this morning, and then he needs to get back to E-tank for a bit. So it would be next week, um, before he could look at this.

**Stone Marballie** [11:33]
Yeah. But it's also a good sign because, um, like, um, it'd be good to, for us, the devs, to have a- another build with the Cake upgrades in a-

**Aksana Rahouski** [11:44]
Uh-huh

**Stone Marballie** [11:45]
... different en- an isolated environment that we can kinda check out branches and see how, you know, how things shake out.

**Aksana Rahouski** [11:55]
Yeah. And I'm thinking, so the only thing that... But I guess we just validated that that's not the case. I guess for beta, let's see. Because if they come back, if they say- Horizon second must go now, and if this is in beta we can't go because we're not shipping this bug to prod. Um, let's see. If we create something on beta just to kinda, uh, something. Kinda wanna copy some data. Something. Okay. Let's search. Go ahead. You get... Oh, they already have this page. Ugh, I don't have tiers. Tier two. Continue. Okay, great. Let's just bump this up. Wow. Just, um, one gig. Leave the rest. One gig.

**Richard Sacco** [13:14]
Hold on, I'm done.

**Aksana Rahouski** [13:15]
Okay, beta's fine. Uh, okay. Okay, um, sounds good. So let's just create a ticket and we need to, like, take a look at that.

**Richard Sacco** [13:28]
Yeah. Let's create the ticket and you can toss it to me and I'll drill into it next week when the smoke's cleared over it, do you think? Yeah.

**Aksana Rahouski** [13:36]
All right, sounds good. Um, is there anything else you guys wanted to discuss today? Um, Laura, I was curious, uh, the, the test that you ran, you posted the comments on, in APW last week, how you found all these, like, 500 pages. How exactly did you test this? Was just curious.

**Laura Perry** [14:03]
Uh, the errors were, were ones that Claude came back with. So I gave, um... I just told Claude kinda like, just like a, you know, I'm clicking around trying to find, you know, basic errors, blah, blah, blah.

**Aksana Rahouski** [14:20]
Mm-hmm.

**Laura Perry** [14:20]
And gave it Noah's file that he created, and then told it to use the Chrome extension-

**Aksana Rahouski** [14:28]
Mm-hmm

**Laura Perry** [14:28]
... and to, like, see what it could find based on, based on those. Um, and the five bugs-

**Aksana Rahouski** [14:37]
Mm-hmm

**Laura Perry** [14:38]
... which I s- I saw that Noah did some updates yesterday. I haven't looked at all of them. I think, uh, at least a few, if not all, but, like, one or whatever, were not really, either not related to Cake or not really bugs at all. Um, but anyway, it, it was just Claude's extension was like, "Oh, I found these 500s," and-

**Aksana Rahouski** [15:00]
Mm

**Laura Perry** [15:00]
... I asked it to create bugs and then that's what we got.

**Aksana Rahouski** [15:05]
Gotcha. I'm curious, this, 'cause this URL, right, that it... Did you guys, did you guys either see what she posted?

**Richard Sacco** [15:19]
Um, I didn't take a look.

**Aksana Rahouski** [15:21]
Or did this-

**Richard Sacco** [15:21]
I'm just sort of, uh, imagining that it's because of the way the forms are done, where it's like they're not really, like, the, the parameters don't show up in the, like, a get request. That's just how, always how, um, it's been. So it basically goes through a post request, while actually some forms do use a get request, so it's a little inconsistent in there. Um, so I'm as- I'm assuming that the form is just like, "Hey, when I do this post and I try to go back to this thing, it's not... I'm getting a 500." That's what I would assume without... I haven't looked at it though. I should probably look at it, but.

**Aksana Rahouski** [15:58]
Yeah, 'cause I kind of followed through and these are not valid URLs that were even buil- And that's why I was, like, curious where... I could imagine from, like, a code base, maybe we have some controllers that are, like, building out these URLs, but that was not done through the code. It's literally done through just, like, go hit this website and... So I don't know if you're just, like, randomly generating these URLs, or how, how does it... I was just curious.

**Laura Perry** [16:27]
Uh, I mean, I can go back to... 'Cause, like, in the conversation that I had, it did give some more, like, FYIs and, and such, I think. So, I mean, like, I could drop those into Slack if-

**Aksana Rahouski** [16:42]
Mm-hmm

**Laura Perry** [16:42]
... you're, or, you know, if you're curious. But-

**Aksana Rahouski** [16:46]
Yeah

**Laura Perry** [16:46]
... to me it was just kind of like, this is a lot of extra noise and I'm just gonna let it create the bugs and pull in what it says is important, and leave it at that.

**Aksana Rahouski** [16:55]
Mm-hmm. Yeah. Yeah, maybe if you just wanna drop it just to me. I'm just curious what else it told you when it did it.

**Laura Perry** [17:05]
Okay.

**Aksana Rahouski** [17:06]
Okay. Richard, did you have a chance to do any testing? 'Cause I know last time we talked you said maybe you'll spend some time also kinda doing similar, I think. You were just kinda go and send-

**Richard Sacco** [17:16]
Yeah

**Aksana Rahouski** [17:16]
... Claude to hit different pages.

**Richard Sacco** [17:18]
I mean, I could still do that, but I, I pretty much thought like, oh, Laura's doing it, so-

**Aksana Rahouski** [17:23]
Yeah, that's what it sounds like, Laura, that... Okay.

**Richard Sacco** [17:25]
Yeah.

**Aksana Rahouski** [17:26]
Yeah. I wonder, like, 'cause it's, like, things like with custom service stamp that I just came across, right? That's a bug, but this is something we can't, we can't catch it without actually paying attention and looking what, and tracking.

**Aaron Diefes** [17:44]
I'm, like, I'm very curious to see, 'cause, like, that's a, uh, functional bug which means, like, Claude might not be testing function and really just only hitting URLs, right? So, like, if we're not testing any function whatsoever, then, like, I guess what's, what's the... Like, y- you know, like, there's so much that's not-

**Aksana Rahouski** [18:03]
I mean, and-

**Aaron Diefes** [18:04]
... just direct

**Aksana Rahouski** [18:05]
... yeah. And you still have to test that we don't get any, like, broken pages, right? And every page loads.

**Aaron Diefes** [18:13]
Yeah.

**Aksana Rahouski** [18:13]
Page that we expect to load, and we're not getting, like, 500s. But yeah, the question of whether that page loads with the right information, so functional requirements are executing properly, that's a, that's-

**Aaron Diefes** [18:26]
Yeah

**Aksana Rahouski** [18:26]
... that's not something you will be able to validate with that kind of test.

**Aaron Diefes** [18:31]
Yeah.

**Aksana Rahouski** [18:32]
Which honestly, that's why, like, the best way is just to kinda... It's c- it's, it's good that we're testing other things, and while we're testing other things, we are finding, like, pay attention to, like, what you're seeing because any misbehavior might be, uh, um, due to a, an underlying bug.

**Richard Sacco** [18:56]
I invest

**Aksana Rahouski** [18:58]
Which I'd be curious-

**Aaron Diefes** [18:59]
Yeah.

**Aksana Rahouski** [18:59]
'Cause Aaron, you still have, uh-

**Aaron Diefes** [19:01]
Mm.

**Aksana Rahouski** [19:01]
You have, um, admin deactivated.

**Aaron Diefes** [19:04]
Right.

**Aksana Rahouski** [19:04]
I think that would be, like, a good thing to run through just to see kind of what else you see a- a- as you kind of comb through this thing.

**Aaron Diefes** [19:13]
Yeah. I bet I could get to that today, actually. Yeah, I could do that today.

**Richard Sacco** [19:18]
No.

**Aksana Rahouski** [19:19]
Okay.

**Aaron Diefes** [19:19]
So.

**Aksana Rahouski** [19:20]
Okay. Um.

**Laura Perry** [19:22]
Okay. So Oksana, I did the... I did it in cowork.

**Aksana Rahouski** [19:27]
Mm-hmm.

**Laura Perry** [19:27]
And I guess I can't s- I can't do, like, a, like, a link out of Claude to share a cowork conversation.

**Aksana Rahouski** [19:37]
Mm-hmm.

**Laura Perry** [19:37]
So it's just gonna be one big, long copy-paste.

**Aksana Rahouski** [19:40]
Yeah. It, it's fine if y- or if it allows you to do, like, a file export, just dump it to me.

**Laura Perry** [19:47]
Okay.

**Aksana Rahouski** [19:47]
Until, like, a f- 'Cause copy-paste might not... Like, uh, Slack has, I think, max to how much you can paste and send.

**Laura Perry** [19:54]
Yeah.

**Aksana Rahouski** [19:55]
So maybe just put it-

**Laura Perry** [19:55]
Yeah

**Aksana Rahouski** [19:55]
... in the file and just-

**Laura Perry** [19:57]
Okay

**Aksana Rahouski** [19:57]
... ship it to me.

**Laura Perry** [19:58]
All right. If there's anything that you see it mention that it doesn't... You know, 'cause it has, like, a, you know, "Oh, I'll do this," and then you can kind of click down to see some of the, like-

**Aksana Rahouski** [20:08]
Mm-hmm

**Laura Perry** [20:08]
... stuff that it took. Anyway, if there's anything not obvious in there that you wanna see more of, let me know.

**Aksana Rahouski** [20:15]
All right. Okay.

**Laura Perry** [20:15]
Um, and I'll do what I can.

**Aksana Rahouski** [20:18]
Okay. Sounds good. Um.

**Richard Sacco** [20:20]
If I finish.

**Aksana Rahouski** [20:22]
Okay. Um, anything else we need to discuss? But I think for now I'm just waiting for them to come back with that email. I think I copied all of you guys on that, uh, regarding Verizon second, and that this, I'll create this bog and we'll just keep testing.

**Laura Perry** [20:41]
Okay.

**Aaron Diefes** [20:42]
Yeah. Sounds good. I'll do about that.

**Aksana Rahouski** [20:46]
Okay.

**Laura Perry** [20:47]
Cool.

**Aksana Rahouski** [20:48]
All right. Thanks, guys.

**Laura Perry** [20:49]
Thanks, guys. Bye.

**Aaron Diefes** [20:50]
All right. Bye.
