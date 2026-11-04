---
title: Global Replication with Postgres
timestamp: 2026-11-04T18:30:01
author:
published: true
description:
tags:
---

## Building a Global Store Using Postgres

[Datadog](https://www.datadoghq.com/) runs a globally distributed infrastructure in which client applications connect to the nearest datacenter, with a strong preference for local access. Supporting this model takes storage that is globally replicated, cost-efficient, highly reliable, quick to recover, and able to keep cross-region replication latency low.
Our team built a new global storage system on Postgres using only its built-in logical replication, with no custom extensions and no forked Postgres.
In this talk, we'll cover:

* Why our legacy global store fell short, and what we needed from a new system
* The architecture, key challenges, and design trade-offs behind the new store
* An AI-first approach to onboarding and operating the system
* How we leveraged Datadog's deep Postgres expertise and ecosystem
* Results and lessons learned from running it at scale

## Speakers

* [Sanketh Balakrishna](https://www.linkedin.com/in/sanketh1024/) is a Senior Engineering Manager at Datadog, where he leads the Managed Replication & Search team within the Transactional Storage group, which provides platforms for replicating and searching data.
* [Maxim Briskin](https://www.linkedin.com/in/maxim-b-1211091/) is a Senior Software Engineer at Datadog on the Data Replication team, which provides a fast, reliable way to replicate data across databases, regions, and systems. He’s also the lead engineer for the Global Storage initiative.

## Length

?? minutes


<!--
<a class="button is-primary" href="">register</a>
-->
