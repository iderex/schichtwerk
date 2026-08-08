# The implantation interface, and what this board asks of the bremsweg board

Decided by issue #41, milestone 6.

## Status

In force from the commit that lands this file.

The naming and numbering of decision records is fixed by issue #2, which is not
decided. This filename is provisional and #2 may change it. Nothing in the
content below depends on the name.

Every fact about the other board was derived on 2026-08-08 against its head at
that time:

    gh api repos/iderex/bremsweg/commits/HEAD --jq '.sha'
    e4b042bca1b32148cf2944c1094869035c3defea

and every fact about this tree against
`03922d9213a108e6f1b37d5bf691a9635bad8534` on `main`. Both move. The commands
are written next to the claims so a later reader re-derives rather than trusts.

## Why this is a decision rather than an implementation detail

The front page of this project says the implantation part is somebody else's
board and that this one depends on it rather than duplicating it. That sentence
is either a tested dependency or a claim nobody checked, and until it is tested
it shapes an entire milestone on the strength of an assumption.

Issue #41 says the same thing in its own words and asks for the contract to be
written in terms of what this project needs rather than in terms of what that
project happens to produce. Written the other way round, a contract is a
description of the producer and every gap in it is invisible, because a
description cannot be short of anything.

So the contract comes first, the test against the other board's accepted records
comes second, and the result of that test is written into this record rather
than kept as a surprise for milestone 6.

## The contract

What this board needs from an implantation calculation, stated from this side.

A depth distribution of the implanted species, for a given ion, energy, dose,
tilt and target, computed on the structure this board holds rather than on a
semi-infinite slab. A real implant goes through a screen oxide and past a mask
edge, and a distribution computed for bulk silicon is a distribution for a case
nobody runs.

A depth distribution of the damage, in the form issue #44 turns into an initial
condition for the point defect equations. This is a different quantity from the
ion distribution, and a producer that returns only the ion profile has left out
the half that decides what the following anneal does.

Enough information to know how good the answer is. The statistical uncertainty
where the method is stochastic, and a statement of what the model assumed about
the target.

Determinism consistent with the numeric contract in issue #7, which for a Monte
Carlo method means the seed is part of the input and is recorded in the manifest
per issue #10.

A fifth item, which issue #41 does not list and which the test below shows it
needs. The dose regime the answer is valid in. An implant heavy enough to change
the target while the beam is still running is ordinary work here rather than an
edge case, and a per ion answer multiplied by a fluence is a different quantity
from a dose dependent one. Adding it as an item of the contract rather than as a
remark, because the failure found against it is the same size as the other two.

## Testing the dependency, item by item

Nothing below is a judgement that the other board is wrong. Each of these is a
limit it states about itself, in a record whose declared purpose is to state
limits before the code exists:

    gh api repos/iderex/bremsweg/contents/docs/decisions/0004-the-target-model-and-what-it-rules-out.md \
      --jq '.content' | base64 -d | sed -n '4p;35p;63p;86p;105p'
    Status: accepted
    The class of question this makes the program unsuitable for is implantation into
    The class of question this makes the program unsuitable for is anything that
    The class of question this makes the program unsuitable for is high fluence: any
    A crystalline mode is not planned and this record does not commit to one. What

### The depth distribution: fails for a crystalline target

Line 35 continues that the class it is unsuitable for is implantation into a
single crystal, which that record names as most of semiconductor processing. The
target there is amorphous by construction, so there is no crystal direction and
no channelling, and the record states the observable and its direction: the
profile is too shallow, and it is missing its tail entirely rather than having
one that is slightly wrong.

That is the first item of the contract, and it is the item this board can take
no substitute for. A channelling tail is not a refinement here. It is the
observable the validation record already singles out as the one its shape metric
exists to catch:

    git grep -n 'channelling tail that is a decade low' -- docs/
    docs/decisions/0012-what-validation-means.md:90:else. This is the number that sees a channelling tail that is a decade low while

So a profile from that board, used for an implant into silicon, is wrong in
exactly the way this board has already decided to measure, and by an amount that
record calls a decade rather than a correction.

Line 105 is what makes this a finding rather than a sequencing note. A
crystalline mode is not planned there, and no issue on that board holds one:

    gh issue list --repo iderex/bremsweg --state all --limit 300 \
      --search "channelling in:title,body" --json number,state,title \
      --jq '.[] | "\(.number) \(.state) \(.title)"'
    4 CLOSED Decide the target model: amorphous binary collisions, and what that rules out

    gh issue list --repo iderex/bremsweg --state all --limit 300 \
      --search "crystalline in:title,body" --json number,state,title \
      --jq '.[] | "\(.number) \(.state) \(.title)"'
    75 OPEN Sputtering yield and surface binding, or a stated exclusion
    4 CLOSED Decide the target model: amorphous binary collisions, and what that rules out

The only issue matching the first is the closed one that ruled it out, and the
one the second adds is about a surface, not about an ordered target.

### The damage distribution: fails in meaning rather than in form

Line 63 names the class as anything depending on what a cascade leaves behind
rather than on how much energy it deposited, and lists the surviving defect
population, defect clustering, and any comparison with a measurement of stable
damage. The stated direction is that a displacement count from chained
independent collisions is too high, and the discrepancy grows with recoil
energy.

What that board will emit is its own separate record, and it is right:

    gh api repos/iderex/bremsweg/contents/docs/decisions/0008-no-displacement-number-without-its-model.md \
      --jq '.content' | base64 -d | sed -n '9p'
    This program never emits a quantity called dpa.

Named quantities, each carrying the model that produced it, rather than one
number whose model has been forgotten. That policy is not the difficulty. The
difficulty is that a named displacement measure is not a surviving defect
population, and the record above puts the step between them outside the model.

Issue #44 needs what survived, and issue #45 needs its clustering. Both are
named in the list line 63 excludes. So the form of the number arrives and its
meaning does not, and whatever converts one into the other is physics this board
owns.

### The dose regime: fails, and both boards exclude it from opposite sides

Line 86 names high fluence as outside that model. The target does not change
while a run proceeds, so there is no accumulated amorphisation and no surface
recession, and the record says the program computes a per ion result, that
multiplying it by a fluence is the operator's step, and that it is linear by
construction.

This board excludes the same regime from the other direction:

    git grep -n -A1 'Amorphisation and solid phase epitaxial regrowth' -- docs/
    docs/decisions/0008-diffusion-model-family.md:124:Amorphisation and solid phase epitaxial regrowth. There is no amorphous phase in
    docs/decisions/0008-diffusion-model-family.md-125:the model, so an implant heavy enough to amorphise is annealed as though the

Two boards, one regime, excluded twice and stated nowhere in one place until
here. That is worth having written down before a recipe with a high dose implant
runs and produces a number that looks ordinary.

### Uncertainty: met, and by more than was asked

That board reports statistical uncertainty per tally in every run, names the
estimator each tally used, keeps coefficient uncertainty separate from it, and
declines to put a number on model uncertainty on the grounds that computing one
would require knowing the answer the approximations are wrong about. The record
is `0010-how-uncertainty-travels-to-a-reported-number.md` in the same directory
as the one quoted above.

Two of those three are more than the third item of the contract asked for. The
contract asked for the statistical uncertainty and a statement of the model's
assumptions; what arrives additionally is the separation of coefficient
uncertainty from statistical, which matters here because a sweep over twenty
energies shares the coefficient part and does not share the statistical part.

### Determinism: met, and stronger than required

Random numbers there come from a counter based generator with one stream per
history, derived from the run seed and the history index, so a history consumes
the same numbers whatever thread ran it and however many threads there were. The
record is `0006-determinism-and-the-random-number-contract.md`.

The position here is bit reproducibility at a fixed thread count. A per history
stream does not depend on the thread count at all, so it satisfies the fourth
item with room to spare, and the seed being part of the input is what issue #10
already records.

## What this board does where the contract fails

### The crystalline implant

The route is not settled here, and saying which route is authoritative for an
implant into silicon would settle it in the wrong document.

What is settled is the placement of the failure, by the rule in the error and
failure policy rather than by preference. A profile computed for an amorphous
target and used for an implant into a single crystal is a number a reader would
mistake for one computed the way they asked, and the first question of that rule
makes that a refusal rather than a recorded substitution. So a run that would
produce one stops and says so, naming the target and the method, and the tie the
policy describes goes the same way: a refusal costs an annoyed user who opens an
issue, and a substitution here costs a junction depth that is too shallow by a
decade in a report somebody has already published.

What is open, and named rather than left to be discovered. Issue #42 is the
analytic route, and its own text says the model has to be able to express a
channelling tail. That inverts the accuracy ordering both issues are written
around, where the Monte Carlo path is the expensive accurate one and the
analytic path is the cheap approximation. For a crystalline target on this
evidence the cheaper path is the one that can express the observable and the
expensive one is the one that cannot. Which route serves that case, and what
that does to the sequencing of milestone 6, is a planning question for the
tracker rather than something this record takes.

Nothing in this record assumes an answer to it. The contract, the connection
below and the placement above hold under either.

### The damage

The interface carries deposited energy and named displacement measures, each
with the model that produced it, and never a bare displacement number. That is
what the producer emits and this board does not ask it for anything else.

The conversion from a named displacement measure to a surviving interstitial and
vacancy population is this board's physics, and issue #44 is where it is named,
cited and given a coefficient with provenance under issue #11. Recording it here
so that the step is visible as a step: reading a displacement measure into a
defect field without one is the failure this whole section exists to prevent,
and it is a failure that produces a plausible profile.

### The dose

Above the amorphisation threshold the tool refuses, which is issue #44's own
done-when and is consistent with the placement rule above.

Below it, the per ion result and the linearity of multiplying it by a fluence
are an assumption of the producer rather than of this board, so the manifest
records the producer, its version and the dose the linear step used, in the
sense issue
#10 fixes for a value that came from outside.

## The connection

Issue #41 offers three shapes and asks for one to be chosen with the choice
argued. The choice is a subprocess with a defined file interface.

One of the three is unavailable today rather than rejected on the merits, and
that belongs against the choice rather than being discovered when a build file
is first written. Neither tree carries terms:

    gh api repos/iderex/bremsweg --jq '.license.spdx_id // "none"'
    none
    gh api repos/iderex/schichtwerk --jq '.license.spdx_id // "none"'
    none

so a library dependency has nothing to be granted under. Entry 1 of issue #1 is
where that changes and no answer to it is assumed here. If it is answered so
that linking becomes available, the argument below still stands on its own and
this record says which parts of it were licensing and which were not.

The licensing part is one sentence and it is the weakest of the three reasons.

The determinism part is stronger. A subprocess boundary is a place where the
whole input document, the seed and the whole output document can be recorded in
the manifest as artefacts, which is what issue #10 already requires of anything
that reaches a result. A library call records what the caller remembered to
record.

The replaceability part is the one the test above made load bearing. This board
now knows it will have more than one implantation route, because the one it
planned to depend on cannot do the crystalline case and the analytic route in
issue #42 is a second producer of the same quantity. A file interface lets a
second producer be substituted without a build change, and it lets the two be
run on the same case and compared, which is issue #42's fifth done-when item.

The cached data path is not a third shape and is not chosen against. It is an
addition on top of the file interface, keyed on the content hash of the input
document together with the producer and its version, and a cache hit is recorded
in the manifest in the same way a value that came from outside is. Adding it
later changes nothing above it, which is the reason it is not decided now.

## What crosses the boundary

Two documents in each direction, and the split between them is the one issue #13
and issue #10 already make for the same reason: structure in a document a schema
can refuse, bulk in a numeric table.

Into the producer: the ion, the energy, the dose, the tilt, the seed, and the
target as the layer stack this board holds at that step rather than as a bulk
material. Every physical quantity carries a value and a unit, both required, per
issue #9, and nothing in this direction is a bare number.

Out of it: a depth distribution per species on a stated depth grid, deposited
energy against depth, the named displacement measures, the statistical
uncertainty per tally with the estimator named, and the producer's identity and
version. The distributions are the numeric table. Everything else is the
document.

What is not fixed here is the spelling of either document, because that is a
negotiation with a producer that has no code yet and a fixed spelling written
from one side is a spelling the other side meets as a constraint rather than as
an agreement. What is fixed is the split, the requirement that every quantity
carries its unit, and that the output identifies the model that produced each
displacement measure.

## What this record does not decide

Which route serves an implant into a crystalline target. Issue #42 is the
analytic route and the planning question above is on the tracker rather than
here.

The conversion from a displacement measure to a surviving defect population.
Issue #44 holds it, and this record only says the step exists and is this
board's.

The licence of either tree. Entry 1 of issue #1, and the connection above is
argued so that its answer changes one sentence rather than the choice.

Whether the other board ever gains a crystalline mode. That is its board's
decision and its accepted record already declines to commit to one.

## What is enforced today

Nothing. There is no implant step, no producer, no manifest and nothing that
reads or writes either document:

    git ls-files -- '*.c' '*.h' '*.cpp' '*.hpp' '*.rs' 'CMakeLists.txt' | wc -l
    0

Every rule here is followed by a person until issue #2 names a language and
issue
#16 supplies something to run. The three failures above are not waiting on that,
because each of them is a limit an accepted record states about a model, and
writing the code as specified on either board would produce them rather than
remove them.

## The means

Markdown in the repository, read by a person and quotable in an issue. A
decision record has to be readable from a fresh clone with nothing installed, it
has to sit in version control so that what was decided and when is recoverable
from git rather than from memory, and it has to survive being disagreed with on
the merits. It adds no language, no runtime and no dependency to a tree that
today holds no build. Nothing outside this repository forces the choice.

The means for the interface the record specifies is the separate question
answered above: a subprocess and two files, because the boundary has to be
recordable in a manifest and replaceable by a second producer, and because the
tighter shape has no terms to be granted under today.
