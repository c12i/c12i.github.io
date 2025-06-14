---
title: "Building bunge-bits, an AI-Powered Summary Pipeline for Kenyan Parliament"
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

I built bunge-bits out of frustration. Kenya’s National Assembly and Senate regularly livestream their proceedings, but these sessions are long, often poorly indexed, and inaccessible to most people. The average citizen doesn’t have the time—or bandwidth—to sit through five-hour videos of parliamentary debates.

At the same time, Kenya is experiencing a generational shift. Young people are increasingly vocal about governance, public spending, and corruption. I wanted to contribute something tangible to that movement—something that makes civic discussions more transparent, digestible, and shareable.

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

```mermaid
    A[Parliament of Kenya<br/>YouTube Livestreams] --> B{Cron Job<br/>Every 12 hours}
    B -->|Check for new streams| C{New streams<br/>detected?}
    C -->|No| B
    C -->|Yes| D[yt-dlp<br/>Download stream audio]
    D --> E[Audio File<br/>.mp3]
    E --> F[ffmpeg<br/>Split into 10-minute chunks]
    F --> G[Audio Chunks<br/>chunk_001.mp3<br/>chunk_002.mp3<br/>...]
    G --> H[OpenAI Whisper<br/>Transcribe chunks]
    H --> I[Raw Transcript Chunks<br/>Text segments]
    I --> J[GPT-4 Algorithm<br/>Generate structured summaries]
    J --> K[Structured Summaries<br/>Coherent final summary]
    K --> L[(PostgreSQL Database<br/>Store summaries)]

    style A fill:#ff6b6b
    style B fill:#4ecdc4
    style D fill:#45b7d1
    style F fill:#96ceb4
    style H fill:#ffeaa7
    style J fill:#fd79a8
    style L fill:#6c5ce7
```

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

-   Context-aware summarization: I inject lists of Kenyan MPs, CSs, Senators, and known figures into the prompt so GPT-4 can correct misheard names with confidence.
-   System/user prompt separation: This helps enforce markdown formatting, avoid emojis, and ensure consistent structure.
-   Linear, contextual summarization pipeline: This was the biggest challenge. To avoid exceeding OpenAI’s context window, I chunk transcripts and summarize each piece separately, feeding each new summary back in as context for the next. The final output is then stitched together in a "combine" step. This closure-based approach let me scale summaries without blowing the token limit.
-   Batching via cron: The job runs every 12 hours, fetching and processing only new streams.
-   Separate cron for cleanup: A lightweight system cron wipes old audio/transcripts after processing to avoid disk bloat.

# What’s Next

There’s a lot more I want to do (in order of priority):

-   A public UI where users can browse summaries and filter by topic or MP.
-   Automatic timestamp injection for key moments (e.g. heated debates, bill passages, speaker interventions) to make summaries easier to navigate — this is a planned feature.
-   Real-time summarization as streams happen.
-   The repo is already open source, but I plan to apply for OpenAI credits, civic tech grants, or journalism fellowships to fund ongoing compute and infrastructure costs.
-   Eventually, I’d love to collaborate with civil society groups, media houses, or transparency orgs to expand its reach.

[View the code on GitHub](https://github.com/c12i/bunge-bits)
