---
layout: post
title: Dead Code is Like Gangrene
excerpt: There's a misconception that dead code causes no harm. Since it's not being used then it has no effect, positive or negative.. or so the logic goes. Wrong! not only is it a serious condition but can be fatal, just like Gangrene.
author: yoni_baciu
---

There's a misconception that dead code causes no harm. Since it's not being used then it has no effect, positive or negative.. or so the logic goes. Wrong! not only is it a serious condition but can be fatal, just like [Gangrene](https://en.wikipedia.org/wiki/Gangrene).

Some may argue that it's good to keep things around for "reference". Some are nice enough to comment out dead code. Don't do any of that! It may serve some very short-term purpose but is almost always quickly forgotten and remains in the codebase indefinitely.  
In reality, nothing we do is ever truly deleted anyway. Git history will save it all so there is no good reason to keep dead code at the tip of your branch.

The most adverse effect dead code has is **negative cognitive load**:
- Greater fear to perform large refactoring when you are not sure whether a piece of code is used or not.
- It's natural for a system to get complicated over time but being complicated due to dead code requires developers to keep more in their brains' 'RAM' and use more 'CPU' for no reason. 
- Onboarding new developers takes more time since there is more explaining to do. It also teaches them a bad habit.
- Team velocity decreases when there is more (dead) code to read and understand.
- General confusion - "If the code is there then it must be for a good reason, right? I'm not sure.. I will just leave it alone"

Treat dead code like a healthy organism treats dead cells - reject it immediately. Too much of it can be fatal!

Next in the series - "Dead features are like Gangrene" and "Barely used features are also like Gangrene"
