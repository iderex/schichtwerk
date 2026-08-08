# How a process recipe is written down and how it is read

Decided by issue #9, milestone 1.

## Status

In force from the commit that lands this file.

The naming and numbering of decision records is fixed by issue #2, which is not
decided. This filename is provisional and #2 may change it. Nothing in the
content below depends on the name.

One item of issue #9 is not settled here and the section on the validator says
which: the record fixes the dialect, the conformance requirement and the
refusal, and does not name a library, because naming one decides issue #2.

## Why this is a decision rather than an implementation detail

The recipe is the surface every user touches first, and it is the one thing that
cannot be changed quietly later, because changing it breaks every file anybody
has written. A format that grew out of whatever the first solver could parse is
a format nobody chose.

## The three shapes

A declarative data format, meaning a document that lists steps and their
parameters with no control flow, is the easiest to validate, the easiest to
diff, the easiest to generate from another tool, and the easiest to refuse
precisely when it is wrong. It costs the parameter sweep. A user who wants
twenty anneal temperatures writes twenty files or reaches for a script that
generates them, and the scripts people write for that are the ones that go
wrong quietly.

An embedded scripting language, which is the route the incumbent tools took and
which issue #9 names FLOOXS as having taken with Tcl, gives loops, sweeps and conditionals for free and
turns every recipe into a program. It costs validation, because a recipe that
only reveals its steps by running is a recipe no check can inspect beforehand.
It costs security in a specific way worth naming rather than gesturing at: a
recipe received from a colleague becomes code that executes on the receiving
machine, with that machine's file system and network reachable from it. Nobody
receiving a process flow expects to be receiving a program.

A declarative format with a separate, explicit sweep description keeps both
properties. The recipe stays inspectable, and the thing that varies it is a
small bounded construct rather than a general language. It costs a second
concept to learn, and it costs the users who genuinely want a conditional and
will not get one.

The third is chosen. The declarative document is the only thing a solver ever
reads, and a sweep is expanded into a set of concrete recipes before anything is
executed, each of them recorded in the run manifest under issue #10.

## The format

JSON.

The property being bought is the one issue #10 already bought for the manifest:
one scalar model and no implicit typing. A recipe is a document of physical
quantities and provenance, and both are places where a silent coercion is a
wrong answer rather than a cosmetic annoyance.

YAML costs exactly that property, and the cost is live here rather than
theoretical. Under YAML 1.1 rules, which several widely used loaders still
implement, an unquoted `on`, `off`, `yes` and `no` are booleans, and this field
writes ambients and switches with exactly those spellings. An unquoted `1e3` is
a string under 1.1 and a float under 1.2, so the same recipe file means two
different anneal times depending on which loader read it. A wafer orientation
written `100` is a number where the schema wants a name. Each of those has a
workaround a careful author knows, and the format's defect is that a careless
one produces a file that loads without complaint and means something else.

TOML is comfortable for flat configuration and awkward for the nested, array
shaped thing a recipe is, and it has no schema language with the maturity the
next section requires. Issue #14 asks for a bad recipe to be refused with a
message naming the offending field, and a hand written validator with hand
written messages is the thing that decays first.

A format of this project's own costs a parser on every reading side, including
the ones that are not this project. A recipe is a thing people generate from
scripts and read from other tools, and every one of those needs a reader.

Using one format for the recipe and for the manifest means one reader, one
writer and one set of habits, and it means a manifest can quote a recipe
fragment without a conversion in the middle.

What JSON costs, stated rather than left for the first user to find. It has no
comments, and a process engineer writing a recipe by hand wants to record why
the anneal is at 950 rather than 1000. It is unpleasant to hand edit, because a
trailing comma is a parse error and there is no block scalar for a long string.
Both are real, and the first is answered below rather than dismissed.

Comments are data rather than syntax. Every object in the schema admits an
optional `note`, a string, at the document, the step and the parameter level. It
is carried into the manifest with the recipe rather than discarded, which is
strictly more than a syntactic comment would give: a comment in syntax is lost
the moment a sweep expands a recipe into twenty concrete ones, and a `note`
field survives into all twenty and into every result document.

That field is the one place a recipe holds free text, so it is the one place a
recipe holds whatever a user typed. Issue #15 owns what that means for personal
data and for what a result document carries out of the operator's machine, and
this record only says that the field exists and where it goes.

## The schema language, and what the validator is

JSON Schema, dialect 2020-12.

It is chosen because it is the one schema language for this format with a
specification, with independent implementations, and with a public conformance
suite those implementations are tested against. Issue #14 requires a refusal to
name the offending field, and JSON Schema's output format carries the instance
location of every failure, so the message is derived from the validation result
rather than written by hand at each site. A message written by hand at each site
is a message that drifts from the rule it describes.

The validating stage is a named thing in this project rather than a step inside
a solver. Its name is `recipe-validates-against-the-schema`. Its subject is a
recipe document as read from disk, before any expansion and before any step is
dispatched. It refuses a document that fails the schema, and its message carries
the instance location, the schema location and the constraint that failed. It
runs to completion and reports every failure rather than the first, because a
user fixing a recipe one refusal at a time is a user running the tool nine
times.

There is a second stage, and it is named separately so that neither absorbs the
other. A schema expresses the shape of a document and cannot express a
constraint that spans two of them or that depends on the parameter set in force.
Whether a named material exists in the parameter set, whether an etch names a
mask that a masking step created, and whether a step's parameters are physically
admissible together are checks that read more than the document. They are
refusals under issue #14 and they run after the schema passes and before the
first step executes. Nothing physical is computed to decide them.

This record does not name a validator library, and that is a gap rather than an
omission. The language and the toolchain are issue #2 and are not decided, so a
library named here would decide #2 from the wrong document. What is fixed here
is what the implementation has to be: a conformant implementation of dialect
2020-12, pinned by version under issue #25, carrying the instance location in
its output, and evaluating the whole document rather than stopping at the first
failure. Issue #2 picks the artefact that meets that; issue #31 is where it
first has to exist.

## The shape of a step

Precise enough that issue #31 can dispatch on it and that the fuzzing issue in
milestone 10 has a grammar to work against. The tracker moves, so re-run this
rather than trusting the paste:

    gh issue list --repo iderex/schichtwerk --state open --limit 300 \
      --json number,title --jq '.[] | select(.number == 31 or .number == 58) | "\(.number) \(.title)"'
    58 Fuzz the recipe reader and the structure reader
    31 The command line: read a recipe, run it, write a result

A recipe document has three required members and nothing else at the top level:
a `schema_version`, a `wafer` describing the starting structure, and `steps`, an
array. An optional `note` is admitted alongside them. Additional members are
refused rather than ignored, at every level of the document, because a
misspelled parameter that is ignored is a recipe that ran with a default nobody
asked for.

`steps` is ordered and the order is the process order. It is an array rather
than a mapping for that reason: a mapping has no order a reader can rely on, and
the one thing a process flow has is a sequence.

A step is an object with one required member, `step`, whose value names the kind
of step. The set of kinds is closed and it is fixed in the schema rather than in
the dispatcher, so an unknown kind is refused by the validator with a location
rather than by a default branch somewhere inside the run. The kinds are
`implant`, `anneal`, `oxidize`, `deposit`, `etch` and `mask`, which are the five
operations issue #5 states against the structure plus the masking step issue #3
places at the boundary with lithography.

Every other member of a step is a parameter of that kind, and which parameters a
kind takes, which are required and what each admits is fixed per kind in the
schema. A step also admits an optional `id`, a string unique within the
document, which a sweep names and which the manifest carries so that a result
can be traced to the step that produced it. And an optional `note`.

A parameter that is a physical quantity is an object with a `value` and a `unit`,
both required. A bare number is refused.

That last sentence is the one this record would defend hardest. Issue #7 exists
because this field mixes centimetres, microns, nanometres and angstroms in one
paragraph, and almost every published wrong number of this kind is a unit
converted twice or not at all. A format that admits a bare number is a format
where a unit is assumed, and an assumption at the input boundary is the one that
no later check can catch, because by then the number is just a number. It costs
verbosity in every recipe, and it is worth it. Which units are admitted for
which quantity, and what the internal system is that they convert into, is issue
#7 and is not restated here.

A parameter that is a name, such as a material, an ambient or a mask, is a
string, and whether the name resolves is the second stage above rather than the
schema.

## Executable recipes

A recipe is never executed. There is no expression, no reference to another
field, no conditional, no loop and no include. The document a user writes is the
document a solver reads, and every value in it is a literal.

What a user asking for loops gets, in order of what they were probably asking
for.

If they want the same flow at several conditions, that is a sweep, and it is the
next section. It is the case that motivates the question almost every time.

If they want a flow assembled from reusable fragments, they generate the recipe
with their own script, in whatever language they already use, and the tool reads
the result. This project supplies no template language, and the reason is that a
template language is the scripting language arriving through a side door. What
this project owes such a user is that the format is trivial to generate, which
JSON is, and that the generated document is validated before anything runs,
which it is.

If they want a conditional on the result of an earlier step, meaning an anneal
whose time depends on a junction depth the previous step produced, they do not
get it, and this is the case where the answer is nothing. That is a real
capability the incumbent tools have and this one will not, and it is refused
rather than deferred. Admitting it makes the recipe a program, and every
property in this record follows from it not being one.

## Sweeps

A sweep is a separate document that names a recipe and the parameters to vary.
It is expanded before anything is executed, and each concrete recipe the
expansion produced is a recipe in the sense above, validated in its own right
and recorded in the manifest of the run that used it.

The constructs are bounded and there are three. A list of literal values for one
parameter. A range for one numeric parameter, given as a first value, a last
value and a count, so that the values are computed from the endpoints rather
than accumulated by repeated addition, which is how a sweep of a hundred
temperatures ends one step short. And a product of several such axes, whose
expansion order is stated in the record of the run rather than left to the
implementation.

There is no expression in a sweep either, and no axis may depend on another. A
user who needs that generates the recipes themselves, which is the same answer
as above and for the same reason.

Whether a sweep runs its cases in one process, in several, or not at all is
issue #31, and nothing about the expansion depends on the answer.

## The versioning rule

`schema_version` is required in every recipe and in every sweep. A document
without one is refused, and no version is inferred from the shape of the
document. Inferring it is how a file written for one meaning gets read under
another.

The version is a major and a minor number. The minor increments when a member is
added that is optional and whose absence means exactly what it meant before, so
that a document written under an earlier minor of the same major has the same
meaning under a later one. The major increments for anything else: a member
removed, a member that becomes required, a member whose admitted values change,
and above all a member whose meaning changes while its name and type stay the
same. That last case is the one this rule is for, and it is the one where the
temptation to call it a minor is strongest.

A tool reading a document whose major it knows and whose minor is at or below
its own runs it. A tool reading a minor above its own refuses it, naming the
minor the document declares and the highest it supports, because a document
using a member the tool does not know is a document whose author expected
something to happen.

A tool reading a major it does not know refuses it by name. It does not attempt
a best effort read. A recipe written today therefore either still runs in five
years, or is refused with a message saying which version it needs, and there is
no third outcome where it runs and means something else.

Support for an old major is dropped by a release deciding to, and the refusal
message says so. Dropping it is a change to this record naming what was dropped
and what a user with such a file does instead.

The schema documents live in the tree, one file per major version, and an old
major's schema stays in the tree after support is dropped, because a user
holding an old recipe needs to know what it meant even when this tool will no
longer run it.

Nothing above is refused by a machine today:

    git ls-files -- '*.c' '*.h' '*.cpp' '*.hpp' '*.rs' 'CMakeLists.txt' | wc -l
    0

There is no schema file in the tree either, and the tree holds no recipe:

    git ls-files -- '*.json' '*.schema.json' | wc -l
    0

Issue #31 is where the reader first exists and issue #58 is where it is fuzzed.
Until then every rule above is read by a person.

## Entry 4 of issue #1

Entry 4 asks how far the recipe language follows the incumbent's. It is open and
this record assumes no answer.

Nothing above changes under any of its three options. A translator from the
incumbent's command language into this format is a separate tool, its output is
an ordinary recipe document, and the format does not move if one is written or
if none is. A mapping table published as documentation is documentation. And an
input format of this project's own with nothing beside it is what this record
describes on its own terms.

One interaction is named rather than left for entry 4 to discover. The largest
of entry 4's options is reading the incumbent's command language directly, and
that language is a command language: it has control flow, and a file in it is a
program. This record's position on execution applies to any input path, so
taking that option means parsing that language rather than executing it, and a
parser for a language with control flow either refuses the control flow or
implements an interpreter. Entry 4 should weigh that cost as part of the option
rather than meeting it afterwards. Saying so assumes no answer, and the reason
it is said here is that the cost is invisible from entry 4's side.

## What this record does not decide

The unit system, the admitted unit spellings, the conversion boundary and the
tolerances are issue #7.

Which parameters each step kind takes, and what each one means physically, are
the milestone issues that implement those steps. This record fixes that the set
is closed, that it is fixed in the schema, and that an unknown member is
refused.

What a starting wafer is, which the `wafer` member describes, is issue #5 with
issue #30. This record fixes only that a recipe names it in the document rather
than depending on an ambient state.

What happens when validation fails, meaning which class the failure is placed
in and what exit status it carries, is issue #14. This record fixes that it is a
refusal before anything runs.

Whether a calibrated parameter set is in force by default, which a recipe would
otherwise have to name, is entry 3 of issue #1. A recipe naming a parameter set
explicitly is valid under either answer.

## The means

Markdown in the repository, read by a person and quotable in an issue. A decision
record has to be readable from a fresh clone with nothing installed, it has to sit
in version control so that what was decided and when is recoverable from git
rather than from memory, and it has to survive being disagreed with on the merits.
It adds no language, no runtime and no dependency to a tree that today holds no
build. Nothing outside this repository forces the choice.

The format the record chooses is a separate means question with its own answer
above, and JSON adds no runtime to this tree either, because nothing in the tree
reads it yet.
