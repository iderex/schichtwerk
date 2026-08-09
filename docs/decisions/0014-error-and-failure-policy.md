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

Amended on 2026-08-09 against `6b59c79a571beb6120146691da4d417745217aa5`. Two
landed records hand a placement to this one and it was silent on both, and the
sentence dividing a refusal from an unreadable input gave one status to three
stops that are fixed by three different people. The sections headed "A parameter
used outside the range it was fitted over" and "Which stop returns which status"
are what the amendment added, and the second says what was wrong with the
sentence it replaces rather than replacing it quietly. Nothing else this record
decided has moved.

Amended again on 2026-08-09 against `2415de065f2b74710c1842cb9070dab24e1d687b`.
The section headed "What is already placed by the records that landed" opened
with a command and a pasted listing of the records in the tree, and the listing
had been true at the commit that landed this file and was not true afterwards.
It is replaced by the command alone, and the section says what was wrong with
the paste and why refreshing it would not be the repair. No placement, class,
status or tolerance in this record is touched by that.

Amended a third time on 2026-08-09 against
`65759e1f1df62f6910bbfa105b9f40860154b214`. Two implant cases were placed
outside this record and one of them was placed twice, in two records that do not
agree, while the record holding the classes said nothing about either. The
sections headed "An implant step that names no method" and "An implant above the
amorphisation threshold" are what the amendment added, and both argue from the
ordered rule rather than asserting a class. Nothing else this record decided has
moved.

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
reader can check them against the files rather than against this paragraph. Each
placement below names the record it came from, and what the set of records is at
any moment is what this prints rather than what this document last saw:

    git ls-files docs/decisions/

Five lines of output were pasted under that command and they are removed rather
than refreshed. They were the whole of the directory at
`ab1dc9c2e7213ee879131a3e5766230c68fadf02`, the commit that landed this record,
and they were correct there. The same command answers with thirteen names at
`2415de065f2b74710c1842cb9070dab24e1d687b`, which is `origin/main` as this is
written:

    git ls-tree -r --name-only ab1dc9c2e7213ee879131a3e5766230c68fadf02 -- docs/decisions/ | wc -l
    5
    git ls-tree -r --name-only 2415de065f2b74710c1842cb9070dab24e1d687b -- docs/decisions/ | wc -l
    13

What the stale paste cost is inside this document rather than beside it. The
amendment of 2026-08-09 added two sections that argue their placements out of
`docs/decisions/0011-parameter-provenance.md` and
`docs/decisions/0009-the-recipe-format.md` and quote both by path, and it left
the paste alone. Neither name is among the five. So the one paragraph offering a
reader the set to check the placements against was telling that reader that two
of the records this document reasons from are not in the tree, and it was doing
it under a command, which is where somebody goes to settle that exact question.

Pasting thirteen names in place of five puts the same defect back at the next
landing, because a record arrives here whenever a decision is taken and nothing
recomputes a paste when one does. The command stays and the snapshot does not.

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

## A parameter used outside the range it was fitted over

The record for issue #11 requires every parameter entry to carry the conditions
its number is good for, requires a use outside them to appear in the manifest and
in the human summary, and hands the class to this record by name:

    grep -n "is issue #14's decision" docs/decisions/0011-parameter-provenance.md
    101:refuses, is issue #14's decision and this record does not take it. What this

This record said nothing about it. That left the case placed nowhere except in the
third item of issue #28's Done-when, which is not where a reader looking for a
failure class goes.

It is a recorded substitution.

The rule above, asked in its own order, is what decides it. The first question is
whether the run could continue and produce a number a reader would mistake for
one computed the way they asked. Taken alone the answer is yes and the case is a
refusal, and the record for issue #11 gives the shape of it:

    grep -n 'produces a plausible number that is wrong' docs/decisions/0011-parameter-provenance.md
    87:700, produces a plausible number that is wrong for a reason nothing in the

A prefactor fitted between 900 and 1100 degrees Celsius, used at 700. That same
record then closes the half of it about the output, because it requires the
excursion to be recorded where the user reads it, and with the disclosure in place
the mistake the refusal class exists against is not the one on offer.

The second question is answered yes. Continuing requires using a coefficient
beyond the conditions it was established over, which the input did not supply and
could not, and a different choice, meaning another set, a lower model rung, or
stopping, gives a different answer.

So the disclosure is load bearing rather than a courtesy beside the placement. A
run that performs the excursion without recording it has not committed a smaller
fault than a wrong class. It has removed the reason the case is admitted at all,
and what it breaks is the substitution rule in this record.

What is recorded is the quantity, the entry it came from, the range that entry
states, the value of the condition at which it was used, and how far outside the
range that is. Recording only that a parameter was used outside its range is not
worth reading: a fit used two degrees past its window and one used two hundred
past it are different facts, and a reader cannot get from the first to the second.

The cost, stated rather than left to be found. A user who does not read the
summary gets a number that is wrong by an amount nothing here bounds, and the
manifest entry does not fix that. What it fixes is that the number is never
separated from the statement that a fit was extended to reach it. The placement
moves to a refusal, by an amendment to this record, if a validation case under
issue #36 or issue #40 is found disagreeing with its measurement because of an
excursion that was recorded and not read. That is observable rather than a date.

Two neighbouring cases, so the boundary is visible rather than assumed. An entry
that states no domain of validity and does not say the domain is unknown never
reaches a run at all, because the record for issue #11 refuses it where a
parameter set is read:

    grep -n 'an entry with no domain of validity' docs/decisions/0011-parameter-provenance.md
    182:kind are not, and an entry with no domain of validity and no statement that the

And an entry whose domain is recorded as unknown produces no excursion to compute,
so the class above cannot fire for it. The only disclosure such an entry carries
is the one the record for issue #11 already requires, which is that the domain is
unknown. That is a residual of admitting the kind rather than a gap this record
can close, and nothing anywhere reads unknown as unbounded.

The neighbouring case that is a refusal. A property absent for a material whose
parameter set otherwise exists is a refusal, with the message naming the material
and the property, which is what the fourth item of issue #28's Done-when asks for.
The case already placed further up this record is the same one a step coarser: a
material for which no parameter set exists is a refusal rather than a run with a
neighbouring material's coefficients, and the reason does not change when the gap
is one property instead of a whole set. There is nothing to substitute from, so
there is nothing a disclosure could carry.

## An implant step that names no method

Two routes produce an implant profile on this board. One is the analytic route in
issue #42, which reads moments from a table indexed by ion, energy and target
material. The other is a transport calculation reached across the interface the
record for issue #41 fixes. A recipe whose implant step names neither leaves the
run to choose between them, and the third item of issue #42's Done-when asks for
that recipe to be refused. This record was silent, so the class sat in a Done-when
and nowhere a reader looking for a failure class would open.

It is a refusal.

The case is one noun away from a case the record for issue #8 places the other
way, which is why it needs the argument rather than a cross reference:

    git grep -n 'A recipe that names no rung is a substituted default' -- docs/decisions/
    docs/decisions/0008-diffusion-model-family.md:100:A recipe that names no rung is a substituted default in the sense of issue #14.

The rungs that record covers are one model family, and a lower rung is the higher
one with terms absent rather than a different model of the same quantity. That
record puts the point defect rung under an obligation to reproduce the charge
state answer where the two meet:

    git grep -n 'Issue #34 carries that obligation' -- docs/decisions/
    docs/decisions/0008-diffusion-model-family.md:67:Issue #34 carries that obligation.

So a run that took the default there produced a number the rung a user would have
named reproduces in a stated limit. That is what makes the default defensible,
which is the condition the substitution class in this record puts before anything
is recorded, and the manifest entry then names something a reader can reason about
without running the other rung.

The two implant routes stand in no such relation. Neither is the other with terms
switched off, and the record for issue #41 finds the accuracy ordering between
them inverted for an implant into a single crystal, then declines to say which
route serves that case:

    git grep -n 'the cheaper path is the one that can express' -- docs/decisions/
    docs/decisions/0041-the-implantation-interface.md:215:evidence the cheaper path is the one that can express the observable and the

    git grep -n 'Which route serves an implant into a crystalline target' -- docs/decisions/
    docs/decisions/0041-the-implantation-interface.md:313:Which route serves an implant into a crystalline target. Issue #42 is the

A default therefore fails in a different direction depending on a property of the
case the run was not told to weigh. Into a single crystal, the route that cannot
express a channelling tail returns a profile that is too shallow and missing the
feature a junction depth is read off. Through a layer stack, the analytic route
approximates what the transport calculation does not, which is the limitation the
sixth item of issue #42's Done-when has to state plainly. While both hold, no
default is defensible, the second question of the rule above does not carry the
case, and the first question and the tie send it the same way.

The refusal names the ion, the energy, the target and both route names. A message
saying only that a method is missing leaves the user to assume the two would have
agreed, which is the assumption this placement exists against.

The condition that moves it is observable rather than a date. If the question the
record for issue #41 leaves open is settled, so that one route is authoritative
for a named target class, a defensible default exists for that class and the
placement becomes a recorded substitution there, by an amendment to this record.
It does not move for a class the settlement does not name.

## An implant above the amorphisation threshold

It is a refusal, and it is written here rather than in a Done-when and one other
record.

Where the case sits in the tree today. The record for issue #41 states the
behaviour and attributes the class to the rule in this record:

    git grep -n 'Above the amorphisation threshold the tool refuses' -- docs/decisions/
    docs/decisions/0041-the-implantation-interface.md:238:Above the amorphisation threshold the tool refuses, which is issue #44's own

The record for issue #8 describes the same case as a run that continues, and the
record above quotes that sentence in a section about the dose regime without
reading it against its own:

    git grep -n 'an implant heavy enough to amorphise' -- docs/decisions/
    docs/decisions/0008-diffusion-model-family.md:125:the model, so an implant heavy enough to amorphise is annealed as though the
    docs/decisions/0041-the-implantation-interface.md:159:    docs/decisions/0008-diffusion-model-family.md-125:the model, so an implant heavy enough to amorphise is annealed as though the

Two landed records disagreeing about a class, with the record that holds the
classes silent, is the state this section ends.

The rule decides it at the first question. An anneal computed as though the
lattice were intact, after an implant that amorphised it, produces a profile that
is smooth and plausible and is about a different problem, and nothing in the
result marks the case. The record for issue #8 refuses a neighbouring case for
that reason in its own words:

    git grep -n 'because a silently substituted material is a wrong' -- docs/decisions/
    docs/decisions/0008-diffusion-model-family.md:137:material's coefficients, because a silently substituted material is a wrong

A lattice the model cannot represent and a material the parameter set does not
hold are the same sentence with different nouns, and the third item of issue
#44's Done-when already asks for the refusal.

The refusal names the ion, the dose, the threshold that was exceeded and the
entry the threshold came from. A threshold quoted without its source is a number
a user cannot argue with, and this is a case where a user with a reason to
disagree is the one the message is for.

What this does not decide. The model still has no amorphous phase and no regrowth
path. That is the record for issue #8's scope statement and nothing here moves
it. What moves is what a run does when it is asked anyway, which is stop rather
than anneal as though the lattice were intact. The sentence in that record
describing the run as continuing is where a reader meets the two statements, and
changing it is an amendment to that record and belongs to issue #8.

The threshold is a number rather than a class, so it is not fixed here. It is a
per material quantity carrying provenance under the record for issue #11, in the
table issue #28 builds, and a run that cannot obtain one for the target material
has no threshold to compare a dose against. That case is already placed in the
section above: a property absent for a material whose parameter set otherwise
exists is a refusal naming the material and the property.

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
different things with them. A refusal is a case that needs a person to look at the
physics or the recipe. An input that could not be read is ordinarily a defect in
whatever produced it, and that thing's author is the one who fixes it. Which stop
falls on which side is the section below, because the sentence that used to stand
here answered it with one word and got it wrong.

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

## Which stop returns which status

The sentence in the section above used to say that an input which could not be
read means a missing file, a malformed recipe, or a parameter set that does not
parse. The middle term was wrong, and this is what was wrong with it rather than a
tidier rewrite of it. A malformed recipe is not one case. A document that is not
the format at all, a document that is the format and fails the schema, and a
document that passes the schema and fails a check reading more than itself are
three stops with three different people fixing them, and one word gave them one
status:

    git show 6b59c79a571beb6120146691da4d417745217aa5:docs/decisions/0014-error-and-failure-policy.md \
      | grep -n 'a malformed recipe'
    187:missing file, a malformed recipe, a parameter set that does not parse, is

The record for issue #9 names two validating stages and calls the failures of both
refusals. Only the second of the two is the class that exits 1 here:

    grep -n 'Issue #14 requires a refusal to' docs/decisions/0009-the-recipe-format.md
    110:suite those implementations are tested against. Issue #14 requires a refusal to

    grep -n 'refusals under issue #14' docs/decisions/0009-the-recipe-format.md
    131:refusals under issue #14 and they run after the schema passes and before the

Which status each stop returns.

A file that is missing, that cannot be read, or that is not the format at all
returns 2. Nothing about the case was established, and the fault is ordinarily in
whatever wrote the file.

A document that parses and fails the schema returns 2. Its message names the
offending field, the instance location, the schema location and the constraint
that failed, which is what the record for issue #9 requires of it, and naming the
field is a property of the message rather than of the status. Who fixes it is the
author of the document or of the script that wrote it, which is the test the
section above gives for this division.

A document declaring a schema version this build will not read returns 2. Nothing
about the case was evaluated and what is wrong is the pairing of the document with
the build, and the record for issue #9 already fixes that such a document is
stopped by name with the version it needs:

    grep -n 'A tool reading a major it does not know' docs/decisions/0009-the-recipe-format.md
    267:A tool reading a major it does not know refuses it by name. It does not attempt

A document that passes the schema and fails a check reading more than itself
returns 1. Whether a named material exists in the parameter set in force, whether
an etch names a mask that a masking step created, whether a step's parameters are
admissible together. Each of those needs a person to look at the physics or the
recipe, which is this record's own test for status 1, and the record for issue #9
places them here already:

    grep -n 'Whether a named material exists in the parameter set' docs/decisions/0009-the-recipe-format.md
    128:Whether a named material exists in the parameter set, whether an etch names a

A parameter set that does not parse returns 2, and that term of the original
sentence was right. A parameter set that parses and lacks a coefficient a recipe
uses returns 1, which is the material case placed further up this record taken a
step finer.

The word, since the two records use it differently and neither is wrong. In this
record a refusal is the class that stops the run and exits 1. In the record for
issue #9 refuse is the ordinary verb for a stage that will not proceed, and that
set is the larger one. Anything asserting a status therefore derives it from the
list above and not from the verb, which is what issue #58 needs before a fuzz
target exists: almost every input a fuzzer produces is a malformed document, so a
target asserting the refusal status would assert 1 against a reader this record
says returns 2. The other half of that trap is the rule in the section above, that
zero is never returned by a run which produced no result set, and it is the half a
target asserting only the absence of a crash walks straight past.

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

The two placements the amendment added are in the same position. Nothing reads a
parameter's domain of validity, which the record for issue #11 says of its own two
checks as well, so a run may extend a fit and record nothing and stay green. And
nothing compares the stop a run made against the status list above, so a reader
who wants to know which one a bad recipe returns has this record and no verdict
from a machine.

The two implant placements are in that position and one step further out. There
is no implant step for a method field to be absent from and no dose for a
threshold to be compared against, so nothing could refuse either even if
something read them. Both are also placed against records rather than against
code, so what stands behind them until an implant step exists is a reader who
opens this file.

## The means

Markdown in the repository, read by a person and quotable in an issue. A decision
record has to be readable from a fresh clone with nothing installed, it has to
sit in version control so that what was decided and when is recoverable from git
rather than from memory, and it has to survive being disagreed with on the
merits. It adds no language, no runtime and no dependency to a tree that today
holds no build. Nothing outside this repository forces the choice.
