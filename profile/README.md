# TRVL

Welcome to the organization! This organization hosts **TRVL**, an AI-powered travel-planning platform where users plan trips through a conversational assistant that generates personalized, map-aware itineraries and share them in a social community of travelers.

The platform pairs a **large language model (OpenAI GPT-4.1-mini)** for natural-language trip planning with the **Google Places & Maps** ecosystem for real-world geocoding, place discovery, and booking, all delivered through a decoupled backend / frontend / infrastructure architecture.

---

## Ecosystem Architecture

The ecosystem is intentionally decoupled, separating the API and business logic from the web client and the deployment infrastructure. It is distributed across the following core repositories:

*   **`trvl-back`**: The core REST API (NestJS + TypeScript). Owns authentication, the AI chat engine, itinerary generation, the social graph, and all Google Places / OpenAI integrations. Backed by PostgreSQL via Prisma.
*   **`trvl-ui`**: The frontend web application (Next.js 15 + React 19 + TypeScript), featuring the conversational chat interface, interactive maps, itinerary views, and social profiles. Deployed on Vercel.
*   **`trvl-infra`**: Centralized infrastructure definitions — Nginx reverse proxy, Let's Encrypt TLS certificates, and Docker Compose environments that host isolated **dev** and **prod** stacks on a single server.

---

## The Product

TRVL turns an open-ended conversation ("I want a 5-day romantic trip to Italy on a mid-range budget") into a concrete, actionable travel plan. The user experience is built around three connected pillars:

### 1. Conversational AI Travel Assistant
A chat-first planning experience powered by **OpenAI GPT-4.1-mini** using OpenAI's stateful *Conversations* API, so context persists across the whole planning session. As the assistant replies, it extracts **structured places** from its own natural-language response, which are then enriched and dropped straight onto the map — the conversation and the map stay in sync.

The assistant is steered by a per-chat **context** (destination, budget level, travel preferences) so recommendations stay tailored to each trip.

### 2. AI-Generated Itineraries
From the chat history, TRVL generates a fully structured itinerary as strict JSON: a titled multi-day plan with dated items, recommendations, and geolocated stops (name + latitude/longitude). Each itinerary is then **auto-tagged** by a second LLM pass into a controlled taxonomy — theme tags (Culture, Beach, Nature, Foodie, Nightlife, Adventure…), a single price level (Low / Mid / High), and a party type (Solo / Couple / Family) — powering search, filtering, and discovery.

### 3. Maps, Places & Real-World Actions
Places surfaced by the assistant are resolved against the **Google Places API** — geocoding, text search, place details, and photos — and rendered on an interactive **Google Map** with marker clustering. TRVL also generates **real-world action links** for each place through a security-allowlisted provider system (official sites, TheFork, OpenTable, Booking, Airbnb), so users can jump from a recommendation to an actual booking.

### 4. Social Layer
Beyond planning, TRVL is a community: user profiles with visited countries, trip counts, and preferences; a follow / followers graph; public vs. private profiles; and an explore feed to discover other travelers' itineraries.

---

## Infrastructure & Tech Stack

A modern, containerized stack focused on scalability, environment isolation, and secure real-world integrations.

| Concern | Technology |
|:---|:---|
| **Backend** | NestJS, TypeScript, Prisma ORM |
| **Database** | PostgreSQL 17 |
| **AI / LLM** | OpenAI GPT-4.1-mini (Conversations & Chat Completions API) |
| **Maps & Places** | Google Places API, Google Maps (`@vis.gl/react-google-maps` + marker clustering) |
| **Frontend** | Next.js 15, React 19, TypeScript, Vite tooling |
| **UI & Styling** | Tailwind CSS 4, Chakra UI, Radix UI, Framer Motion |
| **Authentication** | Firebase Authentication (Firebase Admin on the API) |
| **API Docs** | Swagger / OpenAPI |
| **Containerization** | Docker, Docker Compose |
| **CI/CD** | GitHub Actions |
| **Container Registry** | GitHub Container Registry (GHCR) |
| **Hosting** | Microsoft Azure VMs (backend), Vercel (frontend) |
| **Reverse Proxy / TLS** | Nginx + Let's Encrypt (HTTPS) |
| **Testing** | Jest |

---

## DevOps & CI/CD

Every repository uses automated **GitHub Actions** workflows to guarantee quality and ship continuously, with fully isolated **dev** and **prod** environments running side by side.

### Backend (`trvl-back`)
- **Continuous Integration** — On pushes to `dev` and pull requests, the pipeline installs dependencies, generates and **validates the Prisma schema**, lints, runs the Jest test suite, and builds the app.
- **Continuous Deployment** — On pushes to `main` / `dev`, a multi-stage Docker image is built and pushed to **GHCR**, then the workflow **SSHes into the EC2 host** to pull the new image and restart the corresponding environment (`main` → production, `dev` → development) via Docker Compose. Images run as a non-root user for hardening.

### Frontend (`trvl-ui`)
- **Preview & Production deploys** — GitHub Actions drives **Vercel**: pushes to `dev` publish a preview build, pushes to `main` promote to production.

### Infrastructure (`trvl-infra`)
- An **Nginx reverse proxy** terminates TLS (Let's Encrypt) and routes by path prefix — `/dev` and `/prod` map to separate backend containers, **each backed by its own PostgreSQL instance** — so the two environments share a host and domain without ever sharing data.
- **Docker Compose** orchestrates the proxy, both backend environments, and both databases, pulling versioned images straight from GHCR with health-checked, self-restarting services.
