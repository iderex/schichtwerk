# Security policy

## Reporting

Report privately, through GitHub's private vulnerability reporting on this
repository:

    https://github.com/iderex/schichtwerk/security/advisories/new

That route is open today, which is checkable rather than asserted:

    gh api repos/iderex/schichtwerk/private-vulnerability-reporting
    {"enabled":true}

It goes to the maintainers and nowhere else, it lets you attach a file, and it
gives you a place to talk before anything is public. Please do not open a public
issue for something that would help somebody attack a person who has already
installed this.

If that route is not usable for you, say so in any public issue without the
details, and a private channel will be arranged from there.

## What this project is, so that the surface is clear

This is a process simulator. It reads a recipe, computes, and writes results to
the disk of the machine it runs on. It has no server, no account, no session and
no privileged component.

The surface is therefore narrow, and saying so plainly is more useful than
implying it is broad. The inputs are the recipe, the structure file, the
parameter sets, and any reference data the operator obtained for a validation
case. The assets are the operator's own machine and the confidentiality of the
recipe they are simulating. A process recipe is frequently the most guarded thing
a manufacturer owns, so a defect that copies one anywhere is treated as a serious
finding rather than as a privacy nicety.

Where a run's output may go is not decided by this document. Entry 2 of issue #1
is open, and this policy assumes no answer to it in either direction. The data
protection position is issue #15 and is also open. What is true today is stated
in the next section rather than inferred from either.

## The state of the tree today

There is no code and no release. Both are checkable:

    git ls-files -- '*.c' '*.h' '*.cpp' '*.hpp' '*.rs' 'CMakeLists.txt' | wc -l
    0

    gh api repos/iderex/schichtwerk/releases --jq 'length'
    0

So there is nothing installed anywhere to attack, and no version to fix. The
readers, the parameter handling and the result writing described above are what
this project is being built to have, and the scope below is written for them so
that it is in force on the day the first one lands rather than written
afterwards.

What does exist and is in scope now is this repository itself: the workflow
definitions under `.github/workflows/`, and anything in the tree that would run
on a contributor's machine or in a workflow runner.

## In scope

- The readers, once they exist. A recipe, a structure file, a parameter set or a
  reference data file is untrusted input, and a crafted one that reaches memory
  it should not, executes anything, or reads a file it was not given is a
  vulnerability.
- Anything that reaches outside the machine. A network connection, a file
  written outside the result set, or a path traversal out of a directory the
  operator named.
- Anything that copies a recipe, a parameter set or a result somewhere the
  operator did not ask for, including into a log, a temporary file left behind,
  or a diagnostic.
- The workflow definitions in this repository, and any build or release path
  that a change to them could reach.
- A dependency of this project, where the way this project uses it is what makes
  the defect reachable. A defect in the dependency itself belongs to that
  project first, and telling us as well is welcome.

## Not in scope

A model that is physically inaccurate is not a security report. A wrong junction
depth, a profile that disagrees with a measurement, a coefficient from the wrong
publication and a solver that converges to the wrong answer are correctness
problems, and they belong in the open on the issue tracker where they can be
argued with evidence. Handling them privately would be the opposite of what this
project is for.

The line between the two is what the defect gives an attacker. If a crafted
input file makes the tool do something to the machine or to the operator's data,
report it privately. If it makes the tool print a number that is wrong, open an
issue.

Also not in scope: a report that this project has no licence file, which is
known and is entry 1 of issue #1; a scanner result with no reachable path shown;
and a request to enable a setting that is already enabled.

## What you can expect

This is a small project. What it can honestly promise is this.

Your report will be read and you will get an acknowledgement that it has been.
It will be assessed, and you will be told what the assessment was, including when
the conclusion is that it is not a vulnerability and why. If it is one, it will
be fixed, and you will be told when the fix has landed.

No timetable is promised for any of those steps, because none could be met
reliably and a policy that overpromises is worse than one that does not, at
exactly the moment it matters. If you have a disclosure deadline, say so in the
report and it will be worked to or answered honestly.

You will be credited by whatever name you ask for, or not credited if you prefer
that. Nothing here asks you to keep a report secret indefinitely.

## What this policy does not do

It does not offer a reward, and it does not run a bounty programme.

It does not grant permission to test against anybody else's machine or
installation. Test against your own.

Nothing in it is a mechanism. No check in this repository refuses a change that
contradicts a sentence above, so this document is a statement of intent that a
person keeps, and it is read by a person or not at all.
