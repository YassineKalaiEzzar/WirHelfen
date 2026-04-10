# 🤝 WirHelfen Platform

<div align="center">

![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.x-6DB33F?style=for-the-badge&logo=springboot&logoColor=white)
![Java](https://img.shields.io/badge/Java-17+-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)
![Thymeleaf](https://img.shields.io/badge/Thymeleaf-005F0F?style=for-the-badge&logo=thymeleaf&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![WhatsApp](https://img.shields.io/badge/WhatsApp%20Integration-25D366?style=for-the-badge&logo=whatsapp&logoColor=white)

**A full-stack service marketplace connecting helpers with people who need them — with real-time WhatsApp messaging on the roadmap.**

[Features](#-features) · [Architecture](#-architecture) · [Database Schema](#-database-schema) · [WhatsApp Integration](#-whatsapp-integration-roadmap) · [Setup](#-setup) · [API](#-api-endpoints)

</div>

---

## 📌 Overview

HelperConnect is a RESTful Spring Boot platform where **service providers** post opportunities and **helpers** discover, apply, and manage their applications — all through a clean, secure web interface. The next milestone: **direct WhatsApp conversations** between helpers and providers, authenticated end-to-end.

---

## ✨ Features

| Feature | Status |
|---|---|
| Helpers can browse & apply to service posts | ✅ Live |
| Application withdrawal (POST-secured) | ✅ Live |
| Application status tracking (PENDING / ACCEPTED / CANCELLED) | ✅ Live |
| Error-resilient DB schema (VARCHAR vs ENUM) | ✅ Live |
| RESTful controllers with full exception handling | ✅ Live |
| WhatsApp direct messaging between users | 🔨 Roadmap |
| End-to-end encrypted conversations | 🔨 Roadmap |
| Real-time notification via WhatsApp webhook | 🔨 Roadmap |

---

## 🏗 Architecture

```mermaid
flowchart TB
    subgraph Client["🌐 Client Layer"]
        B[Browser / Thymeleaf Templates]
        WA[📱 WhatsApp App]
    end

    subgraph Gateway["🔐 Security & Routing"]
        SEC[Spring Security Filter Chain]
        TWILIO[Twilio WhatsApp API Gateway]
    end

    subgraph App["⚙️ Application Layer — Spring Boot"]
        HC[HelferController]
        SC[ServicePostController]
        WC[WhatsApp Webhook Controller]
        AS[ApplicationService]
        WS[WhatsApp Messaging Service]
    end

    subgraph Data["🗄 Data Layer"]
        REPO[(Spring Data JPA Repositories)]
        DB[(MySQL Database)]
    end

    B --> SEC
    WA --> TWILIO
    SEC --> HC
    SEC --> SC
    TWILIO --> WC
    HC --> AS
    SC --> AS
    WC --> WS
    AS --> REPO
    WS --> REPO
    REPO --> DB
```

---

## 🗂 Database Schema

```mermaid
erDiagram
    USER {
        Long id PK
        String name
        String email
        String passwordHash
        VARCHAR20 role
    }

    SERVICE_POST {
        Long id PK
        String title
        String description
        VARCHAR20 status
        Long createdById FK
    }

    APPLICATION {
        Long id PK
        Long helperId FK
        Long servicePostId FK
        VARCHAR20 status
        LocalDateTime appliedAt
        LocalDateTime updatedAt
    }

    WHATSAPP_CONVERSATION {
        Long id PK
        Long applicationId FK
        String waThreadId
        Boolean isActive
        LocalDateTime startedAt
    }

    MESSAGE {
        Long id PK
        Long conversationId FK
        Long senderId FK
        String content
        LocalDateTime sentAt
        Boolean delivered
    }

    USER ||--o{ SERVICE_POST : "creates"
    USER ||--o{ APPLICATION : "submits"
    SERVICE_POST ||--o{ APPLICATION : "receives"
    APPLICATION ||--o| WHATSAPP_CONVERSATION : "triggers"
    WHATSAPP_CONVERSATION ||--o{ MESSAGE : "contains"
    USER ||--o{ MESSAGE : "sends"
```

---

## 📊 Application Status Flow

```mermaid
stateDiagram-v2
    [*] --> PENDING : Helper applies
    PENDING --> ACCEPTED : Provider accepts
    PENDING --> CANCELLED : Helper withdraws
    ACCEPTED --> CANCELLED : Helper withdraws post-acceptance
    ACCEPTED --> COMPLETED : Service fulfilled
    CANCELLED --> [*]
    COMPLETED --> [*]

    ACCEPTED --> WA_INITIATED : WhatsApp convo started 🔨
    WA_INITIATED --> COMPLETED : Conversation closed 🔨
```

---

## 📱 WhatsApp Integration Roadmap

The goal is to let helpers and service providers chat **directly via WhatsApp** once an application is accepted — without ever leaving their phone.

```mermaid
sequenceDiagram
    participant H as 👷 Helper
    participant BE as ⚙️ Spring Boot Backend
    participant TW as 🔗 Twilio API
    participant WA as 📱 WhatsApp

    H->>BE: Application accepted (POST /apply)
    BE->>TW: Create WhatsApp conversation thread
    TW-->>WA: Send intro message to Helper & Provider
    WA-->>H: "Your application was accepted! Chat here 👇"

    loop Secure Messaging
        H->>WA: Sends message
        WA->>TW: Webhook event
        TW->>BE: POST /webhook/whatsapp
        BE->>BE: Validate signature (HMAC-SHA256)
        BE->>BE: Persist message to DB
        BE->>TW: Forward to recipient
        TW-->>WA: Deliver to Provider
    end
```

### Security Model

```mermaid
flowchart LR
    MSG[Incoming WhatsApp Webhook] --> SIG{Validate\nTwilio Signature}
    SIG -- ❌ Invalid --> REJ[403 Rejected]
    SIG -- ✅ Valid --> AUTH{User\nAuthenticated?}
    AUTH -- ❌ No --> UNAUTH[401 Unauthorized]
    AUTH -- ✅ Yes --> PERM{Participant\nin Conversation?}
    PERM -- ❌ No --> FORBID[403 Forbidden]
    PERM -- ✅ Yes --> SAVE[💾 Persist & Forward Message]
```

---

## 🛠 Tech Stack

| Layer | Technology |
|---|---|
| Backend | Java 17, Spring Boot 3, Spring MVC, Spring Security |
| ORM | Hibernate / Spring Data JPA |
| Templates | Thymeleaf |
| Database | MySQL (VARCHAR-based schema — no ENUM restrictions) |
| Messaging (roadmap) | Twilio WhatsApp Business API |
| Deployment | Railway / Docker |

---

## 🚀 Setup

### Prerequisites
- Java 17+
- MySQL 8+
- Maven 3.8+
- (Optional for WhatsApp) Twilio account with WhatsApp Sandbox enabled

### 1. Clone & configure

```bash
git clone https://github.com/your-username/helperconnect.git
cd helperconnect
```

```properties
# application.properties
spring.datasource.url=jdbc:mysql://localhost:3306/helperconnect
spring.datasource.username=root
spring.datasource.password=yourpassword
spring.jpa.hibernate.ddl-auto=update

# Twilio (roadmap)
twilio.account-sid=ACxxxxxxxxxxxxxxxx
twilio.auth-token=your_auth_token
twilio.whatsapp-from=whatsapp:+14155238886
```

### 2. Run

```bash
mvn spring-boot:run
```

App starts at `http://localhost:8080`

---

## 📡 API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/helfer/applications` | List all applications for logged-in helper |
| `POST` | `/helfer/apply/{servicePostId}` | Submit an application |
| `POST` | `/helfer/applications/{appId}/withdraw` | Withdraw an application |
| `GET` | `/helfer/applications/{appId}/withdraw` | Redirects to service detail (safe fallback) |
| `POST` | `/webhook/whatsapp` | Receive Twilio WhatsApp events *(roadmap)* |

> ⚠️ Withdrawal must always use `POST` — the `GET` fallback exists only to prevent 405 errors on accidental browser navigation.

---

## 🔐 Security Notes

- Withdrawal actions are `POST`-only, protected by Spring Security CSRF tokens
- WhatsApp webhook will validate `X-Twilio-Signature` using HMAC-SHA256 before processing any payload
- All conversation participants are verified before messages are forwarded
- Passwords stored as bcrypt hashes

---

## 📈 Roadmap

- [x] Application lifecycle management
- [x] Error-resilient VARCHAR schema (no ENUM truncation)
- [x] RESTful exception handling & logging
- [ ] WhatsApp conversation on application acceptance
- [ ] End-to-end secure messaging via Twilio
- [ ] Real-time notification webhooks
- [ ] Mobile-first UI redesign
- [ ] Admin dashboard with application analytics

---

## 👤 Author

**Yassine Kalai Ezzar** — Medieninformatik (B.Sc.) @ Berliner Hochschule für Technik  
Full-stack developer · Java / Spring Boot · React · Flutter  
[GitHub](https://github.com/your-username) · [LinkedIn](https://linkedin.com/in/your-profile)

---

<div align="center">
<sub>Built with ☕ Java and a lot of determination.</sub>
</div>
