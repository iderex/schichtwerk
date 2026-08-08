# The handoff to device simulation

Decided by issue #13, milestone 1.

## Status

In force from the commit that lands this file.

The naming and numbering of decision records is fixed by issue #2, which is not
decided. This filename is provisional and #2 may change it. Nothing in the
content below depends on the name.

## Why this is a decision rather than an implementation detail

The open chain from register transfer level to layout is finished and stops
above the layer where devices are made. Finishing the layer below is only useful
if what comes out of it feeds what already exists. A tool that produces a
structure in a format nothing reads has not closed the gap, it has moved it.

Issue #3 puts device simulation outside the boundary for the reason that open
tools for it exist and the honest job here is to feed them. That exclusion is
what makes this a first class deliverable rather than a convenience, and it is
decided before the structure layer is built, because a format chosen afterwards
is a format bolted on.

## What has to cross

A mesh. A material assignment per region. The electrically active dopant
concentrations as fields on that mesh. And enough boundary information for the
receiving tool to know where its contacts and its insulator boundaries are.

What must not silently cross is the distinction between chemical and active
concentration. A device simulator handed the chemical profile above the
solubility limit computes a device that does not exist, and it computes it
without complaining, because a concentration is a concentration.

## The first consumer, and what it actually needs

DEVSIM is named as the first consumer. What follows is from its own manual
rather than from assumption, and the manual version is named because the manual
moves:

    https://devsim.net/meshing.html

which the page identifies as DEVSIM Manual 2.10.0, and which states

    DEVSIM supports reading version 2.2 meshes from Gmsh. In order to write
    this format, it is necessary to specify the mesh format when writing out a
    mesh file. From the gmsh command line, use the -format msh2 option.

and

    These meshes may only contain points, lines, triangles, and tetrahedra.

and

    When creating the mesh file using the software, use physical group names to
    map the difference entities in the resulting mesh file to a group name.

Regions, contacts and interfaces are each established by naming a Gmsh physical
group, through `add_gmsh_region`, `add_gmsh_contact` and `add_gmsh_interface`,
after `create_gmsh_mesh` and before `finalize_mesh`. A contact there is a
boundary to something outside the device and an interface is an internal
boundary between two regions, and the two are different objects rather than two
names for one.

The load bearing fact is what is absent from that list. Field values do not
arrive with the mesh. DEVSIM's command reference sets node quantities through
`set_node_values` and `node_solution`, taking a list of values, after the mesh
exists. So for the first named consumer the mesh file carries geometry, regions
and boundary groups, and the dopant arrives by a separate route.

That settles a question this record would otherwise have had to guess at.
Smuggling fields into the mesh file as element data would be work done for a
consumer that does not read them there.

One thing was not established from that documentation and is not asserted here.
Whether the node ordering in the mesh file is the ordering `set_node_values`
expects after `finalize_mesh` is not stated on the page above, and this record
does not claim it either way. Issue #54, which runs one flow end to end through
somebody else's simulator, is where that is established, and if the orderings
differ then the companion document below has to carry a key rather than an
implicit row order. The companion document is specified so that adding an
explicit key is an addition rather than a change.

## The mesh format

Gmsh MSH version 2.2, ASCII, with physical group names.

It is chosen because the first named consumer reads exactly that and nothing
else, and choosing a format the first consumer cannot read in order to be modern
would be choosing against the only user this deliverable has. That is the whole
argument, and it is stated as a consumer choice rather than dressed up as a
judgement about the format.

What it costs is that 2.2 is an old version of that format. Gmsh's current
series writes a newer one by default, so a user generating a mesh by hand has to
ask for the old one, and a future consumer may want the newer one. Adding a
second output version is an addition rather than a change: the companion
document below names the format and the version it describes, so a reader always
knows which one they have, and nothing else in this record moves.

The element restriction is inherited rather than imposed. Points, lines,
triangles and tetrahedra are what the consumer accepts, which means an export
carrying quadrilaterals or hexahedra is not exportable to it. Whether the mesh
this project builds is simplicial at all is issue #48's, and this record records
the constraint rather than deciding that issue.

A format of this project's own was weighed and is not chosen. It would express
everything, including the chemical and active distinction, natively. It costs a
converter for every receiving tool, and unmaintained converters are how
interoperability dies. Writing directly into one simulator's own input language
was also weighed. It is the fastest route to one working demonstration and it
makes that tool the only consumer, which is the opposite of the reason this
deliverable exists.

## How the fields travel

In a separate table beside the mesh, with a companion document that says what
every column is.

The table is a plain numeric table with one row per node and one column per
exported field, with a short header naming the columns. It is a table rather
than a structured document because the bulk here is numbers: a node count in the
hundreds of thousands turns a nested document into a file that is slow to write,
slow to parse and awkward in every language that would read it. Every receiving
language reads a numeric table without a library.

The metadata is a structured document, in JSON, for the reason issue #10 gives
for the manifest and issue #9 gives for the recipe. That split is the same one
issue #10 already makes between a manifest and the artefacts it describes: the
structure is where meaning lives and the bulk is where volume lives, and putting
either in the other's format costs something real.

## What the companion document says

It is the actual deliverable of this record. The mesh format is somebody else's
problem and the meaning of the fields is this project's.

It carries its own schema version, under the same versioning rule issue #9
states for a recipe, so a reader five years from now either reads it or is told
which version it needs.

It names the mesh file, its format, its version and its content hash, and it
names the field table and its content hash. A companion document separated from
either is then detectably separated rather than silently wrong.

It states the unit of the coordinates, because the mesh format does not, and a
mesh whose length unit has to be guessed is a device whose dimensions are
guessed.

Per field it states the column, the name, the species, the unit, the materials
it is defined in, what an absent value means in a material where it is not
defined, and the kind. The kind is a required member with an enumerated value,
so a consumer that validates the document cannot silently receive a field whose
kind it never read.

It states the material assignment, as a mapping from each physical group name in
the mesh file to a material name as that material is named in the parameter set
that was in force. Not a material name of this document's own invention, so that
a reader can go from the export back to the coefficients.

It states, for each boundary physical group, whether it is an outer boundary of
the domain or an internal boundary between two named regions, and which regions
it bounds. It does not call anything a contact, and the next section says why.

It carries the provenance issue #10 already fixes for a result: the recipe by
content hash, every parameter set by name, version and hash, and the commit the
executable was built from. An export is a projection of a structure in the sense
issue #10 means, so it declares itself a projection, names the structure it came
from by hash, and names the operation that produced it. A projection separated
from its result set can then still be traced.

It states the row ordering convention of the field table explicitly, rather than
leaving it as whatever the writer did.

## Chemical and active concentration

The rule is that the export carries active concentrations and nothing else,
unless the chemical concentration was explicitly asked for.

That is the part that actually prevents the confusion. A file that does not
contain the dangerous number cannot be read as containing it, and every
mechanism weaker than that depends on the receiving side doing something
correctly.

Where a user does ask for the chemical concentration, because they are
comparing against a measured profile rather than simulating a device, three
things hold together. Both fields are present, never one alone under an
ambiguous name. The names are distinct words rather than a base name and a
suffix, so that a reader truncating or ignoring a suffix does not land on the
wrong one. And the required kind member in the companion document says which is
which, so an automated consumer has a machine readable answer and not only a
naming convention.

What this cannot do is stated rather than left as an implication. Nothing here
prevents a person from loading the chemical column and calling it the active
one. This project can decline to put the number in the file by default, can
refuse to write it under an ambiguous name, and can make the distinction
machine readable. It cannot reach into the receiving tool. Anything stronger
than that would be a claim about somebody else's program.

## What is deliberately not exported

Each with the reason, because a list of omissions without reasons reads as an
unfinished feature list.

The chemical concentration, by default, for the reason above.

The point defect fields. Interstitials, vacancies, pairs and clusters are
internal to the process model. A device simulator does not consume them, and
their values are meaningful only against the model rung that produced them,
which issue #8 fixes. Exporting them invites them being read as a physical
statement about the wafer, which they are not.

The implicit geometry. Issue #5 makes the level set functions authoritative for
the shape inside this project. The export carries a mesh, which is what the
consumer reads, and the round trip that preserves the full structure is issue
#30's file rather than this one. An export is not a save.

The interface unknowns. Segregated dopant as an areal density on an interface
has no place in a device simulator's model, and mapping it into a volume
concentration in order to have something to export would be inventing a
thickness. Where the amount matters it is already in the result set, in the
authoritative structure.

The trajectory. An export is one state of the wafer at one point in the flow.
The time step history, the intermediate structures and the convergence record
stay in the result set under issue #10.

Any uncertainty on an exported number. Nothing computed here carries one. The
uncertainties issue #12 handles belong to measurements a result is compared
against, not to the result, and exporting a field with an uncertainty column
would suggest otherwise.

The coefficients. The export names every parameter set by name, version and
hash and copies none of their numbers, so there is one home for a coefficient
and the export is not a second one. That is issue #11's rule and this record
inherits it.

## Contacts

The export does not declare a contact, and this is a decision rather than an
omission.

A contact is a device concept. It is where the outside world is attached, and
which boundary that is depends on what the device is for. A process recipe as
issue #9 defines it does not carry that intent: it deposits, etches and anneals,
and nothing in it says which piece of metal is a terminal.

So the export states what it knows, which is the boundary structure, and the
user states what it does not, which is which of those boundaries is a contact.
Concretely, every boundary physical group is named and described in the
companion document, and the mapping from a named boundary group to a contact in
the receiving tool is made in the script the user runs, which for DEVSIM is
`add_gmsh_contact`.

Inventing a contact for the user would be guessing at the device, and guessing
wrong produces a device simulation that runs.

## What is not enforced

Nothing above is refused by a machine. There is no exporter, no companion
document, no schema for one and nothing that could read either:

    git ls-files -- '*.c' '*.h' '*.cpp' '*.hpp' '*.rs' 'CMakeLists.txt' | wc -l
    0

The two issues that change that are named rather than remembered, and the
tracker moves, so re-run this:

    gh issue list --repo iderex/schichtwerk --state open --limit 300 \
      --json number,title --jq '.[] | select(.number == 53 or .number == 54) | "\(.number) \(.title)"'
    54 One worked flow, from a recipe to a device simulation somebody else runs
    53 Export a meshed, doped structure a device simulator can read

Issue #53 builds the exporter against this record. Issue #54 runs one flow
through somebody else's simulator, and it is the only thing that can prove any
of this, because a format is proven by a consumer reading it and not by a writer
producing it. Until #54 has run, every claim in this record about what the
consumer will accept rests on that consumer's documentation rather than on an
observation, and the section above says which sentences of it were read.

## What this record does not decide

Whether the mesh this project builds is simplicial, and how it is generated, is
issue #48. This record records what the first consumer accepts.

What a structure is, and what the export is a projection of, is issue #5.

How a result set is laid out and what a projection has to declare about itself
is issue #10, which this record inherits rather than restates.

Whether this project ever reads a commercial process simulator's format is entry
4 of issue #1. This record is about what is written, and nothing in it assumes
an answer to that entry. The commercial formats are not candidates for what is
written here, which issue #13 already states and this record carries.

## The means

Markdown in the repository, read by a person and quotable in an issue. A decision
record has to be readable from a fresh clone with nothing installed, it has to sit
in version control so that what was decided and when is recoverable from git
rather than from memory, and it has to survive being disagreed with on the merits.
It adds no language, no runtime and no dependency to a tree that today holds no
build. Nothing outside this repository forces the choice.

The means for the artefacts the record specifies is a separate question and is
answered where each is chosen: an established open mesh format because a
consumer reads it, a numeric table because the content is bulk, and JSON because
the content is structure and this tree already chose it twice for that reason.
