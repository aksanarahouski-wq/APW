00:31 Devon D'Andrea: I was just on the phone with him, so I know he's coming. Real quick though, before we officially kick it off, we had mentioned having this meeting to discuss the couple of Verizon things, um, amongst, you know, whatever. Um, I am trying to find.

01:06 Adam Curcie: Any.

01:07 Devon D'Andrea: Information that anyone might have. I guess Adam could tell me once he, uh, gets in here. This 1936 I'm not quite sure where this one came from. Is this an old card or is this— I only have— I'm only showing that Laura added it to the backlog on December 11th. I don't have anything else on it other than all of the API, all the API information that somebody dropped in.

01:52 Aksana Rahouski: I think for this board, it's just often, I think from, well, every time it's from our side, it says Laura, I'm pretty sure, because I know even sometimes when I add a comment, it says Laura. But that, I guess I don't remember how it was added.

02:11 Stone Marballie: This one, I think this one was created by Richard. So this was the case where a user bought a SIM-only, brought their own device, and tried to activate the device, and it failed. It failed for some reason. So that's where the whole thing started, right? SIM-only. So that's when I asked to read.

02:34 Devon D'Andrea: So these cards are then basically the same issue, these two?

02:40 Stone Marballie: These cards.

02:42 Devon D'Andrea: Okay, okay.

02:43 Stone Marballie: One was discovering the issue as a bug and one was— I think the.

02:48 Adam Curcie: Original, not that it's super relevant, but I think what happened was customer bought SIM cards, we activated them with a SKU, sent them, they used them, then they deactivated the SIM card because they weren't using the credit card reader it was in, so they wanted to stop the paying. Payment for like the offseason. So then they went into the portal and attempted to set the SIM back to active and they could not. It just wouldn't work because there was no IMEI in the portal for it to hit.

03:17 Devon D'Andrea: Yeah, I was asking what this other card, Verizon card in progress, was about, but it's basically the same.

03:24 Aksana Rahouski: It's the same thing. Yeah, I think they were just kind of reported as two different things. Ultimately, like, the solution that we're going to talk about is It's the same thing.

03:35 Adam Curcie: Okay.

03:36 Devon D'Andrea: Yeah, and I did see, uh, Stone.

03:38 Adam Curcie: The email with the other card number. I'm sorry, I'm late. Every single time I have to come to this Google Meet, I have to completely re-sign into Google. I don't understand why. It's not your fault.

03:49 Devon D'Andrea: Uh, it's 1936, but it basically was, I guess, the origination of finding the bug. And then the other card is basically like the solution where like we need to use the SKU. So I did see your email, Stone, where you did get a null response for those two that have no SKUs.

04:16 Stone Marballie: Yep.

04:19 Aksana Rahouski: And just before we maybe dive in, let me just quickly set the agenda for this meeting. I'm hoping we can accomplish all these things and stay focused. Yes. So Verizon. Activation, obviously, like the work that Stone has been doing and that email that came through. Basically, how do we— what process do we follow when we activate and deactivate devices, which is when it's device plus SIM or just SIM, because both exist, right? So the topic number 1. Topic number 2 is I know you guys just created a ticket that looks like we need to add another Verizon account With that being said, we need to figure out how we're going to differentiate the Verizon devices, which account they like attach to based on what. And I know you said a common perhaps service plan, something for us to discuss. And topic number 3, kind of get back to again the whole like service plan management. We want to show you some concepts after we talked last time and You guys are not quite satisfied with what that UI/UX looks like with now tier size price going from one-to-one to one-to-many. Basically, for the same price, you can have volume of data. You could pay different prices based on what it is. We have some concepts to show you there and we're hoping to make a decision. So, and again, the goal is to give it to you by the end of this week so you could test in beta so we can go to production next week.

06:00 Adam Curcie: Great.

06:02 Aksana Rahouski: Okay, so let's maybe start with that Verizon SKU, that whole— the email that Stone sent. So Stone, I'll let you lead on that one as far as what you need to know to kind of push this across the finish line.

06:24 Adam Curcie: Yeah, absolutely.

06:25 Stone Marballie: Basically, you know, I outlined, you know, like what Devon was alluding to, right? I tested those SIMs. I saw there was a null SKU. You saw the first one was the SKU that you guys gave me from prod, that that one did return a SKU, and I was able to activate the device. So the question really is, for those no IMEI situations, how do you guys want to handle it? Because right now I was trying to, you know, ask Verizon what the SKU is, and the one test case that I did had a SKU, so I thought, okay, that must be the case. You guys mentioned, and, you know, I showed where the API says you can use a single SKU and activate a bunch of devices, right? And you guys were saying that you guys were going to provide a SKU. Is that going to work in all cases?

07:13 Adam Curcie: Yes, it should.

07:16 Devon D'Andrea: You know, again, unless we want to, you know, have you ask for the SKU and then if you don't get a SKU, but like this, you know, I don't know. I mean, we could have you ask, we could have you request the SKU and if you get one, use it. And then if not, just use the one that we provide or we just don't, we just use the one that we provide for any SIM activations.

07:40 Adam Curcie: Yeah, just use the one because it will update if it is able to. Like, it's— yeah, I mean, it will absolutely update it once the box connects and it performs the OTA. If once it recognizes the IMEI of whatever it's on, it will update the SKU for that IMEI over the SKU on file for the SIM. So just use the one SIM, make this pretty simple and straightforward.

08:07 Stone Marballie: Okay, just use the— okay, yeah, and go from there.

08:14 Adam Curcie: Okay, yeah, that's it. It's the one, and I put it in the ticket at some point, I thought.

08:19 Devon D'Andrea: I'm looking— uh, yeah, it's the one in that ticket ending in 0018.

08:24 Adam Curcie: Yep.

08:24 Stone Marballie: Okay, it's in the 1936 ticket.

08:27 Devon D'Andrea: 1939.

08:33 Stone Marballie: Yeah, all right, got it. Let's see, you— okay, the VZW. Okay, okay, that looks like the same SKU that was returned for the one that I queried for.

08:44 Devon D'Andrea: It very well could have been. Yeah, I mean, okay, all right, yeah.

08:49 Stone Marballie: That'S a quick change I can make that fix and, um, we can roll that out.

08:55 Aksana Rahouski: Okay, so we're doing global, that's option 3, right, from your email, and global SKU for everything when we don't have one. Okay, sounds good. I wonder if we kind of should flip-flop to Service Plan and then get back to Verizon because I don't know how big of a worm this new thing might be, and I really want to keep like make progress with the Service Plan. Okay, so let's do that. Let's do Service Plan because we're all pretty close to this. So let's do it in this order. So I want first to show you quickly what Aaron did. He grabbed on to your idea. I was like, how about we take, like you said, we want tabs, right? want We basically, possibly, right? Can we do tabs where we can keep our usage limit, default price, and then all these different sub-pricing that we're introducing per limit, per tier limit, maybe become tabs. The problem there that we kind of like, I didn't want to kind of like zoom in, but Erin will show you that the number of these tabs potentially could be like tens of them because you're talking all models, all carriers, a combination of all those, right? So if it, and again, it's built to kind of scale to the max capacity, right? So ever like you guys will start kind of chipping away and creating different combination of prices, Tier 1, T-Mobile gets one, Verizon gets another, a combination of this model carrier. Why don't you pull it up and kind of show?

10:37 Adam Curcie: I can just make a quick comment. This may, I guess, relieve some of your anxiety with regard to that statement. For our Tier 1, Tier 2, and Tier 3 service plans, I don't anticipate us ever making model differentiated pricing changes. It'll just be carrier.

10:55 Aksana Rahouski: Okay.

10:56 Adam Curcie: The model ones are probably going to only be on that ATM service plan.

11:01 Aksana Rahouski: Yep. So Aaron, why don't you show us quickly that mockup that you did? And again, I think the kind of the less variations we have, the more digestible it's going to be, right? But ultimately, again, if we're at a high level, right, we're shifting here from one-to-one dimension where one tier has one price to one-to-many. Different solutions, we know we showed now the grouping which makes that row grow. Tabbing is another solution and Aaron is going to show us here what at some point it could end up being, which data scattered across many tabs. The more of it you have, the harder it's going to be aggregated altogether.

11:53 Aaron Diefes: I'll just sort of get my tab up here.

11:56 Stone Marballie: Oh, what the heck?

11:57 Aaron Diefes: This is the wrong tab.

11:58 Aksana Rahouski: Don't look at that one.

12:01 Stone Marballie: Let's go here.

12:03 Aaron Diefes: So this is sort of what I mocked up, and so you can quickly see, obviously, I know we're talking about even if it were just service plans, we'd have 5 of these tabs across, including default, right? Even that, if you had 6 across, right, you're basically planning on using this entire— obviously, we don't have a sidebar here, but as soon as you start doing any sort of custom configurations, imagine you have a Verizon plan, but for some reason you want to do Verizon plus I-22. Even just that adds an entirely new tab, right? Because you have to account for that specific information. So I mean, I didn't think that this would be unreasonable until I started planning out, okay, so I took— this is data from actually I mean, this is in review, so obviously I've been playing around with it a lot and just adding a whole lot of customizations. But this, like, in our current view, the way we have it, like, sorted by price tier, doesn't come across as crazy. But in the tab view, it does leave a lot to be desired. I mean, you know, it's one thing to have all of your defaults in one little area, but then if, for example, I'm looking for, like, you know, even like i52 and then dual carrier Verizon and AT&T for 2 gigabytes, like, does that exist? Like, I have to go over here and check it out and see, like, or like, it's kind of required to scrolling across tabs, which is, I think, where the shortfall of tabs comes in. But that's sort of why, like, we kind of began to think about it a little bit more and talked about some additional redesigning that we think might work a little bit better. I think, uh, yeah, and that's where.

13:45 Aksana Rahouski: Again, just kind of want— this is ultimately the worst case scenario, right? Like you said, you are not going to start there, but you could end up being there, right? That's why we kind of don't want to build for something we'll have to possibly maybe change. But at the end of the day, if this is what you guys want to do, we're gonna— we, we have no problem doing it. I want to show you though, like, Stone now, like, we've done this morning, we're like, okay, how can we really like kind of keep the functionality as is, group these tiers by the flavor of pricing that's going to be introduced, but also to allow this short view. How do I look at my page? I can digest what my defaults, because I totally see your concern with the fact that it used to be this cute little table grid, now it's a lot more dimension to this thing and harder to understand. So Stone, can you share? Stone did like a quick demo for us this morning. We didn't implement it yet, but he was like, maybe this is something we could go this way, kind of create these as like collapsible accordions and have the default sitting at that kind of headers of these. So which I think will give you what you're looking for, this like the short version of it in kind of the same— Yeah. Anyways, don't just— Yeah, just show it to us what you got. Your screen is so— His screen is so large.

15:19 Aaron Diefes: I got a big 34-inch widescreen.

15:21 Adam Curcie: Sorry.

15:22 Aksana Rahouski: I'm like sitting here like I can't see.

15:24 Devon D'Andrea: Yeah, no, it's fine. I can see it. And I think if you were to zoom, it would probably not Yeah, but yeah, I can see it.

15:35 Stone Marballie: All right, so basically this is the same view kind of with the cards, and I'm showing you that it should essentially be the same thing that you guys were seeing where you're seeing your usage limit and the price, but now I've got it stacked. So like, for example, for this 400-megabyte one, it says 2 variations. I know it's real small, but now you could collapse it and see the detail. So you don't come in with everything open and all cluttered, right? So if you're looking from the perspective of scanning, this is what you would see. And same thing from the perspective of what your customers or distributors would see, right? Let me see if I can find one that's worth looking at. So for this tier 1 here, whatever, Lucky Lincoln Gaming, we're just going to use him, right? So they get to see this. So it would be the same thing. They would see what their base prices and adjusted prices were, and if they wanted to go in and modify, they can see the variation models and carriers and go set the prices that they want. You know what I mean?

16:39 Devon D'Andrea: Yeah.

16:40 Stone Marballie: So that's kind of the, you know, so this was just something I threw together right before this call to kind of like, you know, wonder if this is closer aligned to what you guys had envisioned.

16:49 Devon D'Andrea: So, um, so I, I, yeah, I mean, I I kind of want to— something that I noticed— this— I'm not— I do like this. Something that I noticed though when Aaron was presenting was, would we get— would we have an additional variation for dual carrier?

17:19 Stone Marballie: So, well, yeah, that Yeah, that variation exists right now.

17:23 Devon D'Andrea: That variation exists, right?

17:25 Aksana Rahouski: So under carriers, do we have the two dual combos?

17:30 Devon D'Andrea: You do have the two dual combos, right? Because.

17:34 Adam Curcie: Yeah, because the carrier checkbox is no longer utilized to associate, um, like, like a pricing change, I believe, right?

17:44 Devon D'Andrea: Or so It's so— it's— see, that's, that's gonna be why, because we have different types of dual carriers.

17:53 Adam Curcie: Well, yeah, I forget where we need to— that's different conversation, I guess.

17:58 Aksana Rahouski: Or maybe— yeah, yeah, it's— I think.

18:01 Devon D'Andrea: It'S, it's a conversation if we're— if that's going to affect the number of tabs or variations or whatever, then it is— it's definitely part of this conversation because I didn't think that we would be needing to make.

18:15 Adam Curcie: Do you want to try to like count the maximum number of possible tabs we would ever have? Because I don't think it's that many.

18:23 Devon D'Andrea: I know, but well, yeah, I don't think there would be either.

18:28 Adam Curcie: Um, I don't even think we'd ever get to 10.

18:31 Aksana Rahouski: But 10 tabs is still a lot. You guys still leaning towards tabs?

18:36 Adam Curcie: I really like the tabs.

18:37 Devon D'Andrea: I'm heavily on the tabs.

18:41 Adam Curcie: Yeah, and there's like, I mean, if Aaron can pull it back up real quick, I mean, I wasn't quite sure if he had a service plan selected because it didn't look like any of the usage tiers from one tab to the other were the same, which I would expect. Like, if we're looking at Tier 1, all of those data, like 1 megabyte, 10 megabyte, I'm sorry, 100 megabyte, 1 gigabyte, those should all be identical regardless of which carrier and model, the only thing that really would be changing is the price associated with that data usage. But that wasn't the case when he just had that up. And I mean, I know it's a mock-up, so I mean, as long as we all understand.

19:18 Aksana Rahouski: Yeah, pull it out. And I will say the next thing for us, I mean, there are quite a few, like, there's some gaps in here for, like, let's say if it's a base plan, adding combination, does that add a tab? So, like, it opens a kind of like a new can of worms, which we can absolutely pursue. Let's finish this conversation. I still think that we should give it into your hands in the way that it is currently. You're reacting so strong, I think, because it's unfamiliar yet. And my hope is that you're going to maybe at some point be like, you know what, it does what I want it to do.

20:00 Adam Curcie: Well, I mean, unless you're going to make an argument that due to time and money invested and time and money required to correct it, I mean, I mean, I don't want to put this just in like a— I guess I really like the tabs. I don't feel as though that my discontent for the system that you guys already have in beta has anything to do with it being unfamiliar. I would also just like to quickly make an additional comment on this screen that we're looking at. So, Aaron, if you could just click over to the first tab.

20:34 Aaron Diefes: Yeah, for sure.

20:35 Stone Marballie: Over here.

20:36 Adam Curcie: Yeah, I mean, as I kind of mentioned, like, the default pricing tiers should all kind of replicate across, right? Because if what we're saying is this is Tier 1— I mean, it says standard data plan, and that doesn't really matter, but, you know, if this is just the difference for this, you know, device and carrier on this plan, all of the same usage tiers should be there. And I would like to maybe interject that we should include the default next to the base as a grayed-out for reference. So like if the base price is $750, right, and the default price is $10, then they can have the framework that they need to make accurate adjustments all on one screen, right? So like if we give a customer a discount, you know, so like their base price $750 for $500, and but the actual default for that tier You know, like, actually, I'm sorry, well, it's very difficult to do in this situation, but you click back on default for me. Do you have a value for 500? Yeah, so it says base price is 750, the adjusted price is 650. So then if you click on that, then the base price is 7, and now the adjusted— so I guess you would show that the— you would have base price 750, and then the default price for this 500 usage is $6.50. So then they could adjust this to, you know, they should see both, right? The base and the default, because they're separate.

22:09 Devon D'Andrea: So right now you're looking at a company's— yeah, that's another part of customized service plan. You're not looking at a distributor's view.

22:21 Aksana Rahouski: Correct.

22:22 Adam Curcie: I got you. So then, yeah, I guess then I.

22:26 Devon D'Andrea: Like the idea of if you're a distributor and setting it up that you could recall back to what the default is for that usage value. And that kind of blends both ideas here. Again, I do have to just bring up the dual character thing again. We built our entire thing around like, and our invoices even show that like the dual carrier is a fee. So yes, we're making a pretty big change here with respect to how we build dual carriers.

23:10 Aksana Rahouski: Yes, it's a conflict right now. We're creating conflict, right? Because you were saying that we have a dual carrier subcharge on a company, or I don't remember, or on the device, like on one of those. But also we're now allowing to just introduce price for these dual carriers, right?

23:26 Stone Marballie: Right.

23:26 Devon D'Andrea: And that's where, again, like, that's where now, Adam, we could be looking at upwards of more tabs because of that, unless we eliminate that. But I don't know how to eliminate that other than, you know, uh, like right now the fee for dual carrier is $4.95. I don't care if you use 1 gig or 10 gigs. You pay an extra $4.95 just because you have dual. The data price is the data price, depending on how much data you use. So the idea of now blending the data, the dual carrier setup, uh, into this, you know, service plan usage pricing, I don't like. But then again, but then I don't know how to, because the problem is, is that we are now selling, starting to sell another device that like, it's almost like we need to just have variations of the dual carrier fee. Like, I don't want dual carrier to be brought into this screen in any way.

24:34 Adam Curcie: Yeah, well, I believe necessarily.

24:39 Aksana Rahouski: So are you saying that we don't really need this like dual carrier price options here?

24:45 Devon D'Andrea: Yes, not here, because this page, all that this, all this has to do with is what are they paying for their data. So a dual carrier device, and I'll just explain it this way to make it hopefully make sense. If I have a device that has Verizon and AT&T and it uses, you know, it uses 5 gigs on Verizon and then 5 gigs on AT&T in a billing cycle, it used 10 gigs total. The next billing cycle, it used Verizon for the entire billing cycle and used 10 gigs. It used 10 gigs total. So like, the price for 10 gigs is the price of 10 gigs in that month or the following month. So like, the dual carrier is just a benefit. It's just to have an extra active SIM card sitting there, a hot SIM card. So like it needs to be pulled out completely from the idea of what they're paying for their usage.

25:46 Adam Curcie: Yeah. And to kind of even simplify this further, we'll probably never have a single thing in this that'll say AT&T only. Like I can say that like as a variation.

26:02 Devon D'Andrea: Yeah, no, I mean you say that now, you say that now, and then Margaret might come back to us next month and just like say, hey, you.

26:08 Adam Curcie: Guys get Well, we've said that for 4 years, so I'm gonna go with 98% certainty.

26:16 Devon D'Andrea: Well, if we can eliminate the dual carrier thing, then we know that that's not gonna add tabs. And then in reality, we're probably only gonna have— for any given service plan, we're probably only gonna have up to maybe 4 or 5 tabs at most. And I think I agree with Adam, and we can kind of blend the best of the two ideas here, which is keep the tab thing. I know Stone has a question, but then also for just when distributors are setting up the subs, they can— they have that— they have that column that has that default that they can refer back to.

26:54 Adam Curcie: Yeah, just for their own, you know, they're like, okay, I make an extra, you know, $2 on a, you know, Verizon compared to T-Mobile or AT&T. Um, Stone, what's your question, man?

27:07 Stone Marballie: Oh, my question was, so when you customize these, are you saying that it's only by models?

27:14 Adam Curcie: No, adding variation models for all non-ATM service plans. I'm gonna tell you, it's going to be incredibly rare that we add a model. It's really going to be Verizon only. And honestly, the requirement for there's a good chance that for Tier 1, Tier 3, we only have Verizon only for quite some time.

27:42 Stone Marballie: Okay, so, um, yeah, are you saying like when you customize it, it's gonna be okay, it's either this carrier which is the primary carrier, and then even if it's dual SIM, it's just whatever the primary carrier is, you don't care if they use it on whichever carrier, the primary, the secondary, because primary is.

28:00 Aaron Diefes: Always a good cause of the day.

28:02 Devon D'Andrea: Yeah, Verizon.

28:03 Stone Marballie: Okay, so the variation is always going to be, okay, this Verizon or this T-Mobile?

28:08 Devon D'Andrea: The variation is always going to be— but yeah, right, for dual carrier, Verizon is always going to be the primary. So we're always going to be dictating the data usage pricing for dual carrier. For dual, if someone has dual carrier, like let's just say somebody's trying to negotiate pricing with us, we will get— and they want to— they've got lofty, you know, numbers that they're throwing at us, and we want to be like, okay, what's the best pricing we can give you? Okay, well, if you need AT&T or T-Mobile by itself, you can live here, and that'll be that person's default price. But what we can do is get you really aggressive pricing on Verizon. So for that new customer, they're only going to have a default and a Verizon only. Or Verizon, I should say. Now, they could then— and we've done this in the past, actually, where we've literally told people— the difference in the past was we didn't have the ability to give them both. We would just go to them and say, hey, we can give you this super aggressive pricing, but you have to buy Verizon or your carrier. You can't buy AT&T, you can't buy T-Mobile. So now we can say, hey, we can give you this really aggressive pricing on Verizon, But if you need AT&T or T-Mobile, the pricing will just be a little different. Regardless if they are buying Verizon or dual carrier, which is going to have Verizon as the primary anyway, they will— the only difference there is that those dual carrier devices just come, need to come with that extra fee that we can dictate, that we can customize for them, just like we can now. Um, you know, and I, I, I, I don't know, Adam, do you think that— I don't know, I don't know how to deal with the dual carrier.

30:01 Adam Curcie: I mean, I, I think it might be in our best interest at some point. I, I don't know if we would actually need to do it to kind of move forward with this, but maybe down the road we can add a second dual carrier price point. So we would rename the current dual carrier fee to dual carrier and then probably in like parentheses AT&T and then have a secondary one for T-Mobile because we would probably always elect to give somebody a slightly more, you know, advantageous reason to use a T-Mobile dual carrier solution than an AT&T. So we can try to control that with price incentives. One thing else I kind of feel like it's worth mentioning In regards to like how and, you know, yeah, regarding how devices do use the dual carrier data, like that's entirely out of our control. So like it's really not up to us which devices use what amount of data on which carriers once they're out in the field. Yeah.

31:03 Aksana Rahouski: I have a question because I think I remember now why we added this dual because we started without them. We started with just 3 carriers.

31:11 Devon D'Andrea: Um, exactly why we did it's because of the ATM plan. It's exactly why, because we have— we're offering 2 basically default pricing structures. Yeah, that is correct, that are varying by model. That's why we have the model in, in this, in this new build, and that's why we have the dual carrier, because You know, because our standard dual carrier rate was $4.95. So no matter what, if you got a dual carrier, if it was on the ATM plan or any other service plan, the dual carrier fee was $4.95. Well, we went ahead and made this Origin device that has the ability to have dual carrier with Verizon and T-Mobile, and we decided that since we're selling Verizon at $4, And if they were to buy it on T-Mobile, it's by itself, it'd be $350. Well, we can't really then sell the dual carrier at a $495 upcharge. It's got to be just the $350. So this was why we had to do this in the first place.

32:22 Adam Curcie: I'm almost wondering if it makes sense just to ask if you could create this in like, I don't know, with some restrictions that might make just the conceptual aspect of it easier for you all to fully be comfortable with, right? Like, we will not need model for Tier 1, Tier 2, Tier 3. We will not be giving customers pricing discounts on gigabytes. Regardless of the model. So I mean, if you want to think about it, like our, you know, kind of flagship ATM plan is $4.95. So like we're— and for reference, that's like 15 megabytes in a month, right? So like our cost to Verizon is under $1. So I mean, if we're taking, you know, a $3.50 profit margin and reducing that slightly, you know,. And for T-Mobile, it's kind of apples for apples. So it's very minimal, right? Just in terms of the overall revenue, it's a very minimal discount just to buy the Origin. So we're not— and we're literally doing that just to get people to buy it. So for data that's billed at the gigabyte on Tier 1, Tier 2, Tier 3, and above, there is never going to be any way we can incentivize somebody by model to have different pricing. It's going to only be by carrier.

33:56 Devon D'Andrea: Yeah, there's really no reason.

33:59 Adam Curcie: And then, you know, if it makes it again easier, just don't charge the dual SIM fee for that. Like, I don't know if that makes it easier. It probably doesn't. But like, you know, like for the Verizon T-Mobile Origin discount that we have been telling everybody is going to be $750 or $795. Yeah, yeah. I mean, we have to either have a different dual SIM fee for— because it's a T-Mobile, not an AT&T. Because every other, you know, basically every dual SIM thing out there right now is AT&T and $495, or their pre-negotiated dual SIM fee price is fine, but specifically for T-Mobile moving forward, if we could add a secondary price of $3, I mean, that solves the problem. If the portal can just understand like, hey, this is a dual SIM box, but it's— and the second SIM that's active on the device configuration screen is T-Mobile, not AT&T, so I'm going to use this price, not that one, then that solves that problem.

35:09 Aksana Rahouski: As far as— so again, just kind of like what I recall, one, at what point we decided to add these duals here, because we couldn't decide like if a device, let's say, is on the plan that has pricing for both Verizon and T-Mobile, and when we calculate actual usage, both have usage, which rate do we apply?

35:34 Devon D'Andrea: That was the concern. But again, if they're getting dual carrier, they are paying for— like, we're selling it as Verizon first, right? So if it fails over to T-Mobile and never goes back to Verizon because there's no Verizon service, That really doesn't matter, um, because what they bought is a Verizon primary, uh, service plan. So that is $4. So you know what I'm saying? So it's like, like that we're selling that at $4, right? And they, they have an extra SIM card in it on T-Mobile. And if we were just If they were just getting T-Mobile from us, they would be paying $3.50. But we don't need to worry about which carrier it's using because what they buy in the beginning is Verizon first, which is a $4 service plan. But to have the backup SIM, regardless if it switches to T-Mobile or stays on Verizon, it's a $3.50 fee.

36:58 Aksana Rahouski: So, so are you saying that in real world, if you have a device sitting on a plan where $4 for Verizon, $3.50 for T-Mobile, even if for whatever reason they're sitting and using T-Mobile data the whole month, we're charging them $4 because they are charging them $7.50?

37:16 Devon D'Andrea: You're charging them $7.50, but yeah, the data is $4, but you're charging them $7.50 because there's a $3.50.

37:25 Aksana Rahouski: Yeah. Yeah, for the dual, right?

37:28 Devon D'Andrea: Yes.

37:28 Aksana Rahouski: But for like the data rate, we always apply Verizon even if they use T-Mobile on a dual SIM device.

37:38 Adam Curcie: I don't know what you mean by data rate. I'm getting a little confused with the terminology.

37:43 Aksana Rahouski: So like this, like that's kind of in relation.

37:46 Adam Curcie: You want to share your screen?

37:48 Aksana Rahouski: Yeah.

37:49 Stone Marballie: So even like— Well, I think what they're saying is, Oksana, that when even though it's a dual SIM, what they're selling the customer is Verizon. So it doesn't matter when it switches over. So as long as we map it like in this configuration here, rip all the models out, and so like what Aaron has on the screen, it would be like i22 Verizon would be the configuration that they sell them the price for, or i22, uh, whatever other carrier. But for the Verizon case, even if it was a dual SIM device, it would get charged a dual SIM device based on the charge that's attached to the company, right? $4.50. And we don't care what— if it flip-flops and use data from whatever, we just need the total data. We're going to charge at the Verizon rate or whatever the cost of the plan is and be done, right?

38:33 Devon D'Andrea: Exactly. So they're gonna buy one of these service plans, and to start, we're really only gonna have two variations that we would build.

38:44 Adam Curcie: Yeah. And just for reference, I just want to reiterate, like when we're talking about the ATM service plan, the actual data used is just irrelevant because it's nothing. I mean, it's just not minimal.

38:56 Devon D'Andrea: Yeah, so, so like when we roll this out, we're gonna have our ATM plan, our standard default ATM plan of $4.95, and then we're going to have, you know, our default tier 1, our default tier 2, and our default tier 3. These, these might not be, these might not be We might not make variations to these based on the— if we do, it's going to be based on the carrier. We've got maybe one customer that we have to set that up for when we roll this out, but other than that, like, no, we don't have any. There's really nobody that we have to do that for.

39:32 Adam Curcie: And regardless of whether or not we have one today or two today or two tomorrow, it's most likely with, again, about 98% certainty, only ever going to be a Verizon only. It'll be one actual— for all the tiers, there's going to be potentially only one discount. Not model, not AT&T, and not T-Mobile. It'll just be one discount for any of these tiers, and it'll be a Verizon only discount.

39:59 Devon D'Andrea: Okay, so now there's gonna be dual carrier to this. Okay, right. So then, so then, so then you have just this dual carrier thing, and I just realized something. So So you've got dual carrier. Adam, what about— what about what?

40:26 Adam Curcie: What did you realize? What's your question?

40:27 Devon D'Andrea: Well, I mean, according to you, we're going to basically stop doing any AT&T dual carrier. What if we start putting T-Mobile in dual carrier on I-22s? What's the fee?

40:37 Adam Curcie: That's why I said the separate dual carrier fee.

40:41 Devon D'Andrea: Well, then now you've got a whole nother one because this one would be T-Mobile only if it's an origin. This one would be any AT&T, and then this would be T-Mobile I-22, and then that would be $4.95.

40:57 Adam Curcie: Does it have to be $4.95, or.

41:01 Devon D'Andrea: I mean I guess not. I don't know, maybe not.

41:04 Adam Curcie: I would just roll it at $350.

41:06 Devon D'Andrea: Just roll it at $350?

41:08 Adam Curcie: Well, yeah, I mean, we're trying to incentivize people to use it, so just.

41:12 Devon D'Andrea: Keep it to— so if it's— if the dual carrier is Verizon, AT&T.

41:18 Adam Curcie: Yeah, that's exactly what I think we should do. But you can obviously— oh, I mean, tell me I'm wrong, I don't mind.

41:26 Devon D'Andrea: Yeah. So we just need to have an extra— we need to go— so, all right, yeah. So we kind of flipped this whole thing on its head today. We have to basically go back to what we originally— the way we originally had it, but with the addition of a variation on the dual carrier fee.

41:48 Aksana Rahouski: So we need two different pricing options for dual carrier? Is that what we're saying?

41:55 Devon D'Andrea: Yep.

41:57 Adam Curcie: Yeah, I think that's going to be the only way we get through this, uh, for a long term and short term with any type of like, uh, yeah, sorry, I don't know what I'm trying to say, but, you know, just.

42:11 Stone Marballie: So I'm going off on a tangent here, right? So what we built was a way to price the, the plan itself based on whether it's dual, right? So and then the dual SIM fee itself is attached to the company, right? But now you're adding, saying that the price that we attach to the company has to be variable depending on the dual SIM configuration, whereas here you could have a predefined dual SIM upcharge, right, that you charge regardless, and then affect the price of the plan itself based on which configuration of dual carrier they use here.

42:48 Adam Curcie: Well, yeah, I mean, like, the way it is right now, um, Devin, just go pull up a box and a company page on Allpoint.

42:56 Devon D'Andrea: Yeah, I was just also pulling up the fact that we've got— yeah, this is zero. That's— that's Bitcoin Depot.

43:08 Adam Curcie: Well, no, go just grab a company page.

43:13 Aksana Rahouski: Yeah.

43:14 Adam Curcie: Me. Why are you using zero? There you go. So for every single customer, they have a dual SIM price here. That's the default. But as you just saw with Bitcoin Depot, we gave them zero. And we have got other people everywhere in between— $2, $3, $4. So, and when we did all that, and it— the way it's worked forever is specifically for AT&T. Because we only ever had 2 SIM options. So if you had dual, it was Verizon and AT&T. So if we broke that in half and called this the customer dual SIM price parentheses AT&T and had a second box right to the right of it that said customer dual SIM fee parentheses T-Mobile and made the default for everybody moving forward $3.50, right, then we could, you know.

44:11 Devon D'Andrea: Before we could just— yeah, then we could just go back and then, and then you just have to, and then you just have to, um, and then you just have to change— we'd also have to change the invoice to reflect multiple duals.

44:26 Adam Curcie: It would really just say— I, I think the only thing you'd have to do is append the, the text to say dual SIM fee for AT&T is this and for T-Mobile is that. Yeah, because you're only going to ever have one dual SIM fee on one of these items on this invoice, right?

44:43 Devon D'Andrea: So I know that, but down the bottom— yeah, I'm saying you would have to have— you could keep the same little whatever the hell this is, and.

44:49 Adam Curcie: It would just— it would just be this. Yeah, for both. Yeah, just need a couple extra words. That's it. You wouldn't need to like add— so.

44:58 Stone Marballie: You want to— you want to keep the configuration specific for the company, but multiple options. When you run the invoice across the device, you would check the configuration to see which one's primary or which one's secondary or whatever variation they have and choose the dual SIM fee accordingly.

45:15 Adam Curcie: Yeah, and for your own— again, we're never going to have an AT&T/T-Mobile device. Just going to also state that. You'll never see a combination of AT&T.

45:25 Devon D'Andrea: Adam likes to say never a lot.

45:28 Adam Curcie: Yeah, and I'd like you to stand by me, Dev.

45:35 Devon D'Andrea: So how do we accomplish this and me also be able to get what you already have built pushed to production before the 6th?

45:53 Aksana Rahouski: I know, that's why.

45:54 Stone Marballie: That's why you need to go back and rebuild it all right now.

45:59 Aksana Rahouski: No, let's not rebuild it all. Okay, so let me share. Let me steal the screen.

46:04 Devon D'Andrea: If they get it done by the 6th, that gives me a couple of days, or not even, to make— to retroactively, you know, go back and alter all of the pricing on the Origins.

46:16 Adam Curcie: Dev, you only need one day to do that.

46:18 Devon D'Andrea: I mean, you're probably right, but what if I get hit by a bus?

46:22 Adam Curcie: I'll do it.

46:25 Aksana Rahouski: Okay, so let's— so what I heard, just to repeat, we're saying I still think if we need to get it out before the 6th, maybe we'll kind of table the tabbing because tabbing will take time and most likely will not get it on the 6th. Let's just get out the door kind of what we have, trimming what we need to trim, and then we'll go back and we lay things out in a way that makes more sense.

46:51 Devon D'Andrea: And if you think that that's the quickest way to get there, I just, I don't know how you guys want to approach the dual fee.

46:56 Aksana Rahouski: Yeah, dual fee. Are we saying that, just to be clear, that we, what we do want to— you probably don't see it. Hold on, let me share my whole screen.

47:04 Devon D'Andrea: Yeah, I'll stop sharing.

47:07 Aksana Rahouski: Um, are we saying then that— okay, um, okay, so Service plan management, we want to take out these two, right? We do not want to price separately for dual. So we really keep it strict to just 3 carriers, and you guys will be able to set these pricing per carrier within the tier, right?

47:35 Stone Marballie: Correct.

47:36 Aksana Rahouski: Additionally, I heard that now on the company we want to kind of have two options for dual carrier, right? We want to have Verizon, AT&T, and Verizon T-Mobile with an option to set two different prices for those two different dual SIMs, right?

47:58 Devon D'Andrea: Yes.

47:59 Aksana Rahouski: Okay. And then when we— so that way we kind of move it, like move the dual pricing from the service plan into the customer setting, right? We're charging them if they have Verizon. Verizon is what's always there being billed on their data. Dual subcharge is applied based on the rate that they set up with on their profile.

48:25 Devon D'Andrea: 100%, absolutely.

48:27 Aksana Rahouski: Okay, and then I'm just thinking for the invoices, Stone, do we have everything we need? Because I do remember like now very clearly why even this was added because when we were just running tests and we're like, if the same device, like let's say this is the scenario, they have both T-Mobile and Verizon, we calculate data for the last billing period and we have both, like do we apply $4 or do we apply $5? So we're saying we always are going to apply $5 plus the dual price that they're set up with.

49:05 Stone Marballie: Yep, right, you're on point.

49:06 Aksana Rahouski: Okay, okay, so let's— so I would.

49:10 Stone Marballie: Say, what about the model though? You didn't mention the model. Are we— you mentioned something about this, uh, what's that model you had in your, uh, your spreadsheet?

49:20 Devon D'Andrea: Um, yeah, we do leave the model variation that you guys built just literally, and that's literally only going to be used for the ATM plan.

49:30 Aksana Rahouski: Yeah, and I think again we We leave model as is. Let's just for now, we know there will only ever be used for one model. But again, we try and build that. I hear what you guys are saying and it makes sense for us to maybe design the final look based on the reality of data. If the reality tells us there will never be 25 tabs, that's fine. But I still don't want to lock us in as only one model ever can be configured. And it's already kind of built, right? We'll give you models, we'll give you carriers, but just 3. You will set up whatever you need to set up. We know how to calculate pricing, and we will expand customer page to have 2 different dual prices. As far as kind of the layout, I would say again, maybe let's just stabilize it in this shape and form, and then like We launch it and then we come back and we rethink maybe like this becomes tabs. And because the problem right now with the tabs, why most likely we'll run out of time, because now you have to not only think as far as like how do you lay that data out in a way that makes sense, but also this like an ability to add and delete, like where does it belong? Is it in the tab? Is it somewhere else? Right? Some decisions we still need to make. Adam, go for it.

50:52 Adam Curcie: What is your response if I were to say that this horizontal, not tab approach, can you put that in for just the ATM service plan? Or is that— because I feel like if you built this in soon, which it sounds like will happen.

51:17 Aksana Rahouski: Yeah.

51:17 Adam Curcie: And did it for only the ATM service plan, we'd probably be fine with that forever because there's like— it's, it's, you know, it's odd, I guess, to kind of think of it where in terms of visual directionality, but like the ATM service plan makes more sense to have all of this horizontal space where all the other ones we need the tabs.

51:45 Devon D'Andrea: Yeah.

51:46 Adam Curcie: And we also don't need any tiered service plans to have any modifications before the upcoming billing cycle. So like, it's like, okay, if you're going to build this all in, which you have to kind of do by the 6th, just is there a way you can change your build plan so that it's really only for the ATM stuff? The ATM service plan, and then we can probably just keep it forever that way. Or if that doesn't make sense and you're— because it sounds like, you know, there's a chance we're going to end up doing things different anyway at some point.

52:19 Aksana Rahouski: So, but yeah, so my concern with that, right, because again, at the end of the day, we're saying service plan management, like in this case, service plan could have multiple tiers. The tiers have multiple pricing, right? And if we start kind of picking and choosing between this service plan gets to have look A, but this service plan gets to have look B. And when you say it's not just the look, it's also the way it functions. Ultimately, where we're going to end up with two different code bases that needs to be supported, maintained. It's like at the end of the day, it's more money for you to pay in the future. Because it's two different flavors of things for the same thing. So every time we test, let's say you add new rules to a service plan, but service plan now has two different existences, right? We need to test both. The build, the maintenance, the testing is going to basically double with two code bases. That's where— that's the reality that I'm trying to avoid. Don't correct me if I'm wrong, because like we would have to— again, is it possible? Yes, it's possible. We could definitely treat it as Service Plan A follow these rules, Service Plan B follow these rules. But then as soon as we introduce that, it's two set of rules for the same data entity to follow. And if there's value in it, absolutely, we could pursue that. But also, I feel like maybe that's why I feel like maybe we just— what I'm worried about, we kind of still cannot predict what the final look of this thing is going to be because I hear there's more to come. And I don't want to create now, create these temporary effort and work to then again change it. It's possible, but it's more money for you guys to spend. That's what I don't think you want.

54:25 Adam Curcie: It, but no, not at all.

54:30 Devon D'Andrea: Yeah, I just think that we need to take the quickest approach to get what we can live and then.

54:42 Adam Curcie: Yeah.

54:43 Devon D'Andrea: Again, we can— let's stick with what it looks like now, and then if, you know, like to Adam's point, if we want to switch to tabs for everything or just the other service plans, you know, like I said, we don't— I have— we have a lot to do for the ATM customers that have already started buying Origins. Okay, that's my priority. Changing the UI for building out these larger service plans and variations of such. You know, we don't— there's not a— there's no rush, right?

55:22 Aksana Rahouski: And that's where, again, you kind of gonna— they're gonna be sitting there, the look is gonna change, which what we don't want, right? But I also, like I said, to kind of keeping two looks to maintain is gonna become expensive as far as maintenance, right? Which ideally, that's not the way that I would guide you, but again, possible. So I'm with you. I feel like we just kind of, let's make sure. And also, like, we still, I feel like as you guys kind of evolve your business, right, to where like more models come in, you try different strategies with pricing, how do you charge for do, like, I don't think this is over yet. Like, I think this, there will be more to come. So I'm like, let's not make it so messy to start with that, like, keep working on it, it's going to become a nightmare.

56:16 Devon D'Andrea: Yeah.

56:17 Aksana Rahouski: But again, if you strongly feel about all the plans staying as they are today, I mean, it's possible. It's just we're kind of— I would expect that we're beyond that point, but we you can restore back other plan, create that double look. But like even like thinking from testing perspective, now we have to go back and we have to make sure that both still work, right?

56:43 Devon D'Andrea: Right.

56:43 Aksana Rahouski: We've already tested this and we know this works, but we don't know if the old one still works with the whole concept.

56:50 Devon D'Andrea: Sure.

56:53 Aksana Rahouski: So maybe like again, like with all these kind of things that I named, Let's complete that, leave this look as is. I would recommend even the go-to-beta, or like while you test, and create scenarios that are the most realistic to what your next year is going to look like, right? To what you said, if it's just the carrier, or for some cases is the model, and then we will look at that real data and we'll figure out what's the best look for that data. And if it's only like 3 tabs and we feel like tabs like resonate a lot better and the data will never grow beyond the 3 tabs, then we'll switch to tabs.

57:38 Adam Curcie: Okay.

57:40 Devon D'Andrea: Yeah, I mean, listen, we might just tell you after we push this live that we want to build the tabs.

57:44 Aksana Rahouski: But either way, yeah, yeah, we'll, we'll do whatever. I'm just trying to kind of prevent us from creating a lot of rework.

57:54 Devon D'Andrea: Sure.

57:55 Adam Curcie: No, I get it.

57:56 Devon D'Andrea: I get it. I'm with you. All right, so I think we're good. And I'm really glad that you decided to talk about service plan stuff instead of continuing on with Verizon stuff. I have a 2 o'clock with Chad.

58:13 Aksana Rahouski: Yeah, I have a hard stop too. How urgent is the Verizon new— do we want to schedule another meeting tomorrow? Can you wait till next week? But we can schedule another meeting.

58:23 Devon D'Andrea: Yeah, I don't think— I don't know what— I don't even know what else we needed to talk about.

58:29 Aksana Rahouski: Verizon, the only thing, the new ticket you submitted about new business account.

58:34 Devon D'Andrea: Oh, shit, I'm sorry, I completely forgot about that. Yeah, yeah, no, that's, uh, that's a joke, Ross. Should we get We could probably try to get back together next week on that.

58:44 Aksana Rahouski: Okay.

58:45 Adam Curcie: Yeah. I mean, just to answer your only question I heard you raise earlier when you glossed over it, that's going to have to be, I think the easiest way to differentiate that is by service plan. I think we have a service plan that we built called Business Internet.

59:06 Aksana Rahouski: Okay.

59:08 Adam Curcie: I'm in beta. Hold on, let me go to production. But, um, yeah, yeah.

59:15 Devon D'Andrea: We just have to figure out if there's anything else we need to do. But I gotta jump to meet Chad.

59:20 Aksana Rahouski: Yeah, let's— and we'll follow up on that one. Uh, we'll schedule something next week.

59:25 Devon D'Andrea: Okay.

59:25 Aksana Rahouski: Okay, so we will regroup internally, we'll kind of wrap up the service plan, and, um, I'm hoping that you guys Let me, but soon. I was hoping before the end of this week. I know we introduced a little bit more, but let me talk to Stone and see how much effort it is.

59:41 Devon D'Andrea: Okay, perfect.

59:42 Aksana Rahouski: All right, perfect.

59:43 Devon D'Andrea: Thank you.

59:44 Aksana Rahouski: Bye.

59:45 Adam Curcie: All right.