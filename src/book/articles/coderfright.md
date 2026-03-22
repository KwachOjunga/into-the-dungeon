## Coder's fright 


[DISCLAIMER] This is a pre-release version of an unsufficiently tested method that i am espousing. I will expound on it when its done.
If you have variations of whatever i am trying to put across, feel free to launch a PR.


### Battling your inner demons

You know, in all the programming lore that i have come across, none of them
talk about how difficult it can be to approach a new codebase.

Ok, it is likely that my observation is biased by the fact that i am a beginner at this and the individuals that get to talk about stuff are experts.
Probably there is a shift with increase in experience.


Since the space that mathematics can describe is essentially infinite, i think the ability to be frightened by a codebase can be quantified by a function 
whose parameters are the relative size of the codebase and the average number of new concepts i will encounter per module.

Of course there are other things like the language in which the codebase is written that will factor in like one's mastery of the language 
and the degree of specificity[^1] that the lanuage allows.

So the size of the codebase will obviously determine the duration you have to spend 
nested in the codebase to have a firm grasp of it.
New concepts establish the neccessity to be explored especially if they are heavily 
referenced or used in the implementation.
These create detours that can often be deemed unneccessary since they don't help in meeting the deadline.

I'll probably come back to the utility of such detours but as for now let us 
explore our newfound fear.

So perhaps you are trying to be a systems engineer and there you are staring at a backend whose default mode of operation is with async executions and has also decided to use a threadpool in their implementation.

If you are sufficiently green, you have a vague idea of what asynchronous execution
means, and if you are lucky enough you have been exposed to it through languages like Go. 
You are unaware of synchronization primitives and your working assumption is all computation that has been 
parrallelized is in optimal working conditions.
Threadpools to you are similar to unicorns.

### Strategizing

At this point the codebase is like a predator and you are its prey.
Its primary intent is to crush your spirit and remind you of your inefficiencies.
It shouts of your incompetence.

You start looking into asynchrony. When you scratch its surface you come across the 
notion of a runtime, an executor and you learn of a way to mimic the operating
system's scheduler.

Now to get to understanding the scheduler you have to go a layer deeper.
It is there that you get a proper introduction to computer organization and its architecture.
If you pay heed to the call of adventure you get thrown into a world filled with
ISAs. The scales fall off your eyes and now what emerges is the realisation that "the 
world is way larger than the computer you are hunched over".

If you are a grown up, you cease the pursuit and retrace your steps. You have work to do.

Personally that is where my kind of trouble begins because i have always found that going
a layer deeper always yielded a better picture.

Now, if you are going to understand a codebase there is getting familiar with the "literature"
and then there is getting to understand the implications of the "literature" as applied or implemented.

Grasping the implications requires a whole roadtrip. You play out the serial execution as it occurs in the codebase
and verify the correctness or quality of emergent logic from the order of executions.

Getting to that point alone can be immensely satisfying but now there is a kick to this kick.
Now you want to modify the logic to enhance either perfomance or enhance usability by probably adding an extension.

If you have ever traced a circuit using probes maybe you'll relate to this. Adding an extension to the codebase is similar to knowing the exact places to tap in a closed circuit.
It is like knowing exactly which points can i attach to and i get the exact expected output.

The real kick is probably in enhancing performance. To enhance perfomance you have to breakdown the implementation detail.

From an operation that was simply responding to requests to access files to picturing 
threads (async or otherwise) that are listening to requests make a malloc call to allocate a buffer,
read a file into that buffer before transmitting it through the sockets.
Now you have to figure out where is it that you wasting time in and why.
Once you have found out you make a point of probably not touching the memory as often if memory was the bottleneck or 
you decide to switch the allocator all together.
If that's not enough and you have sufficient memory, you probably make the opening
of certain frequently accessed texts to be opened early enough to prevent several consecutive syscalls.

There is probably a trick about maintaining socket liveness to escape the handshake thing but anyway.
Those who are serious enough switch to io-uring.

I believe that it is in the pursuit of this kick that optimization engineers live.

[^1]: Certain languages particularly low level ones allow you to have more control of how your data will be computed. This is achieved mostly using compiler intrinsics or lints like those that enforce a particular data alignment.
