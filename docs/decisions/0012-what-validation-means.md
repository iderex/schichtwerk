# What validation means when the reference is a measurement

Decided by issue #12, milestone 1.

## Status

In force from the commit that lands this file, for everything it decides.

One thing issue #12 asks for is not decided by a document and is not here. The
rule that a failing case stays visible has to be implemented in the harness
rather than kept as a habit, and the tree holds no harness and no code that could
become one:

    git ls-files -- '*.c' '*.h' '*.cpp' '*.hpp' '*.rs' 'CMakeLists.txt' | wc -l
    0

So issue #12 stays open on that item. Until it is met, nothing refuses the
removal of an inconvenient case, and the section below headed "What is not
enforced" is the whole disclosure of it.

The naming and numbering of decision records is fixed by issue #2, which is not
decided. This filename is provisional and #2 may change it.

## The reference

A measurement is the reference. A commercial tool is a datum.

Two references are available and they disagree, which is why this has to be
settled before the first case exists rather than case by case afterwards.

Comparing against the incumbent commercial simulators is easy to arrange for
anyone with a licence and it is what a prospective user actually wants to know.
It is also a trap, and the trap is not that the commercial answer is bad. It is
that agreement with it becomes the target, and once it is the target this project
is a reimplementation of a model whose coefficients nobody outside those
companies can inspect. The whole argument for building this is that the open
chain needs a level it can audit. A second copy of a level it cannot audit is not
that.

Measurement is what the models were fitted to in the first place and it is the
only reference that can say which of two tools is right. Secondary ion mass
spectrometry and spreading resistance for profiles, sheet resistance and Hall
measurements for the active fraction, ellipsometry and cross section microscopy
for oxide thickness.

So a validation case is a published measurement with a stated uncertainty, and
the simulation result is reported beside it. Where a comparison against a
commercial tool has been published for the same case, it is reported as a third
line in the same report, labelled as a datum. It is never the criterion, it never
appears in the verdict, and no tolerance is stated against it.

When this tool and a commercial tool disagree and the measurement sits between
them, the report says exactly that. It does not adjust.

What that position costs, stated because a record that only lists advantages is
an advertisement. It is slower: every case needs a measurement found, read and
digitised rather than a run of another tool that produces a curve in minutes. It
is harder to declare done, because a difference from a measurement has to be
explained rather than closed. And it is weaker as marketing, because the sentence
a user wants to hear is that this agrees with the tool they already trust, and
that sentence is not available here.

## The agreement metric

Named before the first case exists, which is the point of deciding it here.
Choosing a metric after a result is known is choosing the metric that the result
passes.

A profile is judged on three numbers and it agrees only if all three are inside
their tolerance. One number is not enough, and each of the three is blind to a
failure the others catch.

The junction depth, taken where the profile crosses a background concentration
that the case states. Judged as the relative difference between the simulated and
the measured depth. This is the number a user acts on, and it is the one a
comparison of curves on a logarithmic plot hides most effectively: two profiles
that look identical on a plot spanning five decades can differ by a factor of two
in junction depth.

The retained dose, taken as the integral of the profile over the depth interval
the case states. Judged as the relative difference. A model can get the junction
depth right by having the wrong amount of dopant distributed the wrong way, and
this is the number that notices.

The shape, taken as the root mean square difference of the base ten logarithm of
the concentration, over the depth interval where the measurement is above its own
detection limit and below any saturation the technique imposes. In decades. The
logarithm is not a presentation choice: the concentration spans orders of
magnitude and a linear residual is a statement about the peak and about nothing
else. This is the number that sees a channelling tail that is a decade low while
the depth and the dose both pass.

The interval each of the three is taken over is part of the case, not part of
this record. A case that does not state its interval is not a case.

For an oxide thickness the metric is the relative difference in thickness, and
where a growth curve is being compared rather than a single point, the same
relative difference evaluated at every published time with the worst one
governing. Not the mean, because a model that is right in the linear regime and
wrong in the thin regime averages to a pass and is wrong exactly where the thin
oxide behaviour that issue #37 exists for lives.

## The tolerance

The tolerance is the measurement's stated uncertainty, or a floor, whichever is
larger.

Stated that way for a reason. A single fixed number is dishonest in both
directions: it declares a disagreement against a measurement whose own scatter is
wider than the criterion, and it declares agreement against a measurement good
enough to have caught the model out. Tying the criterion to what the measurement
actually claims is the only form that stays honest across a suite of sources of
different quality.

The floors, which apply where the source states an uncertainty smaller than them
or states none at all:

    junction depth      10 per cent relative
    retained dose       20 per cent relative
    shape               0.15 decades root mean square
    oxide thickness     10 per cent relative

These four numbers are chosen, not derived, and this record says so rather than
dressing them up. They are set at the scale at which a profiling technique's
depth calibration and concentration calibration are ordinarily argued about in
the sources this suite will draw on, so that the criterion is not tighter than
the thing it judges. No measurement was made here to fix them and none could be:
the tree holds no code, no case and no suite.

The condition that revises them is observable rather than a date. If a case is
ever recorded as disagreeing where the floor rather than the measurement decided
the verdict, or as agreeing where the floor rather than the measurement decided
it, that case is the argument for changing the number, and the change is an
amendment to this record carrying the case that forced it. Widening a floor
because a case failed, with no such case named, is the move this record exists to
prevent.

## Uncertainty that the source does not state

A measurement with no stated uncertainty is used only with an assumed
uncertainty, and the assumption is written into the case as an assumption.

Both halves matter. Refusing such measurements outright would remove a large part
of the older literature, which is where several of the reference profiles in this
field live. Using them while quietly treating the floor as though it were the
source's own figure would put a number in the report that the source never made.

So the case file carries the uncertainty and carries which of the two it is. The
report prints it the same way, next to the number, and a reader can tell a
measurement that claims a figure from one that was assigned one here.

A case whose verdict changes depending on whether the assumed uncertainty is
used is reported as unresolved rather than as agreement. That is a third verdict
and it is deliberate: an agreement that rests on an assumption made in this
repository about somebody else's measurement is not an agreement, and calling it
one is exactly the quiet overstatement the whole board is arranged against.

A profile digitised from a published figure carries a second uncertainty, from
the digitisation, and it is recorded separately from the measurement's own. They
have different sizes, different causes and different fixes: a better scan removes
one and nothing removes the other.

## What happens to a case that fails

It stays in the suite, marked as known to disagree, with the discrepancy
published.

It is not removed. Its tolerance is not widened. Its interval is not trimmed to
the part where it agrees.

A suite that contains only passing cases is a claim about the selection, not
about the code, and a reader has no way to tell one from the other from the
outside. A suite that carries its disagreements is the only form in which a green
result means anything.

The verdicts are three rather than two, and the third is not a failure.

Agrees. Every metric inside its tolerance.

Disagrees. At least one metric outside. The report states which one, by how much,
and what is suspected, and the case remains.

Not evaluated. The case could not be run or the comparison could not be formed,
because an input was missing, because the reference data is not present on this
machine, or because the run refused. This is separate from disagreeing because a
case that did not run is not evidence about the model in either direction, and
collapsing the two lets an absent case be read as a passing one.

## Adjusting a coefficient while a comparison is open

That is fitting, and it is disclosed as fitting.

A coefficient changed while looking at a comparison has been fitted to that
measurement, and the case no longer validates the coefficient. It can still be
carried, as a case the parameter set was fitted on, and it is labelled that way
in the suite and in any report drawn from it. What may not happen is a
coefficient moving during validation work and the case afterwards being counted
as evidence that the model agrees.

Issue #40 already asks its pull request to state which coefficients, if any, were
adjusted during that work and why. This record is where the rule that makes that
question answerable is written down.

## Where the measurement data lives

Entry 5 of issue #1 decides whether reference measurements are redistributed
inside this repository, fetched by citation, or a mixture. It is open and this
record assumes no answer to it in either direction.

What this record fixes is the part that is the same whichever way it goes. A case
names its measurement by full citation and by content hash of the exact bytes the
comparison was made against. A case that cannot obtain its measurement is not
evaluated in the sense above, and it says so rather than being absent. The
verdict of a case therefore never depends on where the bytes came from, only on
whether they were the bytes the hash names.

## What is not enforced

Nothing in this record is refused by a machine. There is no suite, no harness and
no code:

    git ls-files -- '*.c' '*.h' '*.cpp' '*.hpp' '*.rs' 'CMakeLists.txt' | wc -l
    0

The rule that a failing case stays visible is the one that most needs a mechanism
rather than a habit, because it is broken by deleting a file, which is the
easiest thing in the world to do and leaves a green suite behind. Issue #12 holds
that item open. The suites that will carry it are issue #36 and issue #40, both
of which run under the harness in issue #24, and the mechanism belongs with them.

Naming that is not having it. Until it exists, a case can be removed from this
suite and nothing will say so.

## The means

Markdown in the repository, read by a person and quotable in an issue. A decision
record has to be readable from a fresh clone with nothing installed, it has to
sit in version control so that what was decided and when is recoverable from git
rather than from memory, and it has to survive being disagreed with on the
merits. It adds no language, no runtime and no dependency to a tree that today
holds no build. Nothing outside this repository forces the choice.
