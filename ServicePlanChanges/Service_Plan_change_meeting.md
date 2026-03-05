00:00 Trista Smith: it's

00:00 Devon D'Andrea: Hello.

00:01 Trista Smith: Yeah.

00:05 Aksana Rahouski: Hey guys. Welcome

00:08 Devon D'Andrea: Just Adam.

00:15 Trista Smith: There is.

00:16 Aksana Rahouski: Alright.

00:17 Trista Smith: yeah, Adam

00:21 Aksana Rahouski: Not worries. How you guys today?

00:26 Devon D'Andrea: Doing well.

00:26 Adam Curcie: Right.

00:28 Aksana Rahouski: Okay. Sorry, go ahead. We're gonna say something.

00:35 Adam Curcie: are I just said,

00:35 Richard Sacco: Yeah, I'll be back in just a second.

00:38 Aksana Rahouski: reach out to you know, bubble that's like that. That's it then or if you have any updates to use this time for that as well.

01:11 Devon D'Andrea: Yeah, I

01:12 Adam Curcie: I did not get away from them.

01:15 Aksana Rahouski: Okay, all right. Sounds good.

01:17 Adam Curcie: But, I mean, talking about the A little bit about this after the meeting. and what I said to him and I don't recall if he agreed a hundred percent but like If we if we know the way that we do it with AP can be done and works. Well, unless there's like a resources or efficiency, you know, argument that would dictate we don't do it that way. I I like that, probably. A way that we would suggest doing it. Is.

01:56 Aksana Rahouski: Richard ready?

01:57 Richard Sacco: prefer unless there's resources or

02:05 Adam Curcie: Sorry. So we didn't get a official response yet for the API guru at. But just

02:11 Richard Sacco: Okay.

02:13 Adam Curcie: thinking through what we do know, like my suggestion which again you know, is open for interpretation or rebuttal would be to do with the way we do it with in our. Yeah. Unless you know, like there's a specific, you know, efficiency or

02:27 Richard Sacco: AT&T.

02:33 Adam Curcie: resources argument. That would suggest we shouldn't do it that way.

02:37 Richard Sacco: know. Like, I feel like if it's good enough for AT&T, it'll be good enough for a T-Mobile. But the, the good thing about it is, You could actually with that API you could do a hundred devices at a time so we could do it in much less API hits and we can get all the data for T-Mobile. So there there is an advantage actually, to doing it that way. Overdoing it. The other way.

03:25 Adam Curcie: Yeah. I mean if there are you know, like you know, that there's clearly some differences pros cons, you know, for each they seem to be none of them I would

03:32 Richard Sacco: Okay.

03:37 Adam Curcie: obvious reason you don't want to do it that way that we, you know, don't understand or not Aware of. I mean, that would probably just work fine. I'd say, I don't know. Devon, do you agree with that?

04:10 Devon D'Andrea: Yeah, I'd say so.

04:13 Richard Sacco: Okay. I'm fine with doing things that way, there's honestly, nothing wrong with it or anything. The other way is just like I said, one of them is just more.

04:21 Adam Curcie: you know, we're getting a relative, you know, within reason relatively Like how you for? Dating every day. that's that's what's the most important thing, you know, not overextending resources, getting accurate data reporting, so

04:40 Richard Sacco: Okay, that sounds good to me.

04:44 Aksana Rahouski: Subcharge, distributor subcharge the 425, If his sub customers. Has one sub. Customer has one device on Bank account payment.

05:18 Devon D'Andrea: Yeah.

05:19 Aksana Rahouski: sub company that sub company has one device and one payment method, right? But distributor itself has also let's say, A device that also falls into the 795 subcharged? do we want to apply both rules for distributors then when it add the acts like a device owner himself but also like parents to others,

06:24 Devon D'Andrea: I would say, That's a Extremely rare. If not almost just gonna go ahead and say, Never gonna happen edge case.

06:36 Aksana Rahouski: And is it because they never buy themselves. Actually have these devices or never have one or

06:42 Devon D'Andrea: Where paying us for? I mean.

07:06 Aksana Rahouski: Like, and I understand that it never happens. You however, like Think about it when you write the code, right? You need to kind of

07:13 Devon D'Andrea: Yeah.

07:14 Aksana Rahouski: thinking, if technically distributor could have two rules, apply to myself and apply to my like it's based on my children. Do we want to have two like exclude both? Or do we just kind of want to assume that if you're distributor, we'll ever track you for dress that children condition never for yourself.

08:26 Devon D'Andrea: Yeah, I think that would be fine.

08:28 Aksana Rahouski: Okay. Sounds good. Okay, so that gives us what we need. okay, so now we can talk about I'm assuming Service plans and that's something you guys brought up yesterday. Sounds like there was a need for you to set the price. To. Service plan and perhaps.

08:50 Devon D'Andrea: yeah, if you go to service plans, Sorry, Adam's question.

08:56 Adam Curcie: because I'm just trying to think in terms of like a distributor, who's actually Setting up Sub Customers.

09:11 Aksana Rahouski: Mm-hmm.

09:11 Adam Curcie: Who, at least at the You know, at the time of creation are going to only have one device.

09:19 Aksana Rahouski: Okay.

09:19 Adam Curcie: We, we need to make sure That not only is the knowledge of that $4.25, minimum charge known to them, but there would also need to be an option. So like, because again, right? So, if I'm Tyler, and I go in, and I build a sub company service plan Because of the order of operations that these distributors take like it almost needs to be dictated when they built the sub company. Because that's really like, I can't foresee them ever creating a sub company that has more than one device, that will later have one device. Like, it doesn't that just if they start with three and get down to one, I mean, or regardless, right? They still need to determine it at the time of creation, like, it's kind of, because that's just gonna be the most Ideal point in which they decide that like, regardless of what they're predetermined pricing is, they need to be reminded that the minimum amount. Any one payment method can be charged is four dollars and 25 cents.

10:56 Devon D'Andrea: Yeah.

10:57 Aksana Rahouski: Okay.

10:57 Adam Curcie: and what to charge this customer if they only have a single device, Because it's going to be different from any of your other stuff. Any of the other stuff once they have enough devices to exceed, you know, the $4 or seven dollars, whatever. Like, at that point, it doesn't matter. They can charge

11:15 Aksana Rahouski: Yeah.

11:16 Adam Curcie: whatever I'm based on whatever. But you know, and again, right? If it's one customer who only has one device,

11:22 Aksana Rahouski: and just to kind of,

11:23 Adam Curcie: And know when that's gonna be, you know, it's two devices A through devices. Like they should just decide it right when they built that sub customer. Because that way, whether it happens that day that they assigned the first device or if they assign three and then a month later they turn off to and not only have one device that rule and that logics already there they don't. Nobody else has to worry about it.

11:45 Aksana Rahouski: The rule, like, Okay, the rule is is gonna be there, right? Because we're adding it, This flag that I'm showing it only allows turn it off, right? I think What

11:53 Adam Curcie: Yeah.

11:56 Aksana Rahouski: be creation screen or something. Right, it needs message, right? That tells them

12:06 Adam Curcie: Yeah.

12:08 Aksana Rahouski: that make sure that when you subculture inside devices, the minimum, what these guys should be paying, you should be 425, right? No matter how many if it's one or something like that, I don't know what the exact wording should be.

12:22 Adam Curcie: It really should see like the way because this screen you're looking at, this is

12:22 Aksana Rahouski: yeah, this

12:27 Adam Curcie: our view and I don't know if you can easily navigate to a distributor accounts view the page, but they almost have a box. Like that kind of states to them. That like the way our says customer dual sim price, right?

12:43 Aksana Rahouski: But yeah.

12:44 Adam Curcie: It's been like customer single device charge and then in parentheses must be greater than $4.25.

12:52 Aksana Rahouski: Yeah.

12:53 Adam Curcie: minimum. And then again, nobody has to worry about it ever again, moving forward.

13:06 Aksana Rahouski: And it's it. I do totally agree with you. That the fact that This rule in same as like the other rule of 795 South charge, right? These ruler kind of like, baked in the system silently. Yes. There is like there's no indication that these rules exist other than here is just as you can turn it off but like you don't know what it is to start, right?

13:29 Devon D'Andrea: Yeah.

13:30 Aksana Rahouski: So perhaps. Yeah, we could, we could think about where that message belongs when they get to see it. But like, yes, they need to be aware about the fact that if you break the rule, it's right. If we could call these rules, this Earth's gonna be penalty, right? And But today.

13:47 Adam Curcie: What, it's

13:48 Devon D'Andrea: I'm Tyler and I have a $3.75, you know, custom price for ATMs. What? Adam is suggesting is Tyler gets informed. Hey When he's building a sub company. Hey, if this person only has one device,

14:16 Aksana Rahouski: Uh-huh.

14:21 Devon D'Andrea: It, you know, it has to be a, you know, the the commission calculation is going to be based off a minimum of 425. So what Adam is suggesting is if Tyler was gonna have us charged that customer. You know, six dollars.

14:38 Aksana Rahouski: Yeah.

14:38 Devon D'Andrea: For on them, Custom Service Plan page.

14:42 Aksana Rahouski: Uh-huh.

14:43 Devon D'Andrea: Adam's saying, You know, on this page when you're building the sub, is there a price that you want to charge them if they only have one device and maybe that's seven dollars?

14:57 Adam Curcie: If you hear, can I share just for one sec.

14:59 Aksana Rahouski: Yeah, let's see.

15:00 Adam Curcie: To that for a customer. Like, he's like, you know, if he sets it at seven and he's gonna be expecting 350. Well, he's not getting 350. So because he's going to pay like the minimum for that is 425. So just some way that like you know, that when they're adding those subs they can just set it up right there and then and then we don't have to worry about it ever, again, really, or they can just come in and deal with it later. If they feel the need to But you know, just kind of like get it out of the way because it's not something that's gonna be a big deal for most of us. Most the time it's just a big deal right now because we, you know, had this one billing cycle where none of this was accounted for. so,

16:43 Aksana Rahouski: Hmm, are you. So, are you saying, rather than kind of implementing the rule itself, you just set the minimum price for a single. And we always end up basically kind of becomes a rule that is said,

16:56 Adam Curcie: above, 425 for singles.

17:02 Devon D'Andrea: Well, now, I mean you still you they still have to calculate, they still have to calculate it.

17:06 Aksana Rahouski: Yeah, because

17:08 Devon D'Andrea: Based on the 425.

17:12 Aksana Rahouski: well, and also like You still have to do kind of. Because what? If they, okay? What if they start single and become not single or vice versa? Okay, I need to

17:23 Adam Curcie: I mean you have like like you know about I don't know. Five days, we'll see.

17:26 Aksana Rahouski: kind of like think a lot of for a second. Okay, okay.

17:38 Adam Curcie: When, are we running Billings? Devon next Tuesday? Next, Wednesday, we have a week.

17:42 Devon D'Andrea: Yes.

17:43 Adam Curcie: So I mean we don't need to solve the necessarily this on this call but that the

17:49 Aksana Rahouski: Yeah.

17:50 Adam Curcie: understand what I'm saying though like the district need to know that that is

17:52 Aksana Rahouski: I think. Yeah.

17:54 Adam Curcie: their minimum like like this cost of doing the ACh.

17:57 Aksana Rahouski: Yeah.

18:00 Adam Curcie: So like you know, they can't, you know, stipulate that we charge somebody X under the assumption, they're paying why which would be like in Tyler's case? 350, They can't charge seven assuming that they're going to get a $3.50 return based on a $3.50 price. Because there's a there's a dollar or 75 cents lost there on our side. So that's that's kind of like the whole thing.

18:30 Devon D'Andrea: You're why can't he just do that on the when he first sets up the the custom service plan?

18:38 Adam Curcie: Well yeah, I gave me. Technically, it could be there, too. That's fine. Like it

18:42 Aksana Rahouski: Yeah.

18:43 Adam Curcie: could be on either page, really I mean.

18:46 Devon D'Andrea: Like on that page. Yeah, he's gonna see 375 but It'll tell him like Hey it's gonna be automatically if it's for single device it's just to let him know when he's going in and he's setting up the the ATM custom service plan for his sub. He's setting it at six dollars or whatever, and he sees that notice, he might want to be like, all right, I'm gonna charge a little more than Because I don't know how long they're gonna have one device. Why does there have to be a separate field?

19:38 Adam Curcie: Test warehouse. and I'll change the test warehouse phrase space price to Environment you have anymore. Search that there's an add one, the test warehouse, can you, and we will change their adjusted price to $3.50.

20:28 Aksana Rahouski: Adam, do you want to share?

20:29 Adam Curcie: Yeah, I'm just cutting it up and I have too many browsers. I might be good. Yeah. Okay. So you so, Come back here. Let's share button. Okay. All right, so you guys can see tests up to and 350 is the sharing the right tab.

20:54 Aksana Rahouski: So you create enough service plan. Okay.

20:54 Adam Curcie: creating a custom service plan for tests up too, and I want this to seven dollars which is I can charge them seven, right? If you is that if I'm invoicing them one box at seven I'm going to expect to make in Commission 350 Based on 350 seven dollars 350 350, but not 350.

21:30 Aksana Rahouski: Yep. Right? Because you're gonna bump 350 to 425 or and then that's gonna be the

21:36 Devon D'Andrea: Yeah.

21:37 Aksana Rahouski: difference between 7 and that that's gonna be what you're gonna be making.

21:39 Devon D'Andrea: It's the model is it always just need to have something on here to inform them of that.

21:44 Aksana Rahouski: so that's what I like option, a right, which is kind of put that disclaimer somewhere, that say that, In case your Sub-customer. Has one device. No matter what it cost will be charging you for 25, right?

22:02 Adam Curcie: Well, that's not the way I mean. Technically like that's not exactly. He? He's too many generalizations but kind of I mean I understand what you're saying it because it doesn't really not no matter what it cost is not correct. But I mean, I get, you know, if their bills $10, if it's not ATM.

22:24 Aksana Rahouski: Yeah. Yeah. But like still, we are.

22:29 Adam Curcie: you need work as a minimum, like,

22:31 Aksana Rahouski: Yeah.

22:33 Adam Curcie: You could put it here like the problem too, right? With having it here is your depending on them. Adding enough charge. What if they don't, What if they're just going to pass on the 350? What if they never actually take time? Because it's only one box to

22:49 Aksana Rahouski: Right.

22:50 Adam Curcie: come in and set this up?

22:51 Aksana Rahouski: Right.

22:52 Adam Curcie: So again, that's why I think that they just need to be informed of it at the time of creation stipulate. What the stipulate what to charge a customer who's total bill, you know, is under $4 and 25 cents calls. because like that because of the minimum, however, the wording needs to be, we can certainly You know, give you that finalized wording for the text boxes, but I don't know that. Do you like if I assign a company? Like, you know, if I don't have this here, right? If I delete this out,

23:38 Devon D'Andrea: A good charge 350.

23:40 Adam Curcie: why. I think having me when they build the company, And they, you know, they have to set that at the required field, you have to dictate it. You know, it can be predefined.

24:11 Devon D'Andrea: Wasn't that one of my? Wasn't that one of the used to test the test scenarios, you sent aksana? Like if it is if their chart, if their prices if these, if they set their place up price up below 425 Were just automatically gonna charge them for 25.

24:31 Aksana Rahouski: Hmm. Password talking about if yes because if if sub company price is below for 25.

24:39 Devon D'Andrea: Mm-hmm.

24:40 Aksana Rahouski: If you're charging distributor but not sub customer, then distributors in negative.

24:45 Devon D'Andrea: Right.

24:46 Aksana Rahouski: So we're saying that a raise sub customer for 425.

24:51 Devon D'Andrea: Right.

24:53 Aksana Rahouski: Then you basically are subcharging both for 25 and he doesn't make any money.

24:58 Devon D'Andrea: It's just nuts out at zero. Yeah.

25:00 Aksana Rahouski: Yes.

25:01 Devon D'Andrea: but that's in that case, it's fine but it still doesn't solve the problem that I guess Adam is trying to solve is just where we put The mess.

25:10 Adam Curcie: one out and now that I think of it this is actually I'm gonna I'm gonna throw a big wrench. Terrible fault. so, we have this, we have Yeah, people in here, I'm sorry. Yeah, we have

25:31 Aksana Rahouski: No.

25:32 Devon D'Andrea: Okay.

25:34 Aksana Rahouski: It doesn't really with you, is like a car, never ending riddle.

25:38 Adam Curcie: Any ability to even like, vast majority of our customers? Don't even know. We can charge under four dollars and 95 cents. So yeah, that's a big problem. You can't just have that out there. If they don't

26:07 Devon D'Andrea: Yeah.

26:12 Adam Curcie: actually have any discounted pricing because like, We've got probably over a thousand customers who are only going to pay $4.95, if they're on the ATM plan, and if they're not on the ATM plan, they're definitely paying more than 495. So just kind of like tipping our hat to anybody that like Oh yeah, you could definitely have a scenario where you're not even charging you four dollars. You're like is like minimum rebill is 425. So like, then they're gonna be like

26:39 Devon D'Andrea: Right.

26:44 Adam Curcie: out there with some four dollar pricing. And there's there's a really a very high number of customers, we would probably prefer not to know that. so,

27:10 Devon D'Andrea: so,

27:11 Adam Curcie: Although, I guess we could literally just push that all off under the cutie. Now that I think about it again.

27:18 Aksana Rahouski: Another one.

27:19 Adam Curcie: Well, we have this new program where we only charge people 275 for this one device on T-Mobile. So, I guess we could live completely blame any or answer any questions that any customer has with that. So maybe we don't have to worry about it. Actually,

27:36 Aksana Rahouski: charge them for 25 instead of 375 and the customer will still pay $5.99. So they making less money.

28:05 Devon D'Andrea: Correct. Correct. We just fit the only thing we really need to do is just add in

28:08 Aksana Rahouski: The 425 value needs to be.

28:12 Devon D'Andrea: the messaging.

28:15 Adam Curcie: And it's like, a text box where they can set that value. Right.

28:21 Devon D'Andrea: Well.

28:25 Aksana Rahouski: A configured, you're saying not baked in the system.

28:28 Devon D'Andrea: No. No Adams. What Adam's saying is that? I see, I don't know if I agree though.

28:31 Adam Curcie: now not, I mean,

28:35 Devon D'Andrea: Adam like it. That we have to make a whole new checkbox just for like, so he's trying to make it dynamic. Like if customer has single device,

28:41 Aksana Rahouski: Yes.

28:44 Devon D'Andrea: Add in an extra dollar surcharge.

28:47 Aksana Rahouski: Uh-huh.

28:48 Devon D'Andrea: So if he hasn't set up at 599 charge them $6.99.

28:52 Aksana Rahouski: Sure. Because you'll in

28:56 Adam Curcie: It's not that, that's not what I'm stating.

29:01 Devon D'Andrea: So then, what's the correct? So then what's the text field for

29:01 Adam Curcie: I mean. well, they have to have like exactly they need to see, like I don't know, I mean they have the price they have to set the price that they're charging, I just a a sub customer who only has a single device. Because it's going to be handled separately and their standard permission.

29:31 Aksana Rahouski: we will charge you more for the scenario, right? So we want them to stop like

30:01 Adam Curcie: Yes.

30:02 Aksana Rahouski: and like it also like a charge to a customer that they want to add knowing that they will be charged is our worst thing.

30:10 Adam Curcie: Yeah, well I mean yes, the entire reason that we have these commissions give these distributors the discounts and build. All of this is so that these guys

30:22 Aksana Rahouski: Hi.

30:22 Adam Curcie: can make money. Like he's the goal. We want them to charge these people more than we're

30:25 Aksana Rahouski: Yeah.

30:29 Adam Curcie: charging, then we give them discounts, so they can do that. And they competitive so, but we also don't want to like Miss mislead them on on what their cost is because if we set somebody up with a cost of four dollars, 375 350, They need to know that that's only applicable.

30:50 Aksana Rahouski: Uh-huh.

30:50 Adam Curcie: Or sub customers are actually.

30:53 Aksana Rahouski: single case, that's just two devices, you know. Yeah, so I I think I think that's what you're saying. So you're saying again just to kind of keeping it in this class area. Um 599 with the client will be, 375 is the price there on with what when one device will charge them for 25. Well we're saying we won them to set up additional for that single device charge, plus whatever 150. Let's say right.

31:30 Adam Curcie: Well.

31:31 Aksana Rahouski: That is what they configure. So that way when we that condition is met one device, Stop customer gets paid $5.99 plus 1.5 and they are paid for 25. That way they make an additional dollar fifty year. I, I don't know.

31:50 Adam Curcie: than $4.25 is going to have the applicable use case.

32:00 Aksana Rahouski: Yes. Yeah, correct.

32:04 Adam Curcie: and then, but and again, it's not just a like a dollar fifty

32:09 Aksana Rahouski: Mm-hmm.

32:10 Adam Curcie: But I mean it's you know, they just need because it they just need to have the ability to understand that regardless of what they're predetermined pricing is

32:20 Aksana Rahouski: Yep.

32:23 Adam Curcie: the minimum for a single device is 425. So if your customer is going to ever be

32:26 Aksana Rahouski: Yes.

32:29 Adam Curcie: build, you need to pick what that charge will be right now. If you're ever gonna be build under that, Even if they aren't you still have to decide it. Like even if you don't think it is we still need to know what to do. If that happens basically. So, what?

32:45 Aksana Rahouski: What which is again? They're just to like today, right? If we in build this rule in really you're paying more their customer is not paying more and they're

32:51 Adam Curcie: Huh.

32:56 Aksana Rahouski: making less, right? That's what.

32:58 Adam Curcie: That exactly. Yeah. What we don't want to happen though because they're not going to know that.

33:03 Aksana Rahouski: Okay, so what

33:04 Adam Curcie: Way everything works. Now they assume that because their price is 350 or 375 that that's what is going to be considered. And it's not

33:12 Aksana Rahouski: Right. Well that I think they're like yes. So problem one they don't know right? They don't know that they will not be paying 375 in this case but in fact, they'll be paying for 25 and they have no way, well they do have way they can change, I guess. Their costume, service plan 599 to hire but but the truth is they don't want to do it for anybody who's not as like a single orphan, right? So like,

33:42 Adam Curcie: it's Something they don't want to. I mean like They made, I mean they could charge nine dollars, they could charge nineteen, we know that that happens and that's fine. What they want to charge is on them, that's completely up to we, you know?

33:57 Aksana Rahouski: Yeah. Okay.

34:00 Adam Curcie: Just that they they need to you know if they're even if they're charging 19 with a three dollar plan and they're assuming they're gonna make 16, they're actually

34:10 Aksana Rahouski: Yes.

34:10 Adam Curcie: they're making 1525.

34:13 Aksana Rahouski: Yes. But that's but that's again, that's a worth of just kind of assuming that

34:13 Adam Curcie: so,

34:18 Aksana Rahouski: the rule doesn't change, will still apply, it will charge a 425. All we're missing is just language on the portal that tells them that in this case, we'll charge you for 25 instead of 375. She'll be making less. But I think what you're asking for is like, additionally, let them to control kind of what sub customer pays in this case, which Or is it or is it? Now what you're saying?

34:45 Adam Curcie: No. I mean it is they I mean I think it would just be easier to have it all set up in done with right there at the time that the account is created. But I mean, I guess I guess we don't have to do with that way or I don't know, maybe you, maybe it's a checkbox that if they want to dictated, you know, at the time of creation they could I don't know.

35:15 Aksana Rahouski: Okay.

35:15 Adam Curcie: I'll let that play in.

35:18 Aksana Rahouski: Let me ask you just because like we have we have actually stone working on it. Do you want us to pause until we define more clarity? But again, he's working on kind of fact. Big. This rule in the system, we will be upcharging in the single device present for sub customers, right? We, we are our this, to me. It sounds like on top of this rule, which we want to apply, definitely to protect ourselves, right? We also want to give a customer a knowledge of this thing be potentially. Some way for them to optimize their children when that happens.

35:57 Devon D'Andrea: I think for now we just add in the messaging letting them know. I don't I just don't think that we need to add in We're already, we're already. We're already covering ourselves by making sure that it's being calculated against 425. As long as the sub customer, like like, if they want to make money, they got to go into that page, regardless to make them to make the customers. To set the customers up charge, so that's on them. And as long as there's a big message there, just letting them know like You know. Whatever word words, we pick. You know that there's there's this minimum

36:39 Aksana Rahouski: Where do you think it belongs that message?

36:43 Devon D'Andrea: On the mat on the Customs, Managed Customer Service Plan page.

36:47 Aksana Rahouski: Okay. Yeah, that makes sense because it shows the baseline, which is what they think, what they will be paying, but that's where we actually want to know. Yeah, it says it does say 375 there is this exception? That apply 375. So we want to put

37:06 Devon D'Andrea: Right.

37:08 Aksana Rahouski: exception on that page, okay? Okay, and we can start there, and then if we feel like we need to keep rolling again, just kind of wanted to. Like, if, if you feel like, yep, rules still must exist. We're rolling or a kid, pause, until we think more through this.

37:27 Devon D'Andrea: No no, I think everything you guys are already have set up is good. I mean if we want to look at what Adam's talking about, maybe down the road, maybe we can look at that if it's something that a sub company, if it's something that a distributor would find valuable,

37:43 Aksana Rahouski: Okay, we will add the message. How about that? We'll add the message. We'll build this once we review. I think sometimes things like that, they just click, once you see it, you know, and then maybe we'll help us to push through.

37:52 Devon D'Andrea: Yeah. Okay.

37:58 Aksana Rahouski: Hi. Okay, so sounds good. So we have 20 more minutes and but I think we might be actually good. So tell us about kind of what we started with yesterday. It almost sounded like and maybe you guys want to share. I know you had some tables. You were looking at to me it sounds like almost there is a need for us to log me out for when we set up service plan, differentiate price per carrier. Is that what we're talking about? But also like, I'll just take the stage and tell us, we'll problem. We're trying to solve.

38:31 Devon D'Andrea: Sure. so, this right here, That's just a view. so, this page right here,

38:45 Aksana Rahouski: Uh-huh.

38:46 Devon D'Andrea: He? Slightly. Different. So we've got right here, our ATM service plan. Let me just We've got our ATM service plan and globally. Our global pricing for the ATM service plan is 495. so what we want to do is make this add two additional layers, so they'll be model And carrier.

39:23 Aksana Rahouski: Okay.

39:24 Devon D'Andrea: so, right now, For Verizon. so, I guess it would be like, first thing would be like, I-22, and then we would have Verizon 495 or AT&T 495.

39:37 Aksana Rahouski: Uh-huh.

39:41 Devon D'Andrea: That would be like another model origin and it would and these will be the global prices origin origin will. I mean, for this purposes of this conversation, we could just say that that's got a four there and then Buddy will only be, T-Mobile will not have even

39:57 Aksana Rahouski: Okay.

40:00 Devon D'Andrea: options for Verizon or AT&T how Ever. We said to ignore this. Mmm.

40:10 Aksana Rahouski: Yeah, maybe for now. Okay.

40:12 Devon D'Andrea: Too. so, Basically. We need to figure out how that's gonna work. Figure out, figure out how that's gonna work in terms of Existing distributors.

40:34 Aksana Rahouski: Cuz you're because then you're tapping into if you're setting. A base service plan, right? And now you're not just like Your price is based on two. Other inputs is a carrier and model, right?

40:49 Devon D'Andrea: Right.

40:50 Aksana Rahouski: And now Custom Service Blend created off this. it needs to be adjustment for each rates, and if they're creating A distributor creates a sub, customer service plan. 495 might become 695, right?

41:09 Devon D'Andrea: Yeah, so merchant money. So just for example, In Tyler's situation. he's got this, 375, so if we look here and we make these changes In all reality. Deal.

41:31 Adam Curcie: well, I real quick, I mean there is no devices in APC as of today, Other than like a double digit at most number of T-Mobile I-22s that like we don't have any T-Mobile stuff in there. We don't have any origins in there. So like everything that's in there right now is an i-22 on Verizon or at I-218. So like, I don't think we have to account for retroactively adjusting any pre discounted pricing because all of that discounted pricing is based on the existing 495 which isn't changing.

42:08 Devon D'Andrea: so,

42:10 Adam Curcie: so, like if we give either Tyler in this case would need another

42:12 Devon D'Andrea: Right.

42:17 Adam Curcie: Tier that literally would say, instead of usage limit, it would say. T-Mobile i-22 and then it would have a base price of 350 and a different predetermined discounted rate. Like so anything that's in there with any of these guys on the ATM plan that doesn't get touched because that's really gonna be a placeholder for moving forward. What will be the i-22 on Verizon or the I20 on AT&T? We don't have anything assigned as an i-22 on T-Mobile, because it doesn't even exist in the portal. So, like, we don't have to account for the fact that that's a different price. Right.

42:56 Devon D'Andrea: Yes, so what would happen if Like let's just for example, I'm not actually gonna save this but like what if we have a distributor in here? That is currently. So, when we build this, you're saying like, When we, let's say, we have a distributor who doesn't have any discount on the ATM plan, right? So when we build this,

43:20 Adam Curcie: It's gonna be a separate line item.

43:23 Devon D'Andrea: There's gonna be. Separate line item and

43:26 Adam Curcie: We have on the other on like tier one, how we have numerous line items and numerous adjustments.

43:32 Devon D'Andrea: Yeah, so these will be these. These these numbers will be shown here.

43:38 Adam Curcie: I i would assume that's the way it's gonna be done. Based on like because you're adding layers. So you're gonna have to present layers,

43:46 Devon D'Andrea: Right.

43:49 Aksana Rahouski: So, let me go to somebody with layers like other service plans, have more like users.

43:54 Adam Curcie: yeah, like Sorry, what? I hear the last thing you said,

44:01 Devon D'Andrea: This is only for the ATM service plan.

44:03 Aksana Rahouski: Oh, this is this. Okay. Are you saying this is only needed just for this plan?

44:09 Adam Curcie: Yeah.

44:10 Devon D'Andrea: Yeah. Sorry I didn't I don't think I said that.

44:12 Aksana Rahouski: I think, okay.

44:12 Adam Curcie: We, we might really, you know, make this complicated later and add different levels for different carriers on the tiers. But for today, that's not

44:22 Devon D'Andrea: Yeah, and not necessarily so it's just this service plan. In this winter conditions are met. Is it a T-Mobile i-22?

44:35 Aksana Rahouski: Okay.

44:36 Devon D'Andrea: Is a Verizon. Is it a T-Mobile origin is AT&T origin.

44:41 Aksana Rahouski: But then do you need to reset this prices for custom that give you let's say, you have. So let's say we set this up, right? We have our table, sorry, our AT&T plan, Oh gosh, I don't know what I'm saying. You know the TM plan with

44:58 Devon D'Andrea: so, for Tyler,

45:01 Aksana Rahouski: Yeah. Let's find somebody who has cost.

45:04 Devon D'Andrea: so, for Tyler, He's currently a 375. So what we would do we would have to just inform him That. These. Additional. Things in here. He's gonna that we, I don't know. I

45:24 Adam Curcie: Well, I mean it's the same. Okay. So here Dev create him a tier one service

45:28 Aksana Rahouski: Right.

45:29 Adam Curcie: plan. Obviously don't save it, but go back and Because he doesn't have tier one pricing. Right. So It would literally just like Look, that's all default, right? And it would look like that at the top. It would instead of usage limited, it would say origin and then, you know, Verizon T-Mobile origin AT&T, and then those base prices would

45:48 Aksana Rahouski: The. What?

45:53 Adam Curcie: be different.

45:54 Aksana Rahouski: Yeah, and

45:54 Adam Curcie: We don't have to add those layers.

45:56 Devon D'Andrea: So the question that is is, are we going like that's a conversation? We have to have internally though because, you know, we told Tyler, he has an ATM plan price of 375, right? So are we automatically Just gonna give him 375 When they build this, or do we have to inform Tyler and any other distributors that have access to a custom ATM plan? That. These plans, I don't know.

46:36 Aksana Rahouski: Are you are you read about all custom ATM plans that exist today? How we gonna recall calculate and like we see data?

46:45 Devon D'Andrea: He, well, yeah. How are we, like, What are we just gonna like to add? Like, what Adam said is, Are they gonna just be built? But as line items with just, you know, with, with no discount and then Yeah, that's not gonna work. We have to dictate. How many distributors have we have? We have Therefore, is there a way for me to look at just? No, it's not.

47:16 Aksana Rahouski: Customer ATMs for distributors.

47:18 Devon D'Andrea: Yeah.

47:20 Aksana Rahouski: Probably, I don't know throw here but I'm sure we can pull it from. The database.

47:26 Devon D'Andrea: Yeah, I mean I could just pull all this but there's a lot.

47:34 Aksana Rahouski: Or has.

47:35 Devon D'Andrea: Actually watching either your readers.

47:36 Richard Sacco: Yeah. Yeah. One thing, another thing we have to do before we do this, work is remember that. If you edit a base service plan, all the custom ones, go out of whack. I think we got to fix that problem.

47:53 Devon D'Andrea: Yeah.

47:53 Adam Curcie: Well, and that's why that was one of the reasons I said we're not going to be editing the base service plan. We're going to be

47:59 Richard Sacco: Oh, okay.

48:00 Adam Curcie: Yeah. Like that base service plan that exists today isn't changing. We just have to rename it. Because like you guys have it like on that on the plan where the service plans are differentiated. The only thing that really you know is categorized as an attribute to it is the usage. So you're, you know, you have a usage limit, That's the only attribute of that service plan is that it, you know, but it's gonna, I guess get additional attributes but I don't want to edit it in in a sense where you're gonna have to then

48:32 Richard Sacco: you don't want to deal with the base one, you just want to edit all the custom ones to now have this other SIM card option or however we do it pretty much

48:40 Aksana Rahouski: But then, I guess the question that I have pricing that you were showing because today ATM based plan is set up with 495 Dewey, and just want to do a model carrier 495 Everybody gets 495, or, or however build it in some way that there's some kind of like, default value and not leave. If there are not set up granularly per model cross carrier combo, it's gonna be this default value, which is today is Yeah, 94.95, we could build it that is like backwards compatible, right for Is that what you're saying? Like, so but you are showing here that these prices are different to start with, is this already custom result, or is this a base?

49:28 Devon D'Andrea: It's a base.

49:29 Aksana Rahouski: To the artist, But there are different prices, right? So they're all they're not all 495. So we do have to touch base

49:37 Devon D'Andrea: Yeah.

49:38 Aksana Rahouski: Okay. What is the problem Richard maybe like I start, I did.

49:43 Devon D'Andrea: So, we ran into this a while back. Where if you I forget what we did, but we did something.

49:51 Richard Sacco: Basic. Yeah, basically, if you edit one or any of those plans and don't actually do this please, if you edit it, if you edit it, and you change anything and you hit Save, it actually gets a new ID and this is because it has to do with the approval system. So all the custom plans associated with it are no longer associated with it. So yeah, they break it breaks everything. So then I've had

50:14 Devon D'Andrea: In a break.

50:18 Richard Sacco: to go in manually fix these and we've kind of like been kicking. The can down the road on fixing that base issue, but I do think that we should Fix that.

50:28 Aksana Rahouski: Yeah.

50:30 Adam Curcie: Devon click on Create Service Plan. so, I mean, In here is again, where I would envision and you guys can tell me it's not gonna work that way. It doesn't. But like, you would basically, you know, have to enable which carriers, because right now the device group names that that's important for what it does. But you're basically gonna say, Okay, Well, you also

50:56 Aksana Rahouski: Huh.

50:59 Adam Curcie: need to define like which carriers this, this service plan, perhaps,

51:05 Richard Sacco: Apply to.

51:06 Adam Curcie: Yeah, or you do it under the price tier if you add price tier and then instead of again just usage limit, you know, you have model and carrier and then you can define the price that way. I mean that's in like again there's just not a lot of attributes in here because I don't want to change the ID you know as Richard stated so that will be a problem.

51:31 Aksana Rahouski: Yeah, I'm looking at that like that's it sounds like a pretty big problem need to be fixed. but would you said though, like,

51:40 Adam Curcie: I would say it's a pretty big problem. We need to avoid. Oh, fix it.

51:47 Aksana Rahouski: Are you then today never changing these like base plans?

51:51 Devon D'Andrea: We haven't changed them in ever. And the way, the reason why, the last time we broke it. The last time we broke it is because we wanted to add all these little micro tears to tier three.

52:01 Adam Curcie: Yes, that was it. Yes, we added.

52:05 Aksana Rahouski: You know.

52:05 Devon D'Andrea: So our tier three used to start at the half you. And then we went in and we

52:09 Aksana Rahouski: Well, but like if we're saying that ultimately an ATM base plan will have to be

52:11 Devon D'Andrea: added this. These micro tears. So by megabyte and when we hit save, we um oh we

52:13 Adam Curcie: Text.

52:20 Devon D'Andrea: lost all the custom servers.

52:21 Adam Curcie: We is that we created hours of work for Richard.

52:33 Aksana Rahouski: changed, right? Because we need to know account for a carrier model. This will come in issue. We have to face in my in my head. Are you guys thinking some other way around kind of still like judge it? And because one way yeah Richard can go and just patch the data in the database.

52:58 Richard Sacco: Yeah, that's what I've been doing so far. Yeah.

52:59 Aksana Rahouski: Well, and also what? What I want to go back to the sample that we're looking at before that, if let's say,

53:05 Richard Sacco: Of.

53:06 Aksana Rahouski: Part of creating a service plan. Now it's become like which carriers the supports pick all three and now you can set up pricing Prokary or a slack plus model combo, right? But also that kind of goes against ATM. Is the only one, which sounds more like hard coding. We're hard coding, you're only the single And has an option others. Stay as is Is which one is it? Are we do? We want to kind of add that ability to any plan, we create in the future to have that product like granular pricing Called.

53:41 Devon D'Andrea: If it's not gonna make a difference. And it's something that doesn't, you know, break anything and we can just, we don't have to do anything until we decide to do something.

53:53 Aksana Rahouski: Yeah. Okay. Which is well, how we would do it. We would build it backwards. Compatible meaning, whatever exists today works. There's this actual air we add, you could pick it, choose to add it to any service plan. Which in this case will be, just one and we need to make sure that that changing that plan doesn't actually create orphans. Which order does today sounds like

54:18 Devon D'Andrea: Right.

54:19 Aksana Rahouski: Um, Richard go ahead.

54:21 Richard Sacco: Yeah, I think maybe a smart way to go about this and I'm just sort of brainstorming here but is if you had default pricing that didn't depend on carrier but then you were also able to add her carrier pricing and so that way nothing would really be affected until you added the extra attribute.

54:40 Aksana Rahouski: Yeah, it's a trust that's like optional but not required. That would make it

54:43 Devon D'Andrea: Yeah, I like that.

54:47 Aksana Rahouski: backwards compatible because whatever exists today works, we don't have to touch

54:49 Devon D'Andrea: Right.

54:52 Aksana Rahouski: it.

54:53 Devon D'Andrea: Yeah, I agree with that.

54:54 Aksana Rahouski: But isn't that though? Like, I feel like again from a need right before saying

54:56 Devon D'Andrea: I just have to we have we have to make a decision internally on what we want to do with the existing. Distributors that have custom ATM plans. I just, I don't quite know the answer that right off the bat.

55:16 Aksana Rahouski: that base plan supports granular pricing right with Model slash carrier plus carrier, right? Children of that. Should support the same thing. So right like and again I'm just saying a lot to see if you're like Yeah that makes sense because if you then have a base plan for ATM that allows you to set the granularity, any child created, all that allows you to reset. Four to three five to four etc, right? So it's really just like mirrors it to like you can recent about fall. They'll be apples to apples, but you can in like

55:55 Devon D'Andrea: Right.

55:55 Aksana Rahouski: a job dollars if you want.

55:57 Devon D'Andrea: So that would mean that if we want the day one that we go live with this. there would be, we would build one once we build in the additional carrier related and model related charges

56:10 Aksana Rahouski: oh,

56:11 Devon D'Andrea: That would mean that. I would come back into Tyler's this page for Tyler and I will see that he has a T-Mobile I-22 that is not, you know, 95 cents less than 375.

56:27 Adam Curcie: Did. Because that would probably be a big.

56:35 Devon D'Andrea: Yeah, I don't know if I would want that.

56:37 Aksana Rahouski: But look nice to tell me. What would you want in this case like

56:40 Adam Curcie: Good.

56:41 Devon D'Andrea: Okay, I don't know. I don't know. Okay.

56:45 Adam Curcie: I want to just ask, I mean, so we know that if we actually change the service plan that service plan screen for me, not to customize the create service plan,

56:57 Devon D'Andrea: Yeah, we know that.

56:59 Adam Curcie: If we, if we change the service plan,

56:59 Devon D'Andrea: Well.

57:02 Adam Curcie: That's going to create a new service plan ID, basics, right again, right? Like

57:07 Devon D'Andrea: Right.

57:07 Aksana Rahouski: Yeah.

57:08 Adam Curcie: if we add attributes that click on View, sorry,

57:13 Devon D'Andrea: Sure.

57:15 Adam Curcie: like if we add attributes into what are currently defined for price tiers,

57:22 Aksana Rahouski: Yeah.

57:22 Adam Curcie: We don't have to change the actual service.

57:26 Aksana Rahouski: And that I thank you guys like what I think you're doing. You're dancing around this bug right now. And that like If we fix it like touch service plan, whatever it is about, it shouldn't create a new service plan that's like attribute or data or whatever. I think the fact

57:42 Devon D'Andrea: Right.

57:44 Aksana Rahouski: that it like creates new service plan with the new ID is a hundred percent bog and we continue dancing around it. But I feel like it's, it would be cheaper for us to just drop it. It makes things so much easier moving forward.

57:55 Devon D'Andrea: Yeah.

58:00 Adam Curcie: I mean that's fine if you think that that's the best way to go about it. I just I just know. We basically you know only ever change the service plans. Once or twice now in what is it, you know?

58:14 Aksana Rahouski: Huh.

58:15 Adam Curcie: Two and a half, three years, whatever.

58:17 Aksana Rahouski: Yeah, but now we're saying and again the music we still kind of need to like dive in. I don't know if like twice is like Aaron's to service plan and changing

58:22 Adam Curcie: Yeah.

58:26 Aksana Rahouski: those doesn't actually change service plot itself. But like, we are now changing right that the definition of a service plan, the the dimension of this thing is going to change. Meaning

58:38 Adam Curcie: Now.

58:38 Aksana Rahouski: You might need to come in and do it manually. We might want to script the whole thing and just clean knob data with the script, right? Third options there but at the IT we all need to like kind of run with an assumption that touching this thing doesn't doesn't break it completely.

58:55 Devon D'Andrea: Yes.

58:56 Adam Curcie: Excellence. yeah, I just don't know if like again to Devon's point or, you know, which I was gonna raise as well like,

59:03 Aksana Rahouski: Okay.

59:04 Adam Curcie: I don't know if building these additional benchmarks for how we build these carrier model combinations, it kind of sound like you were just gonna take 495 and then it add, You know, a discount onto that based on certain you know conditions if it's an origin on T-Mobile then discount one dollar and you know 45 cents or something which you know that again to the exact point we just looked at with Tyler like that wouldn't work at all. If that's the way we're going to have that. Kind of run within the entire scheme of how all the other distributors are already discounted. so, That's yeah, I mean there's definitely a lot. We have to think about so

59:54 Aksana Rahouski: So do you want to? Okay, so this one, let's just wear. I know where our time. Do we want to? Resume another one, maybe like set up a meeting next week or because I don't think we have other than, you know, we could definitely look into what it's gonna cost us to fix it so please touching it, it doesn't break it, right? And

01:00:11 Devon D'Andrea: Yeah.

01:00:15 Aksana Rahouski: then like if as we expanded, right? What is the smartest way to expanded that creates less work and fits your business model as far as distributor approach and such

01:00:28 Devon D'Andrea: Yeah, I

01:00:30 Adam Curcie: Yeah, we definitely.

01:00:30 Devon D'Andrea: It's not even just it's not even just distributors either. It's also just regular customers that have custom service plans.

01:00:37 Adam Curcie: Yeah, we have a lot of large customers. We just have discounts

01:00:38 Aksana Rahouski: Okay.

01:00:41 Adam Curcie: So we definitely can't have like chord bays. Actually the lowest even lower than Tyler, they pick 350 for ATM. So,

01:00:48 Aksana Rahouski: Yeah. And and it sounds like you don't know yet. How would you want this to act? Your

01:00:57 Devon D'Andrea: Yeah, if we want this to all set. Yeah, how we wanted to offset like day one or do we want to Just determine how many distributors, and

01:01:10 Aksana Rahouski: Uh-huh.

01:01:10 Devon D'Andrea: Customers we have currently. On customized ATM plan. And then just go in and make the adjustments. You know, day one.

01:01:23 Aksana Rahouski: Yeah. Okay.

01:01:29 Richard Sacco: yeah, maybe this can just be like homework and you decide how you want to do it

01:01:32 Devon D'Andrea: Yes.

01:01:34 Richard Sacco: day one and we could Talk amongst ourselves and come up with the solution in terms of, yeah.

01:01:39 Devon D'Andrea: That sounds fantastic.

01:01:43 Aksana Rahouski: Yeah, that's you guys think. And we will think and we can regroup

01:01:46 Devon D'Andrea: Yeah.

01:01:48 Aksana Rahouski: Can we but we can book something maybe next week to look back and

01:01:54 Devon D'Andrea: Yeah, let's loop back.

01:01:57 Aksana Rahouski: Okay.

01:01:58 Trista Smith: A regular check-in next week. Do you want to use that time next?

01:02:01 Aksana Rahouski: Yeah, perfect. Yeah.

01:02:02 Devon D'Andrea: Sure, sure.

01:02:05 Trista Smith: All right. That sounds good.

01:02:07 Devon D'Andrea: Oh Trista. We are just waiting on confirmation from our guy from, in hand about

01:02:07 Aksana Rahouski: oh,

01:02:15 Devon D'Andrea: that. Those dates that you gave us.

01:02:17 Trista Smith: Okay, sounds good.

01:02:19 Devon D'Andrea: But, you know, ASAP.

01:02:20 Trista Smith: Okay, perfect. Thank you.

01:02:23 Devon D'Andrea: You know. All right. Thanks everybody.

01:02:25 Aksana Rahouski: Thank you.