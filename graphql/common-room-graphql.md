# Common Room GraphQL Schema

Common Room is a community intelligence and go-to-market (GTM) platform that aggregates member signals across GitHub, Slack, Discord, LinkedIn, Twitter, and other channels. This conceptual GraphQL schema models the core domain objects exposed through the Common Room REST API and platform capabilities, enabling rich querying of community members, signals, organizations, activities, segments, workflows, and integrations.

## Schema Source

This schema is derived from the Common Room developer documentation at https://docs.commonroom.io/ and the REST API reference at https://api.commonroom.io/docs/community.html. Common Room does not currently publish a public GraphQL endpoint; this schema represents a conceptual GraphQL projection of the platform's data model.

## Core Domain Areas

### Members and Profiles

The `Member` type is the central entity in Common Room. Each member represents a person tracked across one or more community channels. Members have detailed profiles (`MemberProfile`), activity histories (`MemberActivity`), sentiment scores (`MemberSentiment`), tier classifications (`MemberTier`), and tags (`MemberTag`).

### Signals

Signals are the behavioral indicators that Common Room detects from members across integrated platforms. Each `Signal` has a type (`SignalType`), a source (`SignalSource`), and detailed metadata (`SignalDetails`). Signals drive member scoring, tier placement, and workflow triggers.

### Organizations and Accounts

Members belong to organizations (`Organization`), which can be enriched with firmographic data (`OrganizationDetails`). Organizations map to sales accounts (`Account`) with scoring (`AccountScore`) and contacts (`Contact`) with roles (`ContactRole`).

### Community and Channels

A `Community` is the top-level workspace in Common Room. Communities contain channels (`CommunityChannel`) sourced from integrations (Slack workspaces, GitHub repositories, Discord servers, etc.). Each `Channel` has a type (`ChannelType`) and detailed metadata (`ChannelDetails`).

### Activity and Posts

All member interactions are recorded as activities (`Activity`) with typed details (`ActivityDetails`, `ActivityType`). Longer-form interactions are modeled as posts (`Post`), threads (`Thread`), and replies (`Reply`), with reaction tracking (`PostReaction`).

### Segments

Segments (`Segment`) are dynamic or static lists of members that match defined criteria. Segment membership (`SegmentMember`) tracks when and why a member belongs to a segment.

### Workflows, Playbooks, and Alerts

Common Room automates outreach and responses through workflows (`Workflow`) with triggers (`WorkflowTrigger`) and actions (`WorkflowAction`). Playbooks (`Playbook`) are curated workflow templates. Alerts (`Alert`) notify users when conditions (`AlertCondition`) are met.

### Integrations

Common Room connects to Slack (`SlackIntegration`), GitHub (`GitHubIntegration`), Discord (`DiscordIntegration`), Twitter (`TwitterIntegration`), Salesforce (`SalesforceIntegration`), and HubSpot (`HubSpotIntegration`). All integrations share a base `Integration` type.

### API Access

API keys (`APIKey`) and tokens (`Token`) control authentication. Webhooks (`Webhook`) push events to external systems.

### Reports

Reports (`Report`) and report details (`ReportDetails`) provide analytics summaries of community health, member engagement, and GTM pipeline contribution.

## Key Queries (Conceptual)

- `member(id: ID!)` — fetch a single member with full profile and signal history
- `members(filter: MemberFilter, pagination: PaginationInput)` — paginated member list with filtering
- `signal(id: ID!)` — fetch a single signal with source and type details
- `signals(memberId: ID, type: SignalTypeEnum, dateRange: DateRangeInput)` — signals for a member or community
- `organization(id: ID!)` — fetch an organization with firmographic enrichment
- `segment(id: ID!)` — fetch a segment with its member list
- `workflow(id: ID!)` — fetch a workflow definition with triggers and actions
- `report(id: ID!, dateRange: DateRangeInput)` — fetch a community health report

## Key Mutations (Conceptual)

- `createMember(input: CreateMemberInput!)` — create a new member
- `updateMember(id: ID!, input: UpdateMemberInput!)` — update member profile data
- `addActivity(memberId: ID!, input: ActivityInput!)` — record an activity on a member's timeline
- `anonymizeMember(id: ID!)` — GDPR anonymization of a member record
- `addMemberToSegment(memberId: ID!, segmentId: ID!)` — add a member to a segment
- `createWorkflow(input: WorkflowInput!)` — create a workflow
- `triggerWorkflow(workflowId: ID!, memberId: ID!)` — manually trigger a workflow for a member

## Schema File

See `common-room-schema.graphql` for the full type definitions.
