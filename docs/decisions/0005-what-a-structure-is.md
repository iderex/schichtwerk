# What a structure is: geometry, materials, interfaces and the fields on them

Decided by issue #5, milestone 1.

## Status

In force from the commit that lands this file.

The naming and numbering of decision records is fixed by issue #2, which is not
decided. This filename is provisional and #2 may change it. Nothing in the
content below depends on the name.

## Why this is a decision rather than an implementation detail

Everything on this board reads or writes one object. A recipe step is applied to
it, a solver solves on it, it is what gets written to disk, and it is what a
device simulator eventually receives. Nothing else can be built until what it is
has been written down.

The reason it is hard is that a structure here is four things at once and they
change at different rates. There is a geometry, which moves during oxidation,
deposition and etching. There are regions of material, which appear and
disappear during those same steps. There are interfaces between them, which
carry quantities that belong to neither neighbour. And there are fields, which
live on the geometry and have to survive it moving.

A representation chosen after the first solver exists is a representation shaped
by whatever the first solver found convenient, and every later solver inherits
that shape. This record fixes it first.

## The shape

An implicit geometry owns the shape. A boundary conforming mesh owns the fields.
One named transfer connects them, in each direction, and it is tested rather
than assumed.

### What that means concretely

The geometry is a level set: a signed distance function per material, sampled on
its own grid, whose zero contour is the surface of that material. Where a point
is inside more than one, the one with the smallest value owns it, so material
assignment is a comparison rather than a stored tag, and no point can be in two
materials at once by construction.

The materials are a table of named entries, each carrying its properties by
reference to a parameter set rather than by value. A region is not a material.
A region is a connected component of the set of points one material owns, it is
derived from the level set functions rather than stored, and it appears and
disappears without anything having to be told.

The interfaces are their own objects with their own unknowns. An interface is
the locus where two named materials meet, taken in a fixed order so that a
quantity with a direction, such as a segregation ratio, has an unambiguous sign.
An interface carries segregated dopant, trap densities and anything else that
lives on the surface rather than in either neighbour. It is not a property of
the geometry and it is not a thin layer of a material.

The fields are cell quantities on the boundary conforming mesh, which is the
finite volume mesh issue #6 fixes. A field declares its species, its unit and
the materials it exists in, and it has a value on every cell of those materials
and nowhere else. A field that is absent from a material is absent rather than
zero, and the two are distinguishable in the file.

### Why not a layer stack

A layer stack with explicit boundaries is the classical one dimensional answer
and it is genuinely the fastest route to milestone 3. Materials are ordered,
interfaces are points, and a moving interface is a number changing. Every
operation is arithmetic on a short list.

It is not chosen because nothing about it generalises. There is no second
dimension in it to extend: the ordering that makes it cheap is exactly the thing
that has no analogue on a plane. Choosing it means milestone 8 rewrites this
layer and every solver written against it, which is the outcome this record
exists to avoid. The weeks it saves in milestone 3 are borrowed rather than
earned.

The one thing it does better is worth recording so that the cost of not
choosing it is visible. In one dimension a layer stack has no discretisation
error at the interface at all, because the interface is a coordinate. Every
other shape approximates that position, and the approximation is what the
transfer tolerance in issue #6 is about.

### Why not a tagged mesh alone

An explicit mesh with a material tag on each cell, with the interface
reconstructed wherever the tag changes, generalises to two and three dimensions
and is what mesh libraries and mesh formats expect. It has one representation
rather than two, so there is no transfer between them and no accuracy leaking
through one.

It is not chosen for the interface. Reconstructing a surface from where a tag
changes puts that surface on cell faces, which means its position is quantised
to the mesh and its curvature is whatever the cell pattern happens to give.
Dopant segregation and oxide growth both happen exactly there, and both are
sensitive to the position and the normal rather than to the average behaviour
of a neighbourhood. Refining fixes it slowly, at a cost that rises with
dimension, and it never removes the quantisation.

The second reason is the topology change. Two etch fronts meeting, a void
closing during deposition into a trench and a layer pinching off are routine in
this field rather than exotic. A tag based reconstruction handles each of them
as a special case in the reconstruction code, and the list of special cases is
the kind that is never finished.

### Why the implicit shape, and what it costs

It is chosen for the topology change and for the interface position. A level set
merges two fronts, closes a void and pinches a layer off with no special case
at all, because none of those is an event in the representation. And the surface
position is the zero of a continuous function rather than a set of faces, so it
has a position and a normal between cells.

What it costs is a second representation and a transfer every time a step changes
the geometry. That transfer is where accuracy quietly leaks, which is why issue
#6 fixes a tolerance for it before either representation exists, and why this
record makes the transfer a named, tested object rather than a step inside a
solver.

It costs one more thing that is easy to overlook. A signed distance function does
not conserve anything. Volume is a derived quantity of the zero contour, so the
volume of oxide grown is whatever the advection scheme produced rather than what
the reaction consumed. Nothing in issue #6 bounds that, because its tolerance is
on the integral of a transported species and the geometry carries no species.
The bound on it belongs to the oxidation work in issue #38, which is where the
consumed silicon and the grown oxide first have to agree, and this record's
contribution is to say that the two facts are separate rather than one.

## The transfer, and which direction issue #6's tolerance reaches

Issue #6 states that under this shape every step that changes the geometry
performs a transfer in each direction. That is accepted here, and this record
adds which of the two the tolerance is about, because a reader could otherwise
take one number as covering both.

Outward, from the field mesh to the level set grid, what travels is a velocity.
The surface speed comes from the solver, as an oxidation rate from the interface
flux or as an etch rate the recipe states, and it has to be extended off the
surface before the level set can be advanced. Nothing in issue #6 bounds that
direction, because a velocity is not a transported species and there is no
integral of it that has to be preserved. What bounds it is the accuracy of the
extension and of the advection scheme, and that is issue #49's to fix.

Inward, from the level set to the field mesh, the boundary moves and the fields
have to be expressed on the mesh that results. That is the direction issue #6
bounds, at a relative error of 1e-12 per transfer and 1e-9 over a run, and this
record neither restates those numbers nor changes them.

Saying which direction the number reaches is the whole of this section. A single
tolerance quoted over a paired operation is a tolerance that will eventually be
cited for the half it never covered.

## What each operation does to the representation

Stated per operation, because a representation that is only described in the
abstract is one where each solver decides for itself.

An implant deposits into a field. The geometry does not move, no material
appears or disappears, and no interface is created. What changes is the value of
one or more species fields on the cells the profile reaches, and the fields of
the point defects that the damage creates. This is the only operation of the five
that touches nothing but the fields, and it is the reason the implant interface
in issue #41 can be written against a field rather than against a geometry.

An anneal solves on the mesh. The geometry does not move. The species fields and
the point defect fields change together through the coupled system, and the
interface unknowns change with them, because segregation moves dopant across a
boundary that is not moving. No transfer occurs unless the mesh quality rule in
issue #6 fires, which during an anneal it should not, because nothing is
deforming the mesh.

An oxidation moves the geometry and consumes a material. Silicon is consumed and
oxide is produced at the same interface, so two level set functions move against
each other rather than one moving into empty space. Dopant redistributes across
the moving interface by segregation, and both of those are transported quantities
that the balance accounts for rather than transfers. The field mesh deforms to
follow the boundary and is regenerated when it will not, and each regeneration is
a transfer.

A deposition adds a material. A level set function that was empty becomes
non-empty, or an existing one grows, and the region count changes without
anything being told. The fields of the new material begin at whatever the recipe
states, and every field that does not exist in the new material is absent there
rather than zero. Deposition into a trench is the case that closes a void, and it
is handled by the representation rather than by a branch.

An etch removes a material. A level set function shrinks and may vanish, and a
region may split into two or disappear. The fields defined on the removed volume
go with it, and the amount of each species that left the domain is accounted for
at the boundary rather than discarded silently, because a species that leaves
unaccounted is a conservation failure that looks like physics.

Two operations that are not in the list of five are named so that the list is not
read as complete. A masking step changes no field and no material and changes
only which part of the surface a later step may reach, so it is a property of the
step rather than of the structure. A structure read from disk or written to it is
issue #10's and issue #30's, and it changes nothing except which of the two
representations is authoritative on the way out, which is the next section.

## What is authoritative on disk

The level set functions and the interface unknowns are authoritative for the
geometry. The mesh and the fields on it are authoritative for the fields. Both
are written, because either alone loses something the other holds, and neither is
derivable from the other cheaply enough to be regenerated on read.

That means a structure file carries two discretisations and a reader can find
them disagreeing. The rule is that the level set decides where material is, and a
mesh cell that the level set places in a different material than the file says is
a refused file rather than a repaired one. Issue #30 owns the round trip and
issue #29 owns the check.

## The invariants that hold on any structure at rest

At rest means between steps, not inside a solver iteration, and the distinction
matters: an intermediate Newton iterate may violate several of these, and a
structure handed back to the driver may not. Issue #29 turns this list into
checks, and it is a list rather than a prose description so that each entry is
separately refusable.

- Every point of the domain is owned by exactly one material, and the outside of
  every material is the outside of the domain. No gap and no overlap.
- Every material a region names exists in the material table, and every entry in
  the material table resolves in the parameter set that is in force.
- Every interface names two distinct materials, in the fixed order, and both of
  them exist. An interface between a material and itself is refused.
- Every interface object corresponds to a locus where those two materials
  actually meet in the current geometry. A stale interface left behind by a
  material that vanished is refused, and it is the failure an etch produces if
  the etch forgets to remove one.
- Every field declares a species, a unit and the materials it exists in, and it
  has a value on every cell of those materials and no value anywhere else.
- Every field value is finite. Not a number and infinity are refused here as
  well as at the four points issue #14 names, because a structure at rest is a
  place a bad value can be stored rather than merely passed through.
- Every concentration is non-negative. This is an invariant of a structure at
  rest and not of a solver iterate, and stating it here rather than in the solver
  is what stops a negative value being clamped silently on the way out.
- The mesh conforms to the geometry. Every cell lies in exactly one material by
  the level set comparison, and the boundary of the mesh lies within a stated
  tolerance of the zero contour. The tolerance is part of the input and is
  recorded in the manifest under issue #10, because a tolerance loose enough
  turns this invariant off.
- The structure declares its dimension, and every geometric quantity in it has
  the arity that dimension implies. A two dimensional structure holding a one
  dimensional normal is refused rather than broadcast.
- Regions, interfaces, materials and fields each have a canonical order that does
  not depend on the order they were created in. Without that, two runs that
  computed the same answer produce two different files, and the determinism
  position in issue #7 cannot be checked by comparing them.
- Where the reserved tensor field is present, it is symmetric, and its component
  ordering is the one the file format states.

The list is what this record commits to. It is not claimed to be complete, and
the reason is that several of the operations above have not been written yet, so
the invariants they will need are not knowable from here. Issue #29 adds to it
rather than only implementing it, and an addition is an amendment to this record
carrying what the new entry catches.

Nothing refuses any of these today:

    git ls-files -- '*.c' '*.h' '*.cpp' '*.hpp' '*.rs' 'CMakeLists.txt' | wc -l
    0

That is the whole disclosure of it. Every entry above is a rule a person reads
until issue #29 lands.

## The reservation for a tensor field

Issue #3 places mechanical stress outside the boundary of the first release and
requires this layer to reserve a place for a symmetric rank two tensor field on
the same footing as the scalar fields. That requirement has its home in the
record for #3 and is not restated here beyond what is needed to say what it
costs.

The reservation is that the field abstraction is generic over what a field
carries rather than fixed to one number per cell. A field is a species or a
quantity, a unit, a set of materials, and a value type, and the value type admits
a scalar and a symmetric rank two tensor. Nothing populates a tensor field today
and no solver writes one.

What that costs today, stated as cost rather than as nothing.

The field container cannot be an array of one number per cell, so every place
that reads a field reads it through the abstraction rather than through the
array. In a solver inner loop that is a real difference, and whether it is a
measurable one is not known here: nothing exists to measure, and issue #52 is
where the question is answerable rather than guessable.

The transfer operator in issue #6 has to be defined for a tensor before a tensor
field is ever populated, or the reservation is void at the moment it is used. A
scalar transfer that conserves an integral does not automatically conserve a
tensor component under a mesh whose orientation changed, because the components
are stated in a frame. Naming that here is cheaper than discovering it in
milestone 8.

The file format has to carry the value type, the component ordering and the
symmetry convention for a field that no writer produces. A format that gains
those later gains a version, and a version that a reader has to branch on is
the thing the round trip in issue #30 exists to keep honest.

The invariant list above gains one entry, and issue #29 implements a check for a
condition nothing can currently violate. That is the smallest of the four costs
and the one most likely to be dropped as pointless, so it is written down.

None of these four is measured. They are the costs this record can name from the
design, and the first place any of them meets a number is milestone 8.

## The dimensionality assumption

Entry 6 of issue #1 asks what is promised about the third dimension. It is open
and this record does not answer it.

The assumption this record makes, stated as an assumption rather than left to be
inferred: the structure is generic over the dimension from the first line. The
dimension is a property of an instance rather than of the type, the code carries
it as a parameter, and nothing hard codes a count of two or of three. What is
exercised and validated is one dimension in milestone 3 and two in milestone 8.
Three is representable and unexercised.

This is the position entry 6 calls the one that cannot be adopted retroactively
without rewriting this layer, and it is taken here for that reason alone rather
than as a prediction about what entry 6 will decide.

What changes if entry 6 answers differently, in each direction.

If entry 6 says one and two dimensions only and permanently, the genericity above
is a cost already paid for an option nobody exercises. That cost is the awkwardness
of writing every geometric quantity without knowing its arity, and it is paid in
the early milestones. Nothing has to be rewritten, and the honest description is
that this record over-insured.

If entry 6 says two now and three later, this assumption is what makes later
cheap, and the condition entry 6 is asked to state rather than a date is what
decides when. Nothing in this layer changes when that condition is met. What
changes is the mesh generator, the level set grid, the solver kernels and the
memory budget, and none of those is this record.

If entry 6 says three dimensions from the start, nothing in this record moves at
all. The milestones move, because a three dimensional case has to be exercised
and validated rather than merely representable, and the cost lands on issues #48
to #51 rather than here.

The asymmetry is the argument. Being wrong in the first direction costs
awkwardness in the early milestones. Being wrong in the other two costs this
layer and everything written on it.

## What this record does not decide

Whether the surface evolution is computed in this tree or by the existing open
topography library is issue #4, and it is open. This record chooses an implicit
geometry, which is the representation that library uses, so the answer to #4
changes who advances the level set and does not change what a structure is. That
is a deliberate consequence of the choice rather than a coincidence, and it is
the one place where this record makes issue #4 cheaper in one direction than the
other. A tagged mesh or a layer stack would have made depending on that library a
conversion at every step.

The on disk encoding is issue #10 with issue #30. This record fixes what has to
be expressible and not what the bytes look like.

The unit system every quantity above is stated in is issue #7.

What crosses to a device simulator, which is a projection of this object rather
than this object, is issue #13.

The tolerance and the norm for the transfer are issue #6 and are referenced here
rather than restated.

## The means

Markdown in the repository, read by a person and quotable in an issue. A decision
record has to be readable from a fresh clone with nothing installed, it has to sit
in version control so that what was decided and when is recoverable from git
rather than from memory, and it has to survive being disagreed with on the merits.
It adds no language, no runtime and no dependency to a tree that today holds no
build. Nothing outside this repository forces the choice.
