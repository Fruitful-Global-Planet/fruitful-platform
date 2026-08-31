# Fruitful Platform

> Production monorepo for Fruitful™ commerce, portals, kiosks, CI governance,
> Shopify integration, Supabase operations, and controlled Gate 1–5 releases.

## Purpose

Fruitful Platform is the production application repository for the Fruitful™
ecosystem.

It is the governed location for software that can affect live customer,
operator, supplier, franchise, kiosk, device, portal, commerce, payment, and
operational workflows.

This repository is not an experimental sandbox. New concepts are developed,
tested, and reviewed in BuildNest / Fruitful Labs before being promoted here
through a controlled pull request and release process.

## Platform principle

```text
Shopify                 = Commerce engine
Supabase / PostgreSQL   = Secure live operational data plane
Vercel                  = Application hosting and deployments
Cloudflare              = DNS, edge, protection, and routing
GitHub                  = Source control, review, audit, and release control
Fruitful Platform       = The governed application and integration layer
```

## What this repository owns

- Fruitful web portals and operator experiences
- Kiosk and point-of-sale interfaces
- Head Office administration interfaces
- CI Guide and approved brand asset distribution
- Supplier and distribution workspaces
- Shopify application integration, webhooks, and commerce sync
- Supabase schema migrations, RLS policies, Edge Functions, and generated types
- Device registration, configuration, heartbeat, and fleet control
- Gate 1–5 policy logic, release checks, and operating workflows
- Production documentation, runbooks, architecture decisions, and integration contracts

## Architecture

```text
Fruitful Platform Monorepo
        │
        ├── Vercel
        │   ├── Fruitful Portal
        │   ├── Fruitful Kiosk
        │   ├── Fruitful Admin
        │   ├── Fruitful CI Guide
        │   └── Fruitful Supplier Portal
        │
        ├── Shopify
        │   ├── Products and collections
        │   ├── Cart and checkout
        │   ├── Payments and online orders
        │   └── Customer commerce records
        │
        └── Supabase / PostgreSQL
            ├── Portal sites and gates
            ├── Roles and permissions
            ├── Kiosk and device fleet
            ├── CI approvals and compliance
            ├── Operational mirrors of commerce events
            ├── Audit events and workflows
            └── Realtime operational state
```

## Source-of-truth rules

| Domain | System of record |
|---|---|
| Products, collections, carts, checkout, online payments | Shopify |
| Commerce orders and customer commerce records | Shopify |
| Portal sites, gates, operational roles, CI approvals | Supabase |
| Devices, kiosks, heartbeat, configuration, compliance | Supabase |
| Application code, schema migrations, integrations, docs | GitHub |
| Hosted application releases | Vercel |
| DNS, edge routing, WAF, cache policy | Cloudflare |

**Rule:** Shopify owns commerce. Supabase owns Fruitful operations.
Fruitful Platform owns the application logic that connects them.

## Gates

| Gate | Name | Production purpose |
|---|---|---|
| Gate 1 | Foundation | Roles, RLS, sites, consent, audit, controlled schema baseline |
| Gate 2 | Connectivity and devices | Device registration, site readiness, configuration, health and connectivity workflows |
| Gate 3 | Commerce synchronisation | Shopify catalogue, orders, inventory signals, kiosk/menu read models |
| Gate 4 | Transactional operations | Payment confirmation, fulfilment, supplier workflows, controlled order events |
| Gate 5 | Network governance | Franchise CI governance, compliance, supplier operations, fleet control and automation |

A gate is not merely a front-end toggle. A gate must be enforced through
database policy, service authorization, and controlled deployment rules.

## Repository structure

```text
fruitful-platform/
├── apps/
│   ├── portal/                 # Franchisee and operator portal
│   ├── kiosk/                  # Kiosk and POS application
│   ├── admin/                  # Head Office control centre
│   ├── ci-guide/               # Protected CI and brand-guidance portal
│   ├── supplier-portal/        # Supplier and distribution workspace
│   └── shop/                   # Shopify-connected Fruitful storefront
│
├── packages/
│   ├── ui/                     # Shared Fruitful design system
│   ├── brand-core/             # CI tokens, approved assets and layouts
│   ├── auth/                   # Role and permission helpers
│   ├── gates/                  # Gate 1–5 definitions and policy helpers
│   ├── shopify/                # Shopify GraphQL client and webhook logic
│   ├── supabase/               # Supabase client, database types and helpers
│   ├── device-core/            # Device, kiosk and fleet protocols
│   └── integrations/           # Payment, email, logistics and other adapters
│
├── supabase/
│   ├── migrations/             # Versioned SQL migrations
│   ├── functions/              # Edge Functions and secure webhook endpoints
│   ├── seed.sql                # Development-only seed data
│   └── config.toml
│
├── shopify/
│   ├── app/                    # Fruitful custom Shopify app
│   ├── extensions/             # Shopify extensions, where needed
│   └── shopify.app.toml
│
├── docs/
│   ├── architecture/
│   ├── gate-governance/
│   ├── integration-contracts/
│   ├── runbooks/
│   └── security/
│
└── .github/
    └── workflows/
        ├── validate.yml
        ├── preview.yml
        ├── deploy-staging.yml
        └── deploy-production.yml
```

## Environment model

| Environment | Purpose | Shopify | Supabase | Vercel |
|---|---|---|---|---|
| Local | Individual development | Development store or mocks | Local/Sandbox project | Local preview |
| Preview | Pull-request validation | Development store | Staging project | Vercel preview deployment |
| Staging | Release verification | Development or staging store | Staging project | Staging deployment |
| Production | Approved live operations | Production store | Production project | Production deployment |

Production credentials must never be committed to Git. Store secrets only in
GitHub Actions environments, Vercel environment variables, Supabase secrets,
and approved Shopify configuration.

## Promotion model

```text
BuildNest / Fruitful Labs
        │
        │ Experiment, prototype, test, research
        v
Pull request to Fruitful Platform
        │
        │ Automated validation, security scan, review
        v
Staging
        │
        │ Explicit release approval
        v
Production
```

## Security rules

- `main` is protected; direct pushes are prohibited.
- All production changes arrive through pull requests.
- Database migrations require review before production deployment.
- Webhook payloads must be signature-verified before processing.
- Service-role and Shopify Admin tokens remain server-side only.
- RLS is enabled on exposed operational tables.
- Every privileged gate, role, compliance, payment-state, or operational-state change is audited.
- Agents may create branches and pull requests; they do not receive unrestricted production write access.

## Shopify ↔ Supabase integration

Shopify sends signed events to secure Fruitful webhook endpoints:

```text
Shopify event
    → Secure webhook endpoint
    → Signature verification and idempotency check
    → Supabase operational mirror
    → Fruitful portal, kiosk, admin, supplier and device updates
```

Initial webhook topics:

- Product updated
- Inventory updated
- Order created
- Order paid
- Order fulfilled
- Order cancelled
- Customer created
- Refund created

Only the commerce fields needed for Fruitful operational workflows should be
mirrored into Supabase. Do not duplicate Shopify without a defined purpose.

## Development rules

1. Build in a branch.
2. Run validation locally.
3. Open a pull request.
4. Review code, migrations, permissions, and integration behaviour.
5. Deploy to preview/staging.
6. Approve production release.
7. Record the release and any gate state change.

## Status

**Repository state:** Foundation scaffold pending  
**Commerce source of truth:** Shopify  
**Operational data plane:** Supabase / PostgreSQL  
**Application runtime:** Vercel  
**Governance hub:** CodeNest  
**Innovation lane:** BuildNest / Fruitful Labs

## License

Fruitful Shops Proprietary License v1.1 or the current Fruitful Holdings
proprietary license adopted for this repository.

---

Built for controlled growth: one governed platform, many Fruitful experiences.
