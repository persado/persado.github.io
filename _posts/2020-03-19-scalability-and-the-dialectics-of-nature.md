---
layout: post
title: Parallels between system scalability and the Dialectics of Nature
excerpt: Parallels between system scalability and the second 'law' proposed in Engel's work - The Dialectics of Nature
author: yoni_baciu
---

> Dialectics of Nature is an unfinished 1883 work by Friedrich Engels that applies Marxist ideas – particularly those of dialectical materialism – to science.

One of the "laws" proposed in the Dialectics of Nature is the "law of the transformation of quantity into quality and vice versa". This "law" may be traced to as far back as the ancient Ionian philosophers, Anaximenes and Eubulides of Miletus. Eubulides was known for his paradoxes, one of them being the Paradox of the Heap: 
A heap of sand minus one grain is still a heap. If one removes all grains till only one remains, is it still a heap of sand? At what point does the heap stop being a heap?

One of the most common examples of the change of quantity into quality is the transformation of water. Take water at a temperature of say minus 50c - It is a block of ice. Raising the temperature up to 0c, a change in temperature 'quantity', will make the block of ice warmer but it will still be a block of ice. But if we keep raising the temperature, a phase transition will happen and the block of ice will turn into a puddle of water - a change in 'quality'. If we continue with this exercise and keep raising the temperature, the water will keep getting warmer until another phase transition will happen and the water will become gas at 100c, yet another change in ‘quality’.

## So how is all this related to system scalability?

Take a system running on two app servers and a simple relational database. This setup can handle a certain load of users just fine but what happens when the load of users increases (heat 'quantity' rises)? At some point the two app servers are not enough to handle the load so we augment them with stronger CPUs and memory and when we max those out we start adding more app servers as the load keeps increasing. All these changes do not constitute a change in 'quality' as the system makeup is still the same.

After a while, though,  we reach a point where we cannot simply boost the current resources. Our database now has tables with millions of records, updates are slow and running reports with complex joins take forever..
A change in 'quality' is now in order - To keep up with the ever increasing load, we decide to incorporate new technologies, say Elastic Search or Amazon Redshift, to relieve some of the stress on the relational database. This proves to be a robust long term solution but nevertheless should not be mistaken for a permanent one as long as the load keeps increasing and more qualitative transitions will inevitably need to be applied.

Similar to water transitioning between states, at certain points in the ever increasing demand on our system, certain qualitative transitions will need to happen in order to keep the system functioning. The main difference being that water passively undergoes qualitative transitions as the temperature rises while in the case of a software system, it's up to the engineers to prepare for and apply the inevitable changes in quality as pressure (heat) on the system mounts.
<br/>
<br/>
<br/>
**Further reading & Definitions:**

Dialectic - [https://en.wikipedia.org/wiki/Dialectic](https://en.wikipedia.org/wiki/Dialectic)  
Dialectical Materialism - [https://en.wikipedia.org/wiki/Dialectical_materialism](https://en.wikipedia.org/wiki/Dialectical_materialism)  
Dialectics of Nature - [https://en.wikipedia.org/wiki/Dialectics_of_Nature](https://en.wikipedia.org/wiki/Dialectics_of_Nature)  
Sorites Paradox (Heap Paradox) - [https://en.wikipedia.org/wiki/Sorites_paradox](https://en.wikipedia.org/wiki/Sorites_paradox)  
Scalability - [https://en.wikipedia.org/wiki/Scalability](https://en.wikipedia.org/wiki/Scalability) - This definition of scalability is somewhat lacking in my humble opinion as it implies that one can simply add resources in a linear fashion without mentioning breaking points where a transition in quality must occur.
