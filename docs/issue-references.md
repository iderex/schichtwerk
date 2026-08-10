# How an issue reference is written, and what a check could refuse

Owned by issue #61, milestone 10:

    gh issue list --repo iderex/schichtwerk --state open --limit 300 \
      --json number,title --jq '.[] | select(.number == 61) | "\(.number) \(.title)"'
    61 A check that refuses documentation naming a path that does not exist

Derived on 2026-08-10 against `9b09048e46059d2c47e51745ef36a35f29e05433`, and
re-derived the same day against `b0d84c5f9e34f23add4bed633ea36295369c2b2e`, which
is the commit this document is written against. Every count below carries the
command that produced it, so the document is re-derived rather than trusted.

Every command below names that commit where it can, rather than `origin/main`.
The first derivation read the moving reference, and three merges later its pasted
output no longer reproduced where a reader would run it:

    git rev-list --count --merges \
      9b09048e46059d2c47e51745ef36a35f29e05433..b0d84c5f9e34f23add4bed633ea36295369c2b2e
    3

Every count of a reference in it had moved, and one paste showed a line that is
no longer in the tree. The one figure that still reproduced was the count of
source files, which is zero because there is no code. A command naming a commit
reproduces for any reader at any time, and a reader wanting today's number puts
`origin/main` in its place.

The first derivation had a second problem the moving reference hides, and it was
there on the day this landed rather than three merges later. Its commit is the
one the branch was based on, and this document was not in it:

    git ls-tree -r --name-only 9b09048e46059d2c47e51745ef36a35f29e05433 \
      | grep -c 'issue-references' ; echo "exit=$?"
    0
    exit=1

So every count it stated excluded its own text. The counts below include it,
which is why landing any change to this file moves them, and why the commit is
written beside the numbers rather than left to be inferred.

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

Four hundred and seventy two references, in nineteen tracked markdown files:

    git grep -oE '#[0-9]+' b0d84c5f9e34f23add4bed633ea36295369c2b2e -- '*.md' | wc -l
    472
    git ls-tree -r --name-only b0d84c5f9e34f23add4bed633ea36295369c2b2e \
      | grep -c '\.md$'
    19

Twenty five of them sit inside a fenced or an indented block and four hundred and
forty seven are in prose. The two regions are the ones `docs/path-references.md`
already defines, and the split is taken the same way. It reads the files out of
the commit rather than out of a checkout, so that a reader on any branch gets the
same answer:

    git ls-tree -r --name-only b0d84c5f9e34f23add4bed633ea36295369c2b2e \
      | grep '\.md$' \
      | while read -r f; do
          git show "b0d84c5f9e34f23add4bed633ea36295369c2b2e:$f"; printf '\n@@@\n'
        done \
      | awk '
        /^@@@$/{fenced=0; next}
        { if ($0 ~ /^[ \t]*(```|~~~)/){fenced=1-fenced; next}
          n=gsub(/#[0-9]+/,"&")
          if (fenced || $0 ~ /^(    |\t)/) blk+=n; else pro+=n }
        END{print "in skipped regions: " blk; print "read as prose: " pro}'
    in skipped regions: 25
    read as prose: 447

Every number referenced anywhere in the tree resolves to an issue on this board.
None is absent, and none is a pull request:

    comm -13 \
      <(gh issue list --repo iderex/schichtwerk --state all --limit 300 \
          --json number --jq '.[].number' | sort -u) \
      <(git grep -ohE '#[0-9]+' b0d84c5f9e34f23add4bed633ea36295369c2b2e \
          -- '*.md' | tr -d '#' | sort -u)

which prints nothing. The tracker side of it is the one command here that cannot
be pinned, since an issue can be opened after this is written, and a number that
resolves today goes on resolving.

So a leg that refuses a reference which does not resolve would refuse nothing
here. That is the entire mechanical yield of this leg against this tree, and it
belongs at the top rather than in a closing caveat, because a green leg reads as
a tree with no reference defects and this tree has six.

## The failure this tree has instead

A hundred and one lines carry a reference making a claim about what the
referenced issue does rather than standing beside a noun:

    git grep -cE '#[0-9]+ (is|builds|implements|holds|carries|measures|refuses|delivers|runs|asks|names|fixes|owes|pins|covers|does)' \
      b0d84c5f9e34f23add4bed633ea36295369c2b2e -- '*.md' \
      | sed 's/^b0d84c5f9e34f23add4bed633ea36295369c2b2e://' \
      | awk -F: '{s+=$2} END {print s}'
    101

Ninety five of the hundred and one are in prose. The other six are inside a
block, which under the first rule below is part of what a command printed rather
than this tree making a claim:

    git ls-tree -r --name-only b0d84c5f9e34f23add4bed633ea36295369c2b2e \
      | grep '\.md$' \
      | while read -r f; do
          git show "b0d84c5f9e34f23add4bed633ea36295369c2b2e:$f"; printf '\n@@@\n'
        done \
      | awk '
        /^@@@$/{fenced=0; next}
        { if ($0 ~ /^[ \t]*(```|~~~)/){fenced=1-fenced; next}
          if ($0 !~ /#[0-9]+ (is|builds|implements|holds|carries|measures|refuses|delivers|runs|asks|names|fixes|owes|pins|covers|does)/) next
          if (fenced || $0 ~ /^(    |\t)/) blk++; else pro++ }
        END{print "in skipped regions: " blk; print "read as prose: " pro}'
    in skipped regions: 6
    read as prose: 95

The first derivation quoted the grep total as a count of prose references. It is
a count of lines and it reads both regions, and at its own commit the two figures
were already apart:

    git ls-tree -r --name-only 9b09048e46059d2c47e51745ef36a35f29e05433 \
      | grep '\.md$' \
      | while read -r f; do
          git show "9b09048e46059d2c47e51745ef36a35f29e05433:$f"; printf '\n@@@\n'
        done \
      | awk '
        /^@@@$/{fenced=0; next}
        { if ($0 ~ /^[ \t]*(```|~~~)/){fenced=1-fenced; next}
          if ($0 !~ /#[0-9]+ (is|builds|implements|holds|carries|measures|refuses|delivers|runs|asks|names|fixes|owes|pins|covers|does)/) next
          if (fenced || $0 ~ /^(    |\t)/) blk++; else pro++ }
        END{print "in skipped regions: " blk+0; print "read as prose: " pro+0}'
    in skipped regions: 3
    read as prose: 92

Ninety five lines and ninety two of them in prose, so the figure stated as a
count of prose references was three high on the day it was written, before any of
the drift above. Both figures are stated here so neither has to be inferred from
the other.

Six references of that shape have been wrong so far, and every one of them
resolved. Only one of the six was in the tree. The other five are in issue
bodies, which no gate leg reads.

The five are corrected on the tracker, across three issue bodies:

    for n in 9 13 65; do
      gh issue view $n --repo iderex/schichtwerk --json body --jq .body \
        | grep -oE '(One|Two) corrections? to the text above'
    done
    One correction to the text above
    Two corrections to the text above
    Two corrections to the text above

The sixth was in the tree. It was the conservation entry of
`docs/gate-parity.md`, which named a pair of issues where one of the two refuses
nothing about conservation, and it is gone:

    git grep -n '#47 with #48' b0d84c5f9e34f23add4bed633ea36295369c2b2e \
      -- docs/gate-parity.md ; echo "exit=$?"
    exit=1

It was repaired by narrowing the sentence to the issue that carries the refusal
rather than by choosing between the two candidates, since which number was meant
is not resolvable from the tree:

    git log --format='%H %s' -1 14cd49132085d089ea3eabc928237d13cf407e26
    14cd49132085d089ea3eabc928237d13cf407e26 Make the conservation entry claim only what it can show (#85)

That change is argued in issue #85, which is where the choice not to guess is
recorded and which stays open on the half of it that is a sentence in another
record:

    gh issue list --repo iderex/schichtwerk --state open --limit 300 \
      --json number,title --jq '.[] | select(.number == 85) | "\(.number) \(.title)"'
    85 The parity mapping names an issue that refuses nothing about conservation

The repair was made by a person reading the paragraph. Nothing refused it, and
nothing would have, which is the whole of the section below.

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
seven of its nineteen markdown files:

    git grep -l 'gh issue list --repo iderex/schichtwerk' \
      b0d84c5f9e34f23add4bed633ea36295369c2b2e -- '*.md' | wc -l
    7

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

Fifty one of the hundred and one lines carry the second rule already, and all
fifty one are in prose, so forty four of the ninety five prose lines are bare:

    git grep -cE 'issue #[0-9]+ (is|builds|implements|holds|carries|measures|refuses|delivers|runs|asks|names|fixes|owes|pins|covers|does)' \
      b0d84c5f9e34f23add4bed633ea36295369c2b2e -- '*.md' \
      | sed 's/^b0d84c5f9e34f23add4bed633ea36295369c2b2e://' \
      | awk -F: '{s+=$2} END {print s}'
    51

Sixteen of the forty four are in one file:

    git grep -cE '#[0-9]+ (is|builds|implements|holds|carries|measures|refuses|delivers|runs|asks|names|fixes|owes|pins|covers|does)' \
      b0d84c5f9e34f23add4bed633ea36295369c2b2e -- docs/gate-parity.md
    b0d84c5f9e34f23add4bed633ea36295369c2b2e:docs/gate-parity.md:18

Eighteen lines by that grep and sixteen of them in prose, the other two being
pasted output. That file carried nothing at all in the other spelling at the
first derivation and carries one line now, and the line is the sentence the
repair above rewrote, so it arrived with a change to that entry rather than with
a sweep of the file:

    git grep -nE 'issue #[0-9]+ (is|builds|implements|holds|carries|measures|refuses|delivers|runs|asks|names|fixes|owes|pins|covers|does)' \
      68d3bde4e6e25aafe388dddef47c71aed89a7d04 -- docs/gate-parity.md ; echo "exit=$?"
    exit=1
    git grep -nE 'issue #[0-9]+ (is|builds|implements|holds|carries|measures|refuses|delivers|runs|asks|names|fixes|owes|pins|covers|does)' \
      b0d84c5f9e34f23add4bed633ea36295369c2b2e -- docs/gate-parity.md
    b0d84c5f9e34f23add4bed633ea36295369c2b2e:docs/gate-parity.md:218:for issue #6, and the Done-when of issue #48 is where the refusal is asked for:

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

    git ls-tree -r --name-only b0d84c5f9e34f23add4bed633ea36295369c2b2e \
      | grep -cE '\.(c|h|cpp|hpp|rs)$|CMakeLists.txt' ; echo "exit=$?"
    0
    exit=1

Until that changes, every rule here is followed by a person or not at all, and a
reference naming the wrong issue lands green. That is the same state
`docs/path-references.md` reports of itself, and it is the state issue #61 stays
open on.
