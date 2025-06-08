---
layout: post
title: What's Next For cocotb
---

I suppose it's time for a state of the union, at least with respect to my own involvement in the cocotb project.

## History of cocotb

cocotb started in 2013 as a part of a startup by the original authors (Chris Higgs and Stuart Hodgson).
They worked on it until after the dissolution of the venture (date unknown) and a bit longer into 2016.
During that time they released the 1.0 version.
Then there was a period of no activity, prompting the popular issue originally posted by Patrick Lehmman ["Is Cocotb dead?"](https://github.com/cocotb/cocotb/issues/513).
But it was not dead yet...

In late 2018 the project was taken over by FOSSi, under the stead of Philipp Wagner.
He added the first new maintainers: Tomasz Hemperek, Colin Marquardt, and Eric Weiser.
Finally adding me and Marlon James in 2019 and 2020, respectively.
Since then, the cocotb maintainers have done 9 major releases in 7 years, with a tenth (2.0) looming.

The new ownership brought a new development/maintenance system: the shared maintainer model,
where the maintainers share responsibility for reviewing and merging contributions from other users rather than directly developing the code.
This was fine as the project had reached the limit of its scope,
it had hit 1.0,
it just needed bug fixes, documentation improvements, etc.
"Maintenance."

Not to speak ill of the original authors,
they had great a great idea and developed a fairly complete prototype to prove that idea worked.
But as I said, they developed a prototype.
The new maintainers have spent the last 7 years fixing bugs and refactoring the code to increase the code quality, usability, and re-usability of the code,
all while trying not to break the ever-increasing user base.

We are very much trying to be a mature project that doesn't need to massive rewrites that inadvertently break things.
2.0 is well over-due in that regard.
I've wanted to work on an API breaking change for years now,
but haven't been in a good enough place to accomplish that,
nor have the other maintainers.

But now the future looks bright. Well, almost...

## The Future Of cocotb

If you asked me "Is cocotb the future?" I'd say "No. cocotb is ***now***".

"But will cocotb also be the future, say, 10 years from now?" "Almost certainly not in its current form."

It already fails to do what many users need it to do.
People need performance and that comes in the form of moving more into the simulator with DPI and RTL Drivers and Monitors.
People need reuse of existing testbench infrastructure, such as UVM agents and TLM-based Drivers.
People need better reusability for with hardware testing.
People need a more featureful regression system.
And while all of that is currently *possible*, it isn't readily available.
And cocotb is not "developing" any more.

cocotb will someday die, but it will live on, and not just in memory, if it's up to me.
This will be accomplished by making the cocotb codebase more extensible and modular.

## Against Prevailing Sentiments

EDA tooling hasn't quite escaped the monolithic design style that is often valuable in commercial tools
("please become dependent on our walled garden!").
Even some FOSS EDA tools fail to escape that mental box.
But open source __software__ hasn't operated in that way in decades.

Modern open source software is built to do one thing and do it well.
And it does this by reusing and extending existing libraries.
Tools fall out of favor,
but the useful pieces will live on and mature.
Over time the useful pieces will ossify and become "core libraries/tools".
That is how I see cocotb surviving into the next decade.

There are useful reusable pieces of cocotb like
the GPI and PyGPI in any other Python-based cosimulation framework,
and the scheduler, tasks, triggers, etc. in shared HW/simulation testing.
And there are less useful pieces, like the Makefiles and Python runner that most serious cocotb users have already ditched.

However the useful pieces aren't in a state to be reusable right now.
Putting cocotb into that state will be the focus of my efforts on cocotb going forward.
I would hate to see useful software and tens of thousands of hours of development time go to waste.

## Change In Attitude

As I mentioned cocotb uses a shared maintainer model, which is best fit for a post-main-development project.
However, if you ask me the cocotb of ***now*** still has a lot of improvement potential.
To me, how cocotb is maintained and and what I'd like to do with it don't really align.
So I decided early last week that I was going to change how I was going to work on cocotb.

I started [`coconext`](https://github.com/ktbarrett/coconext) with the idea that it would be where I would develop new cocotb features;
where "massive rewrites that cause inadvertent changes" that are necessary for new development would be done.
All while letting the cocotb repo do what it should be doing and maturing.

Of course I will need to do changes to cocotb still, but I will do so even more judiciously than I already have been.
My current modus operandi has been to focus on refactors, bugs fixes, and new features to create improvements over existing functionality.
But I will no longer focus on developing new features in the cocotb repo;
that will be reserved for `coconext`.
The majority of my work going forward will be the necessary changes to cocotb to support features in `coconext`,
as well as refactoring changes to make cocotb more extensible and modular
(plus the obvious bugfixes, documentation improvements, etc.).
