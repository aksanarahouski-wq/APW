00:00 Aksana Rahouski: How do we estimate this?

02:01 Richard Sacco: Yeah, I think this one's pretty simple. I would estimated an hour myself.

02:13 Aksana Rahouski: Wait a stone here. Stone your hair right?

02:16 Stone Marballie: Yeah, I'm here.

02:18 Aksana Rahouski: off, we're not either, we're thinking about completely different solutions or something. It cannot be like seven hour gap between two estimates.

03:00 Noah Bratzel: Yeah, I think Richard's probably pretty close with the one hour I would give it a little bit more. Not closer to what Stone said. But the I guess my only question on this would be, do we want to add A somewhat more global helper. Is this something that they're gonna do often to make it easy to make links that for certain, permissions aren't viewable, and they just see texts instead of a link and stuff like that, because I know I did that, On different projects. And so, probably pretty basic to bring that type of functionality over And then that would make a maybe more like three but it would be easier. The next time we have something that sounds very much like this to do it.

03:43 Richard Sacco: it's like only gonna be Super Edmonds. Only. So, Based on that, do with what you will.

04:00 Noah Bratzel: Okay.

04:04 Aksana Rahouski: Yeah, I would say. Yeah, Aaron get a lot of these but but I do like, how you think because that makes it more scalable, right? Because you are like investing in your future to make your future life easier. So I would say in this is like I'm with Richard. If you feel like just use your judgment, you feel like it's a low hanging fruit to make your life easier down the road. Go for it.

04:25 Noah Bratzel: Yeah. Yeah, I would probably not do it based on that input and then just have it in the back of my mind. If I see a ticket like this again, then I probably would do it because it only takes a couple times for it to be worth it, but

04:41 Aksana Rahouski: Okay, so should we put two hours just kind of line somewhere in the middle?

04:47 Noah Bratzel: Good.

04:48 Aksana Rahouski: Okay so this is something also like Super Minor. They submitted a they requested a while ago and through the email actually. So, they want to remove John from. Whatever email that is oh company deactivation. In it should only go to a service apparently. He's probably not on the list for the service open wireless. But he's, I don't know if he's like copied, but you guys have to look at the code. If it's like, literally, he is like Hardcoded on the list of recipients, or, I don't know, but they want to get him out of this emails.

06:29 Richard Sacco: And so for the for this one, basically.

06:29 Laura Perry: I think Jay Donovan just sorry. Jay Donovan is Jen.

06:35 Richard Sacco: Yeah.

06:35 Aksana Rahouski: Oh, sorry. Yeah.

06:36 Laura Perry: Just just in case it comes up with the client, you start going. He just, you know, it's just

06:42 Aksana Rahouski: First, first letter, and made an assumption. Okay, Jen? We want to remove Jen from

06:48 Noah Bratzel: So sexist assumptions. I can't believe.

06:50 Laura Perry: I mean, I don't think she's only been on like I don't know maybe five calls since I've been on the account so she's not really involved anymore, but

07:01 Aksana Rahouski: She's been on maybe like two since I started and I'm coming up to my year in

07:03 Laura Perry: Yeah. Yeah, every everybody else that we deal with at this at this place, a guy so

07:08 Aksana Rahouski: junior.

07:14 Laura Perry: make sense.

07:15 Richard Sacco: Nice.

07:15 Laura Perry: Sorry, continue.

07:17 Aksana Rahouski: Okay, so

07:20 Richard Sacco: The email to that. Email to wherever it's configured and that'll be it. But I find with Stone's one hour.

08:02 Aksana Rahouski: Yeah.

08:03 Richard Sacco: I don't know if you want to get Noah's input that we want to get all three of our, okay? There you go. We're all at one. Okay.

08:11 Aksana Rahouski: This is the one. Yeah. I like this voting. Yeah, that's

08:15 Noah Bratzel: and just to verify, I think what you said the stuff comes from A config, right? It's not. Is it or is Do we just have hard coded stuff and all these

08:24 Richard Sacco: Well in this case, right it goes to all the Wtm admins.

08:29 Noah Bratzel: Okay.

08:31 Richard Sacco: Is currently is what it's doing but we can just re-route it to go to something. We have in the config because I know that service at all point wireless.com is configured somewhere in Productions App. Local And we should just use that sort of logic.

08:47 Noah Bratzel: Yeah, that makes sense. Okay cool.

08:51 Aksana Rahouski: Okay, so long we have done one. Okay, let's do one more. So, this one. Okay. So, feel chuckins, where you guys remind me where the field chickens live,

09:09 Richard Sacco: It is an admin. Yeah, you're yeah.

09:10 Aksana Rahouski: Browse. Chuck.

09:16 Richard Sacco: Device checking failure. Same kind of a length in

09:19 Aksana Rahouski: So, they just want to be able to export this data.

09:24 Richard Sacco: So I think this depends on if they want all of it or they're okay with the most recent thousand or whatever they want. All of it, it's gonna take longer if they just want the most recent thousand or something, it's going to be easier.

09:36 Aksana Rahouski: care about.

09:50 Richard Sacco: The last two hours can be more than it can handle and request. I think like, the it has to be like the last X thousand. If we're gonna just make it consistent. I like and I would have to test exactly basically if it's more than that, you have to put it in a job and the job has to run in the background and we have to do that. I mean, it's not crazy man extra time but it is extra time.

10:15 Aksana Rahouski: Okay. So now let's do these are kind of the minor one, what we want to get through and if we do, I want to talk about the spike. Richard, if you want to kind of take over right now and walk us through. The two tickets that That you created. Is it? It must be it. I think it's this one, right? It's this one that you created today.

11:15 Richard Sacco: No, no, that was not one of the this is I think a really old one. I want to face. Don't need it. I start goes them.

11:23 Stone Marballie: Or it is a question of previous one. So, is there any functionality right now? For them to see the failed check-in than the GUI? I'm wondering if the reason why they want, the export is just a way to see it.

11:37 Aksana Rahouski: Yeah. No. They the other is this table where they can see out every individual one.

11:42 Stone Marballie: Okay.

11:44 Aksana Rahouski: And right now, I don't know what it pulls, but Well, Beta doesn't have any chicken stew, but it seems like pages. I don't know

11:50 Stone Marballie: Oh, I see.

11:52 Aksana Rahouski: if it's everything. But yeah. So yeah, they do see it. So exporting to figure out a little bit more about like what what it is that they need. Okay, let's do this. Here is the ticket that T-Mobile device, usage issue. So that's the one that you created today.

12:16 Richard Sacco: Yeah.

12:16 Aksana Rahouski: so now on and this bog, about Verizon status, which they emailed it to us last week, saying that they were trying to deactivate The Verizon SIM card on Portal, and it didn't deactivate actual SIM.

12:36 Richard Sacco: Yeah yeah. So why don't we go over the data one first? Because that one's fresh in my mind. But basically what is going on here and then No I know you'll be

12:41 Aksana Rahouski: Care.

12:46 Richard Sacco: they've grabs data today, it's gonna be using 11 10 Right. Which is the start of apw's billing cycle. So the thing is, what is returning then is from the 10th to the 19th every time you query it right now.

14:12 Aksana Rahouski: I have so many questions, but I'm

14:13 Richard Sacco: It should just put in the current day. Because if it puts in the current day is always going to get the most current. Thing, which is what we want. Anyway, we don't care about their billing cycle for the most part. So when it becomes the 19th, what we grabbing it from the 19th on, when it becomes, when it's the 18th, won't be grabbing it up to the 18th. That makes sense.

15:21 Aksana Rahouski: A person.

15:24 Richard Sacco: Yeah, basically we're not grabbing the most recent data, every time we grab right now, and that's the bug. So we want the most recent data up to today from now, because if we do that and we grab it every day, then the differences will be correct.

15:42 Aksana Rahouski: So is the problem like let's just first before solution.

15:45 Richard Sacco: Yeah.

15:47 Aksana Rahouski: I feel like I'm even look a little bit confused on the problem for mobile today.

15:51 Richard Sacco: Mm-hmm.

15:54 Aksana Rahouski: Once a day, right? We hit a T-Mobile API for a device and we get the usage right?

15:59 Richard Sacco: Yes.

16:00 Aksana Rahouski: What we do is that let's say they started with a hundred tomorrow, we run it, a hundred twenty twenty is through today, that's how we make usage per day, right? Is that or

16:12 Richard Sacco: And exactly, yes.

16:14 Aksana Rahouski: The day after we run it again, 130 10. So we subtract from Or I guess do or like the the up-to-date to somehow we calculate the daily rate.

16:24 Richard Sacco: Yeah. Yes, it's not necessarily the daily. It's the total you could sort of do dailies if you would do today's from yesterday and then that would be your daily. But basically over the, what really, what we're worried about is the very first one. The very last one and and there is also sort of like this tricky thing, where is

16:47 Aksana Rahouski: Do we store though? Like daily? Like do we score on like current usage like the

16:52 Richard Sacco: zero is out at one point, right? Because you're gonna get up to the 18 Is going to go to zero and then it's going to continue from there. So we know that there's that bump in the middle there. We store the current and then we calculate the difference from that. So our we

17:09 Aksana Rahouski: max or do we store the difference?

17:17 Richard Sacco: have an algorithm that goes through reads everything and then, calculates, the difference.

17:21 Aksana Rahouski: So our API basically hits Day One hundred to 120 Day 3 1 30 day 4 1 30 day Five

17:25 Richard Sacco: Okay.

17:29 Aksana Rahouski: one forty right? So just grab you just in the saves that and our our logic then calculates daily based and kind of the jumps that it makes like every day that we store, right?

17:42 Richard Sacco: Exactly. Yes.

17:43 Aksana Rahouski: where the problem is?

17:51 Richard Sacco: Well, the thing is, we're grabbing the wrong data because the billing cycle that you pass into gravity, on postman, right on postman, you have to give it a billing cycle date and if you don't, you don't get the data. So you have to put

18:04 Aksana Rahouski: Yeah. Yeah.

18:07 Richard Sacco: something in. So the thing that makes the most sense is just to put the current days date in.

18:14 Aksana Rahouski: Come.

18:14 Richard Sacco: And then, that way, you will always be grabbing the most current billing cycle data.

18:20 Aksana Rahouski: Okay? So you're saying that we calling that API wrong instead of we're passing

18:23 Stone Marballie: Nine is what?

18:25 Aksana Rahouski: the billing cycle and instead we just need to pass today's date.

18:30 Richard Sacco: we're calling the old. Stuff. And so, it's going to be Consistently getting the same value over and over, so there's no difference so all the t-mobile's have a value of zero right now.

18:59 Aksana Rahouski: Okay.

19:00 Richard Sacco: Is what's happening?

19:01 Aksana Rahouski: So we're store. So, what's happening? Because we're every day grabbing, it's the same like 141 41, 41 40s like,

19:08 Richard Sacco: Mm-hmm.

19:10 Aksana Rahouski: There are no usage at all happening daily but it's it, we're getting the same value because we're calling API wrong.

19:17 Richard Sacco: Exactly. So if we were doing to do the fix, the most thing that would make the most sense is we put a value of zero just to start because I already know, we already know it's going to be from then. What is it? November 19th up till now? So it would make sense to put a value of zero to start and then whatever it is

19:35 Aksana Rahouski: Is this?

19:39 Richard Sacco: forward in the future, everything should be good.

19:59 Aksana Rahouski: So, this is a sample of a Noah. I don't know if I shared with you. I will, if not the start team postman for like, APIs calls that we have right now. She's okay. So carrier APIs, like people Verizon AT&T, and I created folder here for our own API. So, just like all these endpoint, the four endpoints that we released, but like this is what the T-Mobile usage one. So, Richard are you saying instead of today, your pat. You're kind of calculating the The billing cycle and you pass the beginning of the end of it.

20:35 Richard Sacco: We've passed the beginning. So we're constantly passing you now. I believe 11:10. So you could test it right now with this device, I don't know if this device has data but

20:44 Aksana Rahouski: And instead, you want to pass now, right? You want to

20:48 Richard Sacco: then it'll really it's getting from even if we pass 12 days, it's gonna be grabbing from the 19th to today because all that matters of the day, is it which billing cycle you're in? It doesn't really matter the anything else. Yeah.

21:06 Aksana Rahouski: So that part, so I think is a two-fold issue what I'm here. And first of all, we're not querying their API properly. But second of all because like you said, our billing like cycles, don't align T-Mobile it like, T-Mobile is 19 through 19 and we're 10 through 10, right? So we have like land in the middle, right? Are you saying we need to like zero out something between 11 and 19th? Or that's the part that I was not clear on

21:33 Richard Sacco: grabbing from What is it? The 11 9, 10, 19, to 11, 19, That's what's being stored in there. Now, that's what's been stored. So, we get to clear that out. That's junk data. That doesn't make any sense. We're gonna put a zero in there and you probably want to do this like via a cake command. You put a zero in there that we only

22:12 Aksana Rahouski: But we're losing daily, right? Because we released a few weeks ago and for this

22:14 Richard Sacco: run one time. you put a zero in there and then you go ahead and grab the current data which is going to be Yeah, which is going to be correctly from the 19th up till today.

22:32 Aksana Rahouski: two weeks, we've been basically calculating usage, wrong Right? Is there a way to patch that if we call it API backwards for every day for the past weeks, we'll allow us to build proper growth or no.

22:47 Richard Sacco: use. I think that might be Like, we could do it. Yeah. It I think it's just a bit excessive but it doesn't. It we can make it work.

23:05 Aksana Rahouski: And I think for now like here's our outside. for now, it doesn't matter because right now we don't give them ever

23:12 Richard Sacco: Okay.

23:14 Aksana Rahouski: Daily view of their usage. We just show three billing cycles so three past

23:20 Richard Sacco: months, right? So we just kind of Started a zero two weeks ago and today's gonna be just like the massive jump with knowing in between, but between doesn't matter right now because we don't report, like, in the last two weeks. These was like your daily growth, right? Yeah. And the data cleanup. Honestly might be something that can happen a little

23:40 Aksana Rahouski: back and do like daily. So okay, so two things bag needs to be fixed data needs to be cleaned up, right? So this ticket really two things so we need to fix the bog and we need to clean up our data. Because right now we're a falsely, reporting usage.

24:10 Richard Sacco: more manually it's because I believe there's only about 20 devices in use right now so it's not anyway but yeah. Basically both we need to happen.

24:23 Aksana Rahouski: Noah, let's say, you would have to take this ticket, there's no way you would be able to do anything. Starting from even, I'll probably understanding what's happening, just by what's in the ticket.

25:02 Noah Bratzel: No, not from what's in the ticket? The I mean, I think I have the basics of the explanation. The Richard's describing, but I'd still have to review how the current code is working, and it remind myself of the data structure is. And it's, I think when I worked on this, we didn't have T-Mobile. So, I think a long time ago, I remember Working at building cycles and things and the basics of what he's talking about.

25:24 Aksana Rahouski: Yeah. Yeah.

25:28 Noah Bratzel: But

25:29 Richard Sacco: yeah, you did AT&T, I remember

25:30 Aksana Rahouski: All right.

25:32 Noah Bratzel: Um, yeah, did I?

25:33 Richard Sacco: Yeah.

25:33 Aksana Rahouski: Yeah, and we because we recording this, right?

25:34 Noah Bratzel: Yeah.

25:39 Aksana Rahouski: Are we recording this? okay, I you see me often in recording meetings that like

25:42 Noah Bratzel: I,

25:42 Laura Perry: Yes.

25:48 Aksana Rahouski: Repeating things out loud because then we can use transcript to summarize this ticket and just copy paste here so we could do this way but let's let's look. Can we size it? At least like obviously still some probably diggin is to happen but in in a question to Is it should you should it be one ticket as a fixed the

26:08 Noah Bratzel: Think it should be.

26:10 Aksana Rahouski: dog? Clean up the data or Do we want two tickets? How do we want to handle it?

26:21 Noah Bratzel: One. But Mostly because it seems related and you shouldn't really do one without doing the other probably.

26:30 Aksana Rahouski: All right, sounds good. So how do we feel about sizing it?

26:36 Richard Sacco: yeah, the one hour by the way, was because I already put time basically or what's researching it So that's that, what that one hour is in there.

26:47 Aksana Rahouski: Okay.

26:49 Richard Sacco: But I put three hours, I know exactly where the code is for the most part. I know how it works. So, I would, I mean, but

26:59 Aksana Rahouski: But it okay. Like right now we're just kind of building this muscle, right? How

27:03 Richard Sacco: Yeah.

27:05 Aksana Rahouski: to work as a three devs team, right?

27:08 Richard Sacco: Yeah.

27:09 Aksana Rahouski: Because look again like if Richard is on the it can only consultant and not do the work. It's it's not Richard's. Three hours. That matters. It's matters. How do we give that requirements to to any Dev on this group? And they feel like they're set up for success?

27:32 Richard Sacco: That makes sense. I mean, I'm happy to connect to another Dev of their Gonna take this one and I think it'll make it easier for them, I think, but

27:42 Aksana Rahouski: Yeah, and sometimes like that for you to, like, sit down, probably still needed for this ticket.

27:48 Richard Sacco: Yeah.

27:49 Aksana Rahouski: and then,

27:53 Richard Sacco: We should probably try to also get it in before next billing cycle.

28:01 Aksana Rahouski: Yeah.

28:01 Richard Sacco: Which is.

28:02 Aksana Rahouski: We need to clean ASAP.

28:04 Richard Sacco: Yeah.

28:05 Aksana Rahouski: okay, but like Do we want? Like No, you said eight.

28:11 Noah Bratzel: I mean that's just by rough estimate. That's I usually give about double what I think it would take when I don't have a clue about and I know I need to look into the code so that's kind of I would, I would kind of agree with my feeling would be what's probably what Richard said, but that would be if I was experienced in and know the code and had had it a good understanding.

28:34 Aksana Rahouski: So Stone, what do you think?

28:38 Noah Bratzel: You threw a nine up there? Eight and a half. What was it?

28:43 Stone Marballie: Nine is what?

28:44 Noah Bratzel: Not.

28:48 Aksana Rahouski: Okay, it's an expensive bug, you guys.

28:53 Noah Bratzel: I mean, I want to guess I would ask about the estimates because

28:53 Aksana Rahouski: Not right now. No client. That's like I told you for this client. They're

29:00 Noah Bratzel: Are we going for? Are we just going for accuracy? Are we going for padding it? That the essence gonna ever go to the client and you know,

29:13 Aksana Rahouski: actually quite forgiving and makes our life a lot easier because we don't ever like size things before to get an approval to move in. Like they just kind of trust us. Go fix the bug, go do the work, right?

29:22 Richard Sacco: And so again a very technical description but I can make it easier. When you change anything on a Verizon SIM status wise, so you change it, active to suspended, active to the activated. It goes through appending process. And in this pending process, um basically what happens is our system, All point command is listening for a callback from Verizon And I believe, Noah you've also worked on this one. Back in the day, I don't know if that's triggering any memories for you.

29:26 Aksana Rahouski: However, right other projects do need it. So like I'm hoping that we can kind of

29:26 Noah Bratzel: ah,

29:31 Aksana Rahouski: like work on this muscle, right? And how do we size things inside? It's always gonna be like, the more I know the last accurate that's gonna be and we can kind of watch it as we go, right? But I'm for now, I'm just curious to kind of, if we start putting some numbers and like, how accurate eventually can we become?

29:52 Noah Bratzel: Okay.

29:52 Aksana Rahouski: But it was a lot of it is going to be based on what information do we need in order to get to that like cleaner estimate, right? Okay.

30:04 Noah Bratzel: Yeah.

30:07 Aksana Rahouski: um, Okay, well, let's just put eight for now. I guess. and then, If it's loss in loss but like at the end of the day somebody needs to take and fix it. Okay. Um, Now, that's other one.

30:32 Richard Sacco: this pending process, um basically what happens is our system, All point command is listening for a callback from Verizon And I believe, Noah you've also worked on this one. Back in the day, I don't know if that's triggering any memories for you.

31:07 Noah Bratzel: Yep.

31:08 Richard Sacco: sparse because the obviously they're not just reporting a lot. My suggestion on how we tackle this as we pull more logging in to see what exactly is going into the API to grab the status and also just just more logging in general. So next time this happens we can just resolve it easily. Um, or we could ask them. Hey, can I mess with that device? And, you know, I don't want to just so we could recreate it but I think it's sort of a niche. Issue. So, this one's gonna tricky.

32:31 Noah Bratzel: Could you could you describe it one more time? I'm having trouble, understanding

32:34 Richard Sacco: Yeah. Yeah.

32:35 Noah Bratzel: what you're describing.

32:36 Richard Sacco: was when it went into pending deactivation, maybe it's because the device is already deactivated. That we got an error right there or I don't, I don't know the exact order of events. To be honest, I would have to do more digging but basically we get we got an error. But the the thing that's an obvious bug, the thing that's like, is that it came back as active. In our system when it was clearly deactivated. So remember at one point after we try after we get a fail, we're like, Okay, what is your true status? We're gonna set you to your true status. So the true status came back as active. When in fact, the true status was deactivated and at the time that I locked, this book, I did double check on its true status, and it did come back as the activated. Well, in the system, you can see in the logs, it's logging it. It's true state was found to be active.

33:48 Aksana Rahouski: When you say true states, do you mean the device or the SIM?

33:53 Richard Sacco: The sin. This is all about the Verizon sin.

33:56 Noah Bratzel: And that's a endpoint that we're hitting to get that.

33:59 Aksana Rahouski: Yeah. So think of

33:59 Richard Sacco: So so what happens is a Verizon is sending the data to us at our endpoint and we're sucking it in and also because of that that's only able to be done in one environment and it's only on production. So you could only see this on production, we can't recreate it on beta or review.

34:20 Noah Bratzel: Okay, in that callback is triggered by when you ask what you do with the deactivation or something, right? And then

34:26 Richard Sacco: Yes, once we send it to Verizon as soon, as Verizon has the response for us, it sends it to our system, Verizon sensors.

34:33 Noah Bratzel: Yeah. And the system case and then that handles actually setting. So you're not hitting an endpoint to get that true state you're saying the callbacks that

34:42 Richard Sacco: failure. If we get a failure from Verizon then we check for its true state.

34:53 Noah Bratzel: Okay.

34:54 Richard Sacco: And yeah.

34:55 Noah Bratzel: So, the failure happened, you, your theory is the fair happened. We try to set something to pending to deactivated that might have already been deactivated. That, that might have been why it failed anyway. And then you what you're saying? The main bug was is that or the clear bug? Is that when we check the real estate via API, it came back and said it was Active.

35:18 Richard Sacco: Yes, rather than deactivated.

35:19 Noah Bratzel: You how to slow. How did you So, how did you know? It was deactivated

35:23 Richard Sacco: So I have basically on DB, I have something called Test Suite, or you could even check the postman, it's an in postman, you could check the ICC ID of the device for Verizon and you could get its actual status.

35:39 Aksana Rahouski: You can when we have that normal save and that's where I'm looking a little used

35:42 Richard Sacco: Yeah.

35:43 Aksana Rahouski: and they send us back and that's where the status gets set or is it. It doesn't work different from how I just described

36:56 Richard Sacco: A little bit. So basically what's happening again? So for just follow me. Forget

36:57 Aksana Rahouski: which,

37:01 Richard Sacco: about the whole deactivating, the sim from that. Let's say the sim is active, it's a Verizon sim. You said it to deactivated, you hit, save what happens from

37:06 Aksana Rahouski: Okay.

37:12 Richard Sacco: there is. It's a Verizon Sim it goes into pending the activated. Ending Deactivated Now, we wait until Verizon sends us a success. To get rid of the pending. Like Verizon has to tell us, it's seated.

37:32 Aksana Rahouski: Pause for a second.

37:33 Richard Sacco: Yeah.

37:34 Aksana Rahouski: Is it async, or is it like immediate response or How do we when you say we receive it? How do we have some kind of like listener or is it like inbound job with process? How do we process this like pending to not pending?

37:47 Richard Sacco: It's pending now, Then we wait. And then Verizon sends it to us. Whenever Verizon knows it sends us every status change, I think and then we grab it. And

38:03 Noah Bratzel: This.

38:05 Richard Sacco: we know, Oh, it's this device. This device succeeded or this device failed. If we don't get a response in 10 minutes and this is like going into the more

38:14 Aksana Rahouski: Yeah.

38:15 Richard Sacco: Niche Stuff. If we don't get any response in 10 minutes, we just assume that it worked. and we get rid of it, depending

38:19 Aksana Rahouski: In. Oh my God, we assume we assume are not. But how do we 10 minutes? Wait, where I cannot imagine we're sitting on this page and spinning for 10 minutes, right? How does

38:35 Richard Sacco: Yeah, it happens in a cron job. It looks for anything with a certain it can tell

38:41 Noah Bratzel: Yeah, it's a server just it's a server server post. So their system posts to our

38:45 Aksana Rahouski: Yeah. Yeah.

38:47 Noah Bratzel: system.

38:48 Aksana Rahouski: So it's like a WAP hog bay so they send us a message. We have a listener. We That's okay. And that's outside of check-in, right? It's something separate from

38:58 Richard Sacco: And yeah, that is nothing to do with chickens. Yeah.

39:02 Aksana Rahouski: And if a confirmation comes back of either true or false make decision, right? And then we restore device to its proper state. If it can't came back as active, we put it back to active DIA would take that but if nothing comes back we'll assume that it. It was a success, whatever that was active or inactive.

39:22 Richard Sacco: Yeah, they don't like depending to be kept there. Because there was some times, I guess where it just didn't come back. And so, we just removed pending after x amount of time.

39:34 Aksana Rahouski: sounds like what it is or Are you saying You don't know yet? Let's just add some logs to catch that error back. At some point.

39:59 Richard Sacco: Yes, I know exactly where it's going wrong. But yeah, I would have to get more logs to see exactly why it's going wrong. I don't know why it's going wrong. I know where it's going wrong. and I know where it's going wrong because when they reported it, I tested it Because when they reported basically when it's we know it fails then we attempt

40:21 Aksana Rahouski: Is this do you think new or is it an old bug because Verizon has been around? Or

40:21 Richard Sacco: to get as true status and it failed and getting the true status. It's log. The true status incorrectly

40:35 Aksana Rahouski: is it do you think it was introduced with it a new code that we launched when we we did?

40:40 Richard Sacco: Right? It's it's really hard to say. I think it's this one. I noticed that the device has like, it doesn't have an imei. It's just a regular sim. So I think it's maybe only happening to this sort of device. I I can only make theories at this point. I did introduce some new logic in there with the T-Mobile stuff

40:57 Aksana Rahouski: Some.

41:02 Richard Sacco: because Now, you have to deal with if the Verizon's. Status sim statuses X. Then maybe we should make the AT&T match if it's also active and stuff like that. So, Yeah.

41:20 Aksana Rahouski: okay, so I feel like it also another ticket where somebody needs to sit down with Richard, I also feel as well already touching it. Let's properly diagram. How Verizon's same day activation process. Looks like it's it's like async Process. Oh, I think we need to look in then. To me, it's like more of a spike than a bug yet, but I don't know what you guys think.

41:54 Richard Sacco: Might be more of a spike because my suggestion is just do more logging at first anyway.

42:00 Aksana Rahouski: Right.

42:00 Richard Sacco: Because that can't hurt anything and then won't be closer to the answer. But

42:06 Aksana Rahouski: Yeah. So I would guess the next thing would be kind of like for somebody just

42:07 Richard Sacco: We we do honor review. Yeah, but the thing is we can only point like I said that

42:10 Aksana Rahouski: sit down with Richard. Get it out of his head. Properly just to validate if we align on the root cause if we can identify the root cause before it can talk about how do we fix it. If the root cause is a known, then we boost logging and kind of monitor it. Just until like, it gets us closer to it.

42:39 Noah Bratzel: Well, can I ask how would we test out this type of thing?

42:45 Aksana Rahouski: That's part of sitting down with Richard and figure it out.

42:50 Noah Bratzel: To be I mean do we have Verizon devices to do this type of testing on? That's not going to screw up real devices or how to how do we even?

43:02 Richard Sacco: status thing at one environment so that makes it. So we that's really gonna curtail our testing quite a bit.

43:11 Noah Bratzel: yeah, so they don't have Okay.

43:15 Aksana Rahouski: But but doesn't matter. Like does it matter for us though? Like Richard because

43:15 Noah Bratzel: so, we don't

43:20 Aksana Rahouski: yes Carrier has only one environment they don't have three to match ours, right. But if a device is a testing device we'll just using it like link into prod to kind of manipulate all the real lives to scenarios like That's how all carrier integrations work. We don't have sandbox for T-Mobile. We don't have sandbox for AT&T and Verizon where we we actually go to produce devices are send boxy. They're not real devices. Now.

43:53 Noah Bratzel: Yeah, so we do. We have we have test devices but then that are

43:54 Richard Sacco: um, Yeah, we have some test devices here.

43:57 Aksana Rahouski: Cut.

44:01 Noah Bratzel: Okay.

44:01 Aksana Rahouski: And that's really just again we've been kind of working on that too, right? Because often Because of the nature of this business, right? Things like that is really hard if we cannot test, we cannot guarantee, right? So like client is very like, Accommodating in that way. They, they do give us devices if we need to the like shifted, T-Mobile device, they'll give us the device. I'll help us to set it up, to be able to test it. So I would say If we're like, what if we're running into a wall and they, we need their help, we just need to raise our hand and ask for that help. So let's let's just put some number on it and like somebody I'm guessing, we'll just split those two tickets. Noah text, one stone, takes another one. And you guys will have to get on Richard's, calendar to sit down with him and get some brainstorming and

45:05 Richard Sacco: Yeah, I think for this one it's kind of hard to estimate because it's like it. Are we estimating just putting the logging or is this going to be just a spike to start? And then maybe we do the logging and stuff. So I think it's just because it's not clear. Exactly for me. What I intended with this one honestly is just to look into Just add more logging for the on, getting its true status. And then from there, we can take a look and see but

45:36 Noah Bratzel: Can you can you describe that again? For me, the getting the truth status. You

45:39 Richard Sacco: Yeah. Yeah.

45:41 Noah Bratzel: said, You're saying that was an endpoint that we're hitting.

45:44 Richard Sacco: So it's just one of the Verizon endpoints. It's just yeah.

45:46 Noah Bratzel: but, But then, when you tested is that different than what you're testing here.

45:53 Richard Sacco: when I first, well, I just basically

45:55 Noah Bratzel: No.

45:58 Richard Sacco: What do you mean?

45:59 Noah Bratzel: I mean. I'm sorry, what I mean is. What? I aksana shows us here and these endpoints. Is that getting it? Is it Getting a true device? Status is that hitting a different endpoint that we're talking about over here that was getting these statuses

46:19 Richard Sacco: so I, I

46:20 Aksana Rahouski: It's right.

46:23 Richard Sacco: Basically what's happening and I don't know if I'm gonna make this clearer more confusing. I'm hoping I'm making it more clear, basically. Okay, we change it from active D, I spending the things you return us. It failed. It failed to change the status. Then from there, we're using this exact same API head that she's using. Well, at least that's what is what I see in the code. We're using This exact thing to see

46:48 Noah Bratzel: Okay.

46:49 Richard Sacco: is actual status, but for some reason the actual status, some production and

46:51 Noah Bratzel: Yes.

46:55 Richard Sacco: that piece of logic, Came back differently than what I actually was seeing by calling it.

47:01 Noah Bratzel: Okay.

47:02 Richard Sacco: Independently of the Production Code.

47:03 Noah Bratzel: Okay, that's that's the part that that you're saying is obvious bug. Is you saying it shows something that then when you tested it, you're seeing something else.

47:11 Richard Sacco: Exactly. Yes.

47:13 Noah Bratzel: And it and it's not just based on when it happened, or do you not know for sure?

47:18 Richard Sacco: and when it happened, um, No, I, I know for sure because what I did is, I manually tried changing it from

47:21 Noah Bratzel: Like, could it?

47:27 Richard Sacco: active deactivated. And I saw at the time that it did match up where I was getting deactive from here and it production was showing active. So I did a live test on it to be able to determine that

47:44 Noah Bratzel: oh,

47:44 Richard Sacco: Yeah.

47:45 Aksana Rahouski: You know. And maybe like, if we have a device, which I mean, this is a, what, what at least, this is what we tested from our saved from the last round of tests. We did for all the Sims, It it pulls something. I don't know if it's a I guess I would assume it's not product device just because we wouldn't have product device here. We just use it for testing. So, I would try to recreate it because if you can recreate a problem, that already means that you could fix it like instead of, like, kind of trying to like, put a band date on something. You don't know where it's bleeding. I'm just gonna try to, like, Yes, right. Try to break it. If you can break it, you can fix it. Perhaps, our endpoint is wrong or maybe our implementation to even to start with is wrong and we need to fix, right? I would say that I would say this ticket is a spike to me. It's like sound, like

48:38 Richard Sacco: Yeah, that's fair. If it's in the case that it's a spike take it out probably

48:41 Aksana Rahouski: we still kind of don't are not quite sure what's causing it.

48:52 Richard Sacco: put like Four on it. I don't know. It's only that.

48:58 Aksana Rahouski: Let's put now just because spikes are a little bit different.

49:03 Richard Sacco: Okay.

49:04 Aksana Rahouski: Time blocks yourself with spikes because like if you feel like you're getting Noah or perhaps we're digging in the wrong direction, we need to change our approach. But if you feel like I'll almost there like just Then keep going.

49:20 Richard Sacco: Got you?

49:22 Aksana Rahouski: But also like again like you guys speak up, right? Because it's like, if it works doesn't work, you you tell me how do you want to approach, but this to me it's like not quite it feels like we're still not quite sure like what's the but with the bug is So I would say, like we shouldn't be talking about fixing bag of food. I don't know what the bug is.

49:49 Noah Bratzel: Yeah, but I guess. Where do we know where do we, where the, how do we know about where the test devices are what or what the test devices are? Because I think feel like the only thing you can do on a ticket like this is, Really try to recreate it. Like you were saying,

50:09 Aksana Rahouski: All our devices, all art has devices are in the office in Frederick. So Richard should be able to Ian's tell us, right Richard? Like which we should have

50:19 Richard Sacco: Yeah, I I, yeah, I believe don't. You also have them and your test documentation to like, which ones on review. You can mess with Australia.

50:26 Aksana Rahouski: Yeah, I think can give you that information or the bottom line if you need a

50:29 Richard Sacco: Yeah.

50:30 Noah Bratzel: Okay.

50:31 Aksana Rahouski: device in Non-prod, that is legit divide, a Verizon device that you need to, like, be able to break. We should we should get that information, we should have it.

50:43 Noah Bratzel: Okay.

50:47 Aksana Rahouski: Okay. um, Okay, so We sized that one before and we're out of time. Okay, let's Just okay. Let's just kind of I feel like for now we have tickets to work on And so that would be this download device, I would say this one, if somebody can

51:12 Richard Sacco: And while we're here, maybe we should also have a quick chat about. I know we're

51:13 Aksana Rahouski: take it right away. The, the config file not being downloadable. This is something client ask For.

51:26 Richard Sacco: out of time, but a quick chat about versioning. So if you think that this should be a hot fix label and hotfix, if you think that it should be in the current version, put it in the current version things of that nature. I did all already have like a talk with the devs so if it's labeled Hotfix, they would know what to do with it. If it's in the current version, they know exactly like what to do. And when to push it to review, none of that stuff is gonna be Up in the air.

51:53 Aksana Rahouski: Okay, so why don't you guys say? Yeah, I don't mind staying a little extra talk. Align on that. Because what what, I want to make sure that Whatever I ask from you, right? Is something that you aligned on internally aligned as a group Because, like, you'll always find me like Pressing more. But I'm only pressing because if you need to push back, I'll see it. You'll push me back. Tell like I said like earlier if I want something to release I'll be like Can you release it ASAP, right? If it's a No it's it's valid answer but like we like I said earlier we need to have an approach and how to kind of not to create these like Cue that blog, everything it blocks each other, right? Did you say anything that we treat as a hot fix? We just want to add a hot fix light bulan and that's gonna have like a separate branch or something that would

52:48 Richard Sacco: Yeah, it has a separate process. It'll bypass like the the whole, like this body of work, needs to be, get put in as one, it'll bypass all that. So once it's done once it's QA, we can just push it separately from everything else. Now, if

53:03 Aksana Rahouski: Okay.

53:03 Richard Sacco: you are, okay, with waiting until our current body of work is ready, and you would just put it in the latest version. And you could make those decisions as you go through the tickets.

53:17 Aksana Rahouski: So, let me ask you this question.

53:19 Richard Sacco: Yeah.

53:20 Aksana Rahouski: Bags make sense, right? Because there are always be those things that are going to come in hot and get out hard right. let's say, so let's say we have two big, Let's say we now like started on data purging, let's say Stone's working on that Noah's working on configs, two, massive epics. And then the middle client is bread. Crumbing us little things that we want to like clean up and poured. Clean up. This clean up that How do we kind of like? Structure that that we allow devs to focus on this like big apex. Right. But also if things ask well unless you're saying that we should treat all bread crumbs at hot fixes

54:07 Richard Sacco: Yeah, I think that's up to us really. We could treat them all like hot fixes or we could just put them or we could just say this will be included in this release. This epic is not going to be included in the next release for a while and that's what we've decided. So it's okay to just release little stuff along the way and then once this epic is ready and we maybe we put it into the next version after we do a launch and then we're ready to go with like launching that one next and won't be focusing on launching that one. Next before we do other stuff that there is other stuff, we can do hot fixes on that.

54:43 Aksana Rahouski: Okay.

54:44 Richard Sacco: Yeah.

54:48 Aksana Rahouski: Anybody else has a strong opinion? About it.

54:55 Noah Bratzel: I mean, what what I said this morning would still apply to this week. If we could get to a place where we're doing feature switches, we wouldn't have to do Hot pics is really, because all the longer term development would be wouldn't matter if it rolled with a bug fix. So you would just nor do your normal process, And I think that, that would in the long term, be a much simpler approach. It just it's gonna add a little bit of complexity up front of making sure that people are understanding of feature switch system and putting things behind that.

55:30 Aksana Rahouski: I like and I know maybe not for this meeting, but I want to like know more about it because I I totally get it. If it's like a new feature, you're adding right. And you can kind of like keep releasing without making it publicly accessible available or whatever. But what's been existing set thing? Let's say Like right. How to in? It's like kind of have big. How do you safely? Keep releasing that like code that is like 40% down without breaking the existing

55:57 Noah Bratzel: Yeah.

56:02 Aksana Rahouski: thing. Like imagine, like what? You just worked on manufacture model, right? Imagine that we have. We could, could we release the scout after you did manufacture crowd, but you didn't finish device. You didn't do device validation yet.

56:23 Noah Bratzel: Well, yes, you could, you have but the whole thing basically, you release the code, but there would be a feature switch. The basically, nobody would see that.

56:34 Aksana Rahouski: so that

56:34 Noah Bratzel: The whole NAV would be hidden, even even accessing, the URL would be would, wouldn't be possible a lot of the underlying code would be possible. And then you would have to, if you're gonna do like a My code if you were gonna do additional changes that hit other sections, you you would have to also have it be smart enough and your changes be structured so that it had um, Conditional. Checks to if use that feature switch logic. So it adds complexity. And then

56:58 Aksana Rahouski: So here.

57:03 Noah Bratzel: after you, after you release it, you kind of remove, you know, this is done. You kind of remove that extra feature logic and stuff that's just there to protect the existing code.

57:16 Aksana Rahouski: You just basically adding feature flags into your code. So you conditionally basically putting needs around things that are new.

57:25 Noah Bratzel: He? Yes and depending how you how you do it that's either very simple like you said for completely new sections, it's a really simple thing to do and but when it

57:33 Aksana Rahouski: Right.

57:34 Noah Bratzel: touches existing stuff that's where it gets trickier and stuff would still be needed to be tested to make sure that that feature switch logs you put in, didn't, you know, is done correctly, you know, it's not it's not perfect but mostly for longer.

57:48 Aksana Rahouski: it's Yeah. I think for longer and for more integrated because like even if we take a simple, let's say you have a button that looks red and needs to be blue, right? You have to put like a feature flag is on or off right, still show it red else showed blue, right. And then at some point we need to like on on remove all these conditions, right? And and yeah,

58:13 Noah Bratzel: Yes, there would be you add basically, your project would add an additional step. That after it goes live, you would have a cleanup set afterwards.

58:21 Aksana Rahouski: Stone, add a question.

58:29 Stone Marballie: Um, I guess I have a different idea of how I think things should work. So I think the strategy and I'm just thinking out loud here, because I think the strategy should be that. Should be that about hot thick strategy in the sense that you always. Any new development item that you start building, it's always off of the tip. Of production or master, right? Then for all these different features, you create a branch that's off the tip of master, and you give it the label or whatever. So, then developers can work on any tickets that they want, and their own branch when they're ready, they merge them. Into that release branch and that. So that way, you can have seven lines of we're going on at any given time and all of them would be safe to go into production at any time because at the starting point was from the head of master because you pour those sticking into those feature branches you can see okay, I'm gonna deploy this feature and you'd be able to just deploy just what you

59:34 Aksana Rahouski: oh,

59:35 Stone Marballie: want at any given time and any. So for example You know, something may came up that, you know, they want in production right away and that ticket was worked on. You could have a release that was based off of, you know, the latest set of production. But those tickets in there and you can ship it. You know that branch to review data pipeline and then that would kind of keep everything isolated. That's the way I kind of you know, see it conceptually not on, that's a better practice but don't let this help you know, in my little

01:00:05 Noah Bratzel: Yeah.

01:00:09 Stone Marballie: pee, bring home envisioning it

01:00:11 Noah Bratzel: The question on that is if you just have basically just doing like hot fish fixed style. How do you keep and manage everything that has to be on review and beta? That's being q8 and tested? Because then you.

01:00:26 Stone Marballie: Right. So, right because that feature brand is what you would check out.

01:00:27 Richard Sacco: Well. Yeah. What why don't we table this discussion for now and discuss this on devs and we can have like regular Dev meetings maybe and we can go through this because I feel like not. I feel like this is not really relevant to everyone,

01:00:44 Stone Marballie: Alright, right. We're

01:00:45 Richard Sacco: but yeah.

01:00:45 Aksana Rahouski: Yeah, no. It's like I do think it's a good idea for you guys. to sing because like again, three cooks in the kitchen, It's gonna it's just harder, right? Make sure that you all girls aligned as far as like breasts, how we move?

01:01:01 Noah Bratzel: Yeah, you have to have to have a strategy that I mean, obviously, you can't have three different strategies you have to have a single strategy, we all have to use that strategy. So

01:01:10 Aksana Rahouski: That's how do we coexist and still move through as a team and not get on each other way, right? Okay. Um okay. So for now, I guess, if anybody's looking for work

01:01:21 Richard Sacco: India.

01:01:26 Aksana Rahouski: Just for the purpose of this. Go pull from to do. um, and then Yeah, I will like, I will organize this tickets like a look at it and

01:01:39 Stone Marballie: Still do is just whatever, is on the sprint board on the left there.

01:01:39 Aksana Rahouski: Yeah, in that order. Yep. In order. So the config download file, cleanup, email

01:01:49 Stone Marballie: Okay, so it's prioritized and most important first.

01:01:53 Aksana Rahouski: Yeah, and then like, this tool bottom ones that we just talked about. Move them to. yeah, and then If we need to keep grooming we have another work session tomorrow. Thursday can always use that 30 minutes for Because I have to fight cards. I mean there's there's work that we need to like start sizing and we have a configs meeting tomorrow. No, we're not pulling on I'm eating yet. We're just we'll get you off to speed but it's soon I promise we'll go bring you in properly. For the client.

01:02:30 Noah Bratzel: Okay. Just one quick question about tickets, statuses in these tags are using. What's

01:02:33 Aksana Rahouski: Huh.

01:02:36 Noah Bratzel: the difference between a ticket being in needs and needs review and estimate status? And the, the tag that you're putting things on is that kind of like the same or different.

01:02:48 Aksana Rahouski: It's no. to be honest, like, I think that's where we all start. Like, we, we should be just using like, Statuses that we already have, right? But I was a little confused when I came in, so I guess smell like five fold, not, like, cleaning up. But just starting yet a new thing. I just kind of created like a space for myself. I'm like, I'm not sure how these status is playing, so I'm just gonna add my own tag, and that's how I keep control of it.

01:03:16 Noah Bratzel: Okay.

01:03:17 Aksana Rahouski: But probably technically anything that has a status need review and estimate. Is it should be that we if we get to the point where it's the same thing, then we can just like assume that is the same thing.

01:03:29 Noah Bratzel: Okay, so basically it should be the same as your grooming tag but you okay just things might not be in. Okay,

01:03:35 Aksana Rahouski: Honestly this this tag, this label I just kind of invented for my just to make my own life easier. So I don't have to like ask a bunch of questions everybody right? I'm like these I have full control over it but they should be yes, status need estimate, should be something that's ready for grooming.

01:03:57 Noah Bratzel: Okay.

01:04:00 Aksana Rahouski: But great question. Keep the mask keep asking number because I guess sometimes even like You always have like a good intentions and then you just like the weight of it just like, eventually you give up right? Whatever. She's gonna start digging in my own hole here. Okay. Anything else we're good. Keep us posted. Keep me posted on what you guys find on these two tickets that You data usage for T-Mobile and a Verizon SIM.

01:04:38 Stone Marballie: Set a question on this data purging one. Sorry I don't mean to hold everybody up. But are we just coming up with the first plan like it's considered like

01:04:47 Aksana Rahouski: Yeah. So what I'm gonna do I created a spike Just Kyle. Let's maybe spend half an hour tomorrow to look through that one. I created. I like a productcorns document because on top of Richard had, I want to look, I told you I want to ask to add two more things, I wanted to be configurable as a like. I want to be able to specify which table how much data to purge like where to trim. And and then so these things need to be like configuration based, not

01:05:16 Stone Marballie: Oh, I don't.

01:05:18 Aksana Rahouski: And I want and I want us to figure out, I mean, I two ways, right? Either we do it manually so I tell you stone go do it. Stone goes, does it every Friday when I tell Stone to do it or we're assume it runs every first of the month? Ideally, we want probably combo like a hybrid, right? That like will agree that we will trim up to 30 like 90 days. The rest is gone. We'll pick a date when to run it, but we need to have like, Like a manual as well. Like if we need to run a purge we can run a purge. Right? So we need like, so again, like kind of like, we still use the concept that Richard already lineup, but I wanted to be more like Framework. Like we, we want to have a process that we can. I can specify a table, I can specify how much data I want to purge. And I want to, I can choose when I want to do it right.

01:06:14 Stone Marballie: Right? So basically you want it modular so you can reuse it, you know, across the board for different tables as they grow and not just so,

01:06:21 Aksana Rahouski: So that's why when I took what Richard created a page I created Product

01:06:22 Stone Marballie: Well.

01:06:26 Aksana Rahouski: Requirements document to kind of specify on top of that script that we have I won this script to I want to blow it up to framework, right? And so so next

01:06:36 Stone Marballie: All right.

01:06:39 Aksana Rahouski: spikes, somebody will take a spy go make a plan. Technical document, how we're going to do it, right? And how, if we need to iterate thread, how we're gonna iterate thread, and that, and we're gonna slice it into our backlog. That's, that's where I'm trying to get ultimately like, because work as a like, tickets come from a developer plan and how to tackle the work That's why I think having proper requirements Doc and how I'm gonna do it leads is directly to the cleaner backlog that is much easier to size.

01:07:14 Stone Marballie: But so basically more you more requirements are going to go in there even though it's a spike and it's a comprehensive solution, I got you.

01:07:23 Aksana Rahouski: Yeah, you can feel free, like I said, this Spike ticket. Already is linked to a product requirements. Document

01:07:30 Stone Marballie: but,

01:07:31 Aksana Rahouski: Which is for the most part, it's defined.

01:07:34 Stone Marballie: Oh, I see. I see it's a link in there. I got it, got it.

01:07:37 Aksana Rahouski: It is there like option, ABC, pick one? And that's where I need you guys to pick, right? Like you need to kind of drive like we recommend going left. Instead of right? I just, like I said, I wanted to be configurable and I wanted to be like automated and on demand

01:07:55 Stone Marballie: All right. I don't know, it sounds like the only fun to hit in there.

01:08:01 Aksana Rahouski: It's fun one. Yeah, a little bit more. Don't worry. Working on it.

01:08:08 Stone Marballie: Aaron, aren't you supposed to be coaching soccer right now?

01:08:13 Aaron Diefes: Season's over. What can I say? You know.

01:08:15 Stone Marballie: Oh, okay.

01:08:17 Aaron Diefes: They're in the winter park.

01:08:20 Aksana Rahouski: So I know we had a live cam meeting today and Ben, who I haven't seen for like a few weeks came and he's like this, like Beard. Like, then what happened to your last two weeks for a rough? Okay.

01:08:34 Aaron Diefes: Man, if you

01:08:35 Aksana Rahouski: Shave.

01:08:36 Aaron Diefes: It's going across the junior business analyst. What can I say? You know, all of us are gonna you'll see Josh next, you know,

01:08:47 Aksana Rahouski: Oh, okay. Okay, well thanks guys. Let me know if you have any questions or anything else.

01:08:55 Aaron Diefes: How?

01:08:55 Aksana Rahouski: I,

01:08:56 Noah Bratzel: See you.