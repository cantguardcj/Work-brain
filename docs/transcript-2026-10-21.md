# Medical Writing AI Tool Strategy Discussion
**Date**: Wednesday, October 21, 2026 at 2:05 PM  
**Duration**: 28:44  
**People**: 2 participants

> Saved as given. The original paste contained this transcript twice; it is stored once here.

[0:00] **Participant 2**: But then there's content elements that they want to see and they c and and they're the co it's context specific to the section, the task and it's just hard to predict where they're gonna end up. I think that that's what's hard about this. I think my my fear my fear with the automation is is what I said in there, right? All we're doing is spending more money on L L M calls and resulting in the same thing, right?

[0:29] **Participant 1**: Like

[0:30] **Participant 2**: you you you tell you you let it rip and it's just like this is not what I wanted at all, but it's now or later. Uh where if you're manually mapping evidence, you know, maybe maybe you're a little bit further and it was frustrating, but you got there. So I don't I don't know. I I I don't know. There's there's one thing that we've talked about, um I talked with Caitlin about a little bit. Um the style element.

[0:58] **Participant 2**: One thing that we could attempt to do is um people we all have preferences on writing, style of writing. We could attempt to capture this in the system during the generation process. So today you're hard coding props, right? And then they they're useful or they're not, and then each time they gotta write words to try to make it better, whatever. Um it doesn't have to be like that. The system could be designed so that it captures their personality personal preferences. And then those preferences are used in subsequent generations.

[1:36] **Participant 2**: One idea that we had talked about with Caitlin was maybe we run an experiment. For example, you show someone two outputs, and then which one do you like better? And like the output, I don't know if it's medical, I don't know if it's whatever, but you show them two outputs, and they say this one. So you start to capture, maybe you ask them two, three, five, ten of these things.

[2:00] **Participant 2**: So the cognitive load in their end is not the writing, but they're but by seeing their clicks, they're you're capturing their latent preferences on text, and then you you use that information in the generation process. So my guess, my theory is Jang  preferences are different than Rebecca's. I do not believe if you show them two things every single time they're gonna click the same thing.

[2:23] **Participant 1**: Not even close.

[2:24] **Participant 2**: But you know what? But you know what they're going to do? They're going to evaluate the criteria based on their own personal preferences. So you coming up with a generic prompt is almost fundamentally doomed.

[2:37] **Participant 1**: Yes.

[2:37] **Participant 2**: Because it's the average it's the average of all opinions and people don't want the average. They want their own.

[2:45] **Participant 1**: Yes.

[2:45] **Participant 2**: So if you build the system to sort of accommodate that, maybe we can get more of the styling in line with what they they want. Um now the big question is, is that true? How hard would it be to implement that? Those are those are open questions, but

[3:02] **Participant 1**: well okay, so it sounds like sorry, let me clear my throat. It sounds like it sounds like you're proposing some sort of A B testing for the sake of um learning style of a medical writer. And are you saying this would happen before, during like when when would this getting of of

[3:31] **Participant 2**: It would happen before. It would happen so what let's say you and I have different preferences on text. We wanna capture that as early as possible, right? You want p potentially before any generation occurs. And then when you generate it, you can use those preferences as you generate the content output. Like what if we go through this A-B testing and we discover that Jang really likes short terse factual information and no fluff?

[3:58] **Participant 1**: Yeah.

[3:59] **Participant 2**: Well then those captured preferences could be part of the system message, they could be part of the whatever, and then it would be embedded in each subsequent call.

[4:09] **Participant 1**: Mm-hmm. No, that that that makes a lot of sense. And personally, I thought about this as a potential solution as well for specifically my p with the with the prompts.

[4:20] **Participant 1**: Um like my idea was um come up with five different similar prompts for a given section specifically like say a demographic section give it to the medical writers um and they evaluate the different outputs in A B testing style um and they and then they say why um and then after we get each of those whys we can then build a better prompt or even save their preferences in that. So I thought about building something like that but I wasn't sure if that was really like in the scope scope of my roles and responsibilities.

[5:06] **Participant 1**: But I definitely have this idea and I do think that it um it could produce good results.

[5:13] **Participant 2**: Good. Reliant. That's a good sign. Um the other things that has been kicked around is okay, so in the call, I don't know if I said this, but I've thought a lot about like reducing cognitive load.

[5:31] **Participant 1**: Yes.

[5:32] **Participant 2**: Like, how hard is it for you to write code now? Probably not that hard because the system does a lot.

[5:37] **Participant 1**: Yeah. When

[5:37] **Participant 2**: they were using this tool, you could tell they they their ears were smoking. Like they were putting a ton of mental energy into the process. process. So how do you reduce the cognitive load on them? One thing that I was thinking of is very similar to what you're describing, but instead of it being a you know something discovery during the build process, could you build that into the product itself? For example, when it generates output, maybe it generates more than one.

[6:07] **Participant 1**: Maybe

[6:07] **Participant 2**: it generates three. And then you pick the one you like best or you pick none of them or so you go from you're moving people from writing to clicking buttons. It's so much easier to click a button than it is to write. So if you

[6:21] **Participant 1**: It feels like we're building into CTD   we're building a different artist.

[6:26] **Participant 2**: Kind of. You know, and that was I that was the other thing I was thinking about. about. You have Claude Code, it's a coding harness. Claude writer, writing harness. I don't know. Like that kind of feels like that's kind of what this it's like it's not that, but it kind of is that.

[6:41] **Participant 1**: It's getting there, for sure.

[6:43] **Participant 2**: Yeah.

[6:44] **Participant 1**: And all the features that we're describing, uh thumbs up, thumbs down, uh uh give it three different outputs and tell like these are these are things that they've already implemented and I'm sure found success in.

[6:57] **Participant 2**: Yeah. So I don't know. But I think a lot of these things are at the end of the day, i listening to their feedback. I think some things can be be automated effectively.

[7:13] **Participant 1**: Yeah.

[7:14] **Participant 2**: And some things are going to be very, very difficult. And how do we detect those? How do we how do we get the system to like how how do we get it to detect a boundary where it shouldn't go or it needs to go deeper or it needs to go shallower or something like that. I think that that's where there's the risk with the automation because it you're just letting the agent do something and just hoping for the best.

[7:45] **Participant 1**: Yeah. To that I say Rebecca's point about doing something perfect herself and then giving it a template like in my experience LLMs perform the best when you give it a golden example. So that feels like the best way to go about that problem specifically.

[8:08] **Participant 2**: Right, but are you gonna get her get her to write a golden example from every single section of every single part of the document? I think that's the hard part is that works for one, how do you scale it up for all of all of the content? Is that actually going to save time? No, Nate Nate's actually mentioned this. So Nate one of the things Nate suggested is is we have this more agenc capability that basically does more long running tasks instead of just point tasks.

[8:43] **Participant 1**: Yes.

[8:44] **Participant 2**: And they provided some input material. He called it a template, but something. Something to the agent that is kind of what this golden example would be. How do the five patterns to look for, you know, like basically this the key elements that the model needs to steer it are in this input artifact. And then you let it go and it says, I'm gonna build the sections like this, and you click the go button and it and you and it rips. And like again.

[9:21] **Participant 2**: it's like how what needs to be in that input artifact so it works effectively? How far do you let it rip? Where where where does it need to stop?

[9:30] **Participant 2**: Um I think that is that is sort of the risk of going that

[9:33] **Participant 1**: route

[9:34] **Participant 2**: is you end up right where you're at now.

[9:39] **Participant 1**: Yeah, exactly. I think though while we're we're here now, I feel like and I feel like I'm gonna sound like a broken record, but if they had some guiding light of key messages or a golden example, it's like I don't think that half of the problems that they're talking about of no continuity within the document, those might be be a lot. Like those those might not exist. And and like yeah.

[10:12] **Participant 2**: Well I mean one of the things that was clear is they were talking about repetitive like a data dump in the content. When I hear them say that, what it tells me is they did not do significant refinement in the product itself.

[10:26] **Participant 1**: 100%

[10:26] **Participant 2**: I could generate they just click the button and they say here it is. Um

[10:32] **Participant 1**: but that's

[10:33] **Participant 2**: your point

[10:33] **Participant 1**: right kind of where we're trying to get them to go, right? Make it easier for them, no cognitive load

[10:44] **Participant 2**: Let me ask you this. they clicked a button and got the exact same output, let's say they click a button and got verbatim what was generated in five minutes. What do you think would have happened? Do you think they would have spent more energy improving the product, or they would have just stopped?

[11:08] **Participant 1**: Can you rephrase your question?

[11:11] **Participant 2**: So Jang  I don't know, 16 hours or something. Rebecca spent 12 hours in the tool.

[11:17] **Participant 1**: So

[11:17] **Participant 2**: they ingested the data, they built the evidence pro section by section, they generated, they did this interaction. So they put a bunch of energy in the workbench itself, and then they clicked the export button, looked at the output. output, and then said it's gonna take 40 hours or whatever to edit it. Now let's say you can erase all of the energy spent in the product. Like you you turn the cognitive load in the product to zero and then you have whatever's generated is what the system can do automatically.

[11:47] **Participant 2**: Do you think they're looking at that and they're thinking do you think they would spend energy in the tool refining it more? Or do you think they would just stop and be like, this is good enough and and like they would edit the same way. Cause like that's kind of what we're getting at is do they do they want to why didn't they spend more energy in the product? product. But they just exhaust it?

[12:16] **Participant 1**: I think it's just

[12:17] **Participant 2**: automatic

[12:17] **Participant 1**: native to them.

[12:19] **Participant 2**: Okay.

[12:20] **Participant 1**: What's native to them is a and me and Sandra were joking about this, they want genuinely like Claude Rider. Like a word clone with AI capabilities. They're they're native to using Word and they prefer it in a lot of ways. So when you're in the tool doing stuff versus when you're in Word, something that they're used to, they're gonna lean towards doing all of the things that they do they've been doing for years and years and years there, if that makes sense.

[12:53] **Participant 1**: And I feel like that's that's like that's intuitive, but but they're not gonna wanna do that. Like yes we could add something in the tool where they can edit it just for the sake of having it better looking in the export, but they're spending that time regardless and all those capabilities are in word.

[13:13] **Participant 1**: I'm not saying that it wouldn't be well actually I don't know what the implementation would be look like, um to add the in tool editor um

[13:23] **Participant 2**: I'm very, very resistant to that and I've been resistant to so you before you started before the C D D tools even built we had this discussion about spending time in the tool. If you remember the C S R tool this is there's basically a barely functional interaction. But it's effectively you put the documents in, you click a button and receive the output. The amount of time people expend in the product is very, very low because we don't want people to use the tool.

[13:52] **Participant 2**: We want them to get the output and then work in Word because that's what they're used

[13:55] **Participant 1**: to. Yes.

[13:56] **Participant 2**: The th the theory behind the CTD tool is we need something that's more capable and dynamic and more in control that is better than Word. But what you're saying is no no no, they just want to use Word at the end of the day. So

[14:13] **Participant 1**: you want a Word clone or something like that.

[14:17] **Participant 2**: So if that's true, then the goal should be compressed. compress time in the tool to as close to zero as possible so they can get to the where they want to edit it, which is Word.

[14:28] **Participant 1**: I don't know if that's what they want. I think some medical writers will want a hundred percent full control. They use this instead of Word, but I do think that some medical writers are do exactly what want, exactly what you described, and that's spend his least time in here and go straight to Word because that's what I'm used to and that's what I want to do.

[15:01] **Participant 2**: Everybody's gotta get whatever they want all the time.

[15:05] **Participant 1**: that's why we have a generic prompt that nobody likes.

[15:12] **Participant 2**: Someone made a made a comment the other day. No, it was like it was my neighbor. He um my neighbor he owns a there's a partner in a like a a design business. Like they do advertising and stuff like

[15:25] **Participant 1**: that. Mm-hmm.

[15:26] **Participant 2**: And we were talking about AI and I was like, do you guys use AI? And he's like, we use it for like storyboarding but not for real creative products. And he said what he said I think is related to what you're saying. He said art AI generated art is the average of

[15:41] **Participant 1**: everything. Yes.

[15:43] **Participant 2**: That's what it is. So you're when when you get these prompts it's it's the average it's not the individual preference and again it's like without a standard without an objective truth the person's preference is the correct thing.

[15:59] **Participant 1**: Yes.

[16:01] **Participant 2**: So that's the world we live in. Like we can't until they have a standard there's there is no there's no objective correct answer.

[16:11] **Participant 1**: So it sounds like we need to make one somehow, some way

[16:16] **Participant 2**: sounds great. I like the idea of that we've been trying to do that for a while. Um

[16:22] **Participant 1**: I'll get back to

[16:23] **Participant 2**: we should try to do

[16:24] **Participant 1**: that.

[16:25] **Participant 2**: I'm more I'm less optimistic that we're gonna achieve that.

[16:29] **Participant 1**: Mm-hmm.

[16:33] **Participant 2**: So if there is no standard then and people want to work in Word, you just need more automation. Get their s get their personal preferences, try to capture it somehow, click the button. Look at the output, be angry about it. But you know what, you got it in two hours and not three days.

[16:56] **Participant 1**: Yeah. Um wait, but w how do you I think the thing that they're angry about or the thing that they really don't like doing is mapping. Like I don't know how we could make that process better. Do you have any ideas?

[17:14] **Participant 2**: do mapping. I I think that there's very high likelihood you could get a very coherent first pass at mapping. And again, I think part of the part of the cognitive load is bootstrapping. Starting from zero. I think it's very I guess you're you're new to the workforce.

[17:31] **Participant 1**: Yes.

[17:32] **Participant 2**: One of the things one of the things that I personally appreciate is when people generate a presentation or content from scratch. Like you went to the energy, you built it, and then I can review it and comment on it. The the energy the activation energy to get from nothing to something that is reviewable is enormous.

[17:54] **Participant 1**: Right.

[17:55] **Participant 2**: The energy for me to review something and scrutinize it is much, much smaller. So how do you m how do you move them into that? Whether scrutinizing or reviewing instead of generating from scratch? 'Cause that feels so much harder.

[18:08] **Participant 1**: Hmm. Okay. I get what you're saying.

[18:13] **Participant 2**: Like what what would you rather do? Would you rather review someone's paper or write it?

[18:19] **Participant 1**: Personally, I'd rather write a paper if I'm passionate about it. I don't like rereading other people's work.

[18:27] **Participant 2**: I hate ri I hate writing.

[18:29] **Participant 1**: I

[18:29] **Participant 2**: can barely write. I don't know if you notice, I I I barely write.

[18:33] **Participant 1**: No, I mean I understand. Um like I'd rather like even even in terms of like coding, I'd rather write code than review someone else's code. Like that idea just feels so like Yeah. I feel like it really just depends on the person.

[18:51] **Participant 2**: Look at coding. People don't write code anymore, it's gone.

[18:54] **Participant 1**: Yes.

[18:56] **Participant 2**: Why do we think writing is why do we think writing is exempt from that?

[19:01] **Participant 1**: This this is very true. I did it write a single line of code for my my feedback tool. But I did explain the ideas in great detail and what I wanted.

[19:12] **Participant 2**: That's what that's the skill. The skill of a coding agents means the instruction and the planning. And you know what? That is a skill. It's legitimately a skill. It's something you you have to learn how to do it so that the system is reliable. But that's a hell of a lot

[19:28] **Participant 1**: easier

[19:29] **Participant 2**: than how do you write a prompt for section two so that it's

[19:34] **Participant 1**: lean and then

[19:35] **Participant 2**: lays out all the evidence. Like

[19:37] **Participant 1**: you

[19:37] **Participant 2**: don't know it is so cognitively difficult.

[19:41] **Participant 1**: I think yeah, no, I don't uh I think that it has to be it has to be the medical writers with that type of experience and want to that would make it successful. I think that we talked about this on like one of our first meetings. I think the medical writers have to really want to and really have that I don't really w I don't know how you would describe that skill.

[20:03] **Participant 1**: Um but they need to have to have that ability. 'Cause if not you put it in someone's hands who's not very tech savvy, who doesn't really want to use the tool, who doesn't really like mapping, they're it's gonna be useless to them.

[20:19] **Participant 2**: Right. But so the th my theory is a lot of people struggle with the tool because it's very hard to learn it how to use it and there's resistance. The more the more you can cognitively offload to the system, I think the more they'll come around to it. Like, for example, if all I can do is write like if I can just like describe what I'm trying to do and the system generates a coherent plan I can inspect and I click a button and look at the output. output. Like my cognitive load there is

[20:52] **Participant 1**: low. Yes.

[20:53] **Participant 2**: But if I have to go into this tool and chat with it and map out, that feels like so much more difficult. And it's like I don't want to do this. Just give me the three word documents and I'll copy and paste it in here. It's faster than this.

[21:05] **Participant 1**: Yeah, I mean, and that's genuinely that that's exactly what I do. Like I transcribe, I explain it out loud with all the um filler words and all of the the bad out bad bad text, but I just describe it in great detail. And then after that, I ask it to create a plan, then I inspect the plan. That's about the most inspection that I that I I like to do, inspecting a plan, and then it usually nine times out of ten it produces something that I like once it creates a plan that I've edited. Um

[21:44] **Participant 2**: and coding agents, like like I said, they only became very, very popular at the beginning of this year. It's when Claude Opus four I think came out. That's when and it was because the system capabilities finally crossed a line. Like this invisible line. People have been used like do you remember GitHub Copilot back in the day? Like a couple of years ago? It was like we'll auto-write a function. We'll auto write a couple lines of code. That was it. Oh

[22:09] **Participant 1**: auto write a function contract. Yeah, exactly.

[22:13] **Participant 2**: That is what this tool that's what this tool's doing right now. Writing these things functions one at a time. That's not when you got great adoption. Great adoption occurred when it could do the whole thing from a plan and the system was powerful enough to actually do it.

[22:28] **Participant 1**: So are we trying to automate med the medical writing as a practice?

[22:37] **Participant 2**: Uh yes. But we don't know how if we're capable of doing it yet.

[22:55] **Participant 1**: Why not?

[22:59] **Participant 2**: I can't promise you this will work. I think the technology I I think the technology exists. Technology exists to just do what I'm describing, these tasks, but I don't know like we lack a lot of information. There's a lot of tacit knowledge in people's heads that is only in their heads.

[23:18] **Participant 1**: Yes. And

[23:19] **Participant 2**: it's not written anywhere. And what you're doing is you're relying on the system to have the average of all medical writers and it's pre training to be able to do the same thing with minimal instructions. You're you're s you're assuming the model has enough alignment to what medical writers want to do that when it generates an output it'll they'll be like, Yes. Exactly.

[23:45] **Participant 1**: I feel like if you put ten senior level medical writers with experience using AI in a room with ten anthropic engineers who are good at building AI harnesses and you put them in a room for twenty four hours, this could get figured out. But the key word is people who understand how to communicate to the medical writers have to know how to communicate to the people who build it.

[24:20] **Participant 2**: Nope. It's the other way around.

[24:22] **Participant 1**: So you're saying that people who are building it need to know how to speak medical writing language?

[24:29] **Participant 2**: Yes. That's the secret sauce. That's why our team is special. That's why you're special. That's why Xander's special. That's why Nate's

[24:38] **Participant 1**: special.

[24:39] **Participant 2**: Because most people can't do that. That's not a skill IT people have. We appear to have that skill.

[24:50] **Participant 1**: I see.

[24:53] **Participant 2**: The Rebecca's a rare.

[24:57] **Participant 1**: Okay. Yeah. Yeah. This is true.

[25:03] **Participant 2**: So was a deep conversation. Deep one. Alright. Any parting words? Parting thoughts?

[25:16] **Participant 1**: Parting thoughts. I'm gonna well since I don't really have much to do now that I just finished my feedback feature today, I'll look into seeing what I can. Um I'll probably set up another meeting to get your thoughts on what it would look like for an A B test tester type thing. Yeah.

[25:51] **Participant 2**: We're in this like purgatory phase. So we're wrapping up the template stuff. And

[25:58] **Participant 1**: then next

[25:59] **Participant 2**: week we're doing the test readiness. Um

[26:09] **Participant 1**: I don't think I'll have any real roles and responsibilities until next week.

[26:14] **Participant 2**: No, I think I think I think that's fine if you want to start if you want to start thinking about that and putting something together. Um what else can we do? So like if you could how would you demonstrate you can can build a system that can take my preferences and use it to generate output?

[26:38] **Participant 1**: I gotta think about it.

[26:39] **Participant 2**: How would you do that? Yeah, think about it and and mock something up and then let's let's see if it's possible to capture and embed preferences for

[26:50] **Participant 1**: for

[26:50] **Participant 2**: this.

[26:51] **Participant 1**: I think

[26:51] **Participant 2**: if you can do again does this doesn't have to be a big thing. Something small I think um I think that could be useful. And like if there's ways that more than what like let's say there's ways in which multiple people can use it and they can have multiple generated outputs, I think that would be that would be interesting to look at. Because again it's still a theory right can you capture through people clicking buttons can you capture capture meaningful preferences that they will appreciate in the output?

[27:21] **Participant 1**: So when you say multiple outputs, you were saying have multiple people with multiple outputs. What does that mean?

[27:28] **Participant 2**: Okay, so let's say you're trying to capture my personal writing preferences.

[27:33] **Participant 1**: Right.

[27:33] **Participant 2**: I'm saying you should show me uh I I don't know how you want

[27:38] **Participant 1**: to

[27:39] **Participant 2**: do this, but let's pretend you show me a piece of like you show me three pieces of output, like it's three different buttons, three pieces of text. And you c I click the one I like, and then a new set of prompts come up, and I click these buttons maybe three, four different times, and then at some point you have information about my preferences now how do you use that information to generate some output that I can assess evaluate something like that.

[28:10] **Participant 2**: I think if you prove that concept it'd be powerful because you would have demonstrated you can you can obtain personal preferences and embed it in generation of output and that could be built into the product itself.

[28:28] **Participant 1**: Yes. Okay. I understand the task now. Thank you. Alright have a good one.

[28:39] **Participant 2**: See you man. Woo

[28:43] **Participant 1**: That matter just
