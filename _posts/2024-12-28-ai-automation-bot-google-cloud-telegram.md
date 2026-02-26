---
layout: post
title: "How I deployed an AI automation bot on Google Cloud with Telegram"
date: 2024-12-28
categories: [Automation]
excerpt: "Taking an AI bot from a local Windows setup to a 24/7 cloud deployment with Telegram as the interface."
read_time: 4
---

Running a bot locally is fine for testing. Running it on your laptop 24/7 is not. After a few months of keeping my AI automation script alive through sleep mode restarts and accidental browser closures, I finally moved it to Google Cloud — and wired Telegram in as the interface. Here's how.

## The original setup

The bot started as a Python script running locally on Windows. It would watch a spreadsheet for new rows, trigger a series of OpenAI API calls, and write results back to the sheet. Simple enough, but fragile: it needed the machine running, it needed the VPN connected, and it crashed silently if anything timed out.

The goal was to get to a setup where the bot runs continuously without babysitting, I can trigger it from my phone, and I get notified when it completes a task.

## Moving to Google Cloud Run

Cloud Run was the right choice for this because the bot doesn't need to run constantly — it responds to triggers. That means I only pay for the seconds it's actually doing work.

The migration involved three steps:

1. **Containerize the script** — Write a `Dockerfile`, package the Python script and its dependencies, push the image to Google Artifact Registry.
2. **Set up environment variables** — API keys, sheet IDs, and Telegram bot tokens all go into Cloud Run environment variables (not hardcoded, not in the container).
3. **Configure a Cloud Scheduler trigger** — For time-based tasks, Cloud Scheduler sends an HTTP POST to the Cloud Run service on a cron schedule.

> "The shift from 'always on' to 'triggered on demand' changed how I think about automation architecture entirely."

## Adding Telegram as the interface

The Telegram bot piece was surprisingly straightforward using the `python-telegram-bot` library. The flow looks like this:

- I send a command to the Telegram bot (e.g., `/run` or `/status`)
- Telegram's webhook forwards it to a Cloud Run endpoint
- The endpoint processes the command and triggers the appropriate function
- When the task finishes, the bot sends me a message with the result

Setting up the webhook is a one-time step:

```
https://api.telegram.org/bot{TOKEN}/setWebhook?url=https://your-cloud-run-url/webhook
```

After that, every message to the Telegram bot hits your Cloud Run service — no polling, no open connections.

## What I'd do differently

A few things I'd change if starting over:

- **Use Secret Manager from day one** instead of environment variables for sensitive values
- **Add structured logging** with Cloud Logging so errors are easier to trace
- **Build a `/help` command** into Telegram immediately — future-you will thank present-you

The end result is a bot that's genuinely always on, costs pennies per month to run, and I can control from my phone wherever I am. That's the kind of infrastructure that actually gets used.
