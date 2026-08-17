# schichtwerk

The open EDA chain from RTL to GDS is closed, and then it ends abruptly below the layout level. For process simulation - implantation, diffusion, oxidation, deposition, etch topography - there is effectively nothing open, and the question is asked in forums without a usable answer as recently as April 2026. This is recorded as a multi-year undertaking before the work starts, so its milestones are small and each lands something usable alone. Its implantation part belongs to the bremsweg board, and this one depends on that board for it.

Planning happens on the issue tracker first. Every decision that shapes
the architecture is written down there with its reasons before the code
that depends on it exists.

## What this simulates, and what it does not

This project simulates the front end of the process flow at the wafer cross
section. It follows the geometry, the materials and the impurity and defect
fields as they evolve under implantation, thermal annealing, oxidation,
deposition and etching, given a recipe. It produces a structure with fields on
it, in a form another tool can read.

Five things sit outside that boundary, and each of them says what you get
instead. Device simulation stays out because open tools for it already exist and
the job here is to feed them, so this project exports to one of them.
Lithography stays out because it is its own field with its own tools, so a
masking step here is given a pattern and does not compute one. Equipment and
reactor scale modelling stays out because it answers a different question for
different users, so an ambient, a pressure or an etch rate enters as a number
the recipe states. Layout extraction and everything above it stay out because
the open chain already covers that layer. The stopping physics of an implant
stays out because [bremsweg](https://github.com/iderex/bremsweg) is the board
for it, and this one depends on that board.

Mechanical stress is outside the first release and is the one exclusion that is
not permanent. It changes diffusivity and it changes the oxidation rate, so a
tool without it is wrong for modern geometries. The structure layer reserves a
place for a tensor field, so it can be added later without rewriting the format,
and nothing computes one today.

What is promised about three dimensions is not settled. The reasoning behind all
of this, including what each exclusion costs, is in
[docs/decisions/0003-the-boundary.md](docs/decisions/0003-the-boundary.md).

## What this does with your data

Nothing computed on your machine leaves it. There is no telemetry, no crash
reporting, no result upload and no shared cache. Nothing is sent anywhere in
order to run a simulation. Personal data never leaves the host unless you
deliberately federate, meaning you configure an outward connection yourself,
knowing what it carries. No default federates.

A result document contains the recipe that produced it, and a recipe is often
the most closely held thing a manufacturer has, so the defaults are set for a
document that gets attached to an email. Your user name and your host name are
not recorded. Paths are recorded relative to a named root rather than as
absolute paths that contain your home directory. If you want the user name and
the host name for your own traceability, you switch them on.

You are the controller of anything personal that ends up in your files. This
project processes nothing on anyone else's behalf.

There is no code here yet, so everything above is a commitment that binds the
work. The reasoning, and the two places where the tree does not yet match it,
are in
[docs/decisions/0015-the-data-protection-position.md](docs/decisions/0015-the-data-protection-position.md).

See [NOTICE.md](NOTICE.md) for the intended-use notice.

To report a security problem privately, see [SECURITY.md](SECURITY.md). A model
that disagrees with a measurement is not a security problem. It is a correctness
problem, and it belongs on the issue tracker in the open.

## License

AGPL-3.0, copyright 2026 Nils Lehnen.

The full text is in [LICENSE](LICENSE).
