# Result document, run manifest and what a reader can reconstruct

Decided by issue #10, milestone 1.

## Status

In force from the commit that lands this file. It supersedes nothing, because
it is the first record of its kind here.

Amended once since, in one field. The host name is no longer recorded by
default, and the section on the host name and the user name is the amendment and
says what it replaced.

The naming and numbering of decision records is fixed by issue #2, which is not
decided. This filename is therefore provisional and #2 may change it. Nothing
in the content below depends on the name.

## Why this is a decision rather than an implementation detail

The number a run produces outlives the run. It goes into a report, a paper or a
design review, and the question asked about it later is not whether it looked
plausible but what produced it. If that question can only be answered by the
person who was at the keyboard, the number is an anecdote.

Deciding the answer after the first solver exists means the manifest is shaped
by what happened to be reachable from the code that got written, which is the
wrong way round. The manifest decides what the code has to keep hold of.

## What a run writes

A run writes one result set. A result set is a directory, and it holds

- one manifest, machine readable,
- one summary, human readable,
- the recipe bytes the run was given,
- one authoritative structure artefact,
- zero or more projections of that structure.

Nothing a run produces lands outside a result set. A run that writes a number
to the terminal and nowhere else has produced no result, and the terminal
output is a convenience rather than an artefact.

## The manifest is JSON, and the summary is generated from it

JSON is chosen for one property that matters more here than expressiveness: it
has one scalar model and no implicit typing. A version string, a dose written
in exponent notation and a material name all survive a write and a read
unchanged. Every language a user might query a sweep from has a reader, and a
writer that sorts its keys produces a file that diffs readably between two runs,
which is what turns two manifests into an answer about what changed.

YAML costs exactly the property that was bought. Its implicit typing rules turn
some unquoted strings into numbers and some into booleans, and the fields this
document exists to protect are provenance fields, where a silent coercion is a
corrupted answer about where a number came from. TOML is comfortable for flat
configuration and awkward for the nested, array shaped thing a manifest is, and
it would be fought at every level of nesting. A format of this project's own
costs a parser on the reading side, and the reading side is somebody else's
script years from now.

The summary is generated from the same manifest object, in one pass, after the
manifest is complete. It is a rendering and never a second writer. Two writers
can disagree, and a reader holding the one that is wrong has no way to tell
which one they have. The cost of this is that the summary can only say what the
manifest holds, which is intended: a fact worth putting in the summary is a fact
worth putting in the manifest first.

## The rule for what the manifest holds

Anything that changes the answer is recorded.

That is stated as a test rather than as a list, so the list can grow without the
argument being had again. A candidate field is put to three questions.

Could the result differ if this had been different? If yes, it is recorded.
This is the whole of the rule and the other two questions only refine it.

Is it already derivable from something the manifest holds, by a route this
record or a later one names? If yes, it is not recorded, and the route is named
where the derived thing would have gone. Recording both a thing and its
derivation creates two places that can disagree.

Does it belong to the run rather than to the answer? Wall clock time and the
amount of memory used change nothing about the numbers. They are recorded, and
they are recorded in a part of the manifest that says on its face that nothing
in it affects the answer, so that a reader comparing two manifests knows which
differences matter.

There is a fourth question and it was missing. Could this field identify a
person? A field can affect nothing and still name somebody, and the three
questions above sort fields on an axis that cannot see that. The host name is
the field where the two axes disagree, and the section below is where it is
settled.

## The host name and the user name

The host name is recorded only when the operator switches it on. The user name
is the same, and it was never in the list above because nothing had proposed it.
Neither is recorded by default, and the switch that records them is documented
beside the statement of the data protection position rather than in a reference
appendix, because a control nobody finds is a control that is not there.

Wall clock time and the amount of memory used are unaffected by this and stay
exactly where the paragraph above puts them. They do not name anybody.

Absolute paths are not recorded either. A path is recorded relative to a stated
root, and the root is named in the manifest as a root rather than expanded.
That keeps a result set portable as well as clean, which is a second reason for
the same rule.

The reason is what a result document is for. It is a thing people attach to
issues, papers and emails, and a default that is safe on the machine that wrote
it and unsafe the moment it is sent is the wrong default for an artefact whose
purpose is to be sent. Recording a host name costs the reader nothing they
cannot ask for, and in an organisation a host name is frequently a person's name
or their asset number.

This paragraph is an amendment. The text it replaced put the host name in the
part of the manifest that carries what belongs to the run rather than to the
answer, and it was right that it belongs there and wrong that it is recorded by
default. The record for issue #15 found the disagreement, wrote it out rather
than smoothing it over, and left the repair here, because a record amended from
outside is a record whose own text no longer says what it decided. The reasoning
is in that record and is not restated here:

    git grep -n 'the conflict with the record for issue #10' \
      -- docs/decisions/0015-the-data-protection-position.md

Nothing refuses a manifest that records a host name anyway. There is no manifest
writer and nothing that could read one:

    git ls-files -- '*.c' '*.h' '*.cpp' '*.hpp' '*.rs' 'CMakeLists.txt' | wc -l
    0

So this is a constraint on the writer that issue #10 owes rather than a
property of anything running, in the same sense as the check this record already
owes further down.

## What the manifest holds under that rule

Grouped by what the group is for rather than by field type.

What the run was asked to do. The recipe, by content hash, with the recipe bytes
stored in the result set beside the manifest rather than embedded in it. Every
parameter set that was in force, by name, by version and by content hash, which
matters whether or not a default set ships, because a default that is not
recorded is a number with no provenance. The model selection, meaning which rung
was active and which species existed, which issue #8 fixes and this record only
carries. The tolerances and the convergence criterion, which issue #7 fixes.
Whatever the determinism position in #7 makes part of the input, which for a
position of bit reproducibility at a fixed thread count includes the thread
count.

What the code was. The commit the executable was built from, and whether the
working tree was clean when it was built, which issue #16 owes the executable.
The build configuration and the compiler identification, because one commit
built two ways is two programs, and a reader trying to reproduce a last-bit
difference needs to know which one they have.

What the run actually did, as distinct from what it was asked to do. The time
steps taken. The Newton iteration counts, and every step that reached an
iteration limit and continued rather than stopping. The achieved conservation
figure for every transfer of a field between two states of the domain, which
issue #6 fixes the tolerance for and which is recorded as the figure achieved
rather than as the fact that it passed. And every substitution.

## A substitution is recorded as a substitution

A default that was used because the recipe did not specify anything is not the
same fact as a value the operator chose, and a manifest that writes them
identically has destroyed the difference. Recording the substitution as a value
is the most common way a result becomes irreproducible while looking complete,
because everything is present and nothing says which parts were asked for.

So a substitution is recorded as what was missing, what was used in its place,
where that replacement came from, and that it was a substitution. It appears in
the summary as well as in the manifest, because a manifest field nobody reads
is not a disclosure.

Issue #14 owns the failure policy and decides which failures are substitutions.
What this record fixes is the other half: recording the substitution is the only
way to perform one, so a code path that substitutes without recording is a
defect against this record rather than a style question.

## The result itself

The full structure is authoritative. A table of depth against concentration is
what most users want and it is a lossy projection of a two dimensional
structure, so it exists and it is not the result.

Every projection carries, on its face rather than in a sidecar, that it is a
projection, the content hash of the structure it was taken from, and the
operation that produced it. A projection that has been separated from its result
set can then still be traced back, and one that cannot be traced says so.

The on-disk encoding of the structure is not decided here. Issue #5 decides what
a structure is and issue #13 decides what a device simulator receives, and the
encoding follows both. What this record fixes is that there is exactly one
authoritative structure artefact per result set, that the manifest names it by
path and by content hash, and that everything else derived from it declares
itself derived.

## What a reader can reconstruct from a result alone

A reader holding only a result set can say what was computed, from which inputs,
under which models, with which tolerances, and from which commit. They can re-run
it, because the recipe bytes and every parameter set are in the result set and
the commit is named. They can tell which of the numbers in front of them the
operator asked for and which the run supplied.

What they cannot do, with the missing thing named in each case rather than
gestured at.

They cannot rebuild the executable. A commit and a compiler identification are
not a compiler. The toolchain is pinned in the tree by issue #16 and the result
set does not carry it.

They cannot predict how much of a difference on another machine is legitimate,
until issue #7 fixes the determinism position. This record does not guess at the
size of that set and will carry whatever #7 decides.

They cannot obtain the measurements a validation claim is compared against,
where issue #12 and entry 5 of issue #1 leave those outside the tree. The
manifest names them; naming is not holding.

They cannot recover why the operator ran this case. A manifest is an account of
what was done.

Neither list is padded. Every entry in the second names the specific thing that
is missing, so that supplying it moves the entry to the first list rather than
requiring this record to be reworded.

## The check this record owes

Milestone 10 owes a check that refuses a result written without a complete
manifest. Its name is `result-carries-a-complete-manifest`. Its subject is a
result set written by a run. It refuses when the result set holds no manifest,
when the manifest omits a field the rule above requires for the run that was
performed, or when a projection names no structure.

Naming it here is not having it. No issue in milestone 10 held it when this
record was written, and the tracker moves, so re-run this rather than trusting
the paste:

    gh api repos/iderex/schichtwerk/milestones --paginate \
      --jq '.[] | select(.number == 10) | "\(.title) open=\(.open_issues) closed=\(.closed_issues)"'
    10. Quality parity program open=0 closed=0

Until that check exists, a result written without a complete manifest is
refused by nothing, and this section is the whole disclosure of that.

## Every number written by a later milestone goes through this path

Any issue in milestones 4 to 9 that produces a number writes it through the
manifest rather than beside it. At the commit that lands this file none has,
because the tree holds no code that could:

    git ls-files -- '*.c' '*.h' '*.cpp' '*.hpp' 'CMakeLists.txt' | wc -l
    0

The obligation therefore binds work that has not started, and it is stated here
so that an issue in those milestones is read against this record rather than
this record being amended to fit an issue.

## What this record does not decide

Whether a calibrated parameter set ships as a default is entry 3 of issue #1.
This record requires the set in force to be named whichever way that goes.

Whether anything a run produces may ever leave the host is entry 2 of issue #1.
Everything above concerns files written to the operator's own disk and decides
nothing about that entry in either direction.

The structure encoding is issue #5 with issue #13. The recipe language is issue
#9. The unit system, the constants, the convergence criterion and the
determinism position are issue #7. The model rung and the species are issue #8.
The conservation tolerance is issue #6. The failure classes are issue #14. In
each case the manifest carries what those records decide, and this record fixes
only that it is carried.

## The means

Markdown in the repository, read by a person. A decision record has to be
readable by somebody who has just cloned the tree and has nothing installed, it
has to sit in version control so that what was decided and when is recoverable
from git rather than from memory, and it has to be quotable in an issue. It adds
no language, no runtime and no dependency to a tree that today holds no build.
Nothing outside this repository forces the choice.
