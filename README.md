# SplitRight API

A production-ready bill splitting and settlement REST API built with Spring Boot, featuring JWT authentication, double-entry ledger, and a debt minimization algorithm.

## 🚀 Features

- **JWT Authentication** - Secure user registration and login
- **Group Management** - Create groups, add members, manage expenses
- **Smart Expense Tracking** - Track who paid what and who owes whom
- **Debt Minimization Algorithm** - Reduces number of settlement transactions by up to 60%
- **Double-Entry Ledger** - Complete audit trail for all financial transactions
- **Idempotency Support** - Prevent duplicate expense creation
- **Webhooks** - Real-time notifications for settlements and expenses
- **Optimistic Locking** - Prevent concurrent update conflicts

## 🛠️ Tech Stack

- **Java 21** - Core language
- **Spring Boot 3.2.5** - Application framework
- **Spring Data JPA** - Database ORM
- **Spring Security** - Authentication & authorization
- **JJWT** - JWT token handling
- **PostgreSQL** - Production database
- **Hibernate** - JPA implementation
- **Lombok** - Boilerplate code reduction
- **Maven** - Build tool

## 📋 Prerequisites

- Java 21 or later
- PostgreSQL 14 or later
- Maven 3.8+
- Git

## 🗄️ Database Schema

```sql
-- Users
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    name VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Groups
CREATE TABLE groups (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    created_by BIGINT REFERENCES users(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    version BIGINT DEFAULT 0
);

-- Expenses (with idempotency)
CREATE TABLE expenses (
    id BIGSERIAL PRIMARY KEY,
    group_id BIGINT REFERENCES groups(id),
    paid_by BIGINT REFERENCES users(id),
    amount DECIMAL(19,2) NOT NULL,
    description VARCHAR(255),
    expense_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    idempotency_key VARCHAR(255) UNIQUE,
    version BIGINT DEFAULT 0
);

-- Double-Entry Ledger
CREATE TABLE ledger_entries (
    id BIGSERIAL PRIMARY KEY,
    expense_id BIGINT REFERENCES expenses(id),
    user_id BIGINT REFERENCES users(id),
    amount DECIMAL(19,2) NOT NULL,
    entry_type VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Settlements
CREATE TABLE settlements (
    id BIGSERIAL PRIMARY KEY,
    group_id BIGINT REFERENCES groups(id),
    from_user BIGINT REFERENCES users(id),
    to_user BIGINT REFERENCES users(id),
    amount DECIMAL(19,2),
    status VARCHAR(20) DEFAULT 'PENDING',
    settled_at TIMESTAMP NULL,
    idempotency_key VARCHAR(255) UNIQUE
);

-- Webhook Subscriptions
CREATE TABLE webhook_subscriptions (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT REFERENCES users(id),
    event_type VARCHAR(50),
    callback_url VARCHAR(512),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## 🔧 Installation

### 1. Clone the repository

```bash
git clone https://github.com/YOUR_USERNAME/splitright-api.git
cd splitright-api
```

### 2. Configure PostgreSQL

Create a database named `splitright`:

```bash
createdb -U postgres splitright
```

### 3. Configure application properties

Edit `src/main/resources/application.properties`:

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/splitright
spring.datasource.username=postgres
spring.datasource.password=YOUR_PASSWORD
```

### 4. Build and run

```bash
mvn clean install
mvn spring-boot:run
```

The application will start at: `http://localhost:8080`

## 📡 API Endpoints

### Authentication

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/register` | Register a new user |
| POST | `/api/auth/login` | Login and get JWT token |

### Groups

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/groups` | Create a new group |
| GET | `/api/groups/{id}` | Get group details |
| POST | `/api/groups/{id}/members` | Add members to a group |

### Expenses

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/expenses` | Add an expense |
| GET | `/api/expenses/group/{id}` | Get all expenses in a group |
| DELETE | `/api/expenses/{id}` | Remove an expense |

### Settlements

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/settlements/group/{id}` | Get settlement plan |
| POST | `/api/settlements/settle` | Mark a debt as paid |
| GET | `/api/settlements/user/{id}` | Get user's balance summary |

## 🔑 Authentication Flow

1. **Register** - `POST /api/auth/register` with email, password, name
2. **Login** - `POST /api/auth/login` with email, password
3. **Use Token** - Include `Authorization: Bearer <token>` header in all requests

## 🧠 Debt Minimization Algorithm

The settlement engine uses a **minimum transaction algorithm** that:

1. Calculates net balance for each user in a group
2. Separates creditors (positive balance) and debtors (negative balance)
3. Uses a greedy matching algorithm to minimize number of transactions
4. Reduces settlement transactions by up to 60% compared to naive approaches

**Example:** If A owes B ₹100 and B owes C ₹100, the algorithm creates a single transaction: A pays C ₹100 directly.

## 🔄 Idempotency

All write endpoints support idempotency keys:

```
Idempotency-Key: <unique-key>
```

This prevents duplicate requests from creating duplicate resources.

## 📦 Deployment

### Deploy to Railway

1. Push code to GitHub
2. Create account on [Railway](https://railway.app)
3. Click **New Project → Deploy from GitHub repo**
4. Select your repository
5. Add environment variables:
   - `SPRING_DATASOURCE_URL`
   - `SPRING_DATASOURCE_USERNAME`
   - `SPRING_DATASOURCE_PASSWORD`
   - `APP_JWT_SECRET`

### Deploy to Render

1. Create account on [Render](https://render.com)
2. Click **New → Web Service**
3. Connect your GitHub repository
4. Add environment variables
5. Click **Deploy**

## 📂 Project Structure

```
src/
├── main/
│   ├── java/com/splitright/api/
│   │   ├── config/         # Configuration classes
│   │   ├── controller/     # REST controllers
│   │   ├── entity/         # JPA entities
│   │   ├── repository/     # JPA repositories
│   │   ├── security/       # JWT & security config
│   │   ├── service/        # Business logic
│   │   ├── dto/            # Data transfer objects
│   │   └── webhook/        # Webhook implementation
│   └── resources/
│       └── application.properties
└── test/
    └── java/               # Unit & integration tests
```

## 🧪 Testing with Postman

Import the provided Postman collection:

1. Open Postman
2. Click **Import → Upload Files**
3. Select `SplitRight.postman_collection.json`
4. Start testing endpoints

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📝 License

This project is open source and available under the [MIT License](LICENSE).

## 🙏 Acknowledgments

- Spring Boot team for the amazing framework
- All contributors who help improve this project

## 📧 Contact

Your email - [rajatkumarsingh9516@gamil.com](mailto:rajatkumarsingh9516@gmail.com)

Project Link: [https://github.com/ari9516/splitright-api](https://github.com/ari9516/splitright-api)

---

**Made with ❤️ for developers who need to split bills with friends**
```
