# The discretisation, the moving domain, and what a field transfer guarantees

Decided by issue #6, milestone 1.

## Status

In force from the commit that lands this file.

The naming and numbering of decision records is fixed by issue #2, which is not
decided. This filename is provisional and #2 may change it. Nothing in the
content below depends on the name.

## Why this is a decision rather than an implementation detail

A process simulation solves on a domain that the thing being solved is
deforming. Oxidation consumes silicon and grows oxide while dopant crosses the
interface that is moving. Deposition adds material over a field that must not
notice. Every one of those steps ends with a field expressed on one
discretisation and needing to be expressed on another.

The failure that arrangement produces is invisible. A field that loses two per
cent of its integral on every transfer still produces a smooth, monotone,
entirely plausible profile, and every test that looks at the shape of a curve
passes. Nothing about the output says which of the numbers were physics and
which were interpolation. That is why the guarantee is written down before the
first mesh exists, as a number a check compares against, rather than after,
where the number would be whatever the first implementation happened to
achieve.

## The spatial discretisation

Finite volume on a boundary conforming mesh, for every transported species.

The property being bought is local conservation by construction. A finite volume
scheme states the balance over each cell in terms of fluxes through its faces,
and the flux leaving one cell is the same number, with the opposite sign, that
enters its neighbour. The integral of the species over the domain therefore
changes only by what crosses the outer boundary, and it does so as an identity
of the formulation rather than as a result that happens to come out close.

Continuous Galerkin finite element is the alternative with the better claim on
everything else in this field, and it was weighed rather than dismissed. It
carries anisotropic and tensor coefficients naturally, which matters because
issue #3 leaves stress dependent diffusivity outside the boundary of the first
release while issue #5 is asked to reserve a place for a tensor field, so the
term arrives later rather than never. It is also what the mesh libraries and the
adaptive refinement literature are built around, so the surrounding machinery is
richer. What it does not give without being built for it is local conservation.
Global conservation is available from the Galerkin statement, and local
conservation needs a flux recovery step whose accuracy is its own subject. In a
tool whose output is a dopant count, a formulation where the balance is
recovered is worse than one where the balance is the statement, even when the
recovered number is good.

Discontinuous Galerkin is locally conservative and handles a discontinuity
across an interface without apology, which is the one place in this field where
a discontinuity is physical. It costs several unknowns per cell where finite
volume costs one, on a system that is already several species coupled through
stiff reaction terms, and it costs a body of implementation experience this
project does not have. It is the right answer for a problem dominated by
advection with sharp fronts. The problem here is dominated by diffusion and
reaction, where its advantage is smallest and its cost is unchanged.

What finite volume costs, stated rather than left for a reader to find. A
tensor or strongly anisotropic diffusivity on a mesh whose faces are not aligned
with it needs a multi point flux approximation rather than the two point one,
which is more work than the equivalent step in a finite element formulation and
is a known source of poor conditioning on distorted cells. Second order accuracy
on an unstructured mesh needs care about where the face value is reconstructed
from, and the naive choice degrades to first order exactly where the mesh is
worst, which is next to the moving interface. Both costs are paid at the
interface, and the interface is where this field's accuracy is decided. They are
accepted because the alternative trades a conservation property that cannot be
recovered for accuracy properties that can be improved.

Interface quantities are not cell quantities. Segregated dopant and any other
quantity that lives on an interface rather than in either neighbouring material
has its own unknowns on the interface, and it enters the cell balances as a flux
rather than as a source spread over a cell. A quantity that is smeared into the
neighbouring cells has been given a thickness it does not have, and the profile
that comes out of it is wrong in the first few nanometres, which for a shallow
junction is the whole answer.

## How the domain moves

Deform the mesh with the boundary while it stays good enough, remesh when it
does not, and print the quality figure that decided which of those happened.

Deforming alone keeps every field on its own nodes, so between two steps there
is nothing to interpolate and nothing to lose. It ends when the deformation
inverts a cell, and in this field it does end: a deep trench etch, a re-entrant
profile and the volume expansion of oxide growth all move a boundary further
than a mesh laid out for the original shape can follow.

Remeshing at every step handles arbitrary deformation and pays the transfer cost
on every step whether or not the step needed it. Since the transfer is the
operation this record exists to bound, paying it unnecessarily is the one
overhead worth avoiding on principle rather than on speed.

The hybrid is what working codes in this field converge on, and it is chosen
here for the reason that it makes the expensive operation rare and countable
rather than continuous and unremarkable.

The quality measure is named rather than tuned in silence. It is the minimum
scaled Jacobian over the cells of the mesh, taken as the ratio of the cell's
Jacobian determinant to the product of its edge lengths, normalised so that an
ideal cell is one and an inverted cell is negative. The threshold below which
the run remeshes is part of the input, it is recorded in the manifest under
issue #10 because it changes the answer, and the achieved minimum is printed for
every step. A run that never remeshed and a run that remeshed at every step are
then distinguishable from the result set alone, which they are not if the
measure lives in a header.

The measure is stated in a form that reads on a line as well as on a triangle or
a tetrahedron, because milestone 3 solves on a line and milestone 8 does not,
and a quality measure that only exists in the second one leaves the first with
an unstated rule.

## What a transfer guarantees

The rule, and then the number.

A transfer is a change of discretisation, not a change of physical state. The
two states of the domain either side of a transfer describe the same material
with the same species in the same places, expressed on different cells. So the
integral of each transported species over the domain is the same number before
and after, and any difference is error.

That is why the guarantee can be stated as a number at all, and it is also why
the guarantee must not be pointed at the physics. Oxidation consumes silicon and
produces oxide, and dopant genuinely moves between materials across a moving
interface. Those are transported by the solver and accounted for in the balance,
and they are not transfers. Confusing the two produces either a check that fires
on correct physics or a check that has been loosened until it fires on nothing.
The check is on the transfer operator and on nothing else.

The tolerance for a single transfer is a relative error of 1e-12 in the integral
of every transported species. Written as a comparison a check can make: with Q the integral of one species over the domain before the transfer and
Q' the integral of the same species after it, the run computes

    e = |Q' - Q| / max(|Q|, Q_floor)

and refuses when `e > 1e-12`.

The tolerance for a run is a relative error of 1e-9 in the same quantity,
accumulated over every transfer the run performed. The two are separate
numbers because they fail for different reasons. A single transfer above 1e-12
is an operator that is not conservative and no number of steps will fix it. A
run above 1e-9 with every individual transfer inside 1e-12 is a mesh history
long enough for rounding to add up, which is a real limit on how long a case may
be and deserves to be named as one rather than discovered.

Where those two numbers come from. A conservative transfer is exact in exact
arithmetic, so what is left is floating point summation over the cells of the
mesh. In double precision, a compensated sum over the cell counts this project
expects sits several orders of magnitude below 1e-12, so the per transfer number
is loose enough that meeting it does not constrain the implementation and tight
enough that it cannot be met by an operator that merely interpolates well.
That is the property being bought: 1e-12 is a number an interpolation cannot
reach by being accurate, so passing it is evidence of the construction rather
than of the mesh being fine. The run number is three orders above it, which
admits a long history without admitting a systematic leak, because a systematic
leak reaches 1e-9 in a thousand transfers and rounding does not.

`Q_floor` exists because a relative error on a species whose integral is zero is
not defined, and a species that has not been introduced yet has an integral of
zero at every transfer before its implant step. It is a per species absolute
floor in the internal unit system, it is part of the input, and it is recorded in
the manifest, because a floor set high enough turns the check off and that is a
change to the answer in the sense issue #10 fixes. A floor that a recipe did not
set is a substituted default and is recorded as a substitution rather than as a
value.

None of these three numbers is measured. The tree holds no code that could
measure them:

    git ls-files -- '*.c' '*.h' '*.cpp' '*.hpp' '*.rs' 'CMakeLists.txt' | wc -l
    0

They are a bound this record sets for the implementation to meet, and the first
place they meet an operator is issue #48. If the transfer that is actually built
cannot meet 1e-12, the answer is that the operator is not conservative and the
issue is not done. Widening the number to fit a result is the move this record
exists to prevent, and any change to these figures is an amendment to this record
carrying what was measured and on which case, not an edit to a constant.

## What the run prints

The manifest under issue #10 already carries the achieved conservation figure
for every transfer, as the figure rather than as the fact that it passed. This
record fixes what that figure is, which is `e` above, per species and per
transfer, together with the tolerance that was in force.

The human summary carries less and never something the manifest does not hold.
Per species, the worst single transfer figure over the run, the accumulated
figure over the run, the two tolerances in force, and the number of transfers
performed. A user reading only the summary can then see how close their own case
ran to the bound, which is the point: a case that passed at 9e-13 on every
transfer is a different thing from one that passed at 1e-20, and only the first
is a warning about the next case.

The count of transfers is in the summary rather than only in the manifest
because it is the number that says whether the mesh history was long. A run that
remeshed four hundred times is a run whose accumulated figure is doing real work.

## What the check has to be, so that it refuses

Issue #32 and issue #48 implement this, and both of their Done-when clauses ask
for a refusal rather than a warning. The tolerance is written above so that
neither has anything left to decide about the comparison. There is a named
quantity, a named norm, a named floor, and a constant. Nothing in it depends on
reading a plot, on a human deciding whether a difference is large, or on a
tolerance that a case may supply for itself.

A violation stops the run. This record fixes that much and no more. Which class
it is placed in, what the message says and what exit status it carries is issue
#14, which is not decided, and this record assumes no answer to it beyond the
run stopping. What it does refuse in advance is the third option: the violation
is not recorded and carried past, because a transfer that has lost dopant has
made every subsequent number in the run a number about a different amount of
dopant, and there is no later point at which the run recovers.

## The refinement criterion

Where the mesh is refined decides where the answer is accurate, so the criterion
is part of this decision rather than a tuning knob.

The criterion is the union of three, and it is a union deliberately, because each
one alone is blind exactly where the next one looks.

An error indicator over the transported fields, normalised per species. The
obvious form of this is the gradient of the concentration, and it is not enough
on its own for the reason the next two exist. It is normalised per species
because dopant and point defect concentrations differ by many orders of
magnitude and an unnormalised indicator refines only for whichever species
happens to be largest.

Geometric refinement at every material interface, unconditionally, to a stated
number of cells on each side. This is the part a solution based indicator cannot
supply. Segregation puts a jump across the interface, and a jump is not a
gradient in the interior of either material: the cells on either side can both be
flat, the indicator sees nothing, and the one place in the domain where the
physics is hardest is the one place the mesh is coarse.

Refinement ahead of a moving front, by a distance derived from the boundary
displacement the step is about to take. Refinement that follows the solution is
refinement that arrives after the front has passed, and within a single step a
fast front crosses cells that were sized for where it used to be.

## What the criterion is known to under-resolve

Each item says what a user who hits it gets, because a list of limitations that
does not say what happens is a list nobody can act on.

A region where the field is flat but the coefficient is not. Concentration
dependent diffusivity through the charge state model, which is the rung issue #8
ships first, makes the diffusivity a strong function of concentration, so a
plateau near solid solubility is flat in the field and structured in the
coefficient. The indicator sees a flat field and coarsens, and the user gets a
plateau whose edge is in slightly the wrong place with nothing in the output
marking it.

A species that is small everywhere. Per species normalisation makes the
indicator relative, and a species whose concentration is near the floor
everywhere produces a large relative indicator from what is numerical noise. The
mesh refines where nothing is happening, and the cost lands on a case that did
not need it. The opposite case is worse and is the same mechanism: a species
that matters only in a thin layer can be normalised into invisibility by its own
large values elsewhere.

The interior of a thick oxide during a long growth. Nothing there has structure
worth resolving, and the geometric rule refines both sides of the interface
whatever the material is, so cells are spent where the answer is constant. This
is a cost rather than an error, and it is named because a user watching a long
oxidation slow down is entitled to know why.

A gradient that is steep in one direction and flat in the other. The indicator
as stated is isotropic, so an anisotropic feature is resolved by refining in both
directions and paying for a mesh that is fine where it did not need to be. In one
dimension this costs nothing. In two it is the difference between a mesh that
fits in memory and one that does not, and it is issue #48's to weigh against
anisotropic refinement, which this record does not foreclose and does not
require.

A feature that exists only inside a step. The front rule looks ahead by the
displacement of the boundary, and a feature that appears and disappears within
one step, such as a transient in the defect fields immediately after an implant,
is not a boundary displacement and is not seen. What limits that case is the time
step control in issue #32 rather than the mesh.

## What is forced by issue #5 and what is not

Issue #5 decides what a structure is, and it is not decided. This record was
written so that the answer moves as little of it as possible, and where it does
move something, that is said here rather than discovered later.

Independent of #5. Finite volume as the discretisation for transported species,
because all three of the shapes #5 weighs carry the fields on cells and the
conservation argument is about the flux statement rather than about where the
shape of the domain is stored. The transfer guarantee and all three numbers,
because a transfer is a change of discretisation whatever produced the new
discretisation. The refinement criterion, because it reads fields and interface
positions, both of which exist under every shape. Interface quantities having
their own unknowns, because that follows from the physics rather than from the
representation.

Forced by #5. How many transfers a step performs. Where the geometry and the
field mesh are one object, a step that moves the boundary performs one transfer
and the count in the summary is the count of remeshes. Where the geometry is
implicit and the fields live on a separate mesh, which is the shape #5 proposes,
every step that changes the geometry performs a transfer in each direction
whether or not the mesh quality required one, and the accumulated figure is
reached from a shorter history. The run tolerance of 1e-9 is unchanged by that,
and what changes is how quickly a case approaches it. Also forced by #5 is where
the interface position comes from that the mesh is made to conform to, which is a
stored boundary in one shape and the zero of a level set function in another, and
that difference decides how the conforming mesh is produced rather than what it
has to guarantee once produced.

Not forced by either, and named so that it is chosen rather than drifted into.
Whether the mesh is generated in this tree or by an existing generator is a build
question that follows issue #2 and is weighed in issue #48, and nothing above
depends on the answer.

## The dimensionality assumption

Entry 6 of issue #1 is open, and this record assumes nothing about it in either
direction.

What is written above holds in one dimension and in two. The quality measure is
stated so that it reads on a line as well as on a triangle. The conservation
argument is about a flux balance over cells and does not count dimensions. The
refinement rules are stated in terms of interfaces and fronts rather than of
faces and edges.

Three dimensions would not change any statement in this record. It would change
the cost of two of them: the isotropic indicator's waste grows with dimension,
and the multi point flux approximation on distorted cells is a harder problem on
a tetrahedron than on a triangle. Neither is a reversal, and both are cost.

The thing entry 6 does reach is the structure in issue #5, and through it the
count of transfers per step above. That dependency is stated in the previous
section rather than resolved here.

## The means

Markdown in the repository, read by a person and quotable in an issue. A decision
record has to be readable from a fresh clone with nothing installed, it has to
sit in version control so that what was decided and when is recoverable from git
rather than from memory, and it has to survive being disagreed with on the
merits. It adds no language, no runtime and no dependency to a tree that today
holds no build. Nothing outside this repository forces the choice.

## The issues this record binds

The tracker moves, so re-run this rather than trusting the paste:

    gh issue list --repo iderex/schichtwerk --state open --limit 300 \
      --json number,title,milestone \
      --jq '.[] | select(.number == 5 or .number == 32 or .number == 48) | "\(.number) m\(.milestone.number) \(.title)"'
    48 m8 The two dimensional mesh, and field transfer that survives remeshing
    32 m4 The diffusion solver core: time stepping, tolerance and what convergence means
    5 m1 Decide what a structure is: geometry, materials, interfaces and the fields on them

Issue #32 discretises as this record says and reports the achieved figure through
the manifest. Issue #48 builds the transfer, and the numbers above are what its
repeated transfer test is measured against. Issue #5 is open and the section
above says which parts of this move with its answer.

Nothing here is refused by a machine today. No code exists to refuse it, and the
check that would compare `e` against the tolerance is owed by the issues named
above rather than held anywhere in this tree. That is the whole disclosure of it.
