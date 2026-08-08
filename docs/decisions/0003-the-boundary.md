# The boundary: what this simulates, and what it hands to somebody else

Decided by issue #3, milestone 1.

## Status

In force from the commit that lands this file.

The naming and numbering of decision records is fixed by issue #2, which is not
decided. This filename is provisional and #2 may change it. Nothing in the
content below depends on the name.

## Why this is a decision rather than an implementation detail

Naming five things, implantation, diffusion, oxidation, deposition and etch
topography, describes a field. It does not say where the field ends, and a
project without that line spends its second year arguing one issue at a time
about whether lithography, stress, device simulation, layout extraction and
equipment modelling are in.

The argument is expensive in a specific way rather than in general. The boundary
decides the shape of the data structures. A project that will one day compute
mechanical stress needs a place for a displacement or a stress field on the mesh
from the beginning. A project that will never simulate a device needs no mobility
model and does need an export another tool can read, which is a different
obligation on the same layer. Getting this wrong is not a document that needs
rewriting. It is the structure layer in issue #5 that needs rewriting, and every
field and every solver written against it.

So the line is drawn before the layer is built, and it is drawn in both
directions, because a boundary that only says what is in is a boundary that
grows.

## What is inside

The front end of the process flow, at the wafer cross section, from a recipe.

Concretely, the evolution of three things under the steps a recipe names: the
geometry, meaning where the material is; the materials themselves, meaning which
region is what and how many regions there are; and the fields, meaning the
impurity and defect concentrations that live on the geometry. The steps that
evolve them are implantation, thermal annealing, oxidation, deposition and
etching.

The output is a structure with fields on it, in a form another tool can consume.
That last clause is part of what is inside rather than a courtesy at the edge of
it, and issue #13 is what makes it a deliverable.

Two things about the phrase "wafer cross section" are load bearing. It is a
cross section, so the lateral extent is bounded by what the recipe describes and
not by a wafer. And it is the wafer rather than the reactor, which is the
exclusion below that most often gets argued.

## What is outside

Each exclusion carries one reason and then what a user gets instead. The three
shapes the alternative can take are an input the recipe supplies, an export
another tool consumes, or nothing, and the third is written as nothing rather
than dressed up.

### Device simulation

Out, because it is solved in the open and the honest job here is to feed those
tools rather than to compete with them. Issue #3 names DEVSIM, Solcore, Genius
and Charon as what exists; that they exist is the reason, and no claim is made
here about what any of them can do, which is issue #13's to establish from their
own documentation.

Instead: an export another tool consumes. Issue #13 decides what crosses,
issue #53 builds it, and issue #54 runs one flow end to end through somebody
else's simulator. The handoff being a first class deliverable is a direct
consequence of this line, not an unrelated nicety.

### Lithography

Out, because aerial image simulation and resist development are their own field
with their own open tools, and what this project needs from them is a pattern
rather than a model.

Instead: an input the recipe supplies. A masking step names the geometry of the
mask, and where that geometry came from is outside. A user who needs the resist
profile computed rather than stated runs a lithography tool and brings its
output in as the pattern.

The cost is named rather than hidden. A stated pattern is a sharp pattern, so
the sidewall angle and the corner rounding a real resist profile has are things
the recipe must supply as numbers if they matter. A user who supplies neither
gets a vertical wall, and that is a modelling choice they made by omission. The
recipe format in issue #9 is where the omission is made visible.

### Equipment and reactor scale modelling

Out, because feature scale is what a process engineer asks about and reactor
scale is what an equipment vendor asks about. They need different physics,
different validation data and different users, and a tool that tries both is
poor at each.

Instead: an input the recipe supplies. Where a reactor scale quantity is needed,
it enters as a boundary condition the recipe states. An oxidation step is given
the ambient, the partial pressure and the temperature; it does not compute the
gas flow that produced them. An etch step is given a rate and an anisotropy; it
does not compute the plasma.

The consequence worth stating is that this project cannot answer a question of
the form "what happens if I change the chamber pressure", except through a
parameter the user has themselves related to chamber pressure. That is a real
limit on who this tool is for, and it follows from the line rather than from an
unfinished feature.

### Layout extraction and anything above it

Out, because the open chain from register transfer level to layout is finished
and this project exists to extend it downward rather than to duplicate it.
Netlist extraction, parasitic extraction, design rule checking and the layout
database are all above the line this project starts at.

Instead: an input the recipe supplies, for the one thing that crosses. The mask
geometry a masking step needs is the only thing this project takes from that
layer, and it takes it as geometry rather than as a layout database. For
everything else above the line, nothing.

### The stopping physics of an implant

Out, because it is a separate board with a separate audience. An ion stopping
model is useful to people who never run a diffusion, and the sibling board
bremsweg exists for it, which is checkable:

    gh api repos/iderex/bremsweg --jq '{visibility, description}'
    {"description":"An open replacement for SRIM, with a stopping-power fit somebody can audit","visibility":"public"}

Instead: an input, from a named source, with the interface itself a deliverable
here. The implant step is inside the boundary; what the ion did on the way in is
not. Issue #41 is where the interface is written and where what this board asks
of that one is stated. Issue #42 is the analytic route for a user who does not
want a second tool in the loop, and the honest statement of what an analytic
profile cannot do belongs there.

This is the one exclusion where the thing excluded is a dependency rather than a
neighbour, and it is the reason the front page says this project depends on that
board rather than duplicating it.

### Mechanical stress

Out for the first release, with a reservation. This one is argued at length
below rather than in a line, because waving it away would make the tool quietly
wrong for anything modern and declaring it in would add a mechanical solve to
milestone 4.

## The position on mechanical stress

Stress is not a refinement. It changes diffusivity, it changes the oxidation
rate, and below roughly the ninety nanometre node it is deliberately engineered
into the device rather than tolerated. A process simulator that cannot represent
it is wrong for the geometries that most of the interested audience actually
works on.

Declaring it permanently out would say that plainly and would be a defensible
smaller tool. Declaring it in now would put a mechanical solve, its constitutive
model, its boundary conditions and its validation data into the same milestone
that first gets diffusion working, and the likely outcome is that neither lands.

The position is the third one, and it has three parts.

Stress is outside the boundary of the first release. Nothing in milestones 3
to 9 computes a displacement, a strain or a stress, and no coefficient in this
project takes a stress argument.

The structure layer reserves for it. Issue #5 is required to make a place for a
symmetric rank two tensor field on the structure, on the same footing as the
scalar fields, so that adding stress later is an addition of a solver and a
coefficient dependence rather than a change to what a structure is. That
requirement is stated here and issue #5's record references it rather than
restating it, so that it has one home.

The reservation is a reservation and not an implementation. No tensor field is
populated, no solver writes one, and a structure written today carries the place
rather than the field. What the reservation costs today is issue #5's to state
in its own record, because the cost is a property of the representation it
chooses and this record does not choose one.

What the reservation does not buy is worth saying, so that it is not mistaken for
the work. Carrying a tensor field makes the data structure ready. It does not
make the coefficient models ready, and every diffusivity and every oxidation rate
constant in this project is fitted on unstressed material and will need refitting
or a stress dependent form. It does not make the validation ready either, because
a stress dependent claim needs measurements on stressed material. The
reservation buys the part that cannot be added later without a rewrite, and
nothing else.

## Three dimensions

Not settled here. Entry 6 of issue #1 governs what is promised about the third
dimension, it is open, and this record assumes no answer to it.

What this record does say is that the third dimension is a question about extent
rather than about the boundary. Everything inside the boundary above is stated
without counting dimensions, and everything outside it stays outside in three as
well. So entry 6 moves the cost of the structure layer and moves what the front
page promises, and it moves no line in this document.

## How a later proposal is tested against this record

A proposal for work outside the boundary is closed against this record rather
than argued on its own merits, and that is the point of writing it down. The test
is two questions in order.

Does the proposed work compute something this record places outside? If yes, the
proposal is closed with a pointer to the exclusion and to what the record says a
user gets instead. Whether the work would be good is not the question, because
every one of the exclusions above would be good.

If it is genuinely not covered, the proposal amends this record before it is
built. An amendment says which line moves, what it costs in the structure layer,
and what the reason given for the original exclusion got wrong. It is a change to
this file with an argument in it, not an exception granted in an issue nobody
finds later.

No sweep of the open tracker against this record was performed when it was
written, and this record does not claim the board is clean. What was read is
every open title and every milestone one body, which is a smaller thing than
every body. The population is this large and it moves, so re-run it rather than
trusting the paste:

    gh issue list --repo iderex/schichtwerk --state open --limit 300 \
      --json number --jq 'length'
    60

Nothing enforces the test. No check reads this file, no check reads an issue
body against it, and the tree holds nothing that could:

    git ls-files -- '*.c' '*.h' '*.cpp' '*.hpp' '*.rs' 'CMakeLists.txt' | wc -l
    0

The test is applied by a person reading this record, and that is the whole
disclosure of it.

## What this record does not decide

Whether this project depends on the existing open topography library for surface
evolution is issue #4, and it is open. Deposition and etch topography are inside
the boundary either way; what is undecided is who computes the surface
evolution, which is a means question rather than a boundary one.

What a structure is, including how the reserved tensor field is expressed, is
issue #5.

What crosses the handoff to a device simulator, in what format, is issue #13.
This record fixes that there is one and that it is a deliverable.

What the recipe supplies for each of the exclusions above, in what syntax, is
issue #9. This record fixes which quantities have to be suppliable and says
nothing about how they are written.

## The means

Markdown in the repository, read by a person and quotable in an issue. A decision
record has to be readable from a fresh clone with nothing installed, it has to sit
in version control so that what was decided and when is recoverable from git
rather than from memory, and it has to survive being disagreed with on the merits.
It adds no language, no runtime and no dependency to a tree that today holds no
build. Nothing outside this repository forces the choice.
