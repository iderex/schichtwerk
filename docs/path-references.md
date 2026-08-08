# How a path is written, so that a check can refuse a wrong one

Owned by issue #61, milestone 10.

Derived on 2026-08-08 against `5e3ce8dc542fae2b3a7c43be0995a9c81ac1dca3`. Every
count below carries the command that produced it, so the document is re-derived
rather than trusted.

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
anyway, or it is named in prose by its bare filename, or it is written as a link
to it. The two foreign filenames in this tree today are written the second way
and are the two the rules above decline to check.

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

## What this does not do, stated rather than implied

It does not read inside a block. A path pasted in a command or in output can rot
with the tree and stay green. What catches that is the command beside it, which
can be run again; nothing mechanical catches it here, and pretending otherwise
would be worse than saying so.

It does not check a bare filename in prose. Two are declined today and both are
correct to decline, but an in tree file named that way would escape the same
rule, and the tree is one rename away from that being a live gap rather than a
theoretical one.

It does not check an extensionless file, or a directory written without a
trailing separator. Both are shapes the last rule cannot tell from ordinary
prose.

It does not resolve a fragment inside a document, so a link to a heading that
was renamed is not caught by this. Cross reference resolution is the third leg
#61 asks for and it is a different subject.

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
