# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A birthday greeting website for "农友维" (Nong Youwei). Two self-contained HTML pages — no build tools, no frameworks, no external deps (except Google Fonts for one heading).

## Pages

- **index.html** — Main page: particle text morphing intro → interactive cake scene (blow out candle via microphone) → blessing reveal → link to surprise page
- **jingxi.html** — Surprise page: heart-shaped popup blessing cards → canvas fireworks finale (normal bursts + heart-shaped firework)

## Architecture

Both files are fully self-contained (inline CSS + inline JS). Key render loops and state machines:

- **Particle text system** (`index.html`): Dots morph between text shapes via SVG text rasterization → canvas rendering. Font fallback chain + retry logic for CJK rendering.
- **Blow detection** (`index.html`): `getUserMedia` + `AnalyserNode` RMS threshold — calls `input stream.stop()` and triggers scene transition.
- **Popup queue** (`jingxi.html`): Blessing cards appear at random positions, then heart-popup cards render along a parametric heart curve (`heartCurve()`). After all popups fade, fireworks trigger.
- **Fireworks system** (`jingxi.html`): Rocket physics → circular or heart-shaped particle explosion. Canvas with trail/decay/gravity. Phased launch schedule with screen flash on heavy bursts.

## No Tests / No Build

No package.json, no test runner, no bundler. Open the HTML files directly in a browser to run.

## Dev Workflow

- Open `index.html` in a browser to preview the full flow
- Edit any inline CSS/JS directly in the file
- Commit with `git commit` (standard git workflow, no hooks)
