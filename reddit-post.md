Well, the journey of my first RFC for the PHP language is nearly over. My first RFC is currently in voting and is likely to be rejected. I was a frequent reader of this subreddit, had worked in PHP for very nearly 20 years, and simply decided one evening "I'm tired of complaining about the language, I want to actually try to improve it". So, I did. Or, well, I tried.

One thing that has struck me is just how obfuscated/difficult it is for someone to start contributing to php-src. Even some of the people I have talked with in internals over the last six months were blocked by problems that I encountered and was able to find solutions for.

In general, I think (based on my limited experience so far) that the RFC process is rather unkind for new contributors. In reality, it's very difficult to get an RFC passed unless you take the time to learn who contributes to what, what some of the personalities are... there's a very large human element to it that just isn't documented anywhere. 

In part, it *can't* be documented. I mean, how would you document something like "so-and-so generally dislikes proposals that do X unless they also do Y"? It's a very real element to getting an RFC passed though. I would suggest that you expect any RFC you propose to fail, because that's just... reality. Most RFC's don't make it to voting, are withdrawn, or are rejected. You can debate whether this is good or bad, but it's the reality of the process, so you need to go into it with your "eyes open" so to speak. Even knowing that your RFC may have long odds, it can still suck quite a lot to have something you worked on just... turned down. "No thanks, we don't want it."

But, the smaller your change is, the better chance it has. Can you implement your change without introducing any new syntax? That's a boost in your chances. Can you do it without any impact on existing code? That's another boost in your chances.

At the same time, your change needs to be significant enough to justify itself. If you want to add a single function to the language that performs a task that can already be done by some function in PHP code, you'll have a lot of people asking why even add it.

# Why are RFCs Rejected?

If you want specific reasons that RFCs are commonly rejected, I would suggest reading the page written by /u/danack on the subject: https://phpopendocs.com/internals/rfc_attitudes

Here, I'm going to talk about the more conceptual reasons that rejecting an RFC isn't a bad thing.

## Maintainability and Contributor Commitment

I do not think that anyone in internals would say they are rejecting something because the person proposing it hasn't contributed to PHP before. I also do not think that anyone in internals would say they are rejecting something because they *expect* the person proposing it to disappear after it gets included, a "drive-by commit".

But... I think this *is* a thing that people must (and probably do) consider. If you propose something that adds lots of complexity but you don't plan on continuing to contribute in the future, the people who are still working on the project afterward will need to maintain your implementation.

Anything you contribute is likely to need maintenance for a minimum of 10 years into the future, because in general PHP's BC guarantee is much stronger than most languages.

Wider reaching changes are more likely to be accepted if they are proposed by people who have a history of contributing to internals. That's perhaps unfair (it will certainly feel that way to you), but it's also very rational.

In other words, choosing something that has large implications on the behavior of the language in general is probably a poor choice for your first RFC, because it will be difficult to pass it even if it is (somehow) an objectively perfect design.

## Language Purpose and Priority

The people who vote on RFCs have opinions about what PHP is for, what it *should* be for, and what sorts of changes are high or low priority, just like anyone else. If your proposal significantly changes the style of the language, or what the language is for, it'll probably meet a lot of resistance.

"If you want to do X then just use language Y."

You'll be told this more than once. Again, whether this is a good or bad, it's reality, so go into the process knowing it.

# How To Start Your RFC

In my opinion, (which again is based on my own experience of going into an RFC with no prior experience in php-src), the absolute first step in writing your RFC should be getting a dev setup going where you can make changes to the engine and see what happens.

You *should* know roughly whether a change you want to make is even possible before bringing it up to others. You don't need to know all the details, but if literally thousands of people were asking the 6-10 people who can simply give an answer, they wouldn't have time for anything else, and none of them are paid to work on PHP full-time (now that Nikita no longer works at JetBrains).

## Getting a Dev Setup

Obviously you need to fork the php-src repository so that you can work on it. After that, you need to get some kind of setup where you can see what all the symbols mean in the C code. I used the CLion IDE, which is also from JetBrains. Currently PHP does not use CMake, which makes it difficult for most IDEs to understand it, however Ben Ramsey wrote an excellent article describing, step-by-step, how you can setup CLion to understand how the PHP project is supposed to build: https://dev.to/ramsey/using-clion-with-php-src-4me0

## What the Different Parts Do

To understand this more completely, I would suggest reading through the PHP Internals Book: https://www.phpinternalsbook.com/

I will give two very basic categories that probably cover most RFCs however:

- If you want to change a function or class in PHP, add a new one, or something like that, it probably is in the /ext folder.
- If you want to change something about the language, such as syntax, what a particular expression does, how things are compiled, etc., it probably is in the /Zend folder.

For my operator overloads RFC, the vast majority of my changes were in the /Zend folder. I was adding new syntax, the `operator` keyword, and I was changing how several of the opcodes work, and I was changing the default behavior/capabilities of objects. All of that kind of stuff is in /Zend. 

I also needed to update Reflection and OPcache, both of which are in the /ext folder, but those changes were relatively minor in the scope of the rest of the work. Mainly, I wasn't adding or changing functions, I was instead working with object structure and language syntax.

## The Mailing List

Subscribing and participating in the mailing list is a core requirement if you want to write an RFC. In fact, to be able to edit the RFC page, you must ask for someone to enable that permission on the mailing list, so you can't even write the RFC without doing this.

You should be aware of several things with the mailing list though:

1. Not everyone who replies to things on the mailing list has voting rights, many are people just interested in PHP development like you.
2. Because of that, it's not always... worth it, I guess, to get into deep technical discussions with everyone on the mailing list.
3. Despite that, you can't simply ignore or not participate. The people who vote reply to the mailing list much less frequently, but if someone new is proposing an RFC and doesn't participate well... again, though no one has ever said this to me, I have to believe that would count against you.

A big part of this is that some voices you should listen to, and some you can pay *less* attention to. Some people will vote no to nearly anything, regardless of the merits (yes, of course those people exist within the PHP project). Some people will vote yes to very large changes even if it's likely to cause problems.

There are a few people that you very much need to talk to in certain situations however, and that can be trouble if they don't have the time or interest to respond. But, most of the people that are of the "critical resource" nature within the project aren't going to just ignore you, no matter how new you are.

If you are going to make changes to /Zend, you probably need to talk to /u/krakjoe, /u/nikic, Bob, and/or Dmitry. Someone who is active in the project currently and has contributed to the /Zend folder before. They are going to have a lot more insight into how your change might be accomplished, whether it's likely to make it past a vote, that sort of thing. They don't exactly get to veto changes, but it's going to be functionally impossible to pass an RFC that touches the /Zend folder without the support of some of these people.

If we take my Operator Overload RFC as an example, Bob voted for it, but Joe, Nikita, and Dmitry all voted against it. Bob actually helped me with part of the implementation, Nikita expressed that he might vote for it with a different syntax, Joe explained that he thought the idea itself was flawed (operator overloads), and Dmitry provided no interaction at all.

Think about it. That's the messy internals part *of* internals. Other people working on the project may not have had to work in it at all, and *know* that their knowledge of the area is limited. If they see all the people who have experience in that area vote against, or express concerns, they are not going to be very likely to vote for a change that affects it. Having most of those people vote no is a strong signal that other voters are likely to follow. So while there's no formal extra power that any of these individuals have, their contributions and knowledge are respected and valued.

However, if you are able to set up a debugger like the one in CLion, the code in /Zend isn't really that difficult to deal with. I had never worked with it prior to last August, and at this point I know more about it than many of the voters (though obviously quite a bit less than the names I mentioned above). So if that's where your feature takes you, don't be scared. It seems quite large and convoluted at first, but it's actually pretty straightforward once you understand the process that everything goes through.

## Your Use Cases (And Voter Use Cases)

In general, it is really difficult for people to understand use cases they don't experience. Maybe you spend all of your time in PHP working on framework based websites. If so, it might be difficult for you to understand or relate to the use case of someone who uses PHP to write command line scripts.

Think about your feature and how it might be used. What are *all* the ways that it might benefit a particular use case? The more use cases you can present, the stronger *your* case is for adding it to the language.

At the same time, be prepared for the idea that some people are going to vote no if your feature doesn't improve *their* use case. You might find it frustrating, or you might relate to the position, but in either case it's the reality of it.

## Reach Out For Help

If you are stuck, reach out for help. The brand new PHP foundation has helping new contributors as one of its goals, and it could definitely be a resource for you. Many of the people who work on the project want to see new contributors, particularly new contributors who seem like they might stick around and continue helping. There are definitely people who are willing to invest some time and some effort in helping you, but you need to ask.

# After Writing the RFC

Strictly speaking, you do not need to provide code that implements your changes. It improves your chances, particularly for a large change, because it shows your own commitment *and* it shows that the change is possible. Anyone *could* write an RFC describing a beautiful Generic Types setup, but no one will vote for that until it is shown that it *can* be done. There's also the issue of "volunteering" someone else's time. 

Everyone who works on PHP does so on a volunteer basis. If you write an RFC and voters pass it without any implementation, *someone* has to actually implement it.

So you could stop here and move on to the next step: putting your RFC on the mailing list.

In general I would suggest trying to get what's called a "playable commit" done though, if you can. This is a commit where the feature you are describing works, even if a few other things are broken. For instance, it's not critical that all tests pass if the failures are in unrelated areas such as opcache.

A playable commit means that voters can check out a build your changes, then write PHP and see exactly how it behaves. They can test it and play with it.

# Open Discussion

To open discussion on your RFC, simply create a new topic on the mailing list. Do not expect much feedback, or that the people who do provide feedback have voting rights. That *might* happen of course, but it shouldn't be something you expect.

For better or worse, very little useful feedback happens via the mailing list. You'll get feedback from active contributors at this stage mainly in two circumstances:

1. Your RFC touches something that one of the main contributors was already looking at doing.
2. Your RFC does something that one of the main contributors is categorically opposed to or thinks is unworkable.

Detailed feedback often comes only once voting happens, and sometimes only through direct conversations. Of course, at that point it's too late to just change your RFC with the advice given but... again, I'm here to describe the *actual* process you can expect.

# Build Consensus

One of the things I was told multiple times is that you're supposed to build consensus before voting happens. "Voting is not a way to build consensus". This might seem... odd. In fact, I think it kind of is. The idea behind it is sound. If you're going to change the language somehow, it's better to build a discussion based consensus prior to voting instead of opening voting and then it barely gets included with some voters feeling blindsighted.

Your main blocker to building consensus is going to be that many of the people you need to build consensus *with* just don't respond to messages. Perhaps they do respond to people that they recognize as "frequent contributors" to the project, I don't know. So your only realistic path forward is to find someone in the project that will both:

1. Actively participate in discussion with you, and;
2. Understands the voters well enough to provide you rough feedback on the likely outcome of a vote.

This is, by far, going to be the most difficult part of your RFC. In part because of the communication barriers, but also because there's no way for you to really measure the level of consensus you have. That's... what the vote is, but also not what the vote is supposed to be used for.

# Open the Vote

If it's been at least 2 weeks since you announced the RFC on the mailing list, then you can open the RFC for voting. In reality, you probably need to wait longer.

Opening the voting involves editing the wiki page for your RFC to include the voting widget, and then announcing it on the mailing list with a separate topic that explains the dates the vote is open, and links to any previous relevant discussions.

---

The php-src codebase is actually pretty easy to work with, in my personal opinion. Most of the difficult parts of this process are all process problems and people problems. But mainly, all the difficult parts are designed to make sure that hasty, unmaintainable things don't make it into the language. Is there collateral damage as part of that? Sure. But overall, it's a good goal that has helped the language continue to be successful and improve.
