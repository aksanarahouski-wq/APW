Mar 27, 2026
Configuration engine sync - Transcript
00:00:00
 
Aksana Rahouski: Uh-huh.
Devon D'Andrea: Similarly,
Aksana Rahouski: Yeah, go
Devon D'Andrea: there are instances where we've seen this in the past where a devices's config got
Aksana Rahouski: ahead.
Devon D'Andrea: messed up and the um API access to the device was for whatever reason disabled.
Aksana Rahouski: Mhm.
Devon D'Andrea: So just like things like that like there are real reasons why the portal could push something to a device the device not really get it. So at that point it's like okay so what do we do after that to
Aksana Rahouski: Mhm.
Devon D'Andrea: know that that that change might still need to happen even though we already tried to make it happen.
Aksana Rahouski: Yes.
Devon D'Andrea: And so we're trying to figure out like,
Aksana Rahouski: Um,
Devon D'Andrea: okay, so do we make a change? Hold on. I'm trying to see where Adam's at.
Aksana Rahouski: was he going to join
Devon D'Andrea: He was I didn't want to waste any time, though.
Aksana Rahouski: us?
Devon D'Andrea: Um so what we were thinking was okay if we stick with like the host name idea right like let's say okay we need to go in and
 
 
00:01:38
 
Aksana Rahouski: Mhm.
Devon D'Andrea: make a global change to let's say primary DNS when we go in and make that change then that would
Aksana Rahouski: Mhm.
Devon D'Andrea: iterate the host name. So let's say all of the host names get iterated by whatever the date whatever that date is, right? So like our our scheme for uh what we make as host names is typically like carrier model um date that we made that config file.
Aksana Rahouski: Mhm.
Devon D'Andrea: some might be carrier model customer name than then date.
Aksana Rahouski: Sure.
Devon D'Andrea: So we try to stick with that right.
Aksana Rahouski: Mhm.
Devon D'Andrea: So if we go in and make a global change let's say it's today then the portal would say okay new host name to be expected pushed and expected for all for everything. So now VZD, VZW22, whatever gets changed to VZD, VZW22 03 27 2026. Same with AT&T 222,
Aksana Rahouski: Mhm.
Devon D'Andrea: like all of the different versions of what it could be based on the the three-way rule.
Aksana Rahouski: Mhm.
Devon D'Andrea: And then and then that way it's like,
 
 
00:03:03
 
Aksana Rahouski: And
Devon D'Andrea: okay, let's say the the the change tried to get pushed to the device and for whatever reason it just didn't get it to it. Well,
Aksana Rahouski: Mhm.
Devon D'Andrea: next time the device comes online and checks in,
Aksana Rahouski: Mhm.
Devon D'Andrea: it'll have an old date. So then the portal would know, oh, actually it didn't get it the last time we tried to send it. You know, I just think this is a very, very big
Aksana Rahouski: Yeah.
Devon D'Andrea: piece.
Aksana Rahouski: Yeah. And I just want you to know that I I recognize that and that's why I think like kind of
Devon D'Andrea: Yeah.
Aksana Rahouski: intentionally so multiple things in my head. first of all to figure out like what is that like con configuration builder itself looks like right and that's that's the part that we're focusing on right
Devon D'Andrea: Right.
Aksana Rahouski: now like how do I build a configuration uh in a way that I I don't need to create a lot of like redundant files and every time I update something like global I have to update every single file that I
 
 
00:04:00
 
Devon D'Andrea: Yes. Right.
Aksana Rahouski: have right once we solve that right just to figure out okay how global
Devon D'Andrea: Right.
Aksana Rahouski: versus model versus three-way rule versus company versus maybe subver, right? Um then from there we see okay once we have um kind of this like cascading mechanism to build config right ultimately device is what get the final product right devices would get the the full set of config that they should receive
Devon D'Andrea: Okay.
Aksana Rahouski: right so once we figure that out I think then the next part was in my radar
Devon D'Andrea: Mhm.
Aksana Rahouski: like how do how like application of these configs right how do we so we have a certain mechanism the trigger like checkins is one of them, right? You could also like manually push onto the device. Do are these enough? Do we need to rethink that? How does that like once we change the configs or or whatever, right? Like that application mechanism, how devices get these configs, whether it needs to be like on demand or kind of they like
 
 
00:05:04
 
Devon D'Andrea: Yeah.
Aksana Rahouski: checkins trigger these, right? Or maybe there there need to be some other options for it. Um it kind of with related to that,
Devon D'Andrea: Right.
Aksana Rahouski: right? How do we track whether or not these pushes actually are successful, right?
Devon D'Andrea: Right.
Aksana Rahouski: Like as they kind of go out and we can like have some kind of logging that tells us, yep, we did send it to that device got it on that date and that was like success failure, right? some kind of reporting about around application but and all and and I think that's where you brought up which is totally like yes this is something we need to like work on next I just wanted for us to first figure like if we can solve how we build them then we move into how do we apply them um additionally I think what we need to
Devon D'Andrea: Yeah.
Aksana Rahouski: is um do we need any sort of versioning right like for example Because what versioning allows us to do is to have like okay let's say I had a configs right like sample that you said something within global right that needs to change do we need to
 
 
00:06:18
 
Devon D'Andrea: Good point.
Aksana Rahouski: somehow kind of like uh version control with an approval process that you can as an admin kind of take your time to work on an update review it approve it now your latest is v2 that's what all the devices will get what versioning does it allow like right if for whatever reason um Adam's here something did not go well you can go back to the version zero one and push piece
Devon D'Andrea: Right. Right.
Aksana Rahouski: right so these are kind of the the main components in my head at least that need to exist in a system such to make it like safe
Devon D'Andrea: Yeah. Yeah, I I agree. Yeah. So, Adam, we we we kind of put the the cart before the horse a little bit. Um this is kind of the what we were discussing earlier today is kind of like the next stage. um what what we want to be focusing on is you know how it's being built and how it's going to be interacted with in the portal and then as far as applying it and validating
 
 
00:07:31
 
Aksana Rahouski: Mhm.
Devon D'Andrea: it that's kind of like the next stage. So,
Aksana Rahouski: Mhm.
Devon D'Andrea: um it's something that we obviously need to collectively agree on how we're going to validate.
Aksana Rahouski: Yes.
Devon D'Andrea: Um but not necessarily at right
Aksana Rahouski: Which Yeah.
Devon D'Andrea: now.
Aksana Rahouski: So my plan was like really just because when I brought you guys V1, right? Overwhelming and that's kind of like I'm so glad that you kind of cut that and you were like okay
Devon D'Andrea: Yeah.
Aksana Rahouski: this is overkill for what we're trying to do scale back.
Devon D'Andrea: Yeah.
Aksana Rahouski: So I I put together this other pro which by the way I don't know did you have a chance to like clicker?
Devon D'Andrea: Oh yeah.
Aksana Rahouski: I think you did based on
Devon D'Andrea: Oh yeah.
Adam Curcie: Oh,
Devon D'Andrea: We've been Yeah,
Adam Curcie: we started Axana,
Devon D'Andrea: we were we were Yeah.
Adam Curcie: but we're probably like still got about four days of work getting through all that.
Devon D'Andrea: No, I mean I mean to be honest like in my personal opinion like I think that it's probably like way far closer to the finish line than than you know than than I would have ever you know expected it to be.
 
 
00:08:36
 
Devon D'Andrea: Like it's like the interface, the way you interact with it. Like I I I I love all of it. Um, we just really had a couple of questions really with regards to, you know, how we're going to transition, you know, having both legacy and new, and then how we're going to choose subsets, batches of things to put on the put on the new one, and then ultimately how we're going to, you know, basically just protect ourselves from accidentally using the legacy system.
Aksana Rahouski: Yes. Yes.
Adam Curcie: Yeah.
Devon D'Andrea: Um and then we also we also started to um we also started to kind of look at there was one section in your documentation regarding uh company override sets um and I
Adam Curcie: Yeah, I put a comment on it, but if you didn't notice,
Devon D'Andrea: think yeah just you that you had mentioned that you can't
Aksana Rahouski: Yeah.
Adam Curcie: Dev,
Devon D'Andrea: have uh multiple company override sets for the same three-way rule. Um,
Aksana Rahouski: Mhm.
Adam Curcie: no multiple company override sets per
Devon D'Andrea: so
 
 
00:09:40
 
Adam Curcie: company.
Aksana Rahouski: Do you think there should be
Adam Curcie: Yes.
Devon D'Andrea: that's what I just said on the same three-way
Adam Curcie: As long No,
Aksana Rahouski: multiple
Devon D'Andrea: rule.
Adam Curcie: but it's But yeah, but you didn't say per company.
Devon D'Andrea: Yeah,
Adam Curcie: That's the issue.
Devon D'Andrea: a multiple company override sets on per Yeah. with the same company as long as there's no conflicting parameters.
Adam Curcie: Per company. Yes.
Devon D'Andrea: So like let's say for example we have a customer like Cord who has a firewall rule and we want to use that firewall override set for other companies as well.
Aksana Rahouski: Yes.
Devon D'Andrea: But then Cord might also have another override set that pertains to something completely different within the config and maybe that only applies to them but not all those other companies. So then cord would have a firewall override set as well as maybe an additional override set with different parameters than the firewall override, but it would be potentially on the same three-way
Adam Curcie: Yeah, like there's certain So,
Devon D'Andrea: rule.
 
 
00:10:40
 
Adam Curcie: like if you go right now to the um page you were just on where you can build a override set,
Aksana Rahouski: this
Adam Curcie: just click click create override set,
Aksana Rahouski: one.
Adam Curcie: right? So, um you can put it whatever in for these, right?
Aksana Rahouski: Yeah.
Adam Curcie: Just test and then I'm a model.
Aksana Rahouski: And this is like you guys right did get the idea that I was building it as a like this is basically like the building block that you're building right to your like it's in it is not instead
Devon D'Andrea: Yeah.
Aksana Rahouski: of having one to one like one override to one company we're like we're building that override block and applying as many companies as we
Devon D'Andrea: Yes.
Adam Curcie: Yeah. Yes.
Devon D'Andrea: Yes.
Adam Curcie: Yes.
Aksana Rahouski: want.
Adam Curcie: But so there's and yeah uh you can just Yeah. Any of these, right? And then click continue. So for now, okay, so listen, actually, I apologize.
Devon D'Andrea: It didn't it didn't let her choose.
Adam Curcie: Wait, wait, wait, wait, wait.
 
 
00:11:29
 
Adam Curcie: Hold on. No,
Devon D'Andrea: She already had it that she already had the logic in there.
Adam Curcie: no,
Devon D'Andrea: So, it didn't even let her choose ATM because there's already Oh,
Adam Curcie: no. We didn't we didn't get to it.
Devon D'Andrea: okay.
Adam Curcie: We're You got click back for a second though.
Devon D'Andrea: That's fine.
Adam Curcie: I apologize.
Devon D'Andrea: Yeah.
Adam Curcie: If you Can you go back to to just name I just want you to name it something different. Name this test firewall just to appease me,
Aksana Rahouski: Okay.
Adam Curcie: right? And then write cord actually. Firewall cord. Um and now go through all of these other things.
Aksana Rahouski: Word anything.
Adam Curcie: Doesn't matter. None of this matter.
Aksana Rahouski: Okay.
Adam Curcie: Yeah.
Devon D'Andrea: Yeah.
Adam Curcie: Um and then so
Aksana Rahouski: And this is not because you can
Devon D'Andrea: No, no, no. Yeah. Yeah. I I Yeah, I thought I thought Yeah.
Adam Curcie: yeah,
Aksana Rahouski: this.
 
 
00:12:07
 
Devon D'Andrea: Again,
Adam Curcie: but for the sake of this so for cord right when we build the chord firewall.
Devon D'Andrea: it doesn't matter.
Adam Curcie: Okay. There.
Aksana Rahouski: Should I click?
Adam Curcie: Yeah. Continue. Yep. So somewhere in here you'll have the firewall.
Aksana Rahouski: Okay.
Adam Curcie: Okay. So we'll do enable firewall. True. Right. And then firewall rule content. Right. It's fine. So that's that's going to be the override set for this, right?
Aksana Rahouski: Yeah. Yeah. Yeah. Yeah.
Adam Curcie: I don't I don't know what the I don't know how you would really define it because it looks like you have it defined as an integer and
Aksana Rahouski: Okay.
Adam Curcie: it should really not be, but that's fine. Just change it to 100.
Devon D'Andrea: It doesn't matter.
Adam Curcie: If the default's 50, change it to 100. So there there's we'll use as our placeholder for this. And then you can save this, right?
Aksana Rahouski: Um actually don't know if it actually will Yeah.
 
 
00:12:54
 
Adam Curcie: I know it doesn't, but you Yeah, you get it, right?
Aksana Rahouski: Yeah.
Adam Curcie: So So now we have one,
Aksana Rahouski: So
Adam Curcie: right? So now I'm going to give you another um if you go back to create
Aksana Rahouski: just
Adam Curcie: one, we're going to create a different thing. And this is going to be called um So
Aksana Rahouski: I get where you're going though. Like so you're saying that
Adam Curcie: So here, let me I'm sorry. I I don't always explain things the best way.
Aksana Rahouski: No. Yeah. Keep going. Keep going.
Adam Curcie: There's a customer we work with right now called Altech.
Aksana Rahouski: Mhm.
Adam Curcie: Altech has got specific rules that that are multiple parameters.
Aksana Rahouski: Mhm.
Adam Curcie: So it's not just a firewall rule, it's it's a firewall rule, it's a DHCP scope,
Aksana Rahouski: Right.
Adam Curcie: and it's also a DNA several. So like all of those can be packaged into one because Realistically,
Aksana Rahouski: Yeah.
Adam Curcie: when we build that, there's almost a less than 1% chance any other customer will need it.
 
 
00:13:59
 
Adam Curcie: That's not Altech or his subs. Now, for the chord firewall rule,
Aksana Rahouski: Okay.
Adam Curcie: that's going to be like something that we are going to call chord firewall. But if chord ever needs something else, that would be like a different override. So like we need to be able to combine override sets as long as the parameters defined within that specific override set don't conflict with one another. Like here's a great example. We have a customer right now that's called Baltech. Baltech uses a a config parameter called scheduler in their config.
Aksana Rahouski: Yeah.
Adam Curcie: Every one of their devices they specifically have requested turns off every night at 3:00 a.m. and comes right back on.
Aksana Rahouski: Mhm.
Adam Curcie: So I want to be able to put Bomb Tech Scheduler on Chord's company if they want that without having to then recreate the chord override. Do you understand?
Aksana Rahouski: Yes. And that is
Adam Curcie: So like they should all be able to work like modularly as long as the
Aksana Rahouski: Yeah.
 
 
00:14:57
 
Adam Curcie: like parameters that are defined within each override set are not conflicting. That should just throw an error which basically says hey you have firewall for both of these. You can't have two firewall overrides same time. So figure it
Devon D'Andrea: Yeah. The other thing that we discovered today um was and I think we may
Adam Curcie: out.
Devon D'Andrea: have Adam may have commented on it was um when we're making global changes there
Aksana Rahouski: Mommy.
Devon D'Andrea: are I mean even company changes perhaps um there is 100% going to be situations where the changes we're making apply to you know all models all configurable
Adam Curcie: Oh yeah.
Devon D'Andrea: models and and all carriers.
Adam Curcie: Yeah.
Devon D'Andrea: So instead of us going in and having to make a global change that
Aksana Rahouski: Hello.
Devon D'Andrea: um only per What were we doing that for? Was that for a company override? I think it was.
Adam Curcie: Yeah, it was a company override like the way you just hit create override.
Devon D'Andrea: Yeah.
Adam Curcie: If you go back one more time.
 
 
00:15:56
 
Aksana Rahouski: Mhm.
Devon D'Andrea: Yeah. the screen you were just on.
Adam Curcie: Yeah,
Devon D'Andrea: When if you go back one create override
Aksana Rahouski: This one.
Adam Curcie: create override set. We feel like for model carrier and service plan there should be an option for either the
Devon D'Andrea: that
Adam Curcie: word any or the word all. So that like if it's a like a because the firewall rule that's never going to change based on any of these.
Aksana Rahouski: But that's the that's the three-way rule, right?
Adam Curcie: No,
Aksana Rahouski: That you're saying.
Adam Curcie: no, no, no. Not the three-way rule for an
Devon D'Andrea: No, it's it's it's agnostic.
Adam Curcie: override.
Aksana Rahouski: Okay.
Adam Curcie: Yeah,
Devon D'Andrea: It's like because what we're what we would have to do is like if we if we
Adam Curcie: for some it will be. Some it won't be, some it will be. So,
Devon D'Andrea: Yeah.
Adam Curcie: we need to be able to differentiate
Devon D'Andrea: Like if we wanted to make a firewall change for let's say cord,
 
 
00:16:43
 
Aksana Rahouski: Yeah.
Devon D'Andrea: right? It why we we what we don't want to have to do is come in and make an override set and go i22 Verizon ATM save.
Adam Curcie: Yeah.
Devon D'Andrea: Go back in I22 AT&T.
Adam Curcie: Well,
Devon D'Andrea: Good
Adam Curcie: and it's not even that we don't like you have to also phrase it,
Devon D'Andrea: night.
Adam Curcie: Devon, because it's not that we don't want to, it's that we actually don't need to have it for that parameter. It it it really is universally applicable to every model service planning carrier.
Aksana Rahouski: for for a specific customer for a specific customer though,
Adam Curcie: So we just need
Aksana Rahouski: right? Is that what we're
Adam Curcie: well no it so we're talking about it in context of a customer due to the relevance of this
Aksana Rahouski: saying?
Devon D'Andrea: Yeah, because
Adam Curcie: parameter but the relevance of this parameter is agnostic of carrier making model I mean carrier um model and service plan. So, and we we just want to be able to define that as like it literally as just an option where like when you create override set, yes, we need to be able to choose is this something specific to a model?
 
 
00:17:52
 
Adam Curcie: If so, pick the model. If not, it applies to all models
Aksana Rahouski: But why is it not global if if it's something that
Adam Curcie: because these are being defined specifically for the customer.
Aksana Rahouski: applies
Devon D'Andrea: It's for the customer,
Adam Curcie: You Yeah.
Devon D'Andrea: but it doesn't matter which model or carrier it
Adam Curcie: But that's but it's not always going to be like that.
Devon D'Andrea: is.
Adam Curcie: There are instances where the things that we have to define for them might be they might. So we just need the option to differentiate it.
Aksana Rahouski: So we're saying that certain parameters um hold on let's go. So we're on the company
Adam Curcie: Yeah.
Aksana Rahouski: override.
Adam Curcie: Specifically for a company override certain parameters may or may not actually have relevance to
Aksana Rahouski: So
Adam Curcie: which model carrier or service plan there are. And like and they all need to work independently because there are some where it literally might be okay so the
Aksana Rahouski: yeah.
Devon D'Andrea: Yeah.
Adam Curcie: model doesn't matter matter the carrier doesn't matter but the service plan does and also same thing with
 
 
00:18:46
 
Devon D'Andrea: Right.
Adam Curcie: model matters carrier service plan doesn't for this it's all going
Aksana Rahouski: So,
Adam Curcie: to
Aksana Rahouski: so we're saying is it the same as if I say like a three-way rule could be a not necessarily like an
Adam Curcie: could be What?
Aksana Rahouski: a so like model need to be specified carrier and a service plan at le one of them need to be specified and all others apply or a combination of two. Could that could that be could it be carry like okay um
Adam Curcie: Well,
Aksana Rahouski: all models but let's okay since it doesn't allow to okay could it be
Adam Curcie: yeah.
Aksana Rahouski: I22 Verizon all service
Adam Curcie: Yeah.
Devon D'Andrea: Axana,
Aksana Rahouski: plans
Devon D'Andrea: real quick, can you are you do you see me trying to come in on my Gmail or does that not let you? I'm trying to switch to my phone.
Aksana Rahouski: hold let me see I'll go to the meeting yes there you You should be in. Okay. So uh so
Devon D'Andrea: Sorry about that.
Aksana Rahouski: we do need only specific model and only specific carrier but all service plans and only specific model and the service plan but all carriers and all carriers like service plan ser carrier and all models.
 
 
00:20:13
 
Adam Curcie: Well, can't you just add an option for all or would that break the logic that you already were kind of
Devon D'Andrea: Well, I mean I I think we we need to think about like is there do we do we need
Aksana Rahouski: Um,
Devon D'Andrea: to define parameters as being allowed to be agnostic, you know what I mean? Like or or does it not really matter? Like I mean there's not really many parameters that like let's say if we said for all models or all carriers, there's not really a whole lot of models that would or parameters that would like break something,
Aksana Rahouski: Yeah,
Adam Curcie: Well, I'm just saying like I can give you a handful of examples right off the top of my head.
Devon D'Andrea: right?
Aksana Rahouski: we
Adam Curcie: I might need a few more seconds to think of others,
Devon D'Andrea: to think
Adam Curcie: but like um you know I22 uh the way
Devon D'Andrea: about.
Adam Curcie: that the the you know power cyclinger works for um one
Devon D'Andrea: Sure.
Adam Curcie: customer if we needed to or um man what if you wanted to do for the ATM plan, right?
 
 
00:21:21
 
Adam Curcie: like a like a heartbeat or you know on the Okay, what about this? Okay, so for Allen Hoover, right, no matter what model or carrier, but for his service plan you had of tier three, you had a 10 gigabyte daily usage limit, right? He's not a good example because he only he doesn't do ATMs. But if you had a customer who was doing, you know, ATM stuff and high usage stuff and they needed the the daily usage limit to be different for just their high data stuff, right? That wouldn't be contingent on the model or the carrier. um for for model specific things, you know, there there's not a ton of stuff that would really be something like a model level override, I guess,
Aksana Rahouski: Let me ask you this question because really what we're back to is our two-way rules
Adam Curcie: but
Aksana Rahouski: and I'm just trying to like see how like what's the best way will there ever be for like let's say customer cord and cord is probably a terrible example but I'll just use because you guys use it a lot um that we want to set let's say daily usage for an ATM unlimited to be one value, but then daily usage for ATM unlimited but for all Verizon carriers and Model I22 to yet have a separate value.
 
 
00:22:55
 
Aksana Rahouski: Do you know what I'm saying? It's like the same the same key value. Does it need to be reset based on a kind of broad rule for everybody who's on this plan for this customer but exception for this carrier on this plan because I think the the problem with like that I see as long as we're saying okay maybe that's not we'll have to figure out how to like then again resolve like as these kind of all kinds of rules which is some of them are three-way rules, but some of them were saying it's not really. It just could be like a one like just give me just the service plan or just the carrier on the service plan or just the model and the carrier, right? and attach it to a customer. And if there are any like u more granular like let's say you set a rule for a model carrier just this customer all models on in this carrier follow this rule and then do you want to like but for a specific service plan reset
 
 
00:24:06
 
Devon D'Andrea: Yeah, this is exactly we're we're definitely back to the to the two-way and one way rules.
Aksana Rahouski: Exactly.
Devon D'Andrea: We came full circle.
Aksana Rahouski: It's like we're back to two-way rules and three-way rules and who wins,
Devon D'Andrea: Yeah.
Aksana Rahouski: right?
Devon D'Andrea: Yeah. Yeah.
Aksana Rahouski: Who
Devon D'Andrea: You know, that's exactly what we're back
Adam Curcie: I'm so confused right now.
Devon D'Andrea: to.
Adam Curcie: So,
Aksana Rahouski: H how about this? Um you guys have like a lot of samples for this, right? I think it's always easier if you could like literally like write up like okay I need to build this rule especially our customer where you see a lot of these kind of resetting values right based on model carrier and the service and and or of these things right model and carrier or carrier and service plan and maybe when we have like an actual like kind of samples to work through we can figure out like how flexible and configurable Does it really need to
Devon D'Andrea: Exactly. I think that I think that with a basis of examples
 
 
00:25:08
 
Aksana Rahouski: be?
Devon D'Andrea: we can extrapolate exactly the flexibility of what it Yeah. I mean
Aksana Rahouski: Yeah. Because otherwise otherwise like what you just describe, it just screams that we need a two-way rule and then we need a three-way rule and then we just build a build a cascading algorithm that knows who wins.
Devon D'Andrea: Yeah.
Adam Curcie: Well, well,
Devon D'Andrea: Yeah. That's why I Yeah,
Adam Curcie: hold on a second. Hold on a second.
Devon D'Andrea: I was kind of go
Adam Curcie: Let pump the brakes. Go back a screen from here.
Aksana Rahouski: Mhm. And I know that this
Devon D'Andrea: back.
Adam Curcie: Yeah. So, we already have all those.
Aksana Rahouski: one.
Adam Curcie: So, like the overrides don't even need it.
Devon D'Andrea: What do you
Adam Curcie: You already have all the three three-way rules.
Aksana Rahouski: Yes. Three-way rules we do.
Devon D'Andrea: mean?
Aksana Rahouski: Yes. But now you're saying we need two-way rules because when you say
Adam Curcie: No, no, no, no.
Aksana Rahouski: I
 
 
00:26:02
 
Adam Curcie: That's not what I'm saying is that the override sets. Am I wrong?
Aksana Rahouski: Mhm.
Adam Curcie: We don't If we type out cord If we type out cord
Devon D'Andrea: But you you're you're saying you're
Adam Curcie: firewall and
Devon D'Andrea: It would have to match an existing It would have to match
Aksana Rahouski: Mhm.
Devon D'Andrea: an existing three-way rule. Like it would have to be a if you're trying to do an override that doesn't have three-way, it would have to be a for parameters that already exist inside of one of the existing three-way rules.
Aksana Rahouski: And and let me let me just like we could definitely I think what you're saying okay let's just kind of reuse this what we call three-way rule except now it's like Xway rule because it could be two it could be three right and but if you have a company and then you say
Adam Curcie: All
Aksana Rahouski: for the same you have a two two overrides created one override take the whole three model carrier service plan and another override take just the service plan just like ATM for customer you have this two overrides right one is three-way rule and one is kind of global per service plan which one wins
 
 
00:27:24
 
Adam Curcie: right, hold on a second.
Aksana Rahouski: well
Adam Curcie: Let me share my screen. Is that okay? because maybe we can get to the bottom of this really quickly. I no promises,
Devon D'Andrea: Oh, and what here's here's what I I I understand what's going on here.
Adam Curcie: but
Devon D'Andrea: And what it is is that it's it's not necessarily that we need to define two-way rules and and and one-way rules, right? Because you're what you're going to do what the system is going to do. Let's say let's say we don't define let's say we say all models right and all carriers but ATM plan only okay on the company override let's say
Aksana Rahouski: Yes.
Devon D'Andrea: we had a drop down item they said all all models or all carriers well then the
Aksana Rahouski: Yep.
Devon D'Andrea: system you know should then be looking for okay all models means okay I need to look if if if the device happens to be an I22 on Verizon. Well, then you go to that three-way rule and you have to figure out which parameter in that three-way rule is needs to be
 
 
00:28:34
 
Aksana Rahouski: Yes. Yeah.
Devon D'Andrea: override overridden and then and then again you go to the I4100 and the
Aksana Rahouski: Yeah. and and
Devon D'Andrea: same thing. So like I I just don't know like maybe maybe you're right.
Aksana Rahouski: yes
Devon D'Andrea: Maybe maybe it does need to be defined. I don't know. But I in my mind it's more like
Aksana Rahouski: always think about it from at the end of the day right as these kind of blocks get built and layers right the vase is the one who decide like it gets the final right and from what you just
Devon D'Andrea: Yeah.
Aksana Rahouski: described let's say you have company cord and that company had two overrides.
Devon D'Andrea: Yeah.
Aksana Rahouski: One override is for ATM in all models and all carriers and another override
Devon D'Andrea: Mhm. Yeah.
Aksana Rahouski: and especially now we're saying a company could have many overrides right and let's say there is another one that's
Devon D'Andrea: Mhm.
Aksana Rahouski: specified for I22 on the ATM plan right so that when
Devon D'Andrea: Mhm.
 
 
00:29:26
 
Aksana Rahouski: device when you look at the device device to device will be different but if you're looking on the i22 device I'm guessing that second override wins because it's more granular but it did say that if you are
Devon D'Andrea: Yes.
Aksana Rahouski: I22 In your ATM, you get FU, right? Any other device who is not I22 will get that default.
Devon D'Andrea: Yeah, I hear what you're saying, but we wouldn't make like there wouldn't be like we wouldn't make multiple and that goes back to what we first talked about. We would not want to or would not want to be able to make a second or third or whatever company override for the same company that is using the same parameters is changing the same
Aksana Rahouski: Yeah.
Devon D'Andrea: parameters as the one that already exists.
Aksana Rahouski: Yeah. Yeah.
Adam Curcie: All right, I'm going to get called the three
Aksana Rahouski: No.
Adam Curcie: most different um let's
Devon D'Andrea: So,
Adam Curcie: see, add files, attach files. I'm gonna give Claude three of the most different configs that we have. So,
 
 
00:30:41
 
Aksana Rahouski: Mhm.
Adam Curcie: we have an Altech and we are going to give
Aksana Rahouski: Mhm.
Adam Curcie: Claude a Curvin.
Devon D'Andrea: Sure.
Adam Curcie: And we're going to give Claude bomb tech. And that that should be enough. And I'm going to tell Claude to pull or determine the key differences
Aksana Rahouski: 22.
Adam Curcie: between these three router configuration
Devon D'Andrea: Did I say Did I say we needed you for 15 to 20 minutes,
Adam Curcie: files because I meant the rest of
Devon D'Andrea: Axana?
Adam Curcie: the day and like when you go to sleep,
Aksana Rahouski: But I know it wasn't 15
Adam Curcie: not when you punch out Like the next six to eight
Devon D'Andrea: Yeah.
Aksana Rahouski: minutes
Adam Curcie: hours is where we're where we're headed at this
Devon D'Andrea: Yeah. Once Axana,
Adam Curcie: rate.
Devon D'Andrea: we have been putting this off because we knew once we got started with this, we would just we're not gonna we're not going to stop.
Aksana Rahouski: a can of worms. They keep It's a It's a gift that keeps
 
 
00:31:49
 
Devon D'Andrea: Yeah. Yes. Oh my god.
Adam Curcie: Yeah, you could certainly phrase it that
Devon D'Andrea: Yes.
Aksana Rahouski: giving.
Devon D'Andrea: Um Oh, by the way, Inhand was um very uh pleased with the meeting. They're happy about doing recurring meetings. um they had a lot of great things to say just about Oricsus and and yourself and um you know just everything that you guys um do from the software side. So just wanted to let you know
Adam Curcie: Uh
Aksana Rahouski: And I do think that literally let us know if you can work with uh Laura to book
Devon D'Andrea: that.
Aksana Rahouski: it.
Adam Curcie: well,
Aksana Rahouski: I think it would really help to like meet maybe like yeah I don't think we need more often than once a
Devon D'Andrea: Yeah.
Aksana Rahouski: quarter just to kind of what it helps for us is just to see what's on your radar so we
Devon D'Andrea: Exactly.
Aksana Rahouski: can
Devon D'Andrea: Yeah. Just last night, Ken was playing around with Gemini and sent me this like promo video of a brand new battery uh uh battery product that they have that we didn't even know about.
 
 
00:32:51
 
Devon D'Andrea: And you know, so yeah, things are just always going to pop up. It's always good to stay stay in the loop. All right, what does Claude say,
Aksana Rahouski: Yeah.
Devon D'Andrea: Adam?
Adam Curcie: Um, I don't like this. This is absent entirely from Altech and Kervin Martin. Did I not give them
Aksana Rahouski: And you did Adam.
Adam Curcie: 22s?
Aksana Rahouski: So you used three well company overrides right
Devon D'Andrea: custom config.
Adam Curcie: Well, these are No.
Aksana Rahouski: for Yeah.
Adam Curcie: So,
Devon D'Andrea: Yes.
Adam Curcie: these are complete config files.
Aksana Rahouski: But the but but like based on the naming of these I'm assuming that they're for the
Devon D'Andrea: Yeah. Correct.
Adam Curcie: Yeah,
Aksana Rahouski: model.
Adam Curcie: these are these are custom company configurations and these are like the most like
Aksana Rahouski: Okay.
Adam Curcie: so here I'm gonna I don't want to here let me I want to show you the
Aksana Rahouski: Mhm.
Adam Curcie: this I want to show you our file explorer that houses all of our custom.
 
 
00:33:41
 
Adam Curcie: So here's all point command. These are all the different configs for all the different models.
Aksana Rahouski: Mhm.
Adam Curcie: And then we have this folder that I named custom because this is where all the custom ones live. The vast majority are ATM guys. The vast majority of them are in here and they're really the only thing that's different amongst all of them is the firewall. That's it. They all have got a firewall and almost the entire firewall is identical except they all have
Aksana Rahouski: Okay.
Adam Curcie: got one entry for their specific server that they host and that's different across those customers. That's it. Everything else, excuse me, is literally nearly identical.
Aksana Rahouski: Mhm.
Adam Curcie: So for the for the all the ATM guys except for Baltech, Balm Tech's got some other customizations which is why I chose them. Now, outside of Baltech,
Devon D'Andrea: Huh?
Adam Curcie: we have nothing besides Altech,
Aksana Rahouski: Mhm.
Adam Curcie: who, as you'll read, has got some really interesting stuff. He's got this Altech has got other stuff in here.
 
 
00:34:48
 
Adam Curcie: Um, but, um, yeah. Um, this is all irrelevant though because it's all disabled. All of these VPNs are disabled except for well this cur this particular K Martin is enabled.
Devon D'Andrea: You're you're you're I don't think you're I don't think you're sharing Claude
Adam Curcie: That doesn't Oh.
Devon D'Andrea: anymore.
Adam Curcie: Oh. Oh yeah.
Aksana Rahouski: Yeah,
Adam Curcie: Sorry.
Aksana Rahouski: we're just seeing the are you are these like so you guys are
Adam Curcie: Thanks.
Aksana Rahouski: keeping I mean obviously all the files that are in like live in the file shareer that are actually system uses you have it like a copy of that is that what it
Devon D'Andrea: Yeah. And past versions of,
Adam Curcie: Oh yeah.
Devon D'Andrea: you know, legacy versions and stuff.
Adam Curcie: We we've got we've got way too many DAT files on our share drive.
Devon D'Andrea: Yeah. Yeah. It's It's a little dangerous,
Adam Curcie: Um so here ignore this.
Devon D'Andrea: but like Is the point you're trying is the point that you're
Adam Curcie: Sorry.
 
 
00:35:39
 
Adam Curcie: Ignore that.
Devon D'Andrea: trying to make that like cuz I you want to make
Adam Curcie: The point I'm trying to make, okay, I guess I'll just try to be very direct. I'm not good at making this point.
Devon D'Andrea: sure
Adam Curcie: None of these configurations, none of the differences and none of the customizations are related to the service plan, the model, or the carrier really that I can identify.
Devon D'Andrea: plan is
Aksana Rahouski: Okay.
Adam Curcie: Well, they're not they're not though.
Aksana Rahouski: Well,
Adam Curcie: They're not they're not dependent on it.
Devon D'Andrea: Oh, those ones. Those ones.
Adam Curcie: None of them.
Aksana Rahouski: well, are
Adam Curcie: Well, except for Yeah.
Devon D'Andrea: But Kervin Martin Kervin Martin has boxes on the ATM plan and he gets the
Aksana Rahouski: we
Adam Curcie: Yeah.
Devon D'Andrea: firewall.
Adam Curcie: Yeah. I I guess I guess. Yeah. Service
Devon D'Andrea: I I think I honestly I think like I I don't know. I know it was good that we kind of like simplified everything,
Adam Curcie: plan.
 
 
00:36:31
 
Devon D'Andrea: but like I think that it wouldn't have taken long to just figure out the two-way rules and the one-way rules. It would have been pretty easy and then this would all be kind of irrelevant. But that was just I don't know. That was my opinion. But maybe maybe not. I I because like no matter how you skin it, the system is going to be like it all it's the the thing about the two-way rules and the one-way rules is like you we can't look at it as like this super complicated thing. All it is is just organizing parameters. That's all we're talking about here,
Aksana Rahouski: Well,
Devon D'Andrea: Adams.
Aksana Rahouski: the the tricky part happens with the resolution tree.
Devon D'Andrea: Yeah.
Aksana Rahouski: That's when when you need to know kind of how it cascades and who wins over who, right?
Devon D'Andrea: Yeah.
Aksana Rahouski: But but let me ask you this question. So like for from a customer perspective, right? If we're saying that there are rules that are need to be set per customer, right?
 
 
00:37:27
 
Aksana Rahouski: despite what carrier model service plan they are on. Is that true? because that could be solved just like again if a parameter need to be exposed on a because same way like when you saw the prototype that I did right on the model level you're saying okay this model is configurable additionally you're saying these 20 parameters are going to be what for that model I can configure right and you could set defaults and defaults per model means that every possible device out there of that model this is what they're going to get it maybe
Devon D'Andrea: That's a one that's a one-way rule,
Aksana Rahouski: cost Yes, exactly. And perhaps customer needs the same,
Devon D'Andrea: right?
Aksana Rahouski: right? Because really for us to say that why do we need to bundle a a rule that applies to all models and all service plan and all all all all um carriers. If if really all it is is we're saying customer this is what you get no matter how you what you look like.
Devon D'Andrea: Right.
Aksana Rahouski: Let's do that.
 
 
00:38:32
 
Devon D'Andrea: Right.
Aksana Rahouski: Adam, can you um can you share send me these three files because I think that's where I need your help you guys to like point me to point me please to these like
Adam Curcie: I see.
Aksana Rahouski: configurations that you feel are like have that I need to like analyze and really kind of look at like well how different are these files actually are and then maybe once we have our list of what the differences are we can then literally talk about like well where where is it that and is it based on because it it's a little bit harder to say like with these samples that you said right that customer has this rule set for every I22 of a Verizon but I but how do I know that there's no another config that is I22 or
Devon D'Andrea: right?
Adam Curcie: See,
Aksana Rahouski: AT&T.
Adam Curcie: I guess my way that I think about it is like when you are looking at this, right, the default for model and carrier and service plan should just be any.
Aksana Rahouski: Mhm.
Adam Curcie: So that if you wanted to build an override that that you can use because like you're making an attribute, right?
 
 
00:39:43
 
Aksana Rahouski: Yeah.
Adam Curcie: And you're basically saying what can this attribute be defined to?
Aksana Rahouski: No.
Adam Curcie: This can go to any model, any carrier or any service
Aksana Rahouski: Yes.
Devon D'Andrea: Yeah.
Aksana Rahouski: No.
Devon D'Andrea: And you and and you have to let you have to take that Adam and you have to look at it in terms of like a tree.
Aksana Rahouski: And
Adam Curcie: plan.
Devon D'Andrea: You have to look at it as like okay if you're saying any then uh what was the class that we had in in uh you did you have a statistics class in high school?
Adam Curcie: No, Deon.
Devon D'Andrea: Probability and statistics different combin combinations and
Adam Curcie: I Yeah,
Devon D'Andrea: permutations.
Adam Curcie: I mean I I I played a lot of poker, so I understand statistics,
Devon D'Andrea: That's that's what that's what we're that's what we're doing here.
Adam Curcie: but continue. No, I get
Devon D'Andrea: And it all just falls into,
Aksana Rahouski: Yes.
Adam Curcie: that.
Devon D'Andrea: you know what I mean?
Aksana Rahouski: And the more combinations you want to support, the more complex it becomes.
 
 
00:40:28
 
Aksana Rahouski: So that's why ideally if you want to kind of keep we want to like find a way to keep it low at
Devon D'Andrea: Yes.
Aksana Rahouski: least to start with. And and Adam you're not wrong that here absolutely we could make it less
Devon D'Andrea: Yeah.
Aksana Rahouski: rigid and we say any up to three needs to be provided right and
Devon D'Andrea: Mhm.
Aksana Rahouski: all is an option right so like you don't have to because like for for because for a service plan for example it it is multi- select right it's pick one model pick one carrier pick all service plans so we apply that concept to a model and carrier as well we just need to then figure out because again because there are these
Devon D'Andrea: Right.
Aksana Rahouski: override blocks there could be many of them that company could own right and yes
Devon D'Andrea: What the
Aksana Rahouski: Deon I heard what you said earlier like we need to be like it's it shouldn't be like we're assuming that override never have a redundant uh parameter right but also like we're not over here while you're building the
 
 
00:41:23
 
Devon D'Andrea: Right.
Aksana Rahouski: thing company is like it's not assigned yet assignment happens later
Devon D'Andrea: Right.
Aksana Rahouski: Right. So, it's it's an assignment piece that needs to decide if you're trying to assign something to a customer, a company, and that rule has a firewall and that customer already has another override that also has a firewall.
Devon D'Andrea: Sorry.
Aksana Rahouski: It needs to reject it, right? I I'm worried that the usability of that thing is going to be a nightmare. So, perhaps then maybe we need to like flatten it and make it one to one. Maybe literally, but then but then if you're back to onetoone, if every company needs to have a dedicated override, we lose this option of apply cord configs of a firewall to 500 customers because they are the same. Why do I need to build 500 of these, right? Let's let me think
Devon D'Andrea: Sorry. My my I My son just got in the car.
Aksana Rahouski: about that.
Devon D'Andrea: I had to step out of the park for a second.
 
 
00:42:29
 
Aksana Rahouski: I actually I also you guys need to run.
Devon D'Andrea: Um
Aksana Rahouski: My my son is taking off. He's driving to Chicago and he's home for like a few days. I need to like go send him off. But let's let's do this kind of let me think about this. I I hear kind of where you're coming from. if you could send some samples of like and because I think you guys have some like really good samples of what are different kind of scenarios we want to like meet to
Devon D'Andrea: Yeah.
Aksana Rahouski: solve in in this like I am not worried about
Devon D'Andrea: The problem is is the problem is is it's so as Adam was just showing you it is really really limited.
Aksana Rahouski: this
Devon D'Andrea: There are not a massive amount of differences. Like 90% of the overrides that we have right now are just for customers with RMS servers in the ATM industry.
Aksana Rahouski: Yeah.
Devon D'Andrea: Like I think it'll give us a good basis to like to look at and see
 
 
00:43:24
 
Aksana Rahouski: Yeah.
Devon D'Andrea: if we're you know which direction to go in. But like we shouldn't design the whole thing around the very very limited amount of customization that we have right
Aksana Rahouski: Oh yes. Because that Yeah.
Devon D'Andrea: now.
Aksana Rahouski: Yeah. That's why again because like the the usability of this thing to cut that
Devon D'Andrea: Mhm.
Aksana Rahouski: down right the complexity is important. So if we can like kind of find a a lean solution that works for 80% of
Devon D'Andrea: Yep.
Aksana Rahouski: our configs and then for 20 if there are these edge cases there has to always be another layer of overwrite that is like super strict right that like we build for that these just this edge cases.
Devon D'Andrea: Yeah.
Aksana Rahouski: Um but I think um but I don't think it needs to be like it could be built in into like an initial but that's for back version one which was
Devon D'Andrea: Right.
Aksana Rahouski: super flexible but also like complex to use
Devon D'Andrea: Right. Exactly.
Aksana Rahouski: right and also um Adam you um we talked with Devon a little bit before uh talking once we kind of at least like align on okay directionally this is correct.
 
 
00:44:39
 
Aksana Rahouski: Let's just kind of keep rolling, right? Uh step two is for us to figure out how do we now like what is our mechanism of applying these configs and reporting on which are successes, which are failures. Um is it like on demand or is it reactive? Is it match to what it is today or perhaps it needs to be some some other mechanism that allows you to push but also
Devon D'Andrea: Right.
Aksana Rahouski: see like it did it did it go through or did it not go through? And also like I I brought up to Devon, I think something like versioning is going to be critical here too. Um and I know you guys had a question about like uh legacy versus not. I think because both will exist while both are supported because I can imagine it's going to be kind of we will hand select devices or maybe and we're like okay this customer let's say the entire fleet
Devon D'Andrea: Yep.
Aksana Rahouski: going onto the new we move right having two first of allows us to have that like fall back if something like we can always go back when we feel like okay
 
 
00:45:32
 
Devon D'Andrea: Yes.
Aksana Rahouski: we legacies out like we know what it is we know how to use it we know how it works we'll yank it out of the system And then
Devon D'Andrea: Right.
Adam Curcie: No, just comment it out. Don't Don't get rid of it.
Devon D'Andrea: Don't get rid of
Adam Curcie: Just just com just disable it.
Devon D'Andrea: it.
Aksana Rahouski: I
Adam Curcie: I'm serious. I don't want you to remove it. I just want you to basically like, you know,
Devon D'Andrea: Yeah.
Adam Curcie: comment it out or just disable it.
Devon D'Andrea: I I don't think they You guys probably That's probably part of your best practices to not
Adam Curcie: Yeah, because in in some cat in some catastrophic emergency later,
Devon D'Andrea: permanently trash something. Yeah.
Adam Curcie: if we ever need it,
Aksana Rahouski: yes
Adam Curcie: I don't want to make Richard build it
Devon D'Andrea: I'm sure it'll take a backup.
Aksana Rahouski: and we will have it in our back pocket for a because it it is our safety net to fall back
Adam Curcie: 10.
 
 
00:46:18
 
Aksana Rahouski: into, right? It absolutely needs to be there.
Devon D'Andrea: Yes.
Aksana Rahouski: And when I say I mean like we're like two years in and it's like smooth sailing at that point. It becomes a tech debt to support and maintain, right? It's like it but but again we'll cross that bridge when we get there.
Devon D'Andrea: Yeah.
Aksana Rahouski: Yes. Nobody's gonna I mean because we recognize how sensitive and important configs are and can be. It's a heartbeat of your product, right?
Devon D'Andrea: Yeah.
Aksana Rahouski: We can't just kind of like turn over overnight, finger cross, sink a swim situation. That's not an option here.
Devon D'Andrea: No, no, no, no. Yeah. We just wanted to make sure that like we got to make sure that like it's just not easy
Aksana Rahouski: So,
Devon D'Andrea: for someone to like I don't know go in and push a button that's like sends a file from like a year or anything.
Adam Curcie: Guys, you guys didn't want to tell me I was just sharing sharing my whole nothing.
 
 
00:47:05
 
Devon D'Andrea: What's that?
Aksana Rahouski: yes.
Adam Curcie: You didn't want to tell me I was just sharing my whole screen while I checking my
Aksana Rahouski: So anyway,
Adam Curcie: emails.
Devon D'Andrea: Yeah.
Aksana Rahouski: so what I was going to say I will like I think let me book a few followup like discoveries for us to talk about apply uh and talk
Devon D'Andrea: Yeah.
Aksana Rahouski: more about um okay let me kind of chew on what you guys gave me but but that's a great feedback that's exactly what I um I need again this is like directionally we just need to make sure that
Devon D'Andrea: Okay.
Aksana Rahouski: it sounds like what you said okay this might be a better directional feed for us. So, let's just keep moving and see what else we are uh we can
Devon D'Andrea: Yeah, but like and I and I'll say this like from the UI standpoint, I know that's obviously not the finished product,
Aksana Rahouski: find.
Devon D'Andrea: but like I I like I said earlier and I think Adam mentioned it earlier as well when we were looking at it is like we really like the way it's we really like the way it it it moves, if you will.
 
 
00:48:06
 
Aksana Rahouski: Yeah,
Devon D'Andrea: Uh yeah,
Aksana Rahouski: it's simple, right? Not as Yeah,
Devon D'Andrea: just like the way the the way the buttons the way the buttons work,
Aksana Rahouski: but
Devon D'Andrea: the way it's just arranged,
Aksana Rahouski: it's
Devon D'Andrea: I I really really love it. Obviously, you know, things to change as far as the, you know, the mechanics,
Aksana Rahouski: Yeah.
Devon D'Andrea: but from a from a you know,
Aksana Rahouski: Yeah.
Devon D'Andrea: from an experience standpoint,
Aksana Rahouski: Yeah. But it's probably Yeah.
Devon D'Andrea: I really like it.
Aksana Rahouski: And it's probably going to be something close to that because that is that was that was like built on on bas on a base that is actually ours native, right?
Devon D'Andrea: Right. Right.
Aksana Rahouski: Because I like scaffold it kind of what the website actually looks like and then build on top of
Devon D'Andrea: Right.
Aksana Rahouski: it. So it's it feels a little bit more like native rather than for
Devon D'Andrea: Yeah. We had uh Axana real quick before we go.
 
 
00:48:50
 
Aksana Rahouski: Okay.
Devon D'Andrea: We had Claude whip us up a uh a uh it revised a proposal that we had created for a potentially could be one of our biggest customers if they become our customer.
Adam Curcie: s***.
Devon D'Andrea: Um we had Claude revise a proposal that we were sending to them and oh man it was
Aksana Rahouski: Yeah,
Devon D'Andrea: nice. It was so nice.
Aksana Rahouski: I know. I sometimes worry here.
Devon D'Andrea: Yeah.
Aksana Rahouski: I was like, how am I going to do my job as AI is not part of
Devon D'Andrea: I mean, listen, we had a proposal that we like because we used to do it ourselves, you know, like just like every word that was written was was,
Aksana Rahouski: I know.
Devon D'Andrea: you know, was ours and uh you know,
Adam Curcie: Handpicked.
Devon D'Andrea: but it's now quite quite enhanced,
Aksana Rahouski: Yeah.
Devon D'Andrea: I'll tell you that.
Aksana Rahouski: Yeah. And it's and and that's where again and I think you guys already see it, right?
Devon D'Andrea: Oh, yeah.
Aksana Rahouski: just like it could do a really good job obviously like rewarding thing polishing for you but it also like it does an excellent job helping you to think through
 
 
00:49:51
 
Devon D'Andrea: Yes.
Aksana Rahouski: things as you
Devon D'Andrea: Oh my god. Like there's parts and even just as something as simple of like the way it's formatted document,
Aksana Rahouski: always
Devon D'Andrea: it's like I wouldn't have even thought to like show it that way.
Aksana Rahouski: Yeah.
Devon D'Andrea: And as I'm looking at it and I look at the old proposal that we created, I'm like, wow, this is so much easier to digest. You know what I mean? And it's like makes me realize like,
Aksana Rahouski: Yes.
Devon D'Andrea: oh, like preparing a document that's got a full page and a half of just
Aksana Rahouski: Yes. Yeah.
Devon D'Andrea: straight text might not land the way we wanted
Aksana Rahouski: Yes. Yeah. And that's not like but even something like that,
Devon D'Andrea: to.
Aksana Rahouski: right?
Devon D'Andrea: Yeah.
Aksana Rahouski: Like you always like when you kind of have your gist, right,
Devon D'Andrea: Well,
Aksana Rahouski: of what you're trying to communicate and then you marry it with my audience is and like like my client is like more let's say visual driven versus more a reader, right?
 
 
00:50:42
 
Devon D'Andrea: Yep.
Aksana Rahouski: And this thing does such a good job, okay? Like build a presentation that looks like this versus look like that.
Devon D'Andrea: It's incredible. It's
Aksana Rahouski: It's it's it's really amazing.
Devon D'Andrea: incredible.
Aksana Rahouski: And I think like the more you use it, the more you recognize how you could use it.
Devon D'Andrea: Yes. 100%.
Aksana Rahouski: I'm literally now I sit like like I feel like just like move things around you feed it file and I have my my setup now. It's like I have it like running and then my conflence and all my comments and it analyzing my comments and reworking my version and pushing it back.
Devon D'Andrea: Yeah.
Aksana Rahouski: It's
Devon D'Andrea: So, we haven't really gotten to the point of like where it's doing doing anything automated or running in the background or whatever. So, we we definitely need to um we're we're beginner. We're very very much beginners, but it's incredible how much we've done been able to do with it just that way,
Aksana Rahouski: Yeah.
Devon D'Andrea: you
 
 
00:51:34
 
Aksana Rahouski: Yeah. And honestly, like like I told you,
Devon D'Andrea: know.
Aksana Rahouski: like just use it. Honestly, just jump into it, eyes closed, feet first,
Devon D'Andrea: Yep.
Aksana Rahouski: and and it's the best way to learn and see what it's capable of.
Devon D'Andrea: I
Aksana Rahouski: So,
Devon D'Andrea: agree.
Aksana Rahouski: yeah, I'm glad you guys are embracing it. And yeah, it's it's this technology is amazing.
Devon D'Andrea: It is. There's nothing better.
Aksana Rahouski: Um,
Devon D'Andrea: It's amazing. Um,
Aksana Rahouski: yes.
Devon D'Andrea: all right. Well, we will get you over some more information about the ways that our configs are customized currently and then we'll um yeah, I don't know when's our next when's our next I mean I know when's our next meeting in general.
Aksana Rahouski: Um,
Devon D'Andrea: Um,
Aksana Rahouski: so we have I also So I'm actually out Monday through Wednesday next week.
Devon D'Andrea: okay.
Aksana Rahouski: I can try to book us maybe something for Friday. Um yeah and maybe kind of like keep iterating on
Devon D'Andrea: Friday works for me.
Aksana Rahouski: this otherwise then we have a sponsor call on the 7th
Devon D'Andrea: Yeah.
Aksana Rahouski: which I want to align like and that one will align on our priorities um especially considering the meeting we had uh this week.
Devon D'Andrea: Yeah.
Aksana Rahouski: Um but I I will book us some like working sessions so we can keep um moving
Devon D'Andrea: Okay.
Aksana Rahouski: on this.
Devon D'Andrea: Sounds good.
Aksana Rahouski: Okay. Okay. Well,
Devon D'Andrea: appreciate
Aksana Rahouski: sounds we'll just keep keep chipping away,
Devon D'Andrea: it.
Aksana Rahouski: but I feel like I feel good. Uh I feel like we're on the right track. We just need to keep fine-tuning and we'll get there.
Devon D'Andrea: I would have never expected us to be as far as we are given the given just the the the the you know how gigantic of a lift this is. I think we're in really good spot. Yeah. Yep.
 
 
Transcription ended after 00:53:42
