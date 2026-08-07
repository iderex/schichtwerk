# The diffusion model family, and what it rules out

Decided by issue #8, milestone 1.

## Status

In force from the commit that lands this file.

The naming and numbering of decision records is fixed by issue #2, which is not
decided. This filename is provisional and #2 may change it.

## The rung

The rung is a property of the model configuration and not of the code. The
solver assembles one coupled system, and a lower rung is that system with
species and reaction terms absent from it.

Absent rather than zeroed. A term left in place with a zero coefficient still
costs the solve, still lets a field nobody is updating reach a convergence
criterion, and leaves a partly coupled state that reads as the lower rung
without being it. Absence is a state something can be asked about. A zero is a
number that has to be trusted.

Two rungs are reached in milestone 4. Concentration dependent diffusivity
through charge states is the first thing that works end to end. Pair diffusion
with the self interstitial and the vacancy solved as their own species is the
target of the same milestone, with those species able to be switched off.

Nothing below is reached as a model of a process. Constant diffusivity with an
Arrhenius temperature dependence remains available as a configuration, because
issue #32 needs a case whose closed form answer is known in order to check the
integrator before any physics is in it. As a model of a real anneal it is wrong
by an order of magnitude and it is not offered as one.

Nothing above is reached in the first release. Full kinetic models with extended
defects, clusters and precipitates are what a production tool for advanced nodes
needs, and they add species faster than they add data to fit them with. Issue
#12 makes measurement the reference, and a species with no measurement behind it
is a fitted knob wearing the clothes of a mechanism.

## Why the point defect rung is the target rather than an aspiration

It is the first rung that can produce transient enhanced diffusion, oxidation
enhanced diffusion and emitter push, and those are what separate a process
simulator from a program that solves the diffusion equation. They are the reason
this project is worth building rather than a feature it would be nice to have.

The rung also cannot be reached by accretion. Adding a defect species to a
solver written for one is not a configuration change, it is a rewrite of the
assembly, the Jacobian and the time step control, because the reaction terms are
stiff relative to the transport terms by many orders of magnitude and the
integrator has to have been built for that. Deciding the rung after the first
solver exists means the rung is whatever the first solver could reach, which is
the bottom two, and the rewrite arrives later under more pressure.

## Why the charge state rung ships first anyway

Milestone 4 has to land something usable while the harder rung is being
validated. The charge state rung needs a few coefficients per dopant, it covers
a large part of what a user actually asks about, and it produces the shoulders
and kinks that a measured high concentration profile shows and a constant
diffusivity never does.

It also gives the higher rung a target that is not a measurement. With no defect
source and enough time, the point defect model has to reproduce the charge state
answer, which is a check on the implementation rather than on the physics.
Issue #34 carries that obligation.

## The species that exist by name

Boron, phosphorus, arsenic and antimony as dopants. The silicon self interstitial
and the vacancy as point defects, solved as their own species when that rung is
active.

The dopant and defect pairing reaction exists at that rung. Whether the pair
concentration is carried as its own unknown or expressed from the dopant and
defect concentrations is a numerical choice inside the same rung and belongs to
issue #34, not here.

The list is short because a species is expensive. Each one costs a parameter set
that has to be sourced and cited under issue #11, at least one validation case
under issue #12, and a place in every result document under issue #10. That cost
is the reason germanium, carbon, fluorine, nitrogen and hydrogen are absent, and
the absence is not free: carbon and fluorine are co-implanted industrially to
suppress transient enhanced diffusion, so a user who co-implants them gets a
number computed as though they had not.

## How a user selects a simpler model

The recipe names the rung.

Not a build flag, because the rung changes the answer and issue #10 requires
anything that changes the answer to be in the manifest, and a manifest field has
to come from something the run was given.

Not an inference from which parameters are present, because that makes adding a
parameter file a silent change of physics, and the user who did it has no reason
to suspect the model moved.

A recipe that names no rung is a substituted default in the sense of issue #14.
It is recorded as a substitution under issue #10 and never as a value, so a
reader can tell a rung the operator chose from a rung the run supplied.

A recipe that names a rung the parameter set in force cannot support is a
refusal, not a quiet downgrade. Falling back to a lower rung would produce a
plausible number for a model the user did not ask for, which is the failure this
whole board is arranged against.

## What this rules out

Each item says what happens to a user who asks anyway.

Extended defect evolution. End of range dislocation loops after a high dose
implant are not represented, so the interstitial supersaturation they store and
release is missing; a user annealing after such an implant gets a profile
computed as though the loops were not there, and the rung recorded in the result
under issue #10 is what tells them which model produced it.

Dopant clustering, precipitation and the deactivation that comes with them.
Above solid solubility the model goes on treating all the dopant as mobile and
electrically active, so both the profile and any sheet resistance derived from
it are wrong in that regime rather than uncertain in it.

Amorphisation and solid phase epitaxial regrowth. There is no amorphous phase in
the model, so an implant heavy enough to amorphise is annealed as though the
lattice were intact. Adding it is a change of rung rather than a parameter, and
it is outside the first release.

Stress dependent diffusivity. Mechanical stress is outside the boundary drawn in
issue #3 for the first release, so a structure whose diffusivity is genuinely
stress modified is computed with the unstressed coefficients and nothing in the
result marks the case as one where that matters.

Any material system for which no parameter set exists, which today is everything
except silicon, and silicon germanium only in part. A recipe naming such a
material is refused under issue #14 rather than run with a neighbouring
material's coefficients, because a silently substituted material is a wrong
answer that looks like a right one.

Anything transient at the charge state rung. A user who selects that rung and
expects enhanced diffusion after an implant gets a monotone anneal, and the
reason is that the mechanism is not in the model at that rung rather than that
it was switched off.

Three dimensions. Entry 6 of issue #1 is open and this record assumes nothing
about it in either direction. The rung is independent of the dimensionality; what
depends on it is the structure in issue #5 and the discretisation in issue #6.

## The parameter set

The charge state rung is computed with the charge state dependent coefficients
published in

    R. B. Fair, "Concentration profiles of diffused dopants in silicon", in
    F. F. Y. Wang (ed.), Impurity Doping Processes in Silicon, North-Holland,
    1981, pp. 315-442.

It is named here so that the name reaches the result document under issue #10
rather than living in a header where nobody reading a number will see it.

The point defect rung's coefficients are not named here. That set is a much
larger object, and choosing it is issue #11, which decides where every parameter
comes from and how it is cited. The review this project reads that rung's model
out of is

    P. M. Fahey, P. B. Griffin and J. D. Plummer, "Point defects and dopant
    diffusion in silicon", Reviews of Modern Physics 61 (1989) 289-384.

Whether either set is distributed with the tree as a default is entry 3 of issue
#1 and is not decided here. What this record fixes is that no set is used
without being named, whichever way that entry goes.

## The issues in milestones 4, 5 and 7, checked against this

The tracker moves, so re-run this rather than trusting the paste:

    gh issue list --repo iderex/schichtwerk --state open --limit 300 \
      --json number,title,milestone \
      --jq '.[] | select(.milestone.number == 4 or .milestone.number == 5 or .milestone.number == 7) | "\(.number) m\(.milestone.number) \(.title)"'
    39 m5 Oxidation enhanced and retarded diffusion, coupled through the point defects
    38 m5 The moving oxide interface, silicon consumption and dopant redistribution
    37 m5 Oxidation kinetics in one dimension, and the thin oxide regime
    36 m4 Diffusion validated against published profiles, with the disagreement published too
    35 m4 Surface and interface conditions, including segregation
    34 m4 Point defect coupled diffusion: interstitials, vacancies and pairs
    33 m4 Concentration dependent diffusivity through the charge state model
    32 m4 The diffusion solver core: time stepping, tolerance and what convergence means

None of them assumes a different rung, so none is rewritten.

#33 names the charge state rung as this record's first shipped rung and says in
its own body that it cannot produce transient behaviour. #34 names the point
defect rung, carries the switch back to #33 that this record requires, and
carries the obligation to reproduce #33 in the no defect source limit. #39
couples oxidation to the defect species and requires that switching the defect
model off disables the coupling cleanly rather than leaving a partly coupled
state, which is this record's rule about absent rather than zeroed terms applied
to a specific case.

#32, #35, #36, #37 and #38 are rung neutral. The solver core is checked against
constant diffusivity, where the answer is known in closed form and no rung is
being asserted. The boundary conditions serve whichever species are active. The
validation suite compares whatever the model produced against a measurement.
Oxidation kinetics and the moving interface are geometry and transport rather
than a statement about how a dopant moves.

Milestone 7 held no issues when this was checked, which is what the command
above shows by returning nothing for it. The tracker is being filled while this
is written, so the paste is a snapshot and the command is the authority.

## The means

Markdown in the repository, read by a person and quotable in an issue. The
artefact is an argument that has to survive being disagreed with, so it has to
sit in version control where what was decided and when is recoverable from git.
It adds no language, no runtime and no dependency, and nothing outside this
repository forces the choice.
