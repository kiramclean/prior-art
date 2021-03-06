---
date: 2021-01-30
title: How state makes software complex
site/tags:
    - complexity
    - state
---

What makes large codebases so hard to reason about, and so hard to extend? I really want to find out because I'm tired of losing my mind working on them. One paper I love called "[Out of the Tar Pit](http://curtclifton.net/papers/MoseleyMarks06a.pdf)", by Ben Moseley and Peter Marks, argues that state is the main source of complexity in large software systems. This article is an overview of a couple of the ideas from that paper, along with some discussion about how we might conceptually think about necessary state in software in order to improve its overall quality.

## How does state cause complexity?

The main issue with maintaining state in a program is that it quickly leads to a proliferation of different internal conditions that can affect the output. The more state there is, the more possible combinations it can be in, the harder it is to predict and wrap our heads around what the whole system is going to do with a given input. Every new piece of state introduced into a system dramatically increases the total number of possible states the whole thing can have, and this number quickly exceeds the capacity of our brains to conceptualize it all at once, which leads us to write code that behaves in unexpected ways. Bugs!

When the output of a program depends not only on its inputs, but also on its _current internal state_, it is inherently unreliable because we cannot guarantee that the inputs we're giving it will always produce the same result. This is how we end up with systems that behave in unexpected ways. Most programs need at least some state, though, so how can we deal with it?

## Hiding state does not count as dealing with it

One classical approach to managing state is encapsulation -- breaking state down into small pieces and putting different parts inside different classes or objects. As Rich Hickey says, though, putting the mess into a container does not fix the problem. It just means [you're in charge of that mess now](https://www.youtube.com/watch?v=dGVqrGmwOAw&t=1322).

Remember the fundamental problem causing complexity is that stateful programs have too many possible internal states which affect their output in ways that are opaque and therefore difficult to understand. Separating the state into pieces and regulating access to it through methods doesn't solve that. There is nothing to stop multiple methods from accessing the same bit of state, or even multiple different objects calling those methods and manipulating the internal state of some other object. With this approach, the behaviour of the program as a whole still ultimately depends on the current internal state of its components.

Most languages do support things like private methods and attributes and encourage best practices like [loose coupling](https://en.wikipedia.org/wiki/Law_of_Demeter), but as Murphy's law says, "anything that can go wrong will". Whatever ways there are in a given language to manipulate the internal state of objects will eventually be used. Good practices are never enough to save us from ourselves.

## Ignoring state does not count as dealing with it

Another classical approach to managing state is basically to reject it -- insisting that useful programs can be modeled as pure (mathematical) functions, i.e. ones where the only thing affecting the output is the input. This is a nice way to think about computation in many ways, but at the end of the day it's not pragmatic. It ultimately just kicks the can down the road.

The vast majority of programs/systems/apps exist to be used by people, and those people are not all identical, so many of them want to use those systems in subtly different ways. This is one reason why most programs need to deal with at least some state.

One typical approach to managing user input or other essential state in programs made mostly (or only) of functions is to add arguments to all the functions that would normally store or access the internal state of the program, effectively passing some initial state to the entry point of the system and threading that state all the way through.

In many such programs these extra arguments end up being large compound values, acting effectively as a single pool of global variables, which in the end can be just as confusing. This is still a huge improvement over the encapsulation approach because at least each _individual_ function can be more easily understood in isolation. It is ultimately simpler to understand something when all of its inputs are explicit and transparent. But, if one of those inputs is a huge map of stuff where only a slice is actually relevant, we're not a whole lot better off in the end.

## Hidden, implicit, mutable state is a major source of complexity

These two approaches to dealing with state reveal some characteristics of the kinds of state that are fundamentally bad to have in a system (because they increase its complexity, which makes it harder to reason about, which makes it less reliable).

- **Hidden (or implicit) state** makes programs more complex because we have to do extra work to figure out what the output is going to be. The mental overhead of figuring out how this hidden state affects the output can quickly become overwhelming and make systems impossible to understand.
- **Mutable state** makes programs more complex because we cannot guarantee the state will be what we expect at any given time. We cannot trust best practices alone to isolate state mutation to only the right places, so we can never be certain what the values of mutable properties are going to be. This also means we have to pause the universe in order to read mutable state, which fundamentally breaks out model time in programs that have it. But that's a topic for another day.

## Make state explicit and immutable to make less complex software

There are obviously loads of things that make programs complex. But state is a big one, specifically hidden, implicit, mutable state.

In any system where the outcome of running some code depends on the internal state of the system, you _have to know_ the state of that system _at the moment_ the code runs in order to know what the output will be. And the more parts of a system you have to pull in to your mental model just to understand how one small piece works, the harder it is to write code that works.

To write less complex (and therefore more reliable) software, make implicit state explicit and use immutable data structures. This minimizes the number of things you have to account for when looking at a single method/function, which makes it easier to write better software.

I daydream about a world where most software is written in [languages](https://en.wikipedia.org/wiki/List_of_programming_languages_by_type#Functional_languages) that enforce these constraints by default, but in the meantime there are libraries in many popular programming languages (like [JavaScript](https://gist.github.com/jlongster/bce43d9be633da55053f), [Java](http://www.functionaljava.org/), and [Python](https://pypi.org/project/immutables/)) that can help.
