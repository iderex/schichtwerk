# The numeric contract: units, constants, convergence and determinism

Decided by issue #7, milestone 1.

## Status

In force from the commit that lands this file.

The naming and numbering of decision records is fixed by issue #2, which is not
decided. This filename is provisional and #2 may change it. Nothing in the
content below depends on the name.

Issue #7 asks for a check that refuses a second definition of a constant. This
record names that check and does not supply it, for the reason the section on
constants gives. The issue stays open on that item.

## Why this is a decision rather than an implementation detail

This field mixes centimetres, microns, nanometres and angstroms in the same
paragraph, quotes concentrations per cubic centimetre while solving on a mesh
measured in microns, and states activation energies in electron volts while a
solver works in joules. Almost every published wrong number in computational
work of this kind is a unit that was converted twice or not at all.

The same shape appears three more times. Two parts of a program that each define
the Boltzmann constant will eventually define it differently. A Newton iteration
stopped on a small update is not the same thing as one stopped on a small
residual, and the difference shows up as a plausible answer to an equation that
was not solved. And a parallel sum reorders floating point additions, which moves
the last bits, which moves an iteration count, which moves the answer visibly.

Four contracts, fixed once here so that every later issue inherits them instead
of deciding each one again in its own corner.

## The internal unit system

SI base units throughout the code. Metre, kilogram, second, kelvin, coulomb, and
everything derived from them. Concentration is per cubic metre, diffusivity is
square metres per second, energy is joules, temperature is kelvin.

This is the less comfortable of the two candidates and it is chosen on three
grounds.

The physical constants are exact in SI and are not exact in anything else. Since
the 2019 revision of the SI, the defining constants have exact decimal values by
definition rather than measured ones with an uncertainty. Converting an exact
constant into a centimetre based system produces a number that is no longer
exact, and it produces it in this project rather than in the source. Keeping the
internal system SI means the conversion, where one is needed at all, happens on
the way out to a report rather than on the way in to the arithmetic.

Every number that enters already carries a declared unit. Issue #9 requires every
physical quantity in a recipe to be a value and a unit with both required, and
issue #11 requires the same of a parameter set. So no number reaches the code
without a stated unit, and conversion is mechanical rather than remembered. That
removes the argument that a centimetre based system minimises hand conversion of
published coefficients, because under those two records nobody hand converts
anything: the coefficient is written in the unit the publication used and the
loader converts it.

Anything outside this project assumes SI or assumes nothing. A mesh generator, a
geometry library and a device simulator are each easier to hand SI than to hand a
mixed system, and issue #13 has to state the units of everything it exports
whatever the internal system is.

What SI costs, stated rather than left to be discovered.

The numbers are unfamiliar. A doping level a process engineer reads as 1e20 is
1e26 internally, and a junction depth read as 50 nanometres is 5e-8. Somebody
debugging a solver sees the second of each pair. The answer is that the
boundaries convert and the summary in issue #10 is written in the reader's
units, not that the internal number is comfortable.

Activation energies are the one place the field's convention is nearly universal
and is not SI. They are electron volts everywhere in the literature. They are
joules internally, electron volts are an admitted input unit under issue #11, and
the conversion factor is the elementary charge, which is exact. The cost is that
a parameter file and a debugger disagree by a factor no reader carries in their
head, and it is accepted rather than special cased, because one exception to a
unit rule is how a unit rule ends.

Conditioning is not delegated to the unit system. A system holding concentrations
near 1e26 and geometric quantities near 1e-8 is badly scaled in any unit system,
and a centimetre based system merely makes it less obviously so. The solver
non-dimensionalises explicitly, with the reference values it used recorded in the
manifest under issue #10, and that is issue #32's to build. Scaling that is a
declared step can be inspected and changed. Scaling that is a side effect of
which unit somebody chose cannot.

## Where conversion happens

In exactly two places, and nowhere between them.

On the way in, when a recipe is loaded under issue #9 and when a parameter set is
loaded under issue #11. Both carry an explicit unit per quantity, both are
converted once, and the converted value is what everything downstream sees.

On the way out, when a result document, a summary or an export is written under
issue #10 and issue #13. Every quantity written out carries the unit it is
written in, in the artefact itself rather than in documentation about the
artefact.

Between those two boundaries there is one unit system and there is no
conversion. A function that takes a length in microns, a constant expressed per
cubic centimetre and a coefficient that is converted at its point of use are all
defects against this record rather than local choices.

The check that would refuse one is named here and does not exist. Its name is
`no-conversion-outside-the-boundary`, and its subject is a conversion factor
appearing anywhere but the two loaders and the writers. What it can actually
refuse depends on the language, which is issue #2 and is not decided, so the
shape of the check is not fixed here either. Naming it is not having it.

## The physical constants

One source, one revision, one home in the tree, cited in the file that holds
them.

The source is the CODATA internationally recommended values of the fundamental
physical constants, 2022 adjustment, as published by NIST. That revision is what
the NIST page carried when this record was written, and the page states a last
data update of May 2024:

    https://physics.nist.gov/cuu/Constants/

The revision is named rather than left as the current values, because the
current values change and a result computed under one set is not a result
computed under another. Moving to a later adjustment is a change to this record
and to the constants file together, and it is recorded in the manifest through
the commit under issue #10 rather than being invisible.

Two kinds of constant are distinguished in the file, because they are different
epistemic things and a file that writes them identically has destroyed the
difference. The SI defining constants have exact decimal values by definition
since the 2019 revision, and the elementary charge and the Boltzmann constant,
which are the two this project uses most, are among them. Everything else is a
measured value with a stated uncertainty, and the file carries the uncertainty
alongside the value even though nothing propagates it today. Carrying it costs a
column and it is the only way a later question about sensitivity has anything to
read.

No constant is restated in this record. A list in a document drifts against the
file that holds it, and the file is the authority.

A model coefficient is not a physical constant and does not live in that file. A
diffusivity prefactor, an activation energy for a dopant and an oxidation rate
constant are parameters with provenance under issue #11, they are versioned and
selectable, and a result names which set produced it. A physical constant is
none of those things. Keeping them in separate homes is what stops a fitted
number acquiring the authority of a defined one.

The check that refuses a second definition of a constant is named
`one-home-per-constant`. Its subject is a numeric literal in the tree matching a
constant the constants file holds. It is the same rule issue #11 states for
parameters and it is deliberately a separate check, because the two have
different subjects and a single check over both would report a parameter defect
as a constant defect.

It does not exist. There is no constants file and there is nothing that could
read one:

    git ls-files -- '*.c' '*.h' '*.cpp' '*.hpp' '*.rs' 'CMakeLists.txt' | wc -l
    0

That is the item issue #7 stays open on, and this paragraph is the whole
disclosure of it.

## Convergence

The residual is authoritative. The update is recorded and is never the criterion.

A Newton iteration stopped because the update got small has stopped iterating.
On a badly scaled or nearly singular system the update gets small while the
residual does not, and the run then reports a converged solution to an equation
it did not solve. Stopping on the residual cannot fail in that direction: a small
residual means the discrete equations are nearly satisfied, which is the thing
the answer depends on.

The norm is the infinity norm of the residual, scaled per equation. Two norms
were weighed. The two norm divides by the number of cells, so one cell with a
large residual among a million good ones is averaged into invisibility, and the
one cell is usually the interface. The infinity norm reports the worst cell,
which is the cell a user needs to know about.

The scaling is per species and per equation, by a reference value the solver
computed and recorded, so that a species held near 1e26 and one held near 1e18
are judged on the same footing. Without it the criterion is a criterion about
whichever species happens to be largest.

The default tolerance is a scaled residual infinity norm at or below 1e-8, with
a per species absolute floor for the same reason issue #6 needs one: a species
that has not been introduced yet has a residual of zero and a relative criterion
on it is not defined. Both the tolerance and the floors are part of the input and
both are recorded in the manifest, because a tolerance loosened to make a case
pass is a change to the answer.

The number 1e-8 is chosen and not measured. Nothing exists to measure it
against, and the reasoning behind the choice is stated so that a later
measurement has something to disagree with: it is far enough below the
discretisation error of any mesh this project will run that the iteration is not
what limits accuracy, and far enough above double precision round off on the
cell counts expected that the iteration can actually reach it. Revising it is an
amendment to this record naming the case where the tolerance rather than the
discretisation decided the answer.

What happens when it is not reached. The iteration has a maximum count, 25 by
default, also part of the input and also recorded. Reaching it is not
convergence. The time step is reduced and the step is retried, and every
occurrence is recorded in the manifest under issue #10, which already requires
the iteration counts and every step that reached a limit. A step that fails to
converge at the smallest permitted time step is a refusal, which is what issue
#14 already decided and this record does not reopen.

What the run prints when it stops. Per step, the achieved scaled residual, the
iteration count, and the time step. Per run, the worst achieved residual over
every step, the total iteration count, and the number of step reductions. The
achieved figure rather than the fact that it passed, for the reason issue #6
gives about the conservation figure: a run that converged at 9e-9 every time and
a run that converged at 1e-14 are different runs, and only the first is a warning
about the next case.

Issue #32 applies this contract to the diffusion solver. It does not get to
choose a different criterion, and its own tolerance work is about the time step
rather than about the iteration.

## Determinism

Bit reproducibility at a fixed thread count. The thread count is part of the
input and is recorded in the manifest.

The three positions were weighed as issue #7 states them.

Bit reproducibility regardless of thread count is the strongest and it costs a
reduction ordering that does not depend on the decomposition. That means a fixed
reduction tree over the whole mesh, computed independently of how the work was
split, and it costs both speed and the freedom to change the decomposition for
any other reason. It is the position to move to if the weaker one turns out to
be insufficient, and moving to it is an amendment rather than a rewrite.

Reproducibility only up to a stated tolerance is the weakest, and it costs every
regression test. A test that compares within a tolerance cannot distinguish a
change in the last bits from a change in the third digit unless somebody chose
the tolerance correctly for that quantity, and nobody chooses it correctly for
every quantity in a suite.

The middle position is chosen. Within a fixed thread count the reduction order is
determined by the decomposition and the decomposition is determined by the input,
so two runs of the same case on the same build with the same thread count sum in
the same order and produce the same bits. Across thread counts the order changes
and the last bits may move.

What that costs, named. A user who runs a case on four threads and then on eight
may see a different number, and the difference can be visible rather than in the
last bits, because a moved last bit can change a Newton iteration count and an
iteration count changes a time step history. This is not a defect and the
manifest is what makes it diagnosable: two manifests differing only in thread
count is the answer to why two results differ. A user who wants the results
comparable pins the thread count, and the tool does not pin it for them, because
a tool that silently serialises to make a comparison work has made a performance
decision on the user's behalf.

Any Monte Carlo path in this project is seeded from the recipe rather than from
the clock. The seed is derived from the content hash of the concrete recipe,
after any sweep expansion, together with a stream index that is a property of
which sampler is being seeded. So the seed is a function of the input alone, it
inherits the position above without a separate rule, and it is recorded in the
manifest like everything else that changes the answer. No sampler reads the
clock, the process id, the address of anything, or an environment variable.

Issue #22 implements the check. What it needs from this record is a criterion it
can apply without a judgement, and the criterion is that the same case, on the
same build, at the same thread count, run twice, produces byte identical result
artefacts, with the parts of the manifest that record wall clock time and memory
excluded, since issue #10 already places those in a part that says on its face
that nothing in it affects the answer. A difference anywhere else is a refusal.
The check is a comparison of bytes and it makes no judgement about size.

## Getting a bit identical result on a different machine

Stated plainly, because the alternative is a sentence that sounds like a promise.

In general a user cannot. The reason is not the thread count, which they can pin.
It is that the elementary functions are not bit reproducible across
implementations. Every Arrhenius term in this project is an exponential, and the
result of an exponential in double precision differs in the last bits between one
standard library and another, and between versions of the same one. Nothing this
project can do inside its own source changes that.

What a user can do, and what each item actually buys.

Use the same container image. This is the only route that gets to a bit identical
answer, because it pins the standard library, the compiler and the flags in one
object. It is the recommendation, and it is a recommendation rather than
something this project has today, because there is no build to put in an image.

Pin the thread count, which is necessary in every case and sufficient in none of
them on its own.

Pin the target architecture and refuse a build tuned to the host machine.
Compiling for whatever instruction set the build machine happens to have means
the same source produces different arithmetic on two machines, most visibly
where a fused multiply add contracts two operations into one with a single
rounding. Contraction is disabled, and that decision belongs to the build under
issue #16 rather than to this record, which states only that it has to be
decided rather than defaulted.

Use the same compiler and the same optimisation level. Issue #10 already records
the compiler identification and the build configuration for this reason, and this
record is why that field is there.

What none of that buys is a bit identical answer against a different standard
library, and no combination of the four reaches it. A user comparing results
across machines compares them under the validation criteria in issue #12 rather
than by equality.

## What this record does not decide

The time step control, the local error estimate and what a converged transient
means as distinct from a converged iteration are issue #32. This record fixes the
iteration criterion and the norm, and issue #32 inherits both.

The conservation tolerance for a field transfer is issue #6 and is a different
quantity with a different number. Nothing here changes it.

Which failure class a refusal above belongs to, what its message says and what
exit status it carries, is issue #14.

The build, the compiler, the flags and the container are issue #16, which this
record constrains and does not decide.

The language and the toolchain are issue #2, which is why the two checks named
above have names and no shape.

## The means

Markdown in the repository, read by a person and quotable in an issue. A decision
record has to be readable from a fresh clone with nothing installed, it has to sit
in version control so that what was decided and when is recoverable from git
rather than from memory, and it has to survive being disagreed with on the merits.
It adds no language, no runtime and no dependency to a tree that today holds no
build. Nothing outside this repository forces the choice.
