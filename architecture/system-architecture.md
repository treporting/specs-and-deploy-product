# System architecture (r3)

The subsystems of treporting, their interfaces, and which subsystems realize each system
requirement (r2).

**Figure overview** - the subsystems of treporting and their interfaces

```mermaid
flowchart LR
  files[(file-based<br>SHAREPOINT, SVN,<br>GIT, JFROG)]
  attrs[(attribute-based<br>DOORS, JIRA,<br>DMS, CODEBEAMER)]
  conf[/configuration<br>of the customer/]
  idp[proxy or identity provider]
  user((user))

  subgraph treporting
    exsys[extractor-SYSTEM<br>one per system]
    excon[extractor-content<br>a package per kind of content]
    conv[converter-trace]
    stor[storage-trace<br>import]
    rep[reporter-trace]
    view[viewer-trace]
  end

  files -- ARC_14 --> exsys
  attrs -- ARC_14 --> exsys
  exsys -- ARC_15 files --> excon
  exsys -- ARC_19 items --> conv
  conf -. ARC_20 .-> conv
  conf -. ARC_20 .-> stor
  excon -- ARC_16 --> stor
  conv -- ARC_13 --> stor
  stor -- ARC_12 --> rep
  rep -- ARC_11 --> view
  view --> user
  idp -- ARC_17 --> rep
  rep -- ARC_18 --> stor
```

**Table subsystems** - the subsystems and their ids

| Subsystem | Id |
|---|---|
| viewer-trace | ARC_1 |
| reporter-trace | ARC_2 |
| storage-trace | ARC_3 |
| converter-trace | ARC_4 |
| extractor-content | ARC_5 |
| extractor-SYSTEM | ARC_6 |

There are two kinds of system:

- **file-based** systems - SHAREPOINT, SVN, GIT, JFROG - only manage files: the content lies in
  the folders and files themselves. extractor-content reads it, with a package for each kind of
  content and no plug-in mechanism.
- **attribute-based** systems - DOORS, JIRA, a DMS, CODEBEAMER - hold items whose attributes are
  customized by each customer. The converter maps those attributes to the model, by a
  configuration file of the customer.

Every system has an extractor of its own that lists and fetches what it holds. Converter,
storage and reporter speak one language, the information model (ARC_21), whose contract is
[attributes.md](attributes.md). Our own data in YAML has the schemas of ARC_22 as its contract, in
[schemas/](https://github.com/treporting/specs-and-deploy-process/tree/main/schemas); each schema names it as `@refines ARC_22`, and is a draft of the gel schema. Both ways end in
the import of the storage, which works the same for every system. A subsystem is either a folder
of a code repo, `treporting/<repo>/<subsystem>`, or a repo of its own, `treporting/<subsystem>`;
the field `repo` of its element says which. The folder of a subsystem in a code repo is its name
without the name of the repo: storage-trace in `trace/storage`. Each product has its own
subsystems, a storage too: later storage-tests in `tests/storage`.

**Decision**: what is delivered on its own and talks to others over an interface is a subsystem, an
element of this architecture. What only divides a task within a subsystem is a component. viewer,
reporter, storage and converter are subsystems, with interfaces between them, such as ARC_11 from
the viewer to the reporter; trace is the code repo that holds them, not an element. trace and
maturity in the viewer, objectcode and baseline in tools are components. The next part is classified
by this rule.

**Why**: an interface between parts delivered on their own is set here, in r3, so each side
can be built and tested against it. Within a subsystem nothing is delivered apart, so a
division there needs no interface of r3.

**Rejected**: trace as the subsystem, with viewer, reporter and converter as its components -
they are delivered on their own and talk over interfaces, which a component has not.
The allocation is in this file: the system requirements allocated to a subsystem are
its requirements, derived directly from r2, with no ids of their own. The requirements of the
architecture on a subsystem or an interface stand beside them, with ids of their own. The document is written
for whoever derives the software requirements of a subsystem (r5) from it.

The terms - system, item, content, work product, release, library, baseline - are those of
[definitions.md](https://github.com/treporting/specs-and-deploy-process/blob/main/definitions.md). The levels follow ISO/IEC/IEEE 12207 as set out in the section
Process of [harness.md](https://github.com/treporting/specs-and-deploy-process/blob/main/harness.md); this file is r3, the architecture definition of
ISO/IEC/IEEE 12207, 6.4.4.

The ids are `ARC_n` for the subsystems, the interfaces and the requirements of the architecture on
them, `ARCR_n` for the rationale and
`ARCD_n` for the decisions; a requirement that a decision made possible names it as its
uplink. The elements, the requirements, the allocation and the ids of rationale and decisions are in
[system-architecture.yaml](system-architecture.yaml), in the format of [formats.md](https://github.com/treporting/specs-and-deploy-process/blob/main/formats.md).

## Interfaces

Each interface between two subsystems, with what passes over it. Its requirements are in
[system-architecture.yaml](system-architecture.yaml).

### ARC_11 REST API (viewer-trace → reporter-trace)

The viewer asks the reporter over a REST API. The technical form is the swagger of the reporter;
this section says what goes across it. Every request names its scope (ARC_29) and may name the
process reference model (ARC_30); the routes on the trace and the states narrow the rows (ARC_31),
by level (ARC_32) and, on the code, by component and unit (ARC_33). Every answer comes as JSON or
CSV (ARC_36); a request that cannot be answered is refused with the reason (ARC_37).

**Table routes** - the routes of the reporter, what each answers, and what a request carries
beyond the scope

| Route | Answers | Parameters beyond the scope | Asked by the viewer |
|---|---|---|---|
| GET /version | the revision and the build of the reporter, the state of its data set and the number of its objects | - | yes |
| GET /health | that the reporter runs: 200, "ok" | - | no - for the operation |
| PUT /configuration | the lists the filters offer for a scope, from the filter sent; release `*` names the newest release | filter (body) | yes |
| GET /configuration/projects | the projects of an organization and how they are split | - (organization only) | yes |
| GET /configuration/baselines | the releases and libraries of a project | - (organization, project only) | yes |
| GET /configuration/artefacts | every work product of a subsystem, with its revision, kind, tool and baseline | process; no baseline | yes |
| GET /configuration/systems | the stores and what each holds | process (organization only) | yes |
| GET /configuration/functions | the functions of a scope | level, process | yes |
| GET /configuration/features | the features of a scope | level, process | yes |
| GET /configuration/safety | the safety classes of a scope | - | yes |
| GET /configuration/components | the components of the code and their units; release `*` names every release | - (no library) | yes |
| GET /maturity/requirements | the trace from a level, one row per object | level, narrowing, component, unit, process, above, keys | yes |
| GET /requirements/features | the trace from a feature | narrowing | yes |
| GET /maturity/features | the maturity per feature | level, narrowing, process | yes |
| GET /maturity/states | the counted states of a level over the libraries | level, narrowing, component, unit, process; no baseline | yes |
| GET /maturity/components | the components and the state of their requirements or tests | level, process | yes |
| GET /maturity/compliance | per clause of a standard what addresses it and its state | process | yes |
| GET /maturity/compliance/states | per release how many clauses are open, addressed and verified | process; no release | yes |

The levels above each row come only on request (ARC_34), so that a view of a large data set stays
within its time (SYS_52). A cell comes as the keys of its objects or as the count per state
(ARC_35): one table serves the single object of the trace (SYS_1) and the count of the states
(SYS_5).

### ARC_12 StorageSource (reporter-trace → storage-trace)

The reporter reads the data sets of the storage through `StorageSource`, a Go interface of the
storage. Behind it, the storage holds the data in one of four modes of operation: in a database
server of its own (ARC_23), in a database run together with the reporter (ARC_24), in a data file
(ARC_25), or as generated data in the memory of the reporter (ARC_26). The reporter answers the
same from each (ARC_27), and from one mode only while it runs (ARC_28).

## Rationale

The principles every decision below follows.

### ARCR_1 Read only

treporting only reads the data of its users. It never writes into a source system
and never holds data that a user enters into treporting itself. What it writes are records of
its own: the data set as read, the derived values beside it, the import records, the record of
every report and the audit trail.

**Example**: a finding of the plausibility analysis is evaluated where the customer works - an
attribute in DOORS, a ticket in JIRA linked to the requirement -, and treporting reads that
evaluation like any other data. The approval of a release is made where the organization makes
it, and names the report it relied on.

**Why**: we do not know how others work, so treporting fits into what is there instead of
prescribing a way of working. What a user decides stays in the system where it is evidence
without treporting. And treporting needs no input of its own - no rights to enter data, no
audit trail of entries, no second place where the truth could be.

**What it rules out**: evaluating findings or approving releases in treporting - it would leave
the maxim, and bring a way of working, an audit trail of entries and a second truth with it.

## Decisions

### ARCD_1 Independent count (SYS_23)

**Decision**: the source system is counted by a query of its own (ARC_14), in the extractor of the system
but in code apart from the reader, and the stored count is taken from the storage after the import. A reader never
reports what it read as the count of the source system. Besides, a verifier of its own, apart from all
subsystems, checks the data of treporting itself (the reference edgedb) and is run on request,
for instance by an assessor; it is not delivered with the product.

**Example**: a CODEBEAMER tracker holds 1 200 requirements, read in pages of 500. A fault in the
paging drops the last page, so 1 000 are read. A reader that counts what it reads reports
1 000 in the source system, 1 000 read, 1 000 stored - all equal, and wrong. The query of its own asks
for the total and gets 1 200; the difference of 200 marks the data set incomplete. On the other
side, two requirements share an id in the source system: 1 200 are read, the second overwrites the first,
and only a count taken from the storage afterwards shows 1 199. In code, an `@implements` written
over two lines is missed by the reader and found by a plain `git grep -c '@implements'`.

**Why**: a count made by the reader shares every error of the reading, so it cannot find one. Only
a count on another way detects the errors of an import with high confidence - which is what tool
error detection TD1 asks for, and what keeps treporting at TCL1 without qualification (see
Integrity in the system requirements). The verifier goes one step further, independent of the
whole product; it costs a repo of its own, so it is kept for the own data and for the cases where
an assessor asks for it, rather than run on every import.

**Rejected**: the reader counting for itself - cheap, but it misses exactly the faults above.

### ARCD_2 Import record (SYS_26, SYS_41)

**Decision**: the import of the storage brings the four counts together - the count of the
source system from the extractor of the system, read and refused from extractor-content or
the converter, stored from the storage itself -, so it sets the state
of the data set - complete, incomplete, or unverified when the source system has no count of its own - and
keeps the import record. The storage keeps the record with the data set,
the reporter hands the state on with every answer, the viewer shows it on every view.

**Example**:

```
import 2026-10-23T14:02:11Z  by m.fischer
  source system codebeamer, tracker 4711, revision 3812
  converter     converter-trace v1.4.0, configuration acme-codebeamer v7
  data set      release-2.1
  counted       requirement: source system 1200, read 1000, refused 0, stored 1000
                link parentOf: source system 3400, read 2890, refused 0, stored 2890
  refused       -
  fingerprints  1000 compared, 0 differ
  state         incomplete  (requirement -200, link parentOf -510)
```

**Why**: the record carries the trace of a report past the data set to the source system and its
revision (REG_FDADI_1), it is the audit trail of a change of data (REG_CFR21-11_1), and it is the evidence that
errors of an import are detected (REG_ISO26262-8_1, REG_IEC61508-3_1, REG_DO178C_1, REG_DO330_1).

### ARCD_3 Original and derived values (SYS_11)

**Decision**: the storage always keeps the original as read from the source system. A value derived at
the import is never the original and is kept beside it under a name of its own, with a
prefix: `M` for a value mapped - unified, translated or adapted to the model; where the original
is empty, the mapped value of the parent -, `I` for an intermediate form, only for the status, where
the original has to be transformed before it can be mapped. When a rule changes, the derived values can be made again on the stored data set,
back in time, so that the data need not be fetched from the source systems again - a requirement
that follows from this architecture, not from a need (SYS_69).

**Example**: the source system says `Status = 'im Review'`; the import keeps it and adds
`MStatus = 'review'`, as the configuration of the customer maps it. A status kept per platform -
`Status = "status_platform-1 = in Review_Vendor` / `status_platform-2 = open"` - is first transformed
into `IStatus = "platform-1 = in Review` / `platform-2 = open"`; `MStatus` is then `review` when the
filter selects platform-1, made for the filter set where it is not stored. A software requirement
without a safety class under parents of ASIL B and QM keeps `Safety` empty and gets
`MSafety = 'ASIL B'`; its test case and test result get `MSafety = 'ASIL B'` as well.

**Why**: what is stored can be compared with the source system (SYS_25) and stays original (REG_FDADI_1).
Where the evidence is that every requirement was classified by a person, the report shows
`Safety`, not `MSafety`; where the evidence is that nothing falls out of a filter, it shows
`MSafety`. A change of a rule needs no new import. Only a derived value that is stored can be
checked for plausibility - the reporter finds an original `Safety = 'QM'` under a parent
with `MSafety = 'ASIL D'` in its analysis (SYS_70); the import reads the data as it is.

**Rejected**:

- the reporter derives the values when it answers, and nothing derived is stored. It keeps the
  rule in one place and the storage plain, but a derived value then exists only in an answer:
  it cannot be checked for plausibility, compared between two reports, or shown to an auditor
  as what the report was made from.
- the storage derives them in a view of the database. It binds the rule to the database, and the
  data file (SYS_21) would need the rule a second time.

### ARCD_4 Configuration instead of code for the customer (SYS_22)

**Decision**: what differs from customer to customer - the customized attributes of an
attribute-based system and the values to be mapped - is a configuration file, read by one
general mechanism: the converter for the attributes, the import of the storage for the
values. Only what a configuration cannot say goes into code of its own, in a private repo per
system and customer, such as converter-doors-acme.

**Example**: one customer keeps the safety class of a requirement in the DOORS attribute
`ASIL_Rating`, another in `Safety Level`, with the values `QM, A, B, C, D` or `none, ASIL A, ...`.
Two configuration files map both to `Safety` and their values to `MSafety`; the code is the same.

**Why**: the formats of a customer are proprietary and must not reach the public repos. A
configuration stays with the customer; the mechanism stays public and is tested once for all.

**Rejected**: a converter in code per system - the proprietary formats would be in code, and the
same mechanics would be written once per system.

### ARCD_5 Identity (SYS_39)

**Decision**: treporting keeps no users and no passwords of its own. It takes the identity from
what the organization already has: preferably from a proxy in front of it that authenticates the
user and hands the identity and the groups on - transparent, treporting needs no logic of its
own for it -, and where possible from the identity provider of the organization over a standard
protocol (OIDC). The organization maps its groups to the roles of treporting by configuration.

**Example**: a supplier runs treporting in its own network; its people log in everywhere with
their company account. When someone leaves, the one account is locked, and treporting is closed
to them at the same moment - there is nothing to lock in treporting.

**Why**: we do not know how others manage their users, so treporting fits into what is there
instead of bringing a second management. A proxy keeps that entirely outside treporting.

**Rejected**: a user management of its own in treporting - a second place to lock an account,
and the duty of keeping passwords.

## Allocation

The allocation of every system requirement to the subsystems, and of every requirement of the
architecture to the subsystems and interfaces, is the field `allocated` of each of them in
[system-architecture.yaml](system-architecture.yaml).

## Open

- `treporting = 4_selfhosted` - **Licensed features**: not decided where their source lives and how
  the licence of an installation is proved. None exists yet. They would be in the product, checked
  per route - role, rights, licence -, under the Elastic License 2.0, in a directory of their own
  per repo.
- `treporting = 1_demo` - **Repos**: the repos of extractor-content and of the extractors per system
  are not created yet. reporter-annotations becomes the package annotations of extractor-content.

- `treporting = 1_demo` - **Interfaces**: the sections of ARC_13 to ARC_20 are not written yet.
  ARC_11 describes the routes there are; no route answers yet the findings of the plausibility
  check (SYS_71), the states of the code (SYS_77), a report of the views chosen (SYS_85), or the
  record of what users did (SYS_90).

## Change history

| Version | Date | Change | Reason | By |
|---|---|---|---|---|
| 1.1.1 | 2026-10-30 | - | #327 | Claude Opus 5.5 and Mathias Fischer, authors; Mathias Fischer, reviewer |
| 2.1.1 | 2026-10-30 | System architecture; Decisions | #334 | Claude Opus 5.5, author; Mathias Fischer, reviewer |
| 3.1.1 | 2026-11-12 | Links to specs-and-deploy, from the repo of its own | specs-and-deploy#640 | Claude Opus 5.5, author; Mathias Fischer, reviewer |
| 4.1.1 | 2026-11-13 | specs-and-deploy is named specs-and-deploy-process | specs-and-deploy-process#641 | Claude Opus 5.5, author; Mathias Fischer, reviewer |
| 5.1.1 | 2026-11-13 | The allocation: r4 gone | treporting/specs-and-deploy-process#653 | Claude Opus 5.5, author; Mathias Fischer, reviewer |
| 6.1.1 | 2026-11-17 | Keys without a leading zero | specs-and-deploy-process#679 | Claude Opus 5.5, author; Mathias Fischer, reviewer |
| 7.1.1 | 2026-11-18 | A subsystem is a folder of a code repo or a repo of its own | #26 | Claude Opus 5.5, author; Mathias Fischer, reviewer |
| 8.1.1 | 2026-11-19 | Where a subsystem lives is the field repo of its element; Table subsystems: no short forms | #28 | Claude Opus 5.5, author; Mathias Fischer, reviewer |
| 9.1.1 | 2026-11-20 | The rule that tells a subsystem from a component; storage-trace; the folder of a subsystem | #30 | Claude Opus 5.5, author; Mathias Fischer, reviewer |
| 10.1.1 | 2026-11-21 | Source system, not source; the interface StorageSource | #33 | Claude Opus 5.5, author; Mathias Fischer, reviewer |
| 11.1.1 | 2026-11-21 | Requirements of the architecture; the modes of operation; Interfaces, with ARC_12 | #34 | Claude Opus 5.5, author; Mathias Fischer, reviewer |
| 12.1.1 | 2026-11-21 | The interface ARC_11 REST API: its routes and its requirements | #37 | Claude Opus 5.5, author; Mathias Fischer, reviewer |
