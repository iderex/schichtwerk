# How a path is written, so that a check can refuse a wrong one

Owned by issue #61, milestone 10.

Derived on 2026-08-08 against `5e3ce8dc542fae2b3a7c43be0995a9c81ac1dca3`. Every
count below carries the command that produced it, so the document is re-derived
rather than trusted.

Amended against `4b06a776b6ad710c75824b9cd259e03d4e4f7d95`, which is the first
commit carrying a document written after these rules existed. The rules held on
that document. What the re-run found is in the region they decline to read, and
it narrows one of them. The section on records in another tree carries it, and
the counts under "The measurement" are the ones the earlier commit produced and
are left as they were.

## Why the convention comes before the check

Issue #61 asks for a gate leg that refuses documentation naming a path that does
not exist, and it asks for the convention for writing an example path to be
defined before the check lands. The order is the point. A convention designed
against imagined cases meets the real ones as false positives, and a check that
refuses its own documentation is the first thing anyone notices about it.

This tree is an unusually hard case for such a check, and the reason is a rule
this project follows on purpose. Every asserted fact carries the command that
produced it, so a record written correctly is full of pasted command lines and
pasted output, and that text looks exactly like paths without being claims about
this tree. A matcher that reads it as claims turns the rule that produces good
records into the thing that reds the gate.

So the convention is derived from what is in the tree rather than from what a
path checker usually assumes.

## What the check reads

Two regions, and only one of them is read.

A fenced block, and a block indented by four spaces or a tab, is not read at all.
That is where commands, their output, quoted external documentation, citations
and tables live. A path written there is part of a command or part of what a
command printed, and neither is this document making a claim about this tree.

Everything else is prose and is read.

The split is not close on this tree:

    lines in skipped regions: 147
    lines read as prose: 3878

which comes from walking every tracked markdown file at the commit named above,
of which there were sixteen:

    git ls-files -- '*.md' | wc -l
    16

The command returns one more than that once this file lands, which is why the
commit it was run at is written next to the number rather than left to be
inferred.

## The rules

**A link destination is a path and is always checked**, unless it is a URL or a
bare fragment. This is the one form that is checked even without a directory
separator, because a link is a promise that clicking it arrives somewhere. Two
of the four link destinations in this tree are top level files with no separator
in them, and both should be checked.

**A path in this tree, written in prose, goes in an inline code span**, written
relative to the repository root, with no leading `./`.

**A candidate contains no whitespace.** This rule exists because of a case the
tree already holds. Two of the required check names on the board that
`docs/gate-parity.md` maps against contain a slash with spaces around it, and a
candidate rule keyed on the slash alone refuses both of them:

    docs/gate-parity.md:81   `Package (JPRM) / Build package`
    docs/gate-parity.md:85   `Package (JPRM) / Generate SBOM`

They are names of checks rather than paths, and no amount of resolving will make
them exist.

**A candidate names a file or a directory.** A file has a dot in its last
segment; a directory ends in a separator. This is what keeps a repository name
from being read as a path. `iderex/jellyfin-plugin-sso` is a repository, it is
written in prose in the parity mapping, it has a separator in it, and it will
never resolve here.

**A path in another tree is never written as a candidate.** It goes inside the
block that carries the command reaching it, which is where a reader wants it
anyway, or it is written as a link to it, or it is named in prose by its bare
filename with the tree named beside it. The last of those three is the form the
rules above decline to check, and the requirement to name the tree was added by
the amendment below rather than being in this rule when it landed.

The tree is named beside the filename and not joined to it. A span reading
`iderex/bremsweg` is a repository under the rule above and is declined, but the
same repository joined onto a filename has a separator and a dot in its last
segment, so it becomes a candidate and can never resolve here.

**An example path goes inside a fenced block.** This is the trap #61 names in
its own body, and the block rule handles it without a marker of its own.

## The measurement

Ten candidates in the tree, and every one of them resolves. Four link
destinations that are not URLs:

    README.md	36	docs/decisions/0003-the-boundary.md
    README.md	59	docs/decisions/0015-the-data-protection-position.md
    README.md	61	NOTICE.md
    README.md	63	SECURITY.md

and six code spans in prose:

    SECURITY.md	59	.github/workflows/
    docs/gate-parity.md	93	.github/workflows/dco.yml
    docs/gate-parity.md	105	.github/workflows/unicode-guard.yml
    docs/gate-parity.md	108	.github/workflows/zizmor.yml
    docs/gate-parity.md	115	.github/workflows/dependency-review.yml
    docs/gate-parity.md	137	.github/workflows/scorecard.yml

Two distinct filenames, on three lines, are candidate-shaped in prose and are
declined by the rules above. Both are files in the board the parity mapping
targets rather than in this one:

    docs/gate-parity.md	65	dotnet.yml
    docs/gate-parity.md	66	build.yml
    docs/gate-parity.md	186	dotnet.yml

Three cases sit in skipped regions and would each be a false positive if the
regions were read. Two lines of `git grep` context output, where the separator
between the file and the line number is a hyphen rather than a colon, so a
matcher that stops at whitespace swallows the line number and the first word of
the text:

    docs/decisions/0015-the-data-protection-position.md:165
    docs/decisions/0015-the-data-protection-position.md:166

and one glob standing where a filename would be, inside a quoted command:

    docs/decisions/0009-the-recipe-format.md:288

The approximation that produced the candidate list reads each file, drops fenced
and indented regions, and keeps link destinations and code spans:

    awk '
      BEGIN{fenced=0} FNR==1{fenced=0}
      { if ($0 ~ /^[ \t]*(```|~~~)/){fenced=1-fenced; next}
        if (fenced || $0 ~ /^(    |\t)/) next
        rest=$0
        while (match(rest, /\]\([^)]+\)/)) {
          print FILENAME"\t"FNR"\tLINK\t"substr(rest,RSTART+2,RLENGTH-3)
          rest=substr(rest,RSTART+RLENGTH) }
        rest=$0
        while (match(rest, /`[^`]+`/)) {
          print FILENAME"\t"FNR"\tSPAN\t"substr(rest,RSTART+1,RLENGTH-2)
          rest=substr(rest,RSTART+RLENGTH) } }' $(git ls-files -- '*.md')

with the candidate rules applied to its output:

    awk -F'\t' '
      $3=="LINK" && $4 !~ /^(https?:|mailto:|#)/ { print $1"\t"$2"\t"$4 }
      $3=="SPAN" && $4 ~ /\// && $4 !~ /[ \t]/ && ($4 ~ /\/$/ || $4 ~ /\.[^\/.]+$/) \
        { print $1"\t"$2"\t"$4 }'

That is an approximation and not the leg. It has no idea what a list
continuation is, it treats every four space indent as a block, and its notion of
a code span is a pair of backticks on one line. It is written down because its
output is the evidence for the rules above, not because it is the
implementation.

## Records in another tree, and why a bare filename carries its tree

These rules landed at `03922d9213a108e6f1b37d5bf691a9635bad8534`. One document
has landed since, it was written after the rules existed, and it is the first
contact they have had with a document that had them available. Re-running the
two commands above at `4b06a776b6ad710c75824b9cd259e03d4e4f7d95` gives eighteen
markdown files, twelve candidates instead of ten, and every one of the twelve
resolves. Both new candidates are in this file. The rules held.

What the re-run turned up is in the region they decline to read. A bare filename
in prose is not checked, and when this file landed it said the gap would go live
the day an in tree file was named that way. It is live now, and a rename has
nothing to do with it.

The other board this tree depends on numbers its decision records in the same
scheme this one uses. Ten numbers exist in both trees, carrying different
subjects in each:

    comm -12 \
      <(git ls-files docs/decisions/ | sed 's#.*/##' | cut -c1-4 | sort) \
      <(gh api repos/iderex/bremsweg/contents/docs/decisions --jq '.[].name' \
        | cut -c1-4 | sort)
    0003
    0005
    0006
    0007
    0008
    0009
    0010
    0011
    0012
    0013

Two of those ten are named in prose in this tree, by bare filename, and they are
the other tree's:

    git grep -n '0006-determinism-and-the-random-number-contract.md\|0010-how-uncertainty-travels-to-a-reported-number.md' -- '*.md'
    docs/decisions/0041-the-implantation-interface.md:171:is `0010-how-uncertainty-travels-to-a-reported-number.md` in the same directory
    docs/decisions/0041-the-implantation-interface.md:185:record is `0006-determinism-and-the-random-number-contract.md`.

This tree holds a 0006 and a 0010 of its own, about different things:

    git ls-files docs/decisions/ | grep -E '/(0006|0010)-'
    docs/decisions/0006-discretisation-and-moving-boundaries.md
    docs/decisions/0010-result-document-and-run-manifest.md

Nothing resolves wrongly today, because the titles differ and the full filenames
differ with them. The hazard is that neither tree consults the other before
numbering, so the two sequences are free to converge, and the failure that
produces is not a red gate. It is a green one over a reference that names a real
file in the wrong tree, which is worse than the rot this check exists against,
because the reader arrives somewhere plausible instead of nowhere.

One more shape came out of the same run, and it is why a bare filename cannot be
resolved by its shape at all. Five spans in prose have no separator and a dot in
their last segment, and one of them is not a file:

    docs/decisions/0041-the-implantation-interface.md	171	0010-how-uncertainty-travels-to-a-reported-number.md
    docs/decisions/0041-the-implantation-interface.md	185	0006-determinism-and-the-random-number-contract.md
    docs/gate-parity.md	65	dotnet.yml
    docs/gate-parity.md	66	build.yml
    docs/gate-parity.md	126	.NET
    docs/gate-parity.md	186	dotnet.yml

Six lines and five distinct spans, at the commit named at the top of this
section. Once this file lands the command returns a seventh line and the same
five spans, because the paragraph above quotes one of them.

`.NET` is a platform name and will never be a path. The other four are files in
two different trees, and nothing in the shape of any of them says which tree, or
that one of them is not a file.

So the rule this adds is for a reader rather than for a check. Naming the tree
beside the filename costs three words and removes the case where a reference
lands on the wrong document. It is not enforceable and it is not claimed to be.

The two references above are brought into conformance in the same change that
adds the rule, because a rule the tree breaks on the day it lands is a rule
nobody will follow. The change to each is the tree's name and nothing else, and
no claim in that record moves. What reads them back once this is on `main` is

    git grep -n 'iderex/bremsweg`' -- docs/decisions/0041-the-implantation-interface.md

with no output pasted beside it, because at the commit this section is written
against there is none. The count of bare filename spans does not move either,
since naming the tree beside a filename leaves the filename bare.

## What this does not do, stated rather than implied

It does not read inside a block. A path pasted in a command or in output can rot
with the tree and stay green. What catches that is the command beside it, which
can be run again; nothing mechanical catches it here, and pretending otherwise
would be worse than saying so.

It does not check a bare filename in prose. Five spans are declined today and
all five are correct to decline, but an in tree file named that way would escape
the same rule. That was written here as a gap a rename would open. The section
above is why it is open already, on a route a rename has nothing to do with, and
the convention answers it with a rule a person follows rather than with a
mechanism.

It does not check an extensionless file, or a directory written without a
trailing separator. Both are shapes the last rule cannot tell from ordinary
prose.

It does not resolve a fragment inside a document, so a link to a heading that
was renamed is not caught by this. Cross reference resolution is the third leg
#61 asks for and it is a different subject. `docs/issue-references.md` is that
subject, for the shape of it this tree is made of, which is a number on the
tracker.

It says nothing about whether a command or a flag shown in documentation is one
the tool accepts. That is #61's second leg and it needs a tool, which needs #2.

It says nothing about whether a sentence is still true. #61 says that plainly of
itself and this convention does not widen the claim.

## What is enforced today

Nothing. No check reads this document, no gate leg exists to hang one off, and
the tree carries no build for one to run in:

    git ls-files -- '*.c' '*.h' '*.cpp' '*.hpp' '*.rs' 'CMakeLists.txt' | wc -l
    0

Until that changes, every rule here is followed by a person or not at all, and a
document naming a path that does not exist lands green. The convention is
written first so that the check, when it arrives, is written against cases this
tree actually produced.
