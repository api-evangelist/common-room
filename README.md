# Common Room

Common Room is a community intelligence platform that aggregates member signals from GitHub, Slack, Discord, LinkedIn, and other social platforms to help go-to-market teams identify buying signals, manage contacts, and automate outreach.

## APIs

- **Core API** — Create and update members, record activity to member timelines, retrieve member information, and anonymize member data for GDPR compliance. Documentation: https://api.commonroom.io/docs/community.html
- **SCIM API** — Manage user provisioning and deprovisioning for community-scoped resources. Documentation: https://api.commonroom.io/docs/scim.html

## Authentication

API tokens are created in Settings > API Tokens and used as JWT Bearer tokens in the `Authorization` header.

## Webhooks

Common Room supports outbound webhooks for six event types: new contact, new organization, new activity, contacts meeting criteria, anonymous website visit, and contact identified from website visit. Webhooks are configured with an optional shared secret delivered via the `x-commonroom-webhook-secret` header.

## Resources

- Website: https://www.commonroom.io/
- Developers: https://www.commonroom.io/developers/
- Documentation: https://www.commonroom.io/docs/
- Pricing: https://www.commonroom.io/pricing/
- Blog: https://www.commonroom.io/blog/
- GitHub: https://github.com/common-room
- LinkedIn: https://www.linkedin.com/company/common-room-hq
- X: https://twitter.com/commonroomhq

## APIs.json

This repository is maintained as an APIs.json profile for Common Room. See [apis.yml](apis.yml) for the full machine-readable index.
