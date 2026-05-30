# PulseSync: Full-Stack AI-Powered Fitness Tracker

PulseSync is a comprehensive full-stack fitness application built on a modern microservices architecture. It allows users to track various physical activities (such as running, cycling, and weight training) and leverages artificial intelligence to provide personalized, real-time workout recommendations and safety guidelines [1, 2].

## 🚀 Architecture & Tech Stack

The application is distributed across multiple independent Spring Boot microservices, communicating both synchronously and asynchronously, and features a responsive Single-Page Application (SPA) frontend [3, 4].

**Frontend:**
*   **React & Vite:** Fast, modern frontend tooling for the web project [5].
*   **Material-UI (MUI):** For responsive, pre-built design components [6].
*   **Redux Toolkit:** For local state management of user authentication and JWTs [7, 8].
*   **Axios:** Configured with custom interceptors to automatically handle authenticated API requests [9, 10].

**Backend Infrastructure (Spring Cloud & Spring Boot):**
*   **API Gateway:** Serves as the single secure entry point for all external client requests, routing them to the appropriate microservices [11, 12].
*   **Eureka Server:** Handles service discovery, allowing microservices to register and locate each other dynamically [13, 14].
*   **Config Server:** Provides centralized configuration management for all microservices in the network [15].

**Microservices:**
*   **User Service (PostgreSQL):** Manages extended user profiles and identity data [16, 17].
*   **Activity Service (MongoDB):** Tracks and stores flexible, schema-less workout metrics [18, 19].
*   **AI Service (MongoDB):** Interacts with external AI models to generate and store JSON-formatted fitness recommendations [20, 21].

## ✨ Core Features

*   **Robust Security & User Synchronization:** The API Gateway intercepts incoming JWTs and validates them against Keycloak [22]. It includes a custom filter that automatically synchronizes newly authenticated Keycloak users into the backend user database using their unique Keycloak ID [22, 23].
*   **Inter-Service Communication:**
    *   **Synchronous:** The Activity Service uses Spring WebClient to synchronously call the User Service to validate a user's existence before saving a workout [24, 25].
    *   **Asynchronous:** Once an activity is saved, the Activity Service publishes an event to a RabbitMQ message queue [26]. The AI Service acts as a consumer, picking up the data asynchronously without blocking the main application thread [27, 28].
*   **Polyglot Persistence:** Each microservice manages its own isolated database technology (PostgreSQL vs. MongoDB) best suited for its specific data structure requirements [29].

## 🧠 AI-Powered Fitness Recommendations

The standout feature of PulseSync is its ability to generate real-time, personalized workout insights using **Generative AI** [2].

*   **Google Gemini API Integration:** The dedicated AI microservice queries the Google Gemini 1.5 Flash API to process user workout metrics [30, 31].
*   **Event-Driven Asynchronous Processing (RabbitMQ):** To ensure the application remains fast and responsive, the AI processing is decoupled [26]. When a user logs a new activity, the Activity Service publishes the record to a **RabbitMQ** message queue [26]. The AI Service acts as a consumer, safely picking up the message and querying the AI model asynchronously [26].
*   **Prompt Engineering & Strict JSON Formatting:** The AI model is fed a highly specific custom prompt containing the user's activity details [32]. It is strictly instructed to return its analysis in a predefined **JSON format** rather than plain text [32, 33].
*   **Data Parsing & Polyglot Storage:** Once the AI responds, the backend parses the JSON payload using Jackson `ObjectMapper` [34]. The structured recommendations are then persisted into **MongoDB** for seamless retrieval [35].

## 🛠️ Local Setup & Installation

*(Ensure you have Docker, Java, Node.js, and Maven installed.)*

**1. Infrastructure Setup (Docker)**
*   **RabbitMQ:** Run `docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:4.0-management` [36].
*   **Keycloak:** Run Keycloak on port `8181` and configure a realm with a public client configured for the SHA-256 PKCE code challenge [37, 38].
*   **Databases:** Ensure local instances of PostgreSQL and MongoDB are running [16, 39].

**2. Backend Microservices**
Start the Spring Boot services in this order:
1.  **Config Server** (Port `8888`) [40]
2.  **Eureka Server** (Port `8761`) [41]
3.  **User Service** (Port `8081`) [42]
4.  **Activity Service** (Port `8082`) [43]
5.  **AI Service** (Port `8083`) [43]
6.  **API Gateway** (Port `8080`) [44]

*Note: Inject your Google Gemini API key into the AI Service configuration (or via the Config Server) to enable recommendations [45, 46].*

**3. Frontend Setup**
Navigate to the frontend directory:
```bash
npm install
npm run dev
