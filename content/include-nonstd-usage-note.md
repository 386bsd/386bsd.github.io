----
Title: Include 'nonstd' Usage Note
Date: 2019-01-15 12:46
Modified: 2019-01-15 13:18
Category: note
Tags: first, 386bsd, tale
Slug: include-nonstd-usage-note
Authors: William Jolitz
Summary: Documenting point of the 1.0+ /usr/include/nonstd "tree" for access to nonstandard include files to not overlay standard ones.

In the rush that nearly every BSD release entailed, things get forgotten. With 1.0, much was going in and much was going out. I'd found that keeping track of all the shibboliths present, in both catagories, was a major nusaince.
It also didn't help that for various reasons things were being dragged off in different directions in the basic include files, where whole trees of them were being retained simply to keep coherence on each of these.

On top of that, one of the points of 1.0 was to seperate out stuff so that old/new could be factored out/in smoothly w/o messing up standards coherence. So part of this was /usr/include/nonstd. One place to collect up these differences.

The basic idea was to keep pathnames solely in build tools, and just allow include/interface files to be embedded in code, where they would be satisfied by include paths directed through nonstd, ahead of other/primary includes.

Put back in the README in /usr/include/nonstd to document this (normally there are no README's in the include hierarchy).

It's not a perfect means of doing this, because of name collisions. But it was "good enough" for the expiedent of getting on with the shuffle of old out, newer in place, then what was desired to replace it refactored in.

Originally, 386bsd started just as a means to keep alive a bit rotted BSD only running on obsolete VAX/CCI Power machines, thus had a lot of cruft that still persists to this day in all other BSDs. The "0." releases were labeled such because they were a holding pattern, where the intent was for the discord of the moment to wane, and people would put things back into "one BSD" afterward. There wasn't supposed to be a "1.0" at all.

(There was a "0.2", as well as "1.n" (and a 2.0). 0.2 got nixed because of increased discord of just about every institution on the planet (0.1 got too successful, others wanted own *bsd, those "helping" didn't like where it was heading so "slow walked" things). The magazine who commissioned 1.0 was scrupluous about avoiding conflicts, with a yanked commitment to follow-on/support while the next changes were under test - so no "1.n" - they just issued the one CDROM without anything else, which was frustrating.)

In general it got hard to fight all the "no's" when there wasn't any "yes's".

So coincident with 1.0 was the need to reflect all the other standards without embedding any of them, because 386bsd wasn't going to follow any of those directions. Thus the need for "nonstd". Anything that didn't fit with the "core" was "compartmentalized" so as to draw a distinction/"bright line".

Is it a good idea on its own? Unsure. Since it was never noticed and thus didn't get any traction, there wasn't any feedback whatsoever. It'll continue with the ".x" release because it still serves its purpose.




