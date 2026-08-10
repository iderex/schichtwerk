# The reference gate, and every deviation from it

Owned by issue #55, milestone 10.

Derived on 2026-08-08. The target moves, so the date is part of the document and
the commands below are how it is re-derived rather than trusted.

## Why the target is somebody else's gate

This board should not invent its own idea of what a finished gate looks like.
The public sibling repository `iderex/jellyfin-plugin-sso` has one that was built
up over a long time against real failures, and it is the target this milestone
adopts.

Adopting a target is not the same as copying it. Two of its elements protect a
property this project does not have, several are the same property through a
different mechanism because the language and the subject differ, and this project
needs checks that board has no reason to have, because a wrong number is its
characteristic failure rather than a wrong authorisation. All three directions
are below.

## How the target was derived

    gh api repos/iderex/jellyfin-plugin-sso/rulesets --jq '.[] | "\(.id) \(.name) \(.target) \(.enforcement)"'
    18802863 Protect main and 5.0 branch active

    gh api repos/iderex/jellyfin-plugin-sso/rulesets/18802863 \
      --jq '{enforcement, bypass:.bypass_actors, rules:[.rules[].type]}'
    {"bypass":[],"enforcement":"active","rules":["deletion","non_fast_forward","required_status_checks","pull_request"]}

    gh api repos/iderex/jellyfin-plugin-sso/rulesets/18802863 \
      --jq '[.rules[]|select(.type=="required_status_checks")|.parameters.required_status_checks[].context]'
    ["build","ABI floor build","Package (JPRM) / Build package","Package (JPRM) / Generate SBOM","CodeQL","Analyze (csharp)","DCO sign-off","Deterministic PR-hygiene checks","Enforce greppable invariants","Reject Trojan Source Unicode","Audit workflows (zizmor)","prettier","dependency-review"]

    gh api repos/iderex/jellyfin-plugin-sso/actions/workflows --paginate \
      --jq '.workflows[] | "\(.name)\t\(.path)"' | sort

which returned twenty-eight entries. Five of them are GitHub's own dynamic
workflows rather than files in that tree, and the twenty-three that are files are
the population mapped below:

    gh api repos/iderex/jellyfin-plugin-sso/actions/workflows --paginate \
      --jq '.workflows[] | .path' | grep -c '^dynamic/'
    5
    gh api repos/iderex/jellyfin-plugin-sso/actions/workflows --paginate \
      --jq '.workflows[] | .path' | grep -c '^\.github/workflows/'
    23

One element is inside a workflow rather than beside it and is derived separately,
because a coverage bar that fails closed is one of the things this milestone
exists to reach:

    gh api repos/iderex/jellyfin-plugin-sso/contents/.github/workflows/dotnet.yml \
      --jq '.content' | base64 -d | grep -n 'coverage bar'
    142:    - name: Enforce the security-surface coverage bar

Two things about the derivation are stated rather than implied.

The ruleset carries no bypass actors and is active, so its required set is what a
merge is actually held to on that board. That is the property this board's own
protection has to reach, and it is separate from the individual checks.

The mapping from a required context to the workflow file that produces it was not
fully resolved. Two contexts, `build` and `ABI floor build`, are jobs whose
workflow file was identified from a comment inside `dotnet.yml`, and there is a
separate `build.yml` in that tree. Nothing in the mapping below turns on which
file emits which context, because the mapping is by property, but a reader
wanting the file for a context should derive it rather than take it from here.

## The blocking set

The thirteen contexts a merge on that board is held to.

`build`. Adopted. The tree compiles and its tests run as a leg of one gate, which
is #17, and the workflow whose check name a protection rule can require is #18.

`ABI floor build`. Not adopted. That board packages a plugin against a host
application's binary interface and proves it still loads on the oldest supported
host; this project has no host and publishes no plugin.

`Package (JPRM) / Build package`. Adapted. The same property is that the shippable
artefact is built by the gate rather than by hand at release time, and here the
artefact is a runnable binary rather than a plugin package, delivered by #63.

`Package (JPRM) / Generate SBOM`. Adopted. #59 carries the bill of materials.

`CodeQL` and `Analyze (csharp)`. Adopted, and it needed an issue. Semantic
security analysis of the source is a different thing from the lint and warning
legs in #19: it runs deeper, it runs on a schedule as well as on a change, and
for a memory unsafe numerical core it is worth more here than it is there. #69
delivers it.

`DCO sign-off`. Adopted and already in this tree, at `.github/workflows/dco.yml`.

`Deterministic PR-hygiene checks`. Adopted, and it needed an issue. #70 delivers
it.

`Enforce greppable invariants`. Adapted. The property is a register of invariants
over the tree that a check refuses rather than a reviewer notices, and here that
register begins with #61, which refuses a document naming a path that does not
exist. The adaptation is that this board has one invariant today and that board
has a set; the mechanism is the same shape and #61 is where it grows.

`Reject Trojan Source Unicode`. Adopted and already in this tree, at
`.github/workflows/unicode-guard.yml`.

`Audit workflows (zizmor)`. Adopted and already in this tree, at
`.github/workflows/zizmor.yml`.

`prettier`. Adapted. The property is that formatting is refused rather than
advised and that there are no per file exemptions, and the tool is whatever
formats the language #2 chooses. #19 delivers it.

`dependency-review`. Adopted and already in this tree, at
`.github/workflows/dependency-review.yml`, and #59 is what makes it read a
change to the pins.

Five of the thirteen are therefore already in this tree, one is refused with its
reason, and the rest have a named home.

## The set that runs outside the merge gate

This half matters as much as the blocking set. A quality program is not only what
blocks a merge.

`.NET`, the test and coverage workflow. Adopted in two parts. The unit test
harness with a first test that can actually fail is #20, and the fail closed
coverage bar is #56. That board's bar is on its security decision surface; the
equivalent surface here is where a wrong number is decided, which is what #56 is
already written against.

`Stryker mutation testing`. Adopted. #57, on the solver core.

`Fuzz (SharpFuzz)`. Adopted. #58, on the recipe reader and the structure reader.

`Scorecard supply-chain security`. Adopted and already in this tree, at
`.github/workflows/scorecard.yml`.

`Repo Invariant Lint (Opengrep)`. Adapted, as above, through #61.

`E2E Login Harness`. Adapted. The property is that the thing a user actually does
is exercised end to end by machine rather than described. Here that is a recipe
going in and a device simulation coming out, which is #54, running under the
harness for what needs hardware, time or the network, which is #24.

`Wiki Lint`. Not adopted. This project's documentation lives in the tree, where
the checks that read the tree already reach it, so a lint for a second
documentation surface has no subject here.

`Manifest freshness`. Not adopted. It keeps a plugin repository manifest in step
with published releases, and this project publishes no such manifest.

`Regenerate manifest`. Not adopted, for the same reason as the line above.

`Nightly betas`. Not adopted. A nightly prerelease serves users who track a host
application's own release train, and this project has no such train to follow.

`Publish Beta`, `Publish JF12 Beta`, `Publish JF12 Stable`, `Publish Release`.
Adapted, as one element with four instances there and one here: the release is
built and published by a workflow that fails closed rather than by a person at a
terminal, which is #62. The four instances exist there because two host lines are
supported at two stability levels, and this project has neither axis.

`Publish failure alert`. Not adopted. It exists to make a silent publish failure
loud, and it is worth revisiting once #62 exists rather than now, because an
alert on a workflow that does not exist has nothing to watch.

`Build`. Adapted, and this is the entry the derivation caveat above touches. A
separate build workflow beside the test workflow is a shape that follows from
that board's packaging; here one gate runs the legs in order, which is #17, and
#18 is the single check name a protection rule requires.

`DCO`, `Dependency review`, `unicode-guard`, `Workflow Security Analysis`,
`Prettier Lint`, `PR Hygiene`, `CodeQL`. Workflow files whose contexts are in the
blocking set above and which are mapped there rather than twice here. Four of
those seven are already in this tree.

That accounts for every file in the target's workflow directory. The two
sections together map twenty-three files and thirteen contexts, and the count of
files is the one derived above rather than a count of the entries below, because
a list in a document drifts against the thing it describes.

## Two checks inside the target's build that are elements in their own right

They are named because a parity exercise that reads only workflow names misses
them, and both are steps inside `dotnet.yml`.

`Vulnerable dependency scan`. Adopted, and it needed an issue. #59 pins the
dependency set and reviews a change to it, and neither of those tells you that a
pin you have been carrying for six months has a published vulnerability. An
unmaintained pin is a security problem rather than a stability feature, which is
#25's own sentence, and nothing on this board reads the pins against an advisory
database. #71 delivers it.

`VEX document check`. Adapted. The property is that a published vulnerability
that does not affect this artefact is answered in a machine readable statement
rather than in a mailbox, and the natural home here is the bill of materials
work, so #59 carries it.

`Jellyfin compatibility metadata check`. Not adopted. It verifies metadata about
a host application this project does not have.

## What this project needs that the target does not have

Presenting parity as the ceiling is the failure a parity exercise usually makes.
That board's characteristic failure is a wrong authorisation. This one's is a
wrong number, and a number can be wrong while every check above is green.

Determinism. The same case, on the same build, at the same thread count, twice,
byte identical. #22 refuses a difference and the record for #7 states the
criterion so that #22 has nothing left to judge. Nothing in the target gate has
this shape, because a wrong authorisation does not usually reproduce differently
on two runs.

Conservation. A field transfer that loses dopant produces a smooth, monotone,
entirely plausible profile, and every check that looks at the shape of a curve
passes. The tolerance is fixed as a number a check compares against in the record
for issue #6, and the Done-when of issue #48 is where the refusal is asked for:

    gh issue list --repo iderex/schichtwerk --state open --limit 300 \
      --json number,title --jq '.[] | select(.number == 48) | "\(.number) \(.title)"'
    48 The two dimensional mesh, and field transfer that survives remeshing

    gh issue view 48 --repo iderex/schichtwerk --json body --jq .body \
      | grep -n 'conservation tolerance'
    17:- The conservation tolerance from #6 is enforced as a refusal per #14, not as a warning.

A second number stood beside that one and the sentence was untrue of the issue it
named. Why one number is named here and not two is below, after this list.

Provenance. Every coefficient that is not derived by the code carries where it
came from, in the same file, and no quantity has two homes. #11 holds it and
names the two checks.

Validation reporting. A model that disagrees with a measurement is reported with
the disagreement rather than with the case removed, and a coefficient moved while
a comparison is open is labelled as fitted to that measurement. #12 holds it, and
#36 and #40 are where it first has a suite to run in.

Headlessness and no network in the ordinary suite. #23. The target board has this
implicitly through where its tests run; here it is a requirement of the plan, and
a requirement that lives only in a sentence lasts until the first test that is
easier to write the other way.

Numeric comparison. No test compares floating point with equality, and tolerances
are named constants with a reason rather than literals in the suite. #21.

Reproducible offline build. #25. The target board pins and reviews; this one also
has to be rebuildable years later by somebody checking a published number.

## Where this mapping and the record for issue #6 do not agree

The conservation entry named two issues and one of them refuses nothing about
conservation. The finding is issue #85, along with the reading that the tree does
not settle which number was meant:

    gh issue list --repo iderex/schichtwerk --state open --limit 300 \
      --json number,title --jq '.[] | select(.number == 85) | "\(.number) \(.title)"'
    85 The parity mapping names an issue that refuses nothing about conservation

The entry above now claims only what it can show, and the second number is not
guessed at. This section is why.

The record the entry cites names its own pair:

    git grep -n 'Issue #32 and issue #48 implement this' \
      -- docs/decisions/0006-discretisation-and-moving-boundaries.md
    docs/decisions/0006-discretisation-and-moving-boundaries.md:208:Issue #32 and issue #48 implement this, and both of their Done-when clauses ask

Read against that record, the second number here is issue #32. Its Done-when asks
for a refusal, and the refusal it asks for is non-convergence rather than a
conservation figure over a tolerance:

    gh issue view 32 --repo iderex/schichtwerk --json body --jq .body \
      | sed -n '/## Done when/,$p' | grep -n 'is a refusal'
    6:- Every exit from the Newton loop reports a reason, and non-convergence at the smallest step is a refusal.

Conservation is not named anywhere in that issue, in its Done-when or outside it:

    gh issue view 32 --repo iderex/schichtwerk --json body --jq .body | grep -ci conserv
    0

So writing it in carries across a claim untrue of the issue it names, which is
the defect being repaired, one number over.

The other candidate is issue #29, where a structure whose accounting does not
close is refused, in one dimension and against this same tolerance:

    gh issue list --repo iderex/schichtwerk --state open --limit 300 \
      --json number,title --jq '.[] | select(.number == 29) | "\(.number) \(.title)"'
    29 Fields on a structure, and the invariants a check refuses

    gh issue view 29 --repo iderex/schichtwerk --json body --jq .body \
      | sed -n '/## Done when/,$p' | grep -n 'accounting does not close'
    7:- The pull request shows the check refusing a structure with a negative concentration, one with a stale value outside the geometry, and one where the accounting does not close.

    gh issue view 29 --repo iderex/schichtwerk --json body --jq .body \
      | grep -oE 'the invariant is that it stays under the tolerance from #6'
    the invariant is that it stays under the tolerance from #6

Writing that one in departs from the record the entry cites. Which of the two was
meant is a question about what this document intended to say, and no reading of
the tree answers it, so neither is written in and the entry names what it can
show instead.

That leaves this mapping and the record for issue #6 naming different sets, and
the paragraphs above are this document's half of saying so. The record's half is
a change to `docs/decisions/0006-discretisation-and-moving-boundaries.md` and
belongs with issue #6.

One more issue carries the tolerance in its own Done-when and is not a third
candidate. Issue #38 asks for the tolerance to be met and the achieved figure
reported, which is not a refusal:

    gh issue view 38 --repo iderex/schichtwerk --json body --jq .body \
      | sed -n '/## Done when/,$p' | grep -n 'conservation tolerance'
    6:- The conservation tolerance from #6 is met and the achieved figure is reported in the manifest.

## Issues opened by this mapping

Three elements had no home on this board, and the done-when of #55 asks for one
to be opened rather than for the gap to be noted:

    gh issue list --repo iderex/schichtwerk --state open --limit 300 \
      --json number,title,milestone \
      --jq '.[] | select(.number == 69 or .number == 70 or .number == 71) | "\(.number) m\(.milestone.number) \(.title)"'
    71 m10 A vulnerability scan of the pinned dependency set, on a schedule
    70 m10 Deterministic pull request hygiene checks
    69 m10 Semantic security analysis of the source, failing closed

Each says in its own body why the issue it is nearest to does not already cover
it, because an issue opened out of a mapping is the kind that gets closed later
as a duplicate of the thing it was deliberately separated from.

## What this document is not

It is not a list of this project's checks. `ops verify` has no counterpart here
yet and neither does anything else, because there is no build:

    git ls-files -- '*.c' '*.h' '*.cpp' '*.hpp' '*.rs' 'CMakeLists.txt' | wc -l
    0

Every element above marked adopted or adapted names an issue that has not
happened. What is in this tree today is the five workflow files listed as already
present, and nothing else.

It is not enforced. No check reads this document, nothing compares this board's
gate against the target, and nothing notices when the target moves. The date at
the top and the commands are what a person uses to redo it. Whether that becomes
a scheduled comparison is a decision for after the elements exist, since a
comparison against a target this board has not reached yet would be red on
purpose from the day it landed.

It is not a judgement that the target is correct. It is a target because it was
built against real failures over a long time, which is a better reason than
taste, and any element of it this board declines is declined in one line above
rather than by omission.
