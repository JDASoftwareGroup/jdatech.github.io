---
layout: single
title: "Java: Giants and Infinite Loops"
date:  2020-09-1 12:00:00
tags:  technology
header:
  overlay_image: assets/images/tech_gear_banner.jpg
  overlay_filter: 0.2
  show_overlay_excerpt: false
author: Andrew Laird
author_profile: true
---

# Java: Giants and Infinite Loops

I have an easy-to-code, hard-to-spot, impossible-to-debug infinite loop puzzle for us to solve.  To help us see the issue, I also have a small handful of amazing people to introduce who have helped me solve numerous problems.  But, first, some introductory quotes:  

>This should terrify you! – Douglas Hawkins, java JVM and JIT master, while introducing this puzzle. 

>If I have seen further it is by standing on the shoulders of Giants – Isaac Newton

>Oh, I’m a moron.  I had a breakpoint set.  Sorry for the panic attack – me


The first quote is me, Mr. Senior Principal Engineer, to a colleague recently after realizing that he did *not*, after all, introduce an infinite loop into a customer-facing software patch.   The second is from a scientist who himself became a giant whose shoulders uncounted others – famous, incredible others – have stood upon.   The third is from a java JVM and just-in-time (aka JIT) compiler master, as quoted from one blurb in a 100+ minute presentation on his area of expertise.  All three are relevant to this blog, but first the puzzle:

## The Seemingly-Impossible Infinite Loop
Let’s start with Hawkins and his "terrifying" situation, who at the time was explaining a trivial-to-code *infinite-loop* bombshell.   

**Shared Variables:**  
```java
Object dataShared = null;  //some data that the producer will create.  
boolean sharedDone = false;  // true only when we have produced dataShared.  
```
**Producer Thread:**
```java
dataShared = …;   // some data is being produced here by the thread
sharedDone = true;  //we are done producing so let the consumer make use of it
```
**Consumer Thread:**
```java
while ( sharedDone == false);   //busy wait spin until producer is finished.   
print (dataShared);   // executes once sharedDone is set to true by the other thread
```
The puzzle is, quite simply, do you see the issue?   Amazingly, if you create a test program and debug this, you will not reproduce the problem.   If you create a trivial test program, you also likely will not reproduce the problem.  If you put this into production with real producer and consumer logic, you most likely *will* reproduce the problem (except this time you were really hoping you would *not*).   What on earth could the issue here be?

This brings me to Newton.   

## Isaac Newton
Newton’s statement reads like a humble admission but is more accurately a **roadmap for success**.  [Brain Pickings](https://www.brainpickings.org/) has a nice article, [Standing on the Shoulders of Giants](https://www.brainpickings.org/2016/02/16/newton-standing-on-the-shoulders-of-giants/), on the background of the Newton quote.  At the time, the notion that one should study past masters while in pursuit of new knowledge was a minority viewpoint.   Somehow the need to rely on others was a bruise to the ego.   Newton understood, however, that revolutionary ideas can come from combining existing knowledge from mundane sources.   Einstein would later call this creative process **“combinatory play”**.  And, indeed, what Newton produced was truly revolutionary.   Let’s take his advice and tour some masters to see if they can help with this puzzle.   

## Martin Thompson
So, if at first glance the code looks clean then maybe there is something hidden in the hardware architecture.  The giant who first introduced me to the great impact of computer architecture on production software is **Martin Thompson**, as found in a one-hour presentation [How to do 100K TPS at Less than 1ms Latency](https://www.infoq.com/presentations/LMAX/) with Michael Barker that is very relevant today despite having been recorded in 2010.  Martin Thompson, borrowing a term coined by racing driver Jackie Steward, explains that you must have some **“mechanical sympathy”** to understand how to write code that runs well on production-quality hardware.   Even if you do not use the software pattern ultimately revealed in this video, the video itself is an excellent display of using mechanical sympathy in software design considerations.  

I will pause a moment while you watch the video…. All good?   Great, so, now we can reveal the first probable cause for the infinite loop above:   We may have multiple copies of “sharedDone” in our L1 to L3 CPU core caches, while one has been set to “true”, the other may still say “false”.   The Consumer Thread, not knowing it has a stale version of the truth, loops on and on.

![cpu with cached sharedDone](/assets/images/2020-09-14-cpu-with-shareddone.png)

Figure 1:  sharedDone is “true” in the blue Producer core and “false” in the green Consumer core.  

## Aleksey Shipilev
But wait -- shouldn’t the java memory model properly update other threads (and the underlying core caches) when a shared variable is updated?   The next giant on our tour is someone who can speak to that: JVM master **Aleksey Shipilev**.   Just reading through his articles in [JVM Anatomy Quarks](https://shipilev.net/jvm/anatomy-quarks/) will amaze you with how much you don’t know you don’t know.  See also his widely-used microbenchmark framework [JMH](https://openjdk.java.net/projects/code-tools/jmh/).  For this specific exercise, we can learn much from another hour-long video titled [Java Memory Model Unlearning](https://www.youtube.com/watch?v=TK-7GCCDF_I).   Please enjoy.   

…Okay, well, if you are like me, you found the start of the video a bit intimidating.   However, if you had the patience to get through it, I am sure by the end you appreciated the journey (and the speaker).  Hey, he’s a giant – these topics are indeed challenging!   As a reward for your patience, you almost certainly now know more about this subject than *anyone else* at your company – a newly minted giant yourself.  

So, now we see we must tell java that more than one thread will be updating and reading this variable by using the keyword “volatile”.  With “volatile”, java knows that any changes to “sharedDone” must be instantly published to all cores with that field in their caches.   This is why you may have already found “volatile” in your career if you have searched for “java double checked locking”.  (My favorite Shipilev quote from the video: “If you are not sure where to put volatile, put it freakin’ everywhere!”)   

Well, we know how to fix the problem, but do we really understand how the absence of "volatile" creates an infinite loop?   With our present knowledge, we suspect that sharedDone was in two places in our CPU caches and java did not know to notify the second thread that the first had set sharedDone to true.   Now, to be honest, I am not *positive* that this will loop infinitely.   Perhaps context switching or other reasons would cause core caches to be flushed to main memory and Consumer would eventually grab the updated value.   So, maybe this would be “confusingly, glacially slow” looping rather than “infinite looping”.  Still catastrophic in most situations, but not quite our smoking gun.  Let's keep digging.

## Douglas Hawkins
Well, I guess I must give up and return to the person who proposed the puzzle, our giant **Douglas Hawkins**.   Of his many presentations (watch them all!), my favorite is the 100+ minute presentation [Understanding the Tricks Behind the JIT](https://www.youtube.com/watch?v=oH4_unx8eJQ) where he mentions this specific puzzle. Here, Hawkins gives an expert and entertaining dive into the java just in time (aka JIT) compiler.   I will again pause for you to watch it, including minute 54 that sparked this post.

…All done watching?   So, now you fully understand the final reason the above will loop infinitely.   The code you write, which is compiled to bytecode, is much closer to a “statement of intent” than a specific "mandate" to do exactly what you say.   The debugging experience will fool you into thinking every line is executed as-is, in code order.   However, in production, this is a lie.   

The JIT employs many optimizations as it transforms bytecode to assembler code.   By “optimization” the JIT means “I’ll rewrite your code for optimal performance”.   One of these optimizations is called “loop invariant hoisting” where a value which cannot change in the current thread can be rewritten as a local constant.   If you do not use “volatile”, the JVM assumes that multi-thread situations need not be considered.

So, expanding on the Hawkins video further for this puzzle, we have:  

1. The Consumer thread code can legally be rewritten by the JIT (in the absence of the “volatile” keyword) as the following:  
```java
boolean localDone = sharedDone; // local constant, will never change
while ( localDone == false);   //infinite spin, clearly localDone is never updated.  
print (dataShared).  //never gonna happen 
```
2.  This rewriting is done by the JIT as it transforms bytecode to assembler at run time, if it so chooses to do this optimization.   If sharedDone is false at optimization time, we loop infinitely.  If it is true at optimization time, you would probably throw an exception repeatedly while trying to use uninitialized objects (because the Producer actually has not completed).
3.  When stepping through the debugger, you are stepping over code in “bytecode land” only, not the rewritten assembler.   So, this infinite loop is impossible to reproduce in the debugger, and suddenly you are questioning your career choice.

Terrifying?   Yes, yes I believe so.

## Chris Newland
Our mystery is solved, but the greater lesson here is that the JIT will rewrite your code under high loads for performance, and a "loose understanding" of java coding and memory-model rules can lead to unintended consequences.   A great (surprisingly short) article to see JIT rewriting in detail is Shipilev's first JVM Anatomy Park article [Lock Coursening and Loops](https://shipilev.net/jvm/anatomy-quarks/1-lock-coarsening-for-loops).   But, we can do even better.  

A fourth giant related to this discussion is **Chris Newland** who created the visualization tool [JITWatch](https://github.com/AdoptOpenJDK/jitwatch) for you to see your code, the compiled bytecode, and the JIT assembler all in one glorious screen (and more!).   With this tool you could verify the infinite loop in the assembly.   Enjoy my favorite presentation of his on his tool called [Understanding HotSpot JVM Performance with JITWatch](http://vimeo.com/181925278).

To really show how different assembly can be from the original, and as an excuse to play with JMH and JITWatch, suppose we have a simpler JMH test program with the following:
```java
    @Param("525600")
    long answer;

    static long howDoYouMeasure_MeasureAYear(
            int daylights, int sunsets,
            int midnights, long cupsOfCoffee,
            int inches, int miles, long laughter, short strife){

        int minutes = 525_600;  //FYI 0x80520 in hex
        long result = daylights+sunsets+midnights+cupsOfCoffee;
        result += inches + miles + +laughter + strife;
        result -=result + minutes;
        return result;
    }

    @Benchmark
    @CompilerControl(CompilerControl.Mode.DONT_INLINE)
    public boolean get() {
        long result = howDoYouMeasure_MeasureAYear(daylights, sunsets, midnights, cupsOfCoffee,
                inches, miles, laughter, strife);
        return result == answer;
    }
```
This method always returns 525_600 regardless of input and convoluted logic.   The first time that the JIT compiles this to assembler, it will transform the method "as-is".   In JITWatch, we see this for the get() method: 
![get_c1](/assets/images/2020-09-14-get-c1.png)
Figure 2:  get() with bytecode and assembly.

The code is on the left, the bytecode for the get() method is in the middle, and the assembly on the right.   Note all the assembly for gathering member variable data in preparation for the call, as well as the "call" command itself to invoke the child method.   In fact, this probably matches your expectations fairly well.   Even the variables are in the same order as the code.

The assembly version above is the output of the C1 JIT compiler, which does fast simple-stupid when possible.   The assembly for the called method, howDoYouMeasure_MeasureAYear(), is equally straightforward so I'll skip showing it, but even the pointless additions and subtractions are included.  

After a few seconds, however, the JIT realizes it has CPU cycles available to take another look.  The output of this advanced JIT processor (called the C4) is far more interesting.   The end result is very different from the original java code (and corresponding bytecode).   Here is the new assembly code for get():
![get_c4](/assets/images/2020-09-14-get-c4.png)
Figure 3:  get() with same bytecode and new optimized assembly

Now, if you stare long enough at the assembly, you'll notice several things about this rewrite:
1.  The JIT is no longer bothering to load member variables into memory in preparation for a call. 
2.  The JIT is no longer even calling our child method.  It copied the relevant assembly into get() itself (an optimization called "inlining", as called out in the popup). 
3.  The JIT realized our child method is completely stupid and always returns the same answer.
4.  The JIT has put the answer it always gets from our child method in memory address 0xffffffffffff7fae0 for get() to use.
5.  The JIT assumes this memory address contents will match our member variable "answer".  If ever wrong, it will jump to the "uncommon trap" area of code and call the original child method.

Note too that the bytecode is unchanged.   So, again, if you try to debug the new optimized code of "get()", you'll step through the (unchanged) bytecode and fail to reproduce the exact same behavior.  

## Conclusion
This post was about an infinite loop puzzle, but also much more.   As software engineers, we often search for magic incantations (code snippets) from online communities for our immediate problem at hand, enjoy introductory videos for the latest technologies or concepts, and certainly (hopefully) peruse online manuals for the same.   However, without studying the software engineering "giants", you will only learn to solve the issues that *you know about*.   You will miss learning the solutions to all the unknown issues you have yet to encounter, often interweaving concepts you have yet to come across.  For me, at the time I was first studying Hawkins, this puzzle was just one such example, and Hawkins was just one of several "giants" introducing me to the vast amount I did not know.  Enjoy studying my giants above, or discover your own, and amaze yourself with how much farther you begin to see.

## Postscript
Oh, I did not mention my quote yet – why did I include that?    Just because we can stand on the shoulders of giants does not mean we will not stumble now and then, and that is perfectly okay.   **Carol Dweck** is a giant in the field of empowering people to worry more about achievements and glorious struggles rather than always "appearing smart".   Her concept is called the “Growth Mindset” and it is legitimately awesome.   I love her 10 minute video [How to Help Every Child Fulfil Their Potential](https://youtu.be/Yl9TVbAal5s) as an (animated!) intro as well as her book “The New Psychology of Success”.    Her “Growth Mindset” is a part of Blue Yonder culture, as touted from CEO down, and that culture is one of the many reasons why 20+ years later I still love coming to work here every day.

Credit to song [Seasons of Love](https://en.wikipedia.org/wiki/Seasons_of_Love) for inspiration for my method.  