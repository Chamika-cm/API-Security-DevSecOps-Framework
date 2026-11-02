# Advanced API Security & DevSecOps Framework - System Architecture

## 1. Executive Overview
This document outlines the system architecture, component boundaries, and data flow of the OWASP crAPI (Completely Ridiculous API) deployment used for this advanced DevSecOps framework. The application utilizes a modern microservices architecture with multiple backend services and databases.

## 2. Technology Stack & Component Breakdown
* **Frontend:** Single Page Application (SPA) serving the user interface.
* **API Gateway / Ingress:** Acts as the single entry point, routing client requests to appropriate microservices.
* **Microservices:**
  * Identity Service (Authentication and User Management)
  * Community Service (Posts, Comments, and Forums)
  * Workshop Service (Vehicle and Mechanic interactions)
  * Mail/Report Service (Background processing and alerts)
* **Databases:**
  * MongoDB (NoSQL database for flexible object storage)
  * PostgreSQL (Relational database for structured transactional data)

## 3. Trust Boundaries & Data Flow
* **Untrusted Zone (Public Internet / User Browser):** 
  * All inbound HTTP/HTTPS requests originate from the user's browser, hitting the API Gateway directly.
* **Application Trust Boundary (Docker Container Network):**
  * Internal microservices communicate securely over isolated Docker bridge networks (`docker-compose`).
  * Database connections are strictly restricted to internal backend services and are not exposed externally to the host network.

## 4. Architecture Diagram Reference
The data flows sequentially from the client browser through the API Gateway, reaching individual microservices, which subsequently interact with MongoDB or PostgreSQL databases.