# How an issue reference is written, and what a check could refuse

Owned by issue #61, milestone 10:

    gh issue list --repo iderex/schichtwerk --state open --limit 300 \
      --json number,title --jq '.[] | select(.number == 61) | "\(.number) \(.title)"'
    61 A check that refuses documentation naming a path that does not exist

Derived on 2026-08-10 against `9b09048e46059d2c47e51745ef36a35f29e05433`. Every
count below carries the command that produced it, so the document is re-derived
rather than trusted.

## Why this is separate from the path convention

Issue #61 asks for three gate legs. One refuses a path that does not exist, one
refuses a command or a flag the tool does not accept, and one refuses a cross
reference that does not resolve. `docs/path-references.md` is the convention for
the first, and it says in its own closing section that cross reference
resolution is the third leg and a different subject. This is that subject.

It is narrowed to one shape, and the narrowing is the point. A cross reference
in this tree is almost always a number on this board's tracker. The other shape
is a link to a heading inside a document, and nothing in the tree carries one
today, so a rule about fragments would be written against an imagined case
rather than a real one. `docs/path-references.md` already declines to resolve a
fragment and this document does not widen that.

The order is the same argument that document makes. The convention comes before
the check, because a convention designed against imagined cases meets the real
ones as false positives.

## What the run found

Four hundred and forty four references, in eighteen tracked markdown files:

    git grep -oE '#[0-9]+' origin/main -- '*.md' | wc -l
    444
    git ls-files -- '*.md' | wc -l
    18

Twelve of them sit inside a fenced or an indented block and four hundred and
thirty two are in prose. The two regions are the ones `docs/path-references.md`
already defines, and the split is taken the same way:

    awk '
      BEGIN{fenced=0} FNR==1{fenced=0}
      { if ($0 ~ /^[ \t]*(```|~~~)/){fenced=1-fenced; next}
        n=gsub(/#[0-9]+/,"&")
        if (fenced || $0 ~ /^(    |\t)/) blk+=n; else pro+=n }
      END{print "in skipped regions: " blk; print "read as prose: " pro}' \
      $(git ls-files -- '*.md')
    in skipped regions: 12
    read as prose: 432

Every number referenced anywhere in the tree resolves to an issue on this board.
None is absent, and none is a pull request:

    comm -13 \
      <(gh issue list --repo iderex/schichtwerk --state all --limit 300 \
          --json number --jq '.[].number' | sort -u) \
      <(git grep -ohE '#[0-9]+' origin/main -- '*.md' | tr -d '#' | sort -u)

which prints nothing.

So a leg that refuses a reference which does not resolve would refuse nothing
here. That is the entire mechanical yield of this leg against this tree, and it
belongs at the top rather than in a closing caveat, because a green leg reads as
a tree with no reference defects and this tree has six.

## The failure this tree has instead

Ninety five of the four hundred and thirty two prose references make a claim
about what the referenced issue does rather than standing beside a noun:

    git grep -cE '#[0-9]+ (is|builds|implements|holds|carries|measures|refuses|delivers|runs|asks|names|fixes|owes|pins|covers|does)' \
      origin/main -- '*.md' | sed 's/^origin\/main://' | awk -F: '{s+=$2} END {print s}'
    95

Six references of that shape have been wrong so far, and every one of them
resolved. Only one of the six is in the tree. The other five are in issue
bodies, which no gate leg reads.

The five are corrected on the tracker, across three issue bodies:

    for n in 9 13 65; do
      gh issue view $n --repo iderex/schichtwerk --json body --jq .body \
        | grep -oE '(One|Two) corrections? to the text above'
    done
    One correction to the text above
    Two corrections to the text above
    Two corrections to the text above

The sixth is in the tree and is not repaired:

    git grep -n '#47 with #48' origin/main -- docs/gate-parity.md
    origin/main:docs/gate-parity.md:218:against, and #47 with #48 is where it is refused.

The sentence it ends is about conservation, and the pair it names is

    gh issue list --repo iderex/schichtwerk --state open --limit 300 \
      --json number,title --jq '.[] | select(.number == 47 or .number == 48) | "\(.number) \(.title)"'
    48 The two dimensional mesh, and field transfer that survives remeshing
    47 Transient enhanced diffusion, and the case that would expose it as wrong

so issue #48 carries the refusal and issue #47 refuses nothing about
conservation. Which number was meant is not resolvable from the tree, for the
reason already recorded on issue #61, and it is not guessed at here.

What the six have in common is what makes them expensive. Each resolved. Each
rendered as a working link. Each landed a reader on a real open issue of this
board, in a neighbouring milestone, with a subject close enough that nothing
about the reference looks wrong from either end. That is worse than rot, because
rot announces itself.

Five of the six sat in an issue body rather than in the tree, so the leg this
issue asks for could not have reached them even in principle.

## The rules

**A number is a reference only in prose.** A number inside a fenced or an
indented block is part of a command or part of what a command printed, and it is
not this document making a claim. This is the same region split
`docs/path-references.md` defines, and taking it twice would let the two drift.

**A reference that makes a claim about the referenced issue says which kind of
thing it names.** The number space is shared with pull requests on this board,
and the tracker renders both the same way. Number 46 is a pull request:

    gh api repos/iderex/schichtwerk/issues/46 --jq '.pull_request.html_url'
    https://github.com/iderex/schichtwerk/pull/46

Nothing in the tree names 46 today, which is why this is a rule about a hazard
rather than about a defect. Writing `issue #58` says what a bare `#58` does not,
and it costs one word.

**A claim-shaped reference carries the command that resolves it, with the output
pasted.** This is the only rule here that reaches the failure the section above
describes, and it is not this document's invention. The tree already does it in
six of its eighteen markdown files:

    git grep -l 'gh issue list --repo iderex/schichtwerk' -- '*.md' | wc -l
    6

The shape is a block holding the query and the titles it returned, with the
prose claim beside it. A reader comparing the pasted title against the sentence
sees a wrong number immediately, and a reader who doubts the paste re-runs the
command. Neither needs the tracker open.

**A reference to another board's tracker names the board.** Both boards this
tree depends on number from one, so a bare number is ambiguous across them in
exactly the way a bare decision record filename already was. Nothing in the tree
names an issue on another board today, so this rule arrives before its case, and
it is written now because `docs/path-references.md` learned the other order: the
bare filename gap was written there as something a rename would open, and it was
already open on a route a rename had nothing to do with.

## What the tree does not yet meet, and why it is not repaired here

Forty six of the ninety five claim-shaped references are bare. Forty nine carry
the second rule already:

    git grep -cE 'issue #[0-9]+ (is|builds|implements|holds|carries|measures|refuses|delivers|runs|asks|names|fixes|owes|pins|covers|does)' \
      origin/main -- '*.md' | sed 's/^origin\/main://' | awk -F: '{s+=$2} END {print s}'
    49

Sixteen of the forty six are in one file, which carries no reference in the
other spelling at all:

    git grep -cE '#[0-9]+ (is|builds|implements|holds|carries|measures|refuses|delivers|runs|asks|names|fixes|owes|pins|covers|does)' \
      origin/main -- docs/gate-parity.md
    origin/main:docs/gate-parity.md:16
    git grep -cE 'issue #[0-9]+ (is|builds|implements|holds|carries|measures|refuses|delivers|runs|asks|names|fixes|owes|pins|covers|does)' \
      origin/main -- docs/gate-parity.md ; echo "exit=$?"
    exit=1

Nothing is edited into conformance by this change. Issue #61 asks that whatever
a first run turns up is reported rather than quietly fixed, and this is that
report. The files holding those references are owned by other issues, and a
second hand in a file is the collision this board's sizing exists against.

That leaves a rule the tree breaks on the day it lands, which
`docs/path-references.md` names as the kind of rule nobody follows. The
difference is that the breach is counted here rather than discovered later, and
the repair is one document at a time by whoever holds each.

## What a check could refuse, and what it never will

Refusable, and all of it cheap once a gate exists. A number in prose that names
nothing on this board. A number in prose that names a pull request where the
sentence says issue. A claim-shaped reference with no resolving command anywhere
in the document.

Never refusable. Whether the issue a reference names is the issue the sentence
is about. That is the whole of what has gone wrong here so far, it is a
judgement about meaning, and no reading of this tree makes it. The resolving
command does not make it either. What it does is put the referenced issue's own
title next to the claim, so a person reading the paragraph is reading both. All
six above were found by a person reading, which is the only route there is.

Also never reached by any leg. Anything written on the tracker rather than in the
tree, which is where five of the six sat.

## What is enforced today

Nothing. No check reads this document, no gate leg exists to hang one off, and
the tree carries no build for one to run in:

    git ls-files -- '*.c' '*.h' '*.cpp' '*.hpp' '*.rs' 'CMakeLists.txt' | wc -l
    0

Until that changes, every rule here is followed by a person or not at all, and a
reference naming the wrong issue lands green. That is the same state
`docs/path-references.md` reports of itself, and it is the state issue #61 stays
open on.
