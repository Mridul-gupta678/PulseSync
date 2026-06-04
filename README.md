<div align="center">

# ⚡ PulseSync

### AI-Powered Full-Stack Fitness Tracker

*Real-time workout recommendations. Microservices architecture. Generative AI at its core.*

[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.x-6DB33F?style=flat-square&logo=springboot&logoColor=white)](https://spring.io/projects/spring-boot)
[![React](https://img.shields.io/badge/React-18-61DAFB?style=flat-square&logo=react&logoColor=black)](https://reactjs.org/)
[![RabbitMQ](https://img.shields.io/badge/RabbitMQ-4.0-FF6600?style=flat-square&logo=rabbitmq&logoColor=white)](https://www.rabbitmq.com/)
[![Keycloak](https://img.shields.io/badge/Keycloak-Auth-4D4D4D?style=flat-square&logo=keycloak&logoColor=white)](https://www.keycloak.org/)
[![Google Gemini](https://img.shields.io/badge/Google%20Gemini-1.5%20Flash-4285F4?style=flat-square&logo=google&logoColor=white)](https://ai.google.dev/)
[![MongoDB](https://img.shields.io/badge/MongoDB-Atlas-47A248?style=flat-square&logo=mongodb&logoColor=white)](https://www.mongodb.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-336791?style=flat-square&logo=postgresql&logoColor=white)](https://www.postgresql.org/)

</div>

---

## 📋 Table of Contents:

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Tech Stack](#-tech-stack)
- [Core Features](#-core-features)
- [AI Recommendations Engine](#-ai-recommendations-engine)
- [Local Setup & Installation](#-local-setup--installation)
- [Service Registry](#-service-registry)

---

## 🧭 Overview

**PulseSync** is a production-grade, full-stack fitness application built on a distributed microservices architecture. It enables users to log physical activities — running, cycling, weight training, and more — and delivers **personalized, AI-generated workout insights** in real time.

The system is built for scalability and resilience: each concern is isolated into its own service, communication is both synchronous and event-driven, and AI processing is fully decoupled to keep the user experience fast and responsive.

---

## 🏗 Architecture

```
                        ┌────────────────────────────────┐
                        │         React Frontend         │
                        │    (Vite + MUI + Redux)        │
                        └────────────────┬───────────────┘
                                         │ HTTPS
                        ┌────────────────▼───────────────┐
                        │           API Gateway          │  ◄── JWT Validation (Keycloak)
                        │        (Port 8080)             │  ◄── User Sync Filter
                        └──────┬──────────────┬──────────┘
                               │              │
              ┌────────────────▼──┐    ┌──────▼────────────────┐
              │   User Service    │    │   Activity Service     │
              │   (Port 8081)     │    │   (Port 8082)          │
              │   PostgreSQL      │    │   MongoDB              │
              └───────────────────┘    └──────┬────────────────┘
                                              │ RabbitMQ
                                       ┌──────▼────────────────┐
                                       │      AI Service        │
                                       │   (Port 8083)          │
                                       │   MongoDB              │
                                       │   Google Gemini API    │
                                       └───────────────────────┘

              ┌──────────────────┐    ┌──────────────────────┐
              │  Eureka Server   │    │    Config Server      │
              │  (Port 8761)     │    │    (Port 8888)        │
              └──────────────────┘    └──────────────────────┘
```

**Communication Patterns:**
- **Synchronous** — Activity Service → User Service via Spring WebClient (user validation)
- **Asynchronous** — Activity Service → RabbitMQ → AI Service (non-blocking AI processing)

---

## 🛠 Tech Stack

### Frontend

| Technology | Purpose |
|---|---|
| **React + Vite** | Fast, modern SPA framework and build tooling |
| **Material-UI (MUI)** | Responsive, accessible UI component library |
| **Redux Toolkit** | Global state management for auth and JWTs |
| **Axios** | HTTP client with custom interceptors for authenticated requests |

### Backend Infrastructure (My fav part)

| Service | Technology | Purpose |
|---|---|---|
| **API Gateway** | Spring Cloud Gateway | Single entry point; JWT validation; request routing |
| **Eureka Server** | Spring Cloud Netflix | Service discovery and dynamic registration |
| **Config Server** | Spring Cloud Config | Centralized configuration management |

### Microservices

| Service | Database | Responsibility |
|---|---|---|
| **User Service** | PostgreSQL | User profiles and identity data |
| **Activity Service** | MongoDB | Flexible workout metric storage |
| **AI Service** | MongoDB | AI recommendation generation and persistence |

---

## ✨ Core Features

### 🔐 Security & User Synchronization
The API Gateway intercepts all incoming requests and validates JWTs against **Keycloak**. A custom gateway filter automatically synchronizes newly authenticated Keycloak users into the backend user database using their unique Keycloak ID — no manual registration required.

### 🔄 Inter-Service Communication

**Synchronous (WebClient)**
Before persisting a new workout, the Activity Service calls the User Service to confirm the user exists, ensuring data integrity across service boundaries.

**Asynchronous (RabbitMQ)**
Once an activity is saved, the Activity Service publishes an event to a RabbitMQ message queue. The AI Service consumes the message independently, keeping the primary request path fast and non-blocking.

### 🗄 Polyglot Persistence
Each service owns its database, choosing the technology best suited for its data structure:
- **PostgreSQL** for structured relational user data
- **MongoDB** for schema-flexible workout metrics and AI responses

---

## 🧠 AI Recommendations Engine

PulseSync's standout capability is its real-time, personalized fitness analysis powered by **Generative AI**.

```
User logs activity
       │
       ▼
Activity Service saves to MongoDB
       │
       ▼
Event published to RabbitMQ ──────────────────────────────────┐
                                                               │
                                             AI Service consumes event
                                                               │
                                                               ▼
                                         Google Gemini 1.5 Flash API called
                                         (custom prompt + user workout data)
                                                               │
                                                               ▼
                                         JSON response parsed by Jackson ObjectMapper
                                                               │
                                                               ▼
                                         Structured recommendation persisted to MongoDB
```

**Key implementation details:**

- **Model** — Google Gemini 1.5 Flash API
- **Prompt Engineering** — A highly specific custom prompt is constructed from the user's activity metrics, with explicit instructions to return structured **JSON output** rather than plain text
- **Parsing** — Jackson `ObjectMapper` deserializes the AI's JSON response into typed Java objects
- **Storage** — Parsed recommendations are persisted to MongoDB for fast retrieval and future reference
- **Decoupling** — All AI processing is asynchronous via RabbitMQ, ensuring zero latency impact on the user-facing API

---

## 🚀 Local Setup & Installation

### Prerequisites

Ensure the following are installed before proceeding:

| Tool | Version |
|---|---|
| Docker | Latest |
| Java (JDK) | 17+ |
| Maven | 3.8+ |
| Node.js | 18+ |

---

### Step 1 — Infrastructure (Docker)

**RabbitMQ**
```bash
docker run -it --rm --name rabbitmq \
  -p 5672:5672 -p 15672:15672 \
  rabbitmq:4.0-management
```

**Keycloak**
```bash
# Run Keycloak on port 8181
# Configure a realm with a public client using SHA-256 PKCE code challenge
```
> See the [Keycloak documentation](https://www.keycloak.org/docs/latest/) for realm and client configuration.

**Databases**

Ensure local instances of **PostgreSQL** and **MongoDB** are running and accessible.

---

### Step 2 — Backend Microservices

Start services **in this exact order** to satisfy dependency resolution:

```bash
# 1. Config Server — must start first (provides config to all services)
cd config-server && mvn spring-boot:run        # Port 8888

# 2. Eureka Server — service registry
cd eureka-server && mvn spring-boot:run        # Port 8761

# 3. User Service
cd user-service && mvn spring-boot:run         # Port 8081

# 4. Activity Service
cd activity-service && mvn spring-boot:run     # Port 8082

# 5. AI Service
cd ai-service && mvn spring-boot:run           # Port 8083

# 6. API Gateway — start last
cd api-gateway && mvn spring-boot:run          # Port 8080
```

> **Important:** Inject your **Google Gemini API key** into the AI Service configuration (directly or via the Config Server) before starting to enable the recommendations feature.

---

### Step 3 — Frontend

```bash
cd frontend
npm install
npm run dev
```

The application will be available at `http://localhost:5173` (or the port configured by Vite).

---

## 📡 Service Registry

| Service | Port | Description |
|---|---|---|
| API Gateway | `8080` | External entry point for all client requests |
| User Service | `8081` | User profile management |
| Activity Service | `8082` | Workout tracking and event publishing |
| AI Service | `8083` | Gemini API integration and recommendation storage |
| Eureka Server | `8761` | Service discovery dashboard |
| Config Server | `8888` | Centralized configuration |
| Keycloak | `8181` | Identity and access management |
| RabbitMQ Management | `15672` | Message broker admin UI |

---

<div align="center">

Built with ❤️ using Spring Boot, React, and the power of Generative AI.
Mridul Gupta

</div>
