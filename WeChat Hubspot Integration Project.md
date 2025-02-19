# Project: Connecting Data Between WeChat and HubSpot for Business Analysis

## 🚀 Problem Statement

At a large international school, different online platforms were used to manage users at different stages. The marketing system (**Parllay**) managed WeChat subscribers, while **HubSpot** stored event registration data and **OpenApply** stored application data. However:
1. Users were anonymous across platforms (**WeChat, HubSpot, OpenApply**).
2. No way to track conversions between **marketing (WeChat), sales (HubSpot), and enrollment (OpenApply)**.
3. Difficult to analyze **ROI on marketing spend** without cross-platform attribution.

## 🛠️ Solution Overview

To link user identities across **WeChat, HubSpot, and OpenApply**, a **unique user code system** was implemented using a **Flask API server** and **SQLite database**.

## 🏗️ Design Decisions & Architecture

| **Component**                | **Design Decision**                                      |
|------------------------------|---------------------------------------------------------|
| **WeChat Integration**       | Used **WeChat OpenID** as a unique identifier.         |
| **API Server (Flask)**       | Exposed an API to **generate, store, and return user codes**. |
| **Database (SQLite)**        | Chose SQLite for **simplicity (single-node, lightweight solution)**. |
| **Data Matching Service**    | Used **crontab-based scheduled jobs** (every 24 hours) to match user codes across platforms. |
| **HubSpot & OpenApply Integration** | Queried **APIs** to find and update matching users. |
| **Automation**               | Fully **automated cross-platform identity matching**.  |

## 📝 Implementation Details

### 1️⃣ User Code Generation & Retrieval
- **Trigger**: User clicks **“Get My Code”** on WeChat.
- **API Server Logic**:
  - Checks if **WeChat OpenID already exists** in the database.
  - **If found** → Returns existing **user_code**.
  - **If not found** → Generates a new unique **user_code**, stores it, and returns it.
- **Storage**: Saves (**WeChat OpenID, user_code**) in **SQLite**.

### 2️⃣ Delivering User Code to WeChat
- **Flask API Server** sends **user_code** to **WeChat API**, which replies to the user.
- **WeChat users manually enter this code** into HubSpot or OpenApply forms.

### 3️⃣ Periodic Data Matching
- **Scheduled job (crontab, runs at 2 AM daily)** queries:
  - **HubSpot user tables**
  - **OpenApply user tables**
- **If a submitted user_code matches an existing entry in SQLite**:
  - The **WeChat OpenID** is written into the respective **HubSpot & OpenApply user records**.
  - This allows **cross-platform identity linking**.

## 📊 Outcome & Impact

✅ **Cross-platform identity resolution**: Enabled tracking of user journeys from **WeChat marketing → HubSpot sales → OpenApply enrollment**.
✅ **Business impact analysis**: Provided **conversion insights** for marketing **ROI evaluation**.
✅ **Data pipeline automation**: Reduced manual data reconciliation via **cron jobs + API integrations**.
✅ **Scalable API-based design**: Made **future expansion** to other CRM or marketing tools possible.

## 💡 Alternative Optimizations

### 1️⃣ **Database Choice**
- **SQLite → PostgreSQL/MySQL** for **better scalability** and **multi-user support**.

### 2️⃣ **Event-Driven Matching (Instead of crontab polling)**
- Use **Kafka / Celery workers** to trigger matching **instantly** when a user submits a form.

### 3️⃣ **Webhook-based WeChat Communication**
- Instead of waiting for the user to click **“Get My Code”**, **WeChat webhooks** could **proactively send OpenID** when the user engages with the account.

## 🚀 Future Scalability & Architectural Enhancements

This project follows a **monolithic API with scheduled batch processing**, but it incorporates some **microservice-oriented elements** and could be further improved with **event-driven architecture**.

### 1️⃣ **Limitations of Monolithic Architecture and Microservices Alternative**

✅ **Monolithic API + Batch Processing**
- The **Flask API server** centralizes all functionality:
  - **User code generation**
  - **Data storage & retrieval**
  - **WeChat API interactions**
  - **HubSpot & OpenApply integrations**
  - **A cron job** (batch processing) periodically updates user records.

✅ **API-Driven Integration**
- WeChat, HubSpot, and OpenApply communicate via **REST APIs**.
- The **Flask server acts as an API Gateway**, facilitating data exchange.

🚫 **Scalability Concerns**
- All functionality runs on a **single Flask API server**.
- **WeChat, HubSpot, and OpenApply interactions are handled in one codebase**.
- **The Flask API may become a bottleneck as it scales**.

🔹 **Proposed Microservices Architecture:**

| **Service**            | **Responsibility**                                      |
|------------------------|---------------------------------------------------------|
| **WeChat Service**     | Handles user interactions and generates user codes. |
| **User Identity Service** | Stores user_code mappings and provides lookup functionality. |
| **Data Matching Service** | Matches user_codes from HubSpot & OpenApply. |
| **Analytics Service**   | Generates business reports based on conversion data. |

### 2️⃣ **Limitations of Batch Processing and Event-Driven Alternative**

🚫 **Batch Processing Constraints**
- **No real-time data synchronization**.
- **Cron jobs handle updates instead of instant reactions**.
- A **true event-driven system** would trigger updates **immediately** upon form submission.

🔹 **Proposed Event-Driven Architecture:**
1. **User submits a form** on HubSpot or OpenApply → Webhook **triggers an event**.
2. **Event Broker** (Kafka / RabbitMQ / AWS SQS) **receives the event**.
3. **Data Matching Service processes it instantly** and updates WeChat OpenID mappings.
4. **Analytics Service updates reports in real time**.

✅ **Benefits of Event-Driven Design**:
- **Instant updates** (no delays from cron jobs).
- **Decoupled services** (independent scaling).
- **Better scalability** (handles high event throughput efficiently).

By transitioning to **microservices and event-driven design**, this system can achieve **better performance, real-time synchronization, and increased scalability**.

