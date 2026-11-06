---
title: "What is a Harness? (structured notes)"
author: "Earendil Product"
source_url: "https://earendil.com/posts/what-is-a-harness/"
published_at: "2026-08-20"
archived_at: "2026-11-06"
category: "background"
archive_note: "Structured research notes for an in-house coding-agent harness engineer. Not a full reprint or sentence-by-sentence translation of the source."
---

## Source

Earendil Product essay (2026-08-20), written for readers who hear “agent harness” in AI feeds but have been too embarrassed to ask what it means. Uses a climbing-harness metaphor throughout. Pre-dates Pi Durable but supplies the vocabulary Durable posts assume.

## Metaphor

A climbing harness supports you, connects you to ropes/carabiners that secure falls and govern route, and lets you attach tools (chalk, nuts, quickdraws). You take it between mountains and modify gear loops for terrain. Agent harnesses share structure and function: support, control, tool attachment, portability/adaptability.

## Definition

An agent harness is software that provides an environment for an AI model to operate within. Unlike most models, end users can **own** their harness. Simplistic industry formula: Agent = Model + Harness. First popular apps were coding agents; harnesses now sit at the core of many agent types. Interfaces vary: terminal (Pi), iMessage/chat, email (Earendil mentions Lefos), etc.

## Four jobs

1. **System prompt** — injected instructions each turn (less baked-in than training-time “soul” documents; more like a new-hire handbook).
2. **Tools** — code capabilities the model may call; harness describes and implements them but usually does not hard-script when to use them.
3. **Agentic loop** — observe → decide → tool → observe → … until done (email “compare local schools” walkthrough: search, maybe re-search, spreadsheet via code tool, compose email).
4. **Translation layer** — talk to many model providers (and mix models in one loop); preserves user choice and local session ownership versus lab-locked apps.

## Making a harness yours / Pi stance

Unlike models, harnesses can be owned and adapted. Pi is minimal (short prompt, few tools), OSS on your laptop, with thousands of shared extensions. Earendil frames neutral open-source harnesses as tools of **human agency** amid concern about large AI companies—harness the tech rather than be harnessed by it.

## Relation to Pi Durable readers

When Durable says “harness,” it means this stack **plus** durable storage, tasks, commits, and multi-client machinery—not only the interactive coding loop. Use this essay to onboard stakeholders who only know chat UIs.

## Quote

> “An agent harness is a piece of software that provides an environment for an AI model to operate within. Unlike most AI models, you as an end user can own your own agent harness.”
> — https://earendil.com/posts/what-is-a-harness/
