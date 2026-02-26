---
layout: post
title: "Building a closed-loop reporting system with HubSpot & Salesforce"
date: 2025-01-12
categories: [Marketing Ops]
excerpt: "A step-by-step guide to connecting your CRM and MAP for true revenue attribution across the full funnel."
read_time: 6
---

Most marketing teams report on leads. Great marketing teams report on revenue. The difference comes down to one thing: whether your marketing automation platform (MAP) and CRM are actually talking to each other in a way that closes the loop from first touch to closed deal.

Here's the system I built to make that happen with HubSpot and Salesforce — and why it's changed how we present results to leadership.

## Why closed-loop reporting matters

The classic problem looks like this: marketing generates 500 leads this month. Sales closes 40 deals. But which 40? And which marketing campaigns, channels, or content pieces drove them? Without a bidirectional sync between HubSpot and Salesforce, you're guessing.

Closed-loop reporting means every closed deal can be traced back to its original marketing source — and every marketing campaign can be measured by the revenue it influenced, not just the leads it generated.

> "The key insight is that attribution data needs to travel with the contact record from the very first touch, not be reconstructed after the fact."

## Setting up the data pipeline

The foundation is getting your field mapping right before you sync a single record. In HubSpot, I created custom contact properties for:

- **Original Source** — the channel (Organic Search, Paid Social, etc.)
- **Original Source Drill-Down 1 & 2** — campaign and ad group
- **First Page Seen** — the landing page that converted them
- **First UTM Source / Medium / Campaign** — raw UTM parameters from the first session

These properties are set once — on the first session — and never overwritten. That's the critical piece most teams get wrong: they update attribution fields on every page view instead of preserving the original source.

In Salesforce, I mapped equivalent Lead and Contact fields so the data flows through when HubSpot creates or updates records.

## Closing the loop with Salesforce

The "closed loop" part requires Salesforce Opportunity data to flow back into HubSpot. Here's how I set it up:

1. **Opportunity Created** — HubSpot gets notified via webhook when a new Opportunity is created in Salesforce, and the Contact's lifecycle stage moves to SQL.
2. **Opportunity Stage Changes** — Key milestones (Demo Scheduled, Proposal Sent) update HubSpot contact properties, letting you segment based on pipeline stage.
3. **Closed Won** — When an Opportunity closes, HubSpot marks the contact as a Customer and records the close date and ARR. This is what enables revenue attribution in HubSpot reports.

With this setup, you can build reports in HubSpot that show exactly how much revenue each campaign, channel, or piece of content influenced — not just how many leads it generated.

## What you can report on now

Once the pipeline is running, the reporting possibilities open up considerably:

- **Revenue by original source** — which channels drive the highest-value customers
- **Time to close by lead source** — does organic convert faster than paid?
- **Content-influenced revenue** — which blog posts, webinars, or ebooks appear in the deal history of your best customers
- **MQL-to-Close rate by campaign** — finally understand which campaigns drive real pipeline

This is the reporting that gets marketing a seat at the revenue conversation, not just the lead count conversation.
