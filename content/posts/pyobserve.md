---
title: "PyObserve: A Proposal"
date: 2025-04-05
draft: false
ShowToc: true
author: Herocharge
---

I don't know if this already exists but it just popped in my head like "Yeah, that would be cool if it existed."

Here's the problem. Whenever I write code in python notebooks, I forget to save the variables that I might wanna analyze later. I forget to use something like like `tqdm` to print a progress bar, so that I know how long the cell will run.  

Sometimes I don't forget and save most of things which I think I need, but later realize that I might want to see what the other variables were storing?  

For example, say I'm running an ML model, I might save some of the intermediate activations using hooks but I might forget to save other data that I might wanna analyze later. Hmmm, so what do we do?

### Can we pause execution?
Python is an intepreted language right? Each line is executed one by one (It's not JIT compiled (unless Im using some opt library, numba, etc)). We can probably just stop at a particular line, stop the interpreter, keep the state and resume when needed. Does pdb do this?

Yes, there a lot of problems with threads, etc. But we can probably agree on a fundamental block to pause on.


### Hook code
Once we pause the execution, it would nice if we can write extra code that can use live variables in the code and that executes in parallel to the actual code, once I resume the execution. This way I can add code which can save variables, or add a progress bar.

We know where the variables live, it should'nt be hard to add this functionality on a separate thread.

But what about the state that we lost before we paused?
Should we store states periodically?  This might take up too much storage and resources, and slow down execution.
