---
layout: post
title: "The UTM tracking framework that actually scales across campaigns"
date: 2024-11-15
categories: [UTM Tracking]
excerpt: "Stop losing attribution data. Here's the system I built and maintain for clean UTM coverage at scale."
read_time: 5
---

UTM parameters are simple in theory and a mess in practice. Every team I've joined has had some version of a UTM spreadsheet that nobody fully trusts, inconsistent naming that makes reports useless, and the occasional panic when leadership asks why 40% of web traffic is showing as "direct."

Here's the framework I built to fix that — and how to maintain it as your campaign volume grows.

## Why UTM hygiene fails

The core problem is usually one of three things:

1. **No naming convention** — "facebook" vs "Facebook" vs "fb" all look like different sources in GA4
2. **No central creation point** — people build links in their own spreadsheets, from memory, or not at all
3. **No enforcement** — even with a good system, old links with bad parameters keep circulating

All three problems are solvable with process, not just tooling.

## The naming convention

I use a strict lowercase, hyphen-separated convention across all parameters:

| Parameter | Format | Example |
|-----------|--------|---------|
| `utm_source` | platform name | `linkedin`, `google`, `newsletter` |
| `utm_medium` | channel type | `paid-social`, `cpc`, `email` |
| `utm_campaign` | campaign slug | `q4-2024-brand`, `webinar-nov` |
| `utm_content` | creative ID | `blue-cta`, `video-30s` |
| `utm_term` | keyword (paid search only) | `marketing-automation-software` |

The key rules: always lowercase, always use hyphens (not underscores or spaces), and never leave a parameter blank if you're going to include it at all.

## The central creation system

> "A single source of truth for UTM links isn't a luxury — it's the only way to keep attribution data clean at scale."

I built a simple Google Apps Script form that generates UTM links from a dropdown of approved values. The form:

- Forces selection from a predefined list for source, medium, and campaign prefix
- Auto-generates the link and copies it to clipboard
- Logs every generated link to a master sheet with timestamp, creator, and destination

The logging piece is underrated. When someone asks "did we ever send traffic to this landing page from LinkedIn?" — the answer is one spreadsheet search away.

## Handling the dark traffic problem

Even with perfect UTM discipline on your outbound links, some traffic will always arrive without parameters. The three main culprits:

- **Dark social** — links shared in Slack, WhatsApp, iMessage appear as direct
- **Email clients** — some clients strip parameters before following links
- **PDF links** — UTMs don't survive copy-paste from PDF documents

For email, the fix is easy: always UTM-tag every link in every email, then use a redirect service that preserves parameters. For dark social and PDFs, you can't fully solve it, but you can track it: a dedicated landing page per channel (e.g., `/from/podcast`) gives you clean data even without UTMs.

## Keeping it maintained

The system only works if it stays current. I run a monthly audit that checks:

- Are all active campaigns represented in the approved campaign list?
- Are there any new sources showing up in GA4 that aren't in our convention? (That means someone went rogue)
- Are the top traffic sources tagged, or is direct/none unusually high?

That last question is the health check. If direct traffic spikes suddenly, something broke — and it's usually a missing UTM somewhere in a new campaign or a tool integration that isn't preserving parameters.

Clean UTM data isn't glamorous work. But it's the foundation that makes every other marketing report trustworthy.
