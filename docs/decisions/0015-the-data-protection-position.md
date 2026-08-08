# The data protection position

Decided by issue #15, milestone 1.

## Status

In force from the commit that lands this file.

The naming and numbering of decision records is fixed by issue #2, which is not
decided. This filename is provisional and #2 may change it. Nothing in the
content below depends on the name.

Two items of issue #15 are not met by this record, and the sections below say
which and why rather than only this one. One of them is a conflict with a
sentence in the landed record for issue #10, and it is written out in full
rather than resolved quietly.

## Why a process simulator needs one

A tool that reads a recipe and writes a result looks like the last place a data
protection position is needed, which is exactly why such tools end up without
one. Nothing about the description involves a person until you look at what
lands in the files.

Three things carry personal data here whether anyone intended it or not.

Absolute file paths. On most machines a home directory contains the name of the
person who ran the simulation, and a result document that records where its
inputs were read from records that name.

Host and account identifiers. A manifest is tempted to record them because they
help somebody reconstruct what happened, and a host name in an organisation is
frequently a person's name or their asset number.

Free text. Issue #9 gives every object in a recipe an optional `note`, carried
into the manifest with the recipe rather than discarded. That field is the one
place a recipe holds whatever a user typed, and what users type in such fields is
names, project codes and internal references. It is a deliberate field and it is
worth having, and it is also the one this position most has to account for.

There is a second category that is not personal data and deserves the same care,
because the harm from losing it is comparable. A process recipe is frequently the
most closely held thing a manufacturer owns: thermal budgets, implant conditions
and layer thicknesses are what distinguishes one product from another. A result
document contains the recipe by construction, because issue #10 requires the
recipe bytes to be in the result set. So a result set is a container of somebody's
most guarded input, and anything that moved a result set off the host would move
that with it.

## The position

Nothing computed on the operator's machine leaves it.

The tool makes no network connection in normal operation. It sends no telemetry.
It reports no crash anywhere. It consults no remote service in order to run, and
it maintains no shared cache, keyed on recipe contents or on anything else.

Personal data never leaves the host unless the operator deliberately federates,
meaning the operator configures an outward connection themselves, knowing what it
carries. There is no default that federates, and there is no configuration in
which federation happens without having been switched on by a person.

Two of those are worth spelling out because they are the specific shapes this
would otherwise take. A crash report containing the input that caused the crash
is an exfiltration of exactly the recipe the paragraph above describes, and it is
the most normal-looking feature a tool of this kind could add. A cache keyed on
recipe contents is the same thing wearing a performance argument, because the key
is derived from the guarded input and the lookup discloses it.

## What that statement is today, said accurately

It is a commitment binding work that has not started. It is not a measurement of
a program.

There is no program:

    git ls-files -- '*.c' '*.h' '*.cpp' '*.hpp' '*.rs' 'CMakeLists.txt' | wc -l
    0

So "the tool makes no network connection" is true in the way that any statement
about a program that does not exist is true, and reading it as a verified
property of something running would be reading more into it than it holds. What
it does is bind every later issue: a change that adds a network call in a normal
run is a change against this record, and it is refused on that basis rather than
argued on its own merits.

Writing it down before the code exists is the point. A position adopted after
telemetry has been added is a position that has to remove something, and removing
a feature is a different argument from never having it.

## Every network access this project can make

Enumerated in one place, which is this section, with what each one sends.

Today the list is empty. Nothing exists to reach the network.

One access is anticipated and it is named now so that it arrives inside a
position rather than beside one. Retrieving reference measurement data, where
entry 5 of issue #1 decides that data is fetched rather than redistributed, would
reach the network. If it exists it is a separate action the operator invokes by
name, never a side effect of running a simulation, and the documentation states
exactly what is requested and from where. What it sends is a request for a named
document at a named location, and nothing about the operator's recipe, results or
machine travels with it.

That entry is open and this record assumes no answer to it. Under the answer
where reference data is redistributed in the tree, the access does not exist and
this list stays empty.

Any access added later is added to this list in the same change that adds it. A
network call in this project with no entry here is a defect against this record.

Nothing enforces that. Issue #15 asks for a check or a test demonstrating that a
normal run makes no network access, and there is nothing to run and nothing to
observe. That item is not met and this paragraph is the whole disclosure of it.
The harness in issue #24 is where such a test would live, since it is the one
that already has to deal with what needs the network.

## What the manifest records, and the conflict with the record for issue #10

The rule this record proposes is that the manifest records what changes the
answer, and that an identifier of a person or of their machine does not.

A user name and a host name change nothing about the numbers, so they are not
recorded by default. A user who wants them for their own traceability switches
them on, and the switch is documented next to the statement of the position
rather than in a reference appendix, because a control nobody finds is a control
that is not there.

Absolute paths are not recorded. A path is recorded relative to a stated root,
and the root is named in the manifest as a root rather than expanded. That keeps
a result set portable as well as clean, which is a second reason for the same
rule.

This is the opposite of the usual default, and the reason is what a result
document is for. It is a thing people attach to issues, papers and emails. A
default that is safe on the machine that wrote it and unsafe the moment it is
sent is the wrong default for an artefact whose purpose is to be sent.

The conflict, stated rather than smoothed over. The landed record for issue #10
says of wall clock time, the host name and the amount of memory used that "they
are recorded, and they are recorded in a part of the manifest that says on its
face that nothing in it affects the answer". On the host name that is the
opposite of the rule above.

    git grep -n -A3 'Does it belong to the run rather than to the answer' \
      -- docs/decisions/0010-result-document-and-run-manifest.md
    docs/decisions/0010-result-document-and-run-manifest.md:79:Does it belong to the run rather than to the answer? Wall clock time, the host
    docs/decisions/0010-result-document-and-run-manifest.md-80-name and the amount of memory used change nothing about the numbers. They are
    docs/decisions/0010-result-document-and-run-manifest.md-81-recorded, and they are recorded in a part of the manifest that says on its face
    docs/decisions/0010-result-document-and-run-manifest.md-82-that nothing in it affects the answer, so that a reader comparing two manifests

Both records agree on the reasoning and disagree on the conclusion. Issue #10's
sentence separates fields by whether they affect the answer, which is the right
axis for a manifest and is the wrong axis for this question: a field that affects
nothing can still identify somebody. Wall clock time and memory used are
unaffected by this record and stay exactly as issue #10 has them. The host name
is the single field in dispute.

This record does not edit that one. An amendment to the record for issue #10 is
owed, saying that the host name is recorded only when switched on, and it belongs
with that record rather than being made from this one, because a record amended
from outside is a record whose own text no longer says what it decided. Issue #15
stays open on that item, since its own done-when asks for the manifest defaults in
issue #10 to match this position and today they do not.

Until that amendment lands, the two records disagree in the tree and a reader
finding only one of them gets a different answer depending on which. Saying so is
what this section is for.

## The controller

The operator is the controller of anything personal that ends up in their files.
This project ships nothing that processes personal data on anyone else's behalf,
because it processes nothing on anyone else's behalf at all.

That is an accurate description rather than a disclaimer, and the difference
matters. A disclaimer tries to move a responsibility. This sentence says where
the responsibility already sits, which follows from the position above: if
nothing leaves the host, there is no second party to allocate anything to.

Saying it plainly in the documentation is worth more than a policy nobody reads,
and it is the register the documentation uses. A user who wants to know what the
tool does with their name should find the answer in a paragraph they understand,
not in a legal appendix.

## Where this is written for a reader

The readme carries the statement, so a reader who never opens the tracker learns
it from the front page. That is done by the change that lands this record.

Issue #15 also asks for it in the user documentation. There is none:

    git ls-files docs/
    docs/decisions/0003-the-boundary.md
    docs/decisions/0005-what-a-structure-is.md
    docs/decisions/0006-discretisation-and-moving-boundaries.md
    docs/decisions/0007-the-numeric-contract.md
    docs/decisions/0008-diffusion-model-family.md
    docs/decisions/0009-the-recipe-format.md
    docs/decisions/0010-result-document-and-run-manifest.md
    docs/decisions/0011-parameter-provenance.md
    docs/decisions/0012-what-validation-means.md
    docs/decisions/0013-the-handoff-to-device-simulation.md
    docs/decisions/0014-error-and-failure-policy.md
    docs/decisions/0015-the-data-protection-position.md

Everything in the tree is a decision record, and a decision record is not user
documentation. The tutorial in issue #64 is the first user facing document this
project will have, and the statement belongs in it together with the switch that
changes the manifest defaults. Issue #15 stays open on that item too.

## Entry 2 of issue #1

Entry 2 asks whether anything a run produces may ever leave the host, and it is
open. This record assumes no answer to it.

The relationship between the two is worth being exact about, because they look
like the same question. Entry 2 is about results and about the improvement loop:
whether this project ever builds a route for a user to send a calibration case
back. This record is about personal data and about defaults, and it says that
nothing federates unless a person switched it on.

Every one of entry 2's three options is compatible with that. Never is the
strongest form of it. Permitted, off by default and disclosed wherever it is on
is what this record already describes, applied to results rather than to personal
data. And the third option, where nothing is ever sent automatically and
contribution happens by a person opening an issue with their case attached,
changes nothing here at all, because a person attaching a file to a tracker is
not this tool sending anything.

What entry 2 would change is the wording available to the documentation. Under
never, the sentence about results can be unconditional. Under the second option
it acquires a qualifier that every later sentence has to carry. This record's own
sentence about personal data is unconditional under all three, and that is why it
is written as its own statement rather than as a consequence of entry 2.

## What this record does not decide

Whether results may ever leave the host is entry 2 of issue #1.

Whether reference data is redistributed or fetched is entry 5 of issue #1, and it
decides whether the network access named above ever exists.

What the manifest holds and how it is structured is issue #10, which this record
amends in one field and otherwise inherits.

Which failure class a refusal in this area belongs to is issue #14.

Nothing here is a statement about applicable law or about what any particular
operator's obligations are. It is a statement about what this software does.

## The means

Markdown in the repository, read by a person and quotable in an issue. A decision
record has to be readable from a fresh clone with nothing installed, it has to sit
in version control so that what was decided and when is recoverable from git
rather than from memory, and it has to survive being disagreed with on the merits.
It adds no language, no runtime and no dependency to a tree that today holds no
build. Nothing outside this repository forces the choice.
