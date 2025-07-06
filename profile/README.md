# TravelFlow360

## Overview
TravelFlow360 is a SaaS platform for travel agencies to manage inquiries, quotes, bookings, and payments. Built with a microservice architecture, it uses **Next.js 15**, **Express.js**, **Python Flask**, **Zustand**, **MongoDB**, and **PostgreSQL**, deployed with **Docker** and **Kubernetes**. The platform supports roles like Organization Owner, Travel Agent, and Client, streamlining travel package creation and management.

## Repositories
Below are the repositories for TravelFlow360’s microservices and frontend:

1. **[travelflow360-frontend](https://github.com/TravelFlow360/travelflow360-frontend)**: Next.js 15 frontend with Zustand for state management, providing role-based dashboards and a public landing page.
2. **[travelflow360-authentication](https://github.com/TravelFlow360/travelflow360-authentication)**: For user authentication and RBAC.
3. **[travelflow360-organization-management](https://github.com/TravelFlow360/travelflow360-organization-management)**: Flask service for managing agency setup, team, and billing with PostgreSQL.
4. **[travelflow360-inquiry-management](https://github.com/TravelFlow360/travelflow360-inquiry-management)**: Flask service for creating and assigning client inquiries.
5. **[travelflow360-quote-management](https://github.com/TravelFlow360/travelflow360-quote-management)**: Flask service for generating quotes with markup calculations using.
6. **[travelflow360-hotel-management](https://github.com/TravelFlow360/travelflow360-hotel-management)**: Express.js service for managing hotel lists.
7. **[travelflow360-transfer-transport](https://github.com/TravelFlow360/travelflow360-transfer-transport)**: Express.js service for managing transfer options.
8. **[travelflow360-booking](https://github.com/TravelFlow360/travelflow360-booking)**: Flask service for creating bookings and vouchers.
9. **[travelflow360-payment](https://github.com/TravelFlow360/travelflow360-payment)**: Express.js service for processing payments (M-Pesa, Bank).
10. **[travelflow360-invoice-reporting](https://github.com/TravelFlow360/travelflow360-invoice-reporting)**: Flask service for generating invoices and reports.
11. **[travelflow360-settings-configuration](https://github.com/TravelFlow360/travelflow360-settings-configuration)**: Express.js service for managing agency settings.
12. **[travelflow360-notification](https://github.com/TravelFlow360/travelflow360-notification)**: Express.js service for sending emails.
13. **[travelflow360-super-admin](https://github.com/TravelFlow360/travelflow360-super-admin)**: Flask service for platform-wide settings.


## System Diagrams

### Workflow
The flowchart below shows the end-to-end process from visitor signup to booking and voucher generation.

```mermaid
graph TD
    A[Visitor Lands on Public Page] -->|View Pricing| B[Sign Up as Org Owner]
    A -->|Login| C[Authenticate]
    B --> D[Choose Plan]
    D --> E[Pay: M-Pesa/Card]
    E --> F[Account Created]
    F --> G[Org Owner: Add Team]
    G --> H[Agents, Operators]
    H --> I[Operator: Assign Inquiries]
    I --> J[Agent: Create Quote]
    J --> K[Add Hotels/Transfers]
    K --> L[Share Quote with Client]
    L --> M[Client: View Quote]
    M -->|Approve| N[Create Booking]
    M -->|Request Changes| J
    N --> O[Generate Voucher]
    O --> P[Send to Client/Operator]
    N --> Q[Client Payment]
    Q --> R[Accounts: Confirm Payment]
    R --> S[Generate Invoice/Reports]
```

### Entity-Relationship Diagram (ERD)
The ERD models the database schema for key entities like inquiries, quotes, and bookings.

```mermaid
erDiagram
    ORGANIZATION ||--o{ USER : owns
    ORGANIZATION ||--o{ INQUIRY : creates
    INQUIRY ||--o{ QUOTE : triggers
    QUOTE ||--o{ BOOKING : converts
    QUOTE ||--o{ HOTEL : includes
    QUOTE ||--o{ TRANSFER : includes
    BOOKING ||--o{ VOUCHER : generates
    BOOKING ||--o{ PAYMENT : requires
    PAYMENT ||--o{ INVOICE : triggers
    ORGANIZATION {
        string org_id PK
        string name
        string plan
    }
    USER {
        string user_id PK
        string org_id FK
        string role
    }
    INQUIRY {
        string inquiry_id PK
        string org_id FK
        string destination
        date travel_dates
    }
    QUOTE {
        string quote_id PK
        string inquiry_id FK
        float full_price
    }
    HOTEL {
        string hotel_id PK
        string org_id FK
        string name
    }
    TRANSFER {
        string transfer_id PK
        string quote_id FK
        float cost
    }
    BOOKING {
        string booking_id PK
        string quote_id FK
    }
    VOUCHER {
        string voucher_id PK
        string booking_id FK
    }
    PAYMENT {
        string payment_id PK
        string booking_id FK
        float amount
    }
    INVOICE {
        string invoice_id PK
        string payment_id FK
    }
```

### User Roles
The class diagram shows the hierarchy and permissions of user roles.

```mermaid
classDiagram
    class Super_Admin {
        +Manage platform settings
    }
    class Org_Owner {
        +Manage team
        +Configure billing
    }
    class Tour_Operator {
        +Assign inquiries
        +Manage hotels
    }
    class Travel_Agent {
        +Create quotes
    }
    class Customer_Service {
        +Create inquiries
    }
    class Accounts_Team {
        +Approve payments
    }
    class Client {
        +View quotes
        +Approve quotes
    }
    Super_Admin --> Org_Owner
    Org_Owner --> Tour_Operator
    Org_Owner --> Travel_Agent
    Org_Owner --> Customer_Service
    Org_Owner --> Accounts_Team
    Tour_Operator --> Travel_Agent
    Travel_Agent --> Client
    Customer_Service --> Client
    Accounts_Team --> Client
```

### System Architecture
The architecture diagram illustrates the microservices, databases, and DevOps tools.

```mermaid
graph TD
    A[Frontend: Next.js 15 + Zustand] -->|REST| B[Kong API Gateway]
    B --> C[Authentication: Express.js]
    B --> D[Org Management: Flask]
    B --> E[Inquiry: Flask]
    B --> F[Quote: Flask]
    B --> G[Hotel: Express.js]
    B --> H[Transfer: Express.js]
    B --> I[Booking: Flask]
    B --> J[Payment: Express.js]
    B --> K[Invoice: Flask]
    B --> L[Settings: Express.js]
    B --> M[Notification: Express.js]
    B --> N[Super Admin: Flask]
    
    C -->|PostgreSQL| O[PostgreSQL]
    D -->|PostgreSQL| O
    E -->|PostgreSQL| O
    F -->|PostgreSQL| O
    I -->|PostgreSQL| O
    J -->|PostgreSQL| O
    K -->|PostgreSQL| O
    N -->|PostgreSQL| O
    
    G -->|MongoDB| P[MongoDB]
    H -->|MongoDB| P
    L -->|MongoDB| P
    
    F --> G
    F --> H
    F --> E
    I --> F
    K --> I
    K --> J
    
    M -->|Email| Q[Resend/SendGrid]
    J -->|Payment| R[M-Pesa, Stripe]
    K -->|PDF| S[reportlab]
    
    M -->|RabbitMQ| T[RabbitMQ]
    F --> T
    I --> T
    K --> T
    
    U[Prometheus] --> C
    U --> D
    U --> E
    U --> F
    U --> G
    U --> H
    U --> I
    U --> J
    U --> K
    U --> L
    U --> M
    U --> N
    V[Grafana] --> U
    W[Kubernetes] --> C
    W --> D
    W --> E
    W --> F
    W --> G
    W --> H
    W --> I
    W --> J
    W --> K
    W --> L
    W --> M
    W --> N
```

## Setup
1. Clone all repositories:
   ```bash
   gh repo clone TravelFlow360/travelflow360-<service-name>
   ```
2. Follow each repository’s README for setup instructions.
3. Configure **Kong API Gateway**, **RabbitMQ**, **Prometheus**, and **Grafana** for integration and monitoring.
4. Deploy using **Docker** and **Kubernetes**.

## Contributing
- See each repository’s `CONTRIBUTING.md` for guidelines.
- Submit issues/PRs to respective repositories.
