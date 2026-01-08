# FRD || Software Releases
Responsible: Karmel Elshinnawi Nick Stanford  
Accountable: Nick Stanford  
Consulted: Lauren Lee Idris Olubisi  
Informed: Mike Ward  
Parent Document: DevX: The Midnight Standard  

## Summary
Software releases are a necessary component of the DApp Development Lifecycle. In order to build and maintain working DApps, DApp builders will need to be aware of software releases and their implications to new and existing code bases. 

To create a clear and effective communication process for release versions intended for DApp Builders, it will be necessary to coordinate a standard for release notes across all relevant tools and aggregate those releases to our developers through a single funnel.


## Feature
A Software release communication protocol for DApp Builders.

## Context
DApp builders will need to maintain different software versions across every necessary tool in the dependency tree in order to maintain working DApps.

## Mechanism

### Standard Template for Release notes

A standard template for release notes across all tools will allow for continuity of necessary information and an easier to consume format for developers. The template should include detailed notes about every change in the release and its implication (as far as is known) to that specific tool as well as any other infrastructure or breaking changes.

Relevant Tools:
1. Midnight Node
1. Indexer
1. Proof server
1. Compact (and supporting tools)
1. Midnight JS
1. Midnight Ledger
1. Lace
1. Wallet SDK
1. DApp Connector API
1. Zswap

A standard template will require buy-in from responsible parties for each tool. Instead of adopting the template wholesale, teams could just include all necessary components in their existing templates.

### Single Communication Channel

A single communication channel that aggregates the release announcements of all relevant tools, so that a user only needs to check one place.

### SOP for Release Issue Escalation
An agreed upon process for raising technical issues with specific releases directly to responsible parties.

### Public Roadmaps (long term)
Public roadmaps are essential in providing necessary detail for DApp developers who may depend on the release of certain features. Roadmaps should be tailored per-tool, to a 12 month timeline (where possible) in order to allow users to best plan the implementation of their use case.

## Goal
Create a clear and effective communication process for software releases across all DApp builder tools in order to enable DApp builders to build and maintain DApps with reduced friction and minimal downtime.

## Assumptions
1. The DApp builder is aware of our communication channel and is actively monitoring it.
1. The communication channel is an aggregate of sources, not a single source. Each team will push to Github, we don’t want to hinder that part of the process, only bring together everything relevant for DApp builders.
1. A standard template will need a certain amount of flexibility for different tools.

### Constraints
1. Coordination across a significant number of teams will provide the primary constraint.
1. Specific tools may need a certain level of flexibility in their release notes

## User Flows
### User Definition
The users considered for Software Releases are:
1. DApp Builders
1. Ecosystem Tooling Builders
1. Infrastructure providers
1. SPOs

For the purposes of this document, the type of user is relatively agnostic to their collective need for a clear and effective communication process. If that process is applied across all relevant tools, it should serve all user types.

### User Journey
For the purposes of the user journey, all users will encounter the Software Release process in some way. The journey to the information system is not necessarily relevant here – so long as the information system is openly accessible to all types of users.

## Open Questions
Enforcement and release authority 

## Requirements
### Standard release note template across all tools MUST HAVE 
    
1. A .md template exists covering:
    1. New Features – comprehensive new feature description
    1. Changes – comprehensive notes covering changes to any existing code
    1. Breaking Changes – significant changes that may have an effect on other ecosystem tools.
    1. Appropriate handoff documentation, enabling Documentation Writers to craft user facing documentation. (migration guidance)
    1. Compatibility matrix

1. The template has been agreed upon by responsible parties

### Aggregate communication channel MUST HAVE 
1. A single communication channel exists where users can find all release information (and only release information) relevant to building or maintaining a DApp on Midnight.
1. A single source has been designated by each tooling team (Github is preferred)
1. A similar channel exists for internal stakeholders (MNF <> Shielded)
1. Each tooling component has a designated responsible party present

### SOP for issue escalation MUST HAVE
1. An SOP exists for the purpose of raising issues encountered by MNF (or the Community) directly to responsible parties for each tool.
1. The internal stakeholder channel is utilized to raise initial issues with specific releases to responsible parties
1. Responsible parties agree upon SOP
### Tool specific public roadmaps NICE TO HAVE[short term] MUST HAVE[long term]
1. A public roadmap exists for each of the stakeholder tools
    1. Midnight Node
    1. Indexer
    1. Proof server
    1. Compact (and supporting tools)
    1. Midnight JS
    1. Midnight Ledger
    1. Lace
    1. Wallet SDK
    1. DApp Connector API
    1. Zswap
1. The roadmap covers major features and enables a product focused MNF roadmap through unlocking use cases
