# Research

The established work behind each principle in [SKILL.md](SKILL.md). Name a candidate's design from here, in one line with author and year. A source outside this list is citable only after you have fetched and read it this session.

## Screaming

- **Robert C. Martin, "Screaming Architecture" (blog, 2011).** A system's top-level structure should announce its use cases, not its framework; the framework is a detail kept at arm's length.
- **John Sweller, "Cognitive Load During Problem Solving: Effects on Learning" (_Cognitive Science_, 1988).** Working memory is small; effort spent on how material is presented (extraneous load) is taken from understanding it. A tree that must be decoded before it is understood adds extraneous load.
- **Xin Xia et al., "Measuring Program Comprehension: A Large-Scale Field Study with Professionals" (_IEEE TSE_, 2018).** Professional developers spent roughly 58% of their time on program comprehension, so structure that speeds comprehension pays off on most working hours.

## Slices and cohesion over layering

- **W. P. Stevens, G. J. Myers, L. L. Constantine, "Structured Design" (_IBM Systems Journal_, 1974); Yourdon & Constantine, _Structured Design_ (1979).** Define cohesion and coupling: modules should be strongly related internally and weakly connected externally. Layer folders scatter one feature across many modules, which lowers cohesion and raises coupling between them.
- **David L. Parnas, "On the Criteria To Be Used in Decomposing Systems into Modules" (_CACM_, 1972).** Decompose by the decisions likely to change, not by processing steps. A feature is a unit of change; a technical layer is a processing step.
- **Robert C. Martin, _Agile Software Development: Principles, Patterns, and Practices_ (2002), Common Closure Principle.** Classes that change together belong in the same package.
- **Jimmy Bogard, "Vertical Slice Architecture" (blog, 2018).** Organise around requests or features, coupling within a slice and minimising coupling between slices.
- **Melvin E. Conway, "How Do Committees Invent?" (_Datamation_, 1968).** A system's structure mirrors the communication structure of the organisation that builds it. Slices that match team ownership stay intact; slices that cut across teams erode.

## Public surface

- **Parnas (1972), information hiding.** Each module hides a design decision behind an interface; the rest of the system depends only on the interface.
- **John Ousterhout, _A Philosophy of Software Design_ (2018).** Deep modules offer a small interface over a large implementation; information leakage across modules is a key sign of bad design.
- **Eric Evans, _Domain-Driven Design_ (2003), Bounded Context.** A model applies within an explicit boundary; the boundary is where translation happens.

## Dependency direction

- **Robert C. Martin (2002), Acyclic Dependencies Principle and Stable Dependencies Principle.** The package dependency graph must have no cycles; depend in the direction of stability.
- **Robert C. Martin, _Clean Architecture_ (2017), the Dependency Rule.** Source dependencies point inward, toward higher-level policy; entry points and frameworks are outer details.
- **Alistair Cockburn, "Hexagonal Architecture" (2005).** The application core talks to the outside world through ports; adapters on the edge depend on the core, never the reverse.

## Naming

- **Evans (2003), Ubiquitous Language.** Code uses the same terms as the domain experts, one term per concept, within a bounded context.
- **Dawn Lawrie et al., "What's in a Name? A Study of Identifiers" (ICPC, 2006).** Full-word identifiers were understood better than abbreviations and single letters.
- **Ousterhout (2018), naming.** A good name is precise and consistent, and creates an image of what the thing is; a name that is hard to choose signals an unclear design.
