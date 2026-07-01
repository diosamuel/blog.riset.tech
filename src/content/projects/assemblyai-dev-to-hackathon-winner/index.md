---
title: "Winning the Dev.to x AssemblyAI Winter Speech-to-Text Challenge with ReportSOS"
description: "How ReportSOS, an AI-powered SOS emergency caller, won the LLM-powered application category in the AssemblyAI Challenge"
date: "Jan 09 2025"
draft: false
demoURL: "https://report-sos.vercel.app/"
repoURL: "https://github.com/diosamuel/reportSOS"
---

## Overview

ReportSOS is an AI-powered SOS emergency caller that helps dispatchers handle and interpret emergency reports faster and smarter. The app lets users report incidents in just a few taps — location, type of emergency, even voice summaries — cutting through the noise to deliver the right help to the right place, right on time.

This project won the **LLM-powered application category** in the [Dev.to x AssemblyAI Winter Speech-to-Text Challenge](https://www.assemblyai.com/blog/dev-to-x-assemblyai-winter-speech-to-text-challenge-winners) — selected from 75 submissions across three categories.

## The Challenge

The hackathon had three tracks:

1. **Universal Speech-to-Text** — building a sophisticated STT application
2. **Streaming Speech-to-Text** — real-time transcription apps
3. **LLM-powered application** — building an LLM-powered feature on top of speech data using AssemblyAI's LeMUR

I entered the third category. The prize: $1,000 and a six-month Dev++ membership, plus exclusive AssemblyAI gifts.

## What I Built | 🚨ReportSOS 🆘

> Help! Help! Emergency here! 🗣

Dispatchers handle hundreds of emergency calls every single day. With ReportSOS, emergencies get sorted faster and smarter. Users can report incidents in just a few taps, providing crucial details like location, type of emergency, and even summaries — giving dispatchers a superpower to cut through the noise.

### User Features

- **Start Page** — press "Help Me" or shake your phone vigorously (4 vibrations) to trigger reporting
- **Voice Recorder** — use the microphone to report emergencies, or manually upload audio files
- **Location** — automatic capture of precise position for rapid responder dispatch
- **Camera** — capture images/video of the emergency situation via React-Webcam

### Dispatcher Features

- **Authentication Page** — secure login with username, password, and AssemblyAI API key
- **Dashboard Page** — highlights the latest emergencies with detailed information: location, images, audio, AI-generated summary, emergency status, casualty details, key points

## Demo Video

<iframe width="100%" height="400" src="https://www.youtube.com/embed/UBQYGRWfKcc" title="ReportSOS Demo" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

Watch directly on YouTube: [ReportSOS Demo](https://www.youtube.com/watch?v=UBQYGRWfKcc).

## How It Works

### AssemblyAI LeMUR Integration

- Audio is converted to text and processed through AssemblyAI entity detection + IAB category detection to identify topics
- The transcript is then fed to LeMUR with a custom prompt designed to keep the output in strict JSON format:

```json
{
  "context": "This is an emergency call for assistance.",
  "answer_format": "{ \"activities\":\"very short sentence\", \"summary\": \"\", \"key_points\": [], \"tone\": \"\", \"casualty\":\"\", \"status\":\"Critical\"|\"Urgent\"|\"Non-Urgent\" }"
}
```

- The structured output enables the dashboard to display a triage-ready summary, severity classification, and key points at a glance

### Emergency Mapping

- **React-Leaflet** for spatial visualization
- **Overpass API (OpenStreetMap)** to locate nearby hospitals, fire stations, and police stations:

```overpass
[out:json][timeout:25];
  nwr["amenity"~"^(hospital|fire_station|police)$"](south, west, north, east);
  out geom;
```

- After reporting, users see first-aid guidance and the nearest emergency offices

## Tech Stack

- **React + Vite** — frontend with react-voice-visualizer and react-webcam
- **Tailwind CSS** — styling
- **Express.js** — backend API ([morgan](https://www.npmjs.com/package/morgan), [helmet](https://www.npmjs.com/package/helmet), [cors](https://www.npmjs.com/package/cors))
- **Neon (PostgreSQL)** — database for deduplication and persistence to save API credits
- **AssemblyAI Universal + LeMUR** — transcription + LLM-powered extraction
- **OpenStreetMap** — Overpass API for nearby emergency locations, Nominatim for reverse geocoding

## Winning Announcement

AssemblyAI officially announced the winners in their [Winter Speech-to-Text Challenge blog post](https://www.assemblyai.com/blog/dev-to-x-assemblyai-winter-speech-to-text-challenge-winners):

> Finally, Diosamual won the LLM-powered application category with ReportSOS: AI-powered SOS Emergency Caller. With ReportSOS, emergencies get sorted faster and smarter. The app lets users report incidents in just a few taps, providing crucial details like location, type of emergency, and even summaries.

## Lessons Learned

- **Strict prompt constraints pay off** — forcing LeMUR output into JSON format made integration trivial
- **Deduplicate early** — storing transcripts in Neon prevented redundant API calls and saved credits
- **Seconds matter** — real dispatchers triage as soon as they have location + rough details, so the workflow should not block on optional inputs like a photo
- **Solo is hard but doable** — this was a one-person build; the time-boxed format forced ruthless prioritization

## Links

- [AssemblyAI Winner Announcement](https://www.assemblyai.com/blog/dev-to-x-assemblyai-winter-speech-to-text-challenge-winners)
- [Dev.to Build Submission](https://dev.to/diosamuel/reportsos-enhancing-dispatcher-efficiency-and-response-52c6)
- [Demo Video on YouTube](https://www.youtube.com/watch?v=UBQYGRWfKcc)
- [Frontend Repository](https://github.com/diosamuel/reportSOS)
- [Backend Repository](https://github.com/diosamuel/reportsos-be)
- [Live Demo](https://report-sos.vercel.app/)