# The error and failure policy: what refuses, what warns, what runs on

Decided by issue #14, milestone 1.

## Status

In force from the commit that lands this file, for everything it decides.

Two of the things issue #14 asks for are not decided by a document and are not
here. The substitution class has to be wired into the manifest so that recording
a substitution is the only way to perform one, and one failure of each class has
to be shown actually occurring with the output it produces. Both need a program,
and the tree holds none:

    git ls-files -- '*.c' '*.h' '*.cpp' '*.hpp' '*.rs' 'CMakeLists.txt' | wc -l
    0

So issue #14 stays open on those two items. The section headed "What is not
enforced" is the whole disclosure of what that leaves unrefused.

The naming and numbering of decision records is fixed by issue #2, which is not
decided. This filename is provisional and #2 may change it.

## Why this is decided before the first solver exists

A long simulation that hits a problem has three honest options and one dishonest
one. It can stop. It can carry on and say clearly what it did instead. It can
carry on and record the substitution where the user will read it. The dishonest
one is to carry on quietly.

The dishonest one is also the one that happens by default in numerical code, and
not through carelessness. The alternative to a step that will not converge is a
smaller step, and taking a smaller step feels like recovery rather than like a
decision. By the time somebody notices, the run has taken four thousand of them
and the answer is about a different problem.

Written down after the first solver exists, the class of each failure is whatever
the person at that keyboard chose while trying to get a case to run, which is the
worst moment at which to choose it.

## The classes

Three, and a fourth thing that is deliberately not one of them.

A refusal is anything where continuing would produce a number that looks valid
and is not. The run stops. It names what was refused and what would fix it, and
it exits with a status that a script can tell apart from a crash.

A recorded substitution is anything where a defensible default exists and using
it changes the answer. The run continues. The substitution appears in the
manifest as a substitution rather than as a value, in the sense issue #10 fixes,
and it appears in the human summary too, because a manifest field nobody reads is
not a disclosure.

A warning is anything that does not change the answer but tells the user
something about their case. A mesh that is coarse relative to a gradient. A
requested output that duplicates another. Warnings are the smallest class
deliberately: a code with fifty warnings has none, because the fifty-first is
invisible. A warning that fires on nearly every run is deleted or promoted to one
of the other two, and staying at fifty is not an option this record leaves open.

The fourth thing is a defect in this program: an unhandled fault, an assertion,
an out of memory. It is not a class here because it is not a decision anybody
took. It is deliberately left to produce whatever the runtime produces, and the
reasoning is in the section on exit statuses below.

## The rule for placing a new failure

Asked in this order, and the first yes decides it.

Could the run continue and produce a number that a reader would mistake for one
computed the way they asked? If yes, it is a refusal. This is the whole of the
rule and the rest only refines it.

Does continuing require choosing a value, a coefficient or a model term that the
input did not supply, where a different choice would give a different answer? If
yes, it is a recorded substitution.

Does it leave the answer unchanged while telling the user something about their
case that they could act on? If yes, it is a warning. If it leaves the answer
unchanged and tells them nothing they could act on, it is not reported at all,
and adding it is how the warning class gets to fifty.

Where the answer is not clear, it is a refusal. The tie goes that way on purpose,
because the two mistakes are not the same size. A refusal that should have been a
substitution costs an annoyed user who opens an issue, and the issue is where it
gets fixed. A substitution that should have been a refusal costs a number that is
wrong in a way nobody can find later, in a report somebody has already published.

A class is not changed because a case keeps hitting it. Moving a failure from
refusal to substitution, or from substitution to warning, is an amendment to this
record that says what was wrong with the original placement. It is not a thing
done in the pull request that was trying to get a case to pass.

## What is already placed by the records that landed

Stated here so that the classes are concrete rather than abstract, and so that a
reader can check them against the files rather than against this paragraph:

    git ls-files docs/decisions/
    docs/decisions/0006-discretisation-and-moving-boundaries.md
    docs/decisions/0008-diffusion-model-family.md
    docs/decisions/0010-result-document-and-run-manifest.md
    docs/decisions/0012-what-validation-means.md
    docs/decisions/0014-error-and-failure-policy.md

A field transfer that violates the conservation tolerance is a refusal. The
record for issue #6 fixes that the run stops and leaves the class to this one,
and the reason it cannot be anything else is that a transfer which has lost
dopant has made every later number in the run a number about a different amount
of dopant, with no point at which the run recovers.

A recipe naming a model rung that the parameter set in force cannot support is a
refusal, from the record for issue #8. A recipe naming no rung at all is a
recorded substitution, from the same record.

A material for which no parameter set exists is a refusal rather than a run with
a neighbouring material's coefficients, also from that record.

A Newton iteration that has not converged at the smallest step the integrator
will take is a refusal, which issue #32 carries. A step accepted at its iteration
limit and reported as success is the single defect this project can least afford,
because it is invisible in every output.

A validation case that could not be run is not a failure of any of these three
classes. It is the third verdict in the record for issue #12, and it lives there
rather than here, because it is a statement about evidence rather than about a
run.

## Not a number and infinity

They are refusals, not values to be propagated.

A not a number in a concentration field propagates to everything the field
touches, and the profile that comes out of it is not wrong in a way a user can
see, it is absent in a way that some plotting routines will quietly skip. An
infinity is worse, because arithmetic will sometimes turn it back into a finite
number.

Where the check happens is a decision and it is not on every operation. It runs
at four named points: after a residual is assembled, after a linear solve
returns, after a field transfer, and before any field is written into a result
set. Those are the boundaries at which a bad value has just been produced by
something and has not yet been spread by anything, so the message can name the
step, the species and the place, which is the difference between a refusal a user
can act on and one that says only that something went wrong.

No global floating point trap is enabled. Three reasons, and the third is the one
that decides it. Enabling one is platform and compiler specific in a way that
makes the behaviour differ between two builds of the same commit. It changes the
behaviour of library code this project did not write and cannot fix. And a trap
raised inside such a library arrives as a fault rather than as a refusal, which
converts a diagnosable stop into an undiagnosable one, so the mechanism would
work against the thing it was added for.

The cost has not been measured. There is no code to measure and no case to
measure it on, and this record will not put a figure in that was not produced by
a command. What can be said without measuring is the shape of the claim rather
than its size: each check is a pass over the unknowns, and it sits next to a
linear solve whose cost grows faster than linearly in the same unknowns, so the
ratio is expected to fall as cases get larger. That is a claim and not a
measurement. Issue #52 measures where the time goes before anything is optimised,
and this is one of the things it should attribute rather than fold into the
solver total.

If that measurement shows the cost is not small, the answer is not to remove the
check. It is to state the cost here, next to the reason the check exists, and let
a reader see both.

## Exit statuses

The status is part of the contract with whoever automates this, so it belongs in
this record and in the documentation rather than in the code alone. Somebody
running a sweep of two hundred cases overnight reads statuses and nothing else.

    0    the run completed and wrote a result set
    1    a refusal
    2    the input could not be read
    3    the filesystem the run was given could not be used

The list is short and the reasons for each division are the responses they
produce, not tidiness.

A refusal and an unreadable input are separated because a sweep script does
different things with them. A refusal is a case that needs a person to look at
the physics or the recipe. An input that could not be read at all, meaning a
missing file, a malformed recipe, a parameter set that does not parse, is
ordinarily a defect in the script that generated it, and the script's own author
is the one who fixes it.

The filesystem gets its own status because it is neither of those. A directory
that cannot be written or a file that may not be read says nothing about the case
and everything about where it was run, and a sweep that returns two hundred of
them has one cause.

Zero is never returned by a run that refused, and never by a run that produced no
result set. A run that completed with substitutions and warnings returns zero,
because it did what it was asked and the disclosure of what it substituted is in
the result rather than in the status. A status is too small an object to carry
that, and encoding it there would make a sweep script's success condition depend
on a number nobody can read.

Nothing above 3 is assigned by this project. That is deliberate. A program that
maps an unhandled fault onto a clean status of its own has hidden a defect behind
a number that looks considered, and the sweep script cannot tell it from a real
refusal. A crash exits however the runtime exits, it looks like a crash, and it
gets reported as one.

## What a refusal leaves behind

A refusal writes its account, in the directory the run was given, using the same
manifest shape the record for issue #10 fixes, holding what the run was asked to
do and what was refused and why.

That directory is not a result set and is not called one. A result set holds one
authoritative structure artefact, and a refused run has no answer to put in it.
Calling it a result set would let a script that counts result sets count a refusal
as a result, which is the same error as counting an unevaluated validation case
as a passing one.

The reason the account exists at all is that a refusal on the two hundredth case
of an overnight sweep has to be diagnosable in the morning, and a message that
went to a terminal nobody was watching is not.

## The substitution rule, and the half of it that is owed

The record for issue #10 already fixes that recording a substitution is the only
way to perform one, so a code path that substitutes without recording is a defect
against that record rather than a style question.

This record adds the other direction. Every substitution is one of the class
above, so it is placed by the rule in this document before it is recorded, and a
default that changes the answer may not be introduced as a warning or as nothing
at all in order to avoid the manifest entry.

Making that impossible rather than merely required is the item issue #14 holds
open, and it is a property of the code that writes the manifest rather than of
either record. Until it exists, both records are read by a person.

## What is not enforced

Nothing here is refused by a machine. No code exists that could refuse it:

    git ls-files -- '*.c' '*.h' '*.cpp' '*.hpp' '*.rs' 'CMakeLists.txt' | wc -l
    0

Specifically, and named rather than gestured at: nothing stops a substitution
being performed without being recorded, nothing stops a refusal being softened
into a warning, nothing checks that a refusal message names what would fix it,
and nothing checks that the statuses above are the ones actually returned. The
first of those is the item issue #14 holds open. The rest have no issue of their
own today, and saying so is what this section is for.

## The means

Markdown in the repository, read by a person and quotable in an issue. A decision
record has to be readable from a fresh clone with nothing installed, it has to
sit in version control so that what was decided and when is recoverable from git
rather than from memory, and it has to survive being disagreed with on the
merits. It adds no language, no runtime and no dependency to a tree that today
holds no build. Nothing outside this repository forces the choice.
