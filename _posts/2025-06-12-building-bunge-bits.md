---
title: "Building bunge-bits, an AI-Powered Summary Pipeline for the Parliament of Kenya"
layout: post
date: 2025-06-12 19:00
image: /assets/images/bunge-bits.png
headerImage: true
tag:
    - Rust
    - AI
    - CivicTech
    - Kenya
    - GPT-4
    - Whisper
star: true
category: blog
author: Collins Muriuki
description: A deep dive into how I used Rust, Whisper, and GPT-4 to build a reliable AI pipeline that summarizes livestreams of Kenya’s Parliament, making civic proceedings more accessible to everyday citizens.
---

# Motivation

I built bunge-bits out of frustration. Kenya’s National Assembly and Senate regularly livestream their proceedings, but these sessions are long, often poorly indexed, and inaccessible to most people. The average citizen doesn’t have the time, or bandwidth, to sit through up to five-hour videos of parliamentary debates. Most people end up relying on the media to highlight key moments, often missing the full context or nuance.

At the same time, Kenya is experiencing a generational shift. Young people are increasingly vocal about governance, public spending, and corruption. I wanted to contribute something tangible to that movement, something that makes civic discussions more transparent, digestible, and shareable.

bunge-bits is my attempt to bridge that gap: an AI-powered pipeline that listens to Parliament so citizens don’t have to, and summarizes the highlights in plain markdown.

# The Stack

This project leans heavily on modern AI tools and a reliable Rust backbone:

-   `Rust`: For performance, error handling, and concurrency.
-   `yt-dlp`: To fetch livestream recordings from YouTube.
-   `ffmpeg`: For slicing audio into manageable chunks.
-   `OpenAI Whisper`: To transcribe spoken audio to text.
-   `GPT-4`: To summarize transcripts into clear, structured markdown.
-   `Postgres`: For storing metadata and summaries.
-   `Cron jobs`: For scheduling runs and cleaning up old artifacts.

Here’s how data flows through the system:

<p align="center">
  <img src="/assets/images/bunge-bits-pipeline.png" alt="bunge-bits Pipeline Diagram" width="200">
</p>

# Engineering Challenges

Some parts of this were painful:

-   Whisper often mangled Kenyan names: "John Mbadi" became "John Birdy", and it only got worse with native pronunciations.
-   Bad transcriptions created hallucinated summaries: GPT-4 would fill in gaps with confident nonsense.
-   OpenAI context limits: Long transcripts could easily exceed GPT’s token window, making summarization error-prone or impossible.
-   Multiple streams per day: My VPS filled up fast with raw audio and transcripts.
-   Chunk order and retries: Ensuring chunks were in sequence and retried correctly took careful design.
-   Summary formatting: I needed markdown output with clean headings, bullets, and time-stamped key moments.

# Solutions and Architecture Decisions

To tackle these, I made a few key calls:

-   Name mangling and hallucinations: To improve recognition of Kenyan public figures, I enrich GPT-4 prompts with curated lists of MPs, Senators, and Cabinet Secretaries. This leverages both GPT’s internal knowledge and its ability to ground responses via web search, reducing hallucinations and transcription errors.
-   System/user prompt separation: This helps enforce markdown formatting, avoid emojis, and ensure consistent structure.
-   Linear, contextual summarization pipeline: This was the biggest challenge. To avoid exceeding OpenAI’s context window, I chunk transcripts and summarize each piece separately, feeding each new summary back in as context for the next. The final output is then stitched together in a "combine" step. This closure-based approach let me scale summaries without blowing the token limit.
-   Batching via cron: The job runs every 12 hours, fetching and processing only new streams. To conserve storage on the VPS, each run processes a fixed number of unprocessed streams (currently capped at 3), prioritized by upload date. This prevents overload during periods of high parliamentary activity.
-   Separate cron for cleanup: A lightweight system cron wipes old audio/transcripts after processing to avoid disk bloat.

# What’s Next

There’s a lot more I want to do (in order of priority):

-   A public UI where users can browse summaries and filter by topic or MP.
-   Automatic timestamp injection for key moments (e.g. heated debates, bill passages, speaker interventions) to make summaries easier to navigate — this is a planned feature.
-   The repo is already open source, but I plan to apply for OpenAI credits, civic tech grants, or journalism fellowships to fund ongoing compute and infrastructure costs.
-   Eventually, I’d love to collaborate with civil society groups, media houses, or transparency orgs to expand its reach.

[View the code on GitHub](https://github.com/c12i/bunge-bits)
