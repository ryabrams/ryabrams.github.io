---
layout: post
title: "Google Calendar Sync"
date: 2026-03-24
categories: [Projects]
excerpt: "A small script that mirrors my personal-calendar events onto my work calendar as [Personal] holds, so colleagues can't book over my commitments."
read_time: 2
published: false   # <- draft: not built or visible on the live site. Set true (or remove) to publish.
---

So this project was small yet plugs an annoying hole in my calendar management process.

## The Issue

To have a unified view of my work and personal calendars, I share my personal calendar to my work email. At work, having this view enables me to see if I have any personal commitments while scheduling work meetings.

Simple enough. I'm sure everyone does this.

But to block my colleagues from sending meeting invites for times when I have a personal commitment, I had to manually put a hold on my work calendar.

Repetitive. Inefficient. Annoying.

## The Solution

Using Claude Code (obviously), I built a really simple script.

The script checks my personal calendar and if it detects an event, it creates an event on my work calendar to essentially block off that time and show me as busy to any work colleague trying to book a meeting with me.

## How the Script Works

1. It scans my personal calendar every 15 minutes for any events within the next 30 days.

2. If an event is detected, it creates an event on my work calendar on that date and during the time of the calendar event.

## Settings I chose

- Scan every 15 minutes

- Check for events within 30 days

- Do not create events on my work calendar for all day personal events (ie. Vacation — those are usually handled differently at work so I don't want to apply it for those events)

- It creates an event on my work calendar called [Personal]. I don't want any of the information from my personal calendar showing up on my work calendar)
