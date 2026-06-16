# Metabase

[Metabase](https://www.metabase.com) is the easy, open-source way for everyone in your company to ask questions and learn from data.

## Architecture

```mermaid
flowchart TB
    User([User / Browser]):::actor

    subgraph Frontend["frontend/ — TypeScript + React"]
        UI[UI Components<br/>Dashboards, Question Builder, Admin]:::fe
        EmbedSDK[Embedding SDK]:::fe
    end

    subgraph Backend["src/metabase/ — Clojure Backend"]
        API[api / api_routes<br/>REST endpoints]:::gateway
        Auth[auth_provider<br/>session • api_keys]:::gateway
        Perms[permissions]:::gateway

        subgraph Core["Core Domain"]
            Models[models<br/>Card, Dashboard, User, DB...]:::core
            Queries[queries / dashboards / collections]:::core
            Search[search]:::core
            Notif[notification / pulse<br/>Alerts & scheduled reports]:::core
            Settings[settings]:::core
            Cache[cache]:::core
        end

        QP[query_processor<br/>MBQL → SQL/Native]:::pipeline
        DriverAPI[driver / driver_api<br/>Driver interface]:::pipeline
    end

    subgraph Drivers["modules/drivers/ — Database Connectors"]
        D1[Postgres]:::driver
        D2[MySQL]:::driver
        D3[BigQuery]:::driver
        D4[Snowflake]:::driver
        D5[... 20+ more]:::driver
    end

    subgraph Enterprise["enterprise/ — Commercial features"]
        EE[SSO, Sandboxing, Audit,<br/>Advanced Embedding, Serialization]:::ee
    end

    DB[(App DB<br/>H2 / Postgres / MySQL)]:::store
    DataDBs[(User Data<br/>Warehouses)]:::store

    User --> UI
    User --> EmbedSDK
    UI -->|HTTP/JSON| API
    EmbedSDK -->|HTTP/JSON| API

    API --> Auth
    API --> Perms
    API --> Core
    Core --> QP
    QP --> DriverAPI
    DriverAPI --> Drivers
    Drivers --> DataDBs

    Models --> DB
    Settings --> DB
    Notif --> DriverAPI

    Enterprise -.extends.-> Backend

    classDef actor    fill:#CFD8DC,stroke:#37474F,color:#102027,stroke-width:3px
    classDef fe       fill:#BBDEFB,stroke:#1565C0,color:#0D47A1,stroke-width:3px
    classDef gateway  fill:#B2DFDB,stroke:#00796B,color:#004D40,stroke-width:3px
    classDef core     fill:#C8E6C9,stroke:#2E7D32,color:#1B5E20,stroke-width:3px
    classDef pipeline fill:#DCEDC8,stroke:#689F38,color:#33691E,stroke-width:3px
    classDef driver   fill:#FFE0B2,stroke:#E65100,color:#BF360C,stroke-width:3px
    classDef ee       fill:#E1BEE7,stroke:#6A1B9A,color:#4A148C,stroke-width:3px
    classDef store    fill:#ECEFF1,stroke:#37474F,color:#102027,stroke-width:3px

    style Frontend   fill:#F5FAFE,stroke:#1565C0,stroke-width:2px,color:#0D47A1
    style Backend    fill:#F6FBF6,stroke:#2E7D32,stroke-width:2px,color:#1B5E20
    style Core       fill:#FBFEF8,stroke:#558B2F,stroke-width:1.5px,color:#33691E
    style Drivers    fill:#FFF8F2,stroke:#E65100,stroke-width:2px,color:#BF360C
    style Enterprise fill:#FBF6FD,stroke:#6A1B9A,stroke-width:2px,color:#4A148C
```

## Get started

The easiest way to get started with Metabase is to sign up for a free trial of [Metabase Cloud](https://store.metabase.com/checkout).

You get expert support, backups, upgrades, an SMTP server, SSL certificate, SoC2 Type 2 security auditing, and more (plus your money goes toward improving a major open-source project). Check out our quick overview of [cloud vs self-hosting](https://www.metabase.com/docs/latest/cloud/cloud-vs-self-hosting). If you need to, you can always switch to [self-hosting](https://www.metabase.com/docs/latest/installation-and-operation/installing-metabase) Metabase at any time (or vice versa).

## Key Features

- [Set up in five minutes](https://www.metabase.com/docs/latest/configuring-metabase/setting-up-metabase) (we're not kidding), or have us [host Metabase for you](https://www.metabase.com/cloud/) so you don't even need to think about it.
- Let anyone on your team [ask questions](https://www.metabase.com/docs/latest/questions/introduction) without knowing SQL.
- Use the [SQL editor](https://www.metabase.com/docs/latest/questions/native-editor/writing-sql) for more complex queries.
- Ask AI: [Metabot](https://www.metabase.com/docs/latest/ai/metabot) gives you answers you can trust, helps you write queries, and more. Or build your own [AI agent](https://www.metabase.com/docs/latest/ai/agent-api) to query your data.
- Build handsome, interactive [dashboards](https://www.metabase.com/docs/latest/dashboards/introduction) with filters, auto-refresh, fullscreen, custom click behavior, and more.
- Use [documents](https://www.metabase.com/docs/latest/documents/introduction) for long-form data analysis, and invite people to comment.
- [Transform](https://www.metabase.com/docs/latest/data-studio/transforms/transforms-overview) raw data into analytics-ready tables, track down broken dependencies, and define canonical metrics in Metabase's [Data Studio](https://www.metabase.com/docs/latest/data-studio/overview).
- Set up [alerts on your data](https://www.metabase.com/docs/latest/questions/alerts), or schedule [dashboard subscriptions](https://www.metabase.com/docs/latest/dashboards/subscriptions) to email, Slack, or even a webhook.
- Curate content in a [Library](https://www.metabase.com/docs/latest/data-studio/library), and [version your work with Git](https://www.metabase.com/docs/latest/installation-and-operation/remote-sync).
- [Embed Metabase in your app](https://www.metabase.com/docs/latest/embedding/introduction), with components for charts, dashboards, data browser, AI chat, and more. You can even put [an entire Metabase](https://www.metabase.com/docs/latest/embedding/interactive-embedding) in your app.
- Set granular [permissions](https://www.metabase.com/docs/latest/permissions/introduction) that work for both internal teams and embedded analytics, whether you co-locate your customer data, or give each customer their own database.
- Dark mode, content translations, and way more stuff than we can list here.

Take a [tour of Metabase](https://www.metabase.com/learn/metabase-basics/overview/tour-of-metabase).

## Supported databases

- [Officially supported databases](./docs/databases/connecting.md#connecting-to-supported-databases)
- [Community drivers](./docs/developers-guide/community-drivers.md)

## Installation

Metabase can be run just about anywhere. Check out our [Installation Guides](https://www.metabase.com/docs/latest/installation-and-operation/installing-metabase).

## Documentation

The [Metabase handbook](https://www.metabase.com/docs/latest/).

## Contributing

To contribute to Metabase, see our [Developer docs](./docs/developers-guide/start.md).

## Extending Metabase

Hit our API to integrate analytics. Check out our guide, [Working with the Metabase API](https://www.metabase.com/learn/metabase-basics/administration/administration-and-operation/metabase-api).

## Internationalization

We want Metabase to be available in as many languages as possible. See which translations are available and help contribute to internationalization using our project over at [Crowdin](https://crowdin.com/project/metabase-i18n). You can also check out our [policies on translations](https://www.metabase.com/docs/latest/administration-guide/localization.html).

## Security Disclosure

See [SECURITY.md](./SECURITY.md) for details.

## License

This repository contains the source code for both the Open Source edition of Metabase, released under the AGPL, as well as the [commercial editions of Metabase](https://www.metabase.com/pricing/), which are released under the Metabase Commercial Software License.

See [LICENSE.txt](./LICENSE.txt) for details.

Unless otherwise noted, all files © 2026 Metabase, Inc.

## Community

- [Discourse](https://discourse.metabase.com/)
- [Twitter](https://x.com/metabase)
- [LinkedIn](https://www.linkedin.com/company/metabase/)
- [YouTube](https://www.youtube.com/@metabasedata)
- [Reddit](https://www.reddit.com/r/Metabase/)

## Metabase Experts

If you'd like more technical resources to set up your data stack with Metabase, connect with a [Metabase Expert](https://www.metabase.com/partners/?utm_source=readme&utm_medium=metabase-expetrs&utm_campaign=readme).
