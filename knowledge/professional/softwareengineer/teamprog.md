---
layout: master
title: Programming in a Project Team
---

## Big Project 

Big projects require more than one person (or long, long, long time)

Time/work estimation is hard

Can a project be efficiently partitioned?

- Partitionable task decreases in time asyou add people
- But, if you require communication:
- Mythical person-month problem:

## Partitioning Tasks

### Functional

- Person A implements threads, Person B implements semaphores, Person C implements locks…
- Problem: Lots of communication across APIs
If B changes the API, A may need to make changes

### Task
Person A designs, Person B writes code, Person C tests
May be difficult to find right balance, but can focus on each person’s strengths (Theory vs systems hacker)


## Communication

More people mean more communication, Changes have to be propagated to more people

Miscommunication is common

Who makes decisions?

- Individual decisions are fast but trouble
- Group decisions take time
- Centralized decisions require a big picture view (someone who can be the “system architect”)

Often designating someone as the system architect can be a good thing

- Better not be clueless
- Better have good people skills
- Better let other people do work 

## Coordination

More people = no one can make all meetings!

They miss decisions and associated discussion

What about project slippage?

Hard to add people to existing group

People are human.  Get over it.

People will make mistakes, miss meetings, miss deadlines, etc.  You need to live with it and adapt

It is better to anticipate problems than clean up afterwards. 

**Document, document, document**

Why Document?

- Expose decisions and communicate to others
- Easier to spot mistakes early
- Easier to estimate progress

What to document?

Everything (but don’t overwhelm people or no one will read)

- Project objectives: goals, constraints, and priorities
- Specifications: the manual plus performance specs
- Meeting notes
- Schedule: What is your anticipated timing?
- Organizational Chart


Standardize!

One programming format: variable naming conventions, tab indents,etc.

Comments (Requires, effects, modifies)—javadoc?

