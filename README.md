# Telegram Content Publishing Bot

![Node.js](https://img.shields.io/badge/Node.js-22.x-339933?logo=node.js)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript)
![Telegraf](https://img.shields.io/badge/Telegraf-Telegram-26A5E4?logo=telegram)
![Supabase](https://img.shields.io/badge/Supabase-PostgreSQL-3ECF8E?logo=supabase)
![PM2](https://img.shields.io/badge/PM2-Production-2B037A)
![License](https://img.shields.io/badge/License-MIT-blue)

> Production-ready Telegram automation service built with **Node.js**, **TypeScript**, **Telegraf**, and **Supabase** for scheduled multi-channel content publishing.

The system uses a private Telegram channel as content storage, automatically categorizes uploaded media, maintains independent publishing queues, builds photo/video albums, and distributes content across three Telegram channels according to scheduled publishing rules.

The production source code and content infrastructure are private. This repository serves as a technical showcase of the architecture and implementation.

---

## ✨ Features

- Automated content ingestion from a private Telegram storage channel
- Tag-based content classification
- Four independent content categories
- Photo and video processing
- Automatic Telegram media album creation
- Three destination publishing channels
- Independent content queues with persistent cursors
- Scheduled publishing with `node-cron`
- Duplicate-safe content ingestion
- Queue locking to prevent overlapping publishing jobs
- Telegram support message routing
- Supabase / PostgreSQL persistence
- Production deployment on Contabo VPS
- PM2 process management

---

## ⚙️ How It Works

### 1. Content Storage

Content is uploaded manually to a private Telegram storage channel.

The administrator can switch the active category by sending a predefined tag.

All following photos or videos are automatically associated with that category until another tag is selected.

For every media item, the bot stores:

- Telegram storage channel ID;
- message ID;
- Telegram `file_id`;
- media type;
- content tag;
- media group ID when applicable;
- creation metadata.

The actual media files remain inside Telegram.

Supabase stores only the metadata required to organize and publish the content.

---

### 2. Content Queues

The publishing system maintains several independent queues based on:

- content category;
- media type;
- destination channel.

Each queue has its own persistent cursor stored in Supabase.

The cursor tracks the last successfully published item using its timestamp and Telegram message ID.

This allows the bot to continue publishing from the correct position after a restart without re-sending previously processed content.

---

### 3. Album Generation

Different queues use different publishing strategies.

For example:

- individual videos can be copied directly;
- several photos can be combined into a new Telegram media group;
- existing Telegram media groups can be reconstructed and published as complete albums.

The publisher preserves the original content order using message IDs and stored media group identifiers.

---

### 4. Scheduled Publishing

Content distribution is automated using `node-cron`.

Each destination channel has its own publishing schedule.

Depending on the queue, scheduled jobs can publish:

- single videos;
- generated photo albums;
- complete stored media groups.

All schedules run using an explicitly configured timezone.

Publishing operations are wrapped in error handling so that a failed scheduled job does not stop the scheduler.

---

### 5. Queue Protection

Each queue uses an in-memory lock while publishing.

If another scheduled task attempts to process the same queue before the previous operation has finished, the duplicate execution is skipped.

This prevents concurrent jobs from publishing the same content or moving the same cursor simultaneously.

---

### 6. Support Flow

The bot also provides a lightweight support system.

When a user sends a private message:

1. the message is forwarded to a dedicated support chat;
2. the relationship between the support message and Telegram user is stored in Supabase;
3. an administrator can reply directly to that message;
4. the bot routes the response back to the original user.

This allows support conversations to be handled from a centralized Telegram chat without exposing internal infrastructure.

---

## 🏗 Architecture

```text
Private Telegram Storage Channel
             │
             │ photos / videos
             │ + category tags
             ▼
      Channel Ingestion
             │
             ▼
     Supabase / PostgreSQL
             │
             │ metadata
             │ queue cursors
             │
             ▼
       Content Publisher
        ┌────┼────┐
        │    │    │
        ▼    ▼    ▼
     Queue  Queue  Queue
        │    │    │
        └────┼────┘
             │
             ▼
        node-cron
             │
     ┌───────┼───────┐
     ▼       ▼       ▼
Channel A Channel B Channel C
```

---

## 🛠 Tech Stack

### Backend

- Node.js 22
- TypeScript
- Telegraf
- Telegram Bot API

### Database

- Supabase
- PostgreSQL

### Automation

- node-cron
- Independent scheduled publishing jobs
- Persistent queue cursors
- Queue locking

### Infrastructure

- Contabo VPS
- Ubuntu
- PM2

---

## 📦 Project Structure

```text
src/
└── bot/
    ├── services/
    │   ├── channelContent.ts
    │   ├── publisher.ts
    │   ├── scheduler.ts
    │   └── support.ts
    │
    ├── shared/
    │   ├── constants.ts
    │   ├── types.ts
    │   └── utils.ts
    │
    ├── bot.ts
    ├── index.ts
    └── supabase.ts
```

### Main Services

**`channelContent.ts`**  
Processes new posts from the private storage channel, detects category tags and media types, and stores content metadata in Supabase.

**`publisher.ts`**  
Manages content queues, persistent cursors, media albums, queue locks, and publishing to destination channels.

**`scheduler.ts`**  
Defines recurring publishing jobs using `node-cron` and routes each queue to the appropriate Telegram channel.

**`support.ts`**  
Handles user support messages and routes administrator replies back to the correct Telegram user.

---

## 🗄 Persistence

Supabase is used to persist application state.

The main stored data includes:

- content metadata;
- Telegram file and message identifiers;
- media group identifiers;
- queue cursors;
- support message mappings.

A key part of the architecture is the persistent publishing cursor.

Instead of loading content from the beginning each time, every queue remembers its last published position and requests only newer items.

---

## 🚀 Deployment

The production bot runs continuously on a **Contabo VPS**.

The deployment includes:

- Ubuntu server environment;
- PM2 process management;
- automatic restart after crashes;
- environment-based configuration;
- persistent Supabase database;
- scheduled cron jobs;
- private Telegram storage infrastructure.

Because publishing state is stored in Supabase, restarting the Node.js process does not reset the content queues.

---

## 📈 Technical Highlights

- Multi-channel Telegram publishing architecture
- Telegram used as primary media storage
- Tag-driven content ingestion
- Supabase-backed publishing queues
- Persistent per-queue cursors
- Automatic media album generation
- Ordered media publishing
- Independent photo and video workflows
- Cron-based publishing schedules
- Queue concurrency protection
- Duplicate-safe database ingestion
- Telegram support routing
- Production VPS deployment

---

## 🔒 Privacy & Source Code

The production source code is private because the application is actively deployed and connected to private Telegram channels and production infrastructure.

This repository is intended to demonstrate the project's architecture, system design, deployment approach, and main engineering decisions.

No private channel identifiers, bot tokens, user data, production media, payment data, or environment configuration are included.
