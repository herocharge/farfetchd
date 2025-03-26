---
title: "where tf is the print statement"
date: 2025-03-26
draft: false
ShowToc: true
author: Herocharge
---


I was revisting one of my projects from a long time ago. It was written in python and I wanted to quickly run and see the output. But once I run it I am immediately bombarded with a tonne of "debug print" statements. Well.... I don't use debuggers for python.  

I commented out all the debug prints. Luckily I always include a "\[HERO\]" tag in my debug prints (I should've used loggers instead). Anyway, I ran the code again, and.... there is still a debug print remaining. I can see it in the output. I went through the code again and again but couldnt find it.  

Then it hit me, I must've edited one of the library codes. Pip, fantastically enough, downloads the source code as long as it is in python so if I want to really deeply debug something I can always add a print statement inside that source code. I usually refrain from this because it would affect anything using that environment and I can't predict where that code is being called internally.  

But looks like that's the only explanation. Now I don't exactly remember where I added this print statement and there are 100s of library files to look at, what do I do?

I know that the print happens in before any other print to stdout. I can use this information!
Python, it turns out, let's you change what needs to happen when something prints to stdout

So I did this  

```py
import sys

def exit_on_print(message):
    raise Exception("Exiting program due to print statement")

sys.stdout.write = exit_on_print
```

Now everytime someone tries to write to stdout, this function is called! and since I raise an exception, I get the stacktrace! And the stacktrace will contain the file name and line number of the print statement which tried to write to stdout.

confidence++

bye
