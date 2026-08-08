# Where every model parameter comes from, and how it is cited

Decided by issue #11, milestone 1.

## Status

In force from the commit that lands this file.

The naming and numbering of decision records is fixed by issue #2, which is not
decided. This filename is provisional and #2 may change it. Nothing in the
content below depends on the name.

Issue #11 asks for a parameter failing the rule to be refused by a check rather
than by a reviewer noticing, and for a deliberately unprovenanced value to be
shown being refused. This record names the checks and supplies neither, for the
reason the section on enforcement gives. The issue stays open on that item.

## Why this is a decision rather than a convention

The models in this project are a small amount of mathematics and a large amount
of coefficients. Diffusivity prefactors and activation energies per dopant per
charge state, oxidation rate constants per orientation per ambient, segregation
coefficients per interface, defect formation and migration energies, solid
solubility curves. The mathematics can be checked by reading it. The
coefficients decide the answer and cannot.

The failure this prevents has hollowed out more than one open scientific code. A
number appears in a table. Nobody remembers where it came from. It is copied
into two other places. One of them is corrected and the others are not. By then
nobody can tell which is right, and the only honest response is to stop trusting
all three.

It is a decision rather than a convention because two of its consequences are
structural. Parameter sets have to be objects a result can name, which is a file
format and a loading path. And one home per quantity has to be refusable, which
is a check.

## The provenance rule

Every number in the tree that is not derived by the code carries, in the same
file, where it came from.

In the same file is the load bearing part. Provenance in a document beside the
data is provenance that drifts from the data, and the first correction to either
one separates them. A reader looking at a coefficient must be able to see its
origin without leaving the line.

## The three kinds

A quantity declares its provenance kind, and the kind is one of exactly three.
An entry whose kind is outside the set is refused rather than accepted with an
empty field, for the same reason issue #12 keeps three verdicts rather than two:
"came from somewhere else" and "cannot be placed" are opposite statements and
collapsing them loses the second.

A measured or published value cites the publication. The citation carries enough
to find the specific table, figure or equation the number was read from, not
only the paper: authors, title, venue, year, a persistent identifier where one
exists, and the locator within the work. A citation to a paper without a locator
is a citation to a search, and the person doing that search is the one who
inherits this code.

A fitted value names the fit and the command that reruns it. The name identifies
the fit as an object, the command is what reproduces the number from the data,
and the data is identified by content hash whether or not it lives in this tree.
A fitted value whose fit cannot be rerun is not a fitted value in this sense; it
is an assumed value that happens to have come from a regression, and it is
declared as the third kind with that as its reason.

An assumed value says it is assumed and says why. This kind is legitimate and it
is not a placeholder. Some coefficients genuinely have no measurement, and a
model needs a number anyway. What makes it legitimate is that it says so, so
that a reader comparing two results knows which of the numbers behind them were
chosen. An assumed value with no reason is refused, because the reason is the
only part of it that carries information.

Every kind also carries who recorded it and when, and every entry carries the
unit, under the rule in issue #7 that no number exists in this project without a
declared unit.

## The domain of validity

Every entry carries the conditions the number is good for, and this is part of
the provenance rather than a separate nicety.

A diffusivity prefactor fitted between 900 and 1100 degrees Celsius, used at
700, produces a plausible number that is wrong for a reason nothing in the
output mentions. That is the same failure shape as a missing citation, one step
later: the value is right and its applicability is not, and neither the value
nor the code knows.

So an entry states the ranges it was established over, in the quantities that
matter for it, which for a thermal coefficient is a temperature range and for a
concentration dependent one is a concentration range as well, along with the
material, the orientation and the ambient where those apply.

Using a parameter outside its stated range is not silent. It is recorded in the
manifest under issue #10 and it appears in the human summary, in the same way a
substitution does and for the same reason: a manifest field nobody reads is not
a disclosure. Which failure class that belongs in, meaning whether it warns or
refuses, is issue #14's decision and this record does not take it. What this
record fixes is that the range is recorded and that leaving it is visible.

An entry whose domain of validity is unknown says that, and unknown is a
different statement from unbounded. A number recorded as valid everywhere
because nobody checked is the assumed kind wearing the first kind's clothes.

## A parameter set is a named, versioned object

Not loose constants compiled into the code. A user must be able to say which set
produced a result, and to run the same recipe against two sets and see the
difference, and neither is possible if the numbers live in the source.

A set has a name, a version, and a content hash computed over its files. The
name identifies the set as a thing, the version identifies which revision of it,
and the hash is what a result records so that two sets claiming the same version
cannot be confused. Issue #10 already requires every parameter set in force to
be recorded by name, by version and by content hash, and this record supplies
the object that requirement refers to.

A set may extend exactly one base set. An entry in the extending set replaces
the base entry completely and carries its own full provenance rather than a
delta, because a delta is a number whose origin is half in another file. An
extending entry that names no entry in the base is refused, which catches the
case this rule exists for: a misspelled key that silently defines a new quantity
nobody reads while the base value continues to be used.

The manifest records the extending set and the base, both by name, version and
hash, because a result produced under an override is not a result produced under
the base.

## The file format

JSON, one or more files per set, with a set level document naming the set, the
version, the base it extends where there is one, and the files that belong to
it.

The reason is the reason issue #10 gives for the manifest and issue #9 gives for
the recipe: one scalar model and no implicit typing. A parameter file is a file
of numbers and provenance strings, which is precisely where a silent coercion
turns a correct record into a false one. Using the same format across the recipe,
the manifest and the parameter sets means one reader, one writer, and a manifest
that can quote a parameter entry without converting it.

An entry is an object rather than a bare number. It carries the value, the unit,
the provenance kind and that kind's required members, the domain of validity,
and an optional note. A bare number as a parameter value is refused, in the same
way and for the same reason that issue #9 refuses a bare number as a recipe
quantity.

The keys that identify a quantity are structured rather than a single string. A
diffusivity prefactor is identified by the species, the material, the charge
state and the mechanism, not by a name somebody composed out of those with
underscores, because a composed name is a key two files will spell differently.

## One home per quantity

A number that appears twice is a defect even when both copies agree, because
they will not agree forever.

The rule has two halves and they have different subjects.

Within a resolved parameter set, meaning after any extension is applied, every
quantity is defined exactly once. Two entries with the same structured key are
refused, and this holds within a set and across the files that make one up. An
override is not a second definition, because resolution leaves one.

In the source, a numeric literal that stands in for a quantity a parameter set
holds is refused. That is the rule that stops a coefficient being inlined into a
solver for convenience, which is how the second copy is created in practice.

The check is named `one-home-per-parameter`. Its subject is a quantity in a
resolved parameter set and a numeric literal in the source that shadows one. It
is deliberately separate from `one-home-per-constant` in issue #7, which has a
different subject, because one check over both would report a parameter defect
as a constant defect and the two are fixed in different places.

A second check is named separately rather than folded in. Its name is
`every-parameter-carries-provenance`, its subject is an entry in a parameter set
file, and it refuses an entry with no provenance, an entry whose kind is not one
of the three, an entry whose kind is present but whose required members for that
kind are not, and an entry with no domain of validity and no statement that the
domain is unknown. Two checks rather than one because they fail for different
reasons and a run should say which.

## What is not enforced, and the item this issue stays open on

Neither check exists. There is no parameter set, no parameter file and nothing
in the tree that could read one:

    git ls-files -- '*.c' '*.h' '*.cpp' '*.hpp' '*.rs' 'CMakeLists.txt' | wc -l
    0
    git ls-files -- '*.json' | wc -l
    0

Issue #11 asks for a deliberately unprovenanced value to be shown being refused,
which is the right thing to ask for and is exactly what proves a guard bites.
Nothing in this tree can refuse anything today, so there is no refusal to show
and this record does not claim one. Issue #11 stays open on that item alone.

The material property table in issue #28 is the first place a parameter set
actually exists, and issue #19 and issue #16 are where a check could first run.
That is the whole disclosure: until then, every rule above is read by a person,
and a parameter added without provenance lands green.

## Entry 5 of issue #1

Entry 5 asks whether reference measurement data is redistributed in this
repository or fetched, and it is open. This record assumes no answer and works
under either.

The rule is stated so that it survives both. Provenance is recorded whether or
not the data itself is redistributed. A citation is a citation whether the paper
is in the tree or in a library, and a fit names its data by content hash whether
those bytes are vendored here or obtained by the reader.

What does move with entry 5 is what a reader can do rather than what is
recorded. Where a digitised copy is in the tree, the fit named by a fitted entry
is rerunnable from a fresh clone with nothing else obtained, and the content hash
in the entry is the hash of a file the reader has. Where it is not, the same
entry names the same hash and the reader has to obtain the bytes before the
command runs, and if what they obtain hashes differently they know before they
have a wrong number rather than after.

An entry says which of those two it is, so that a reader can tell whether a fit
is reproducible for them without trying it. That single member is the whole of
what this record needs from entry 5, and it is recorded either way.

There is a genuine tension here that this record records rather than resolves. A
citation is not reproducible for a reader without access to the paper, which is
most readers. Digitising the curve makes it reproducible and makes this project
the publisher of somebody else's data. That is entry 5's argument to have, and
the reason it is not had here is that the answer differs source by source and
depends on terms a person has to read.

## What this record does not decide

The physical constants are issue #7. They are not parameters, they do not live
in a parameter set, and the separation is what stops a fitted number acquiring
the authority of a defined one.

Whether a calibrated set ships as a default is entry 3 of issue #1. Every
requirement above holds under all three of its options: a default set is a named,
versioned set like any other, and issue #10 already requires the set in force to
be recorded whichever way that entry goes.

What a validation case does with a parameter, and what it means when a
coefficient moves while a comparison is open, is issue #12, which already
decides that such a case no longer validates the coefficient. This record
supplies the object that makes the question answerable, which is a versioned set
a result names.

The material property table itself, meaning the first actual numbers, is issue
#28.

## The means

Markdown in the repository, read by a person and quotable in an issue. A decision
record has to be readable from a fresh clone with nothing installed, it has to sit
in version control so that what was decided and when is recoverable from git
rather than from memory, and it has to survive being disagreed with on the merits.
It adds no language, no runtime and no dependency to a tree that today holds no
build. Nothing outside this repository forces the choice.

JSON for the parameter sets themselves is a separate means answer, given above,
and it adds no runtime to this tree either, because nothing here reads it yet.
