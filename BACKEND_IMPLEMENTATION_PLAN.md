# Backend Implementation Plan - PROJECT AQUA THISTLE

## Tech Stack Summary
- **API Framework**: Node.js + Express.js with TypeScript
- **Database**: PostgreSQL (user profiles, categories, financial data)
- **Cache/Sessions**: Redis (session management, conversation cache, real-time data)
- **AI Integration**: LangChain (via n8n workflows)
- **Architecture**: Hybrid (Custom Express API + n8n for AI workflows)

## Project Structure
```
backend/
├── src/
│   ├── config/          # Environment & database config
│   ├── controllers/     # Route controllers
│   ├── middleware/      # Auth, validation, error handling
│   ├── models/          # Database models (Sequelize/TypeORM)
│   ├── routes/          # API route definitions
│   ├── services/        # Business logic layer
│   ├── utils/           # Helper functions, calculators
│   ├── validators/      # Request validation schemas
│   ├── websockets/      # WebSocket/SSE handlers
│   └── app.ts           # Express app setup
├── db/
│   ├── migrations/      # Database migrations
│   └── seeds/           # Seed data (test user profile)
├── tests/               # Test files
├── n8n/
│   └── workflows/       # n8n workflow exports
└── docs/                # API documentation
```

---

## Phase 1: Project Foundation & Setup

### 1.1 Initial Project Structure
- [ ] Initialize Node.js project with TypeScript
- [ ] Set up package.json with dependencies
- [ ] Configure TypeScript (tsconfig.json)
- [ ] Set up ESLint and Prettier
- [ ] Create folder structure as outlined above
- [ ] Set up Git and .gitignore

### 1.2 Database Setup (PostgreSQL)
- [ ] Install PostgreSQL locally or use Docker
- [ ] Choose ORM (Sequelize or TypeORM)
- [ ] Configure database connection
- [ ] Create database schema design:
  - **Users table**: id, email, name, profile_data (JSON), created_at, updated_at
  - **Categories table**: id, user_id, name (Finance/Education/Family/Friends/Vacation), budget_allocated, spent_amount
  - **Transactions table**: id, user_id, category_id, amount, description, date, type (income/expense)
  - **Financial_Goals table**: id, user_id, goal_type, target_amount, current_amount, deadline
  - **Conversations table**: id, user_id, session_id, messages (JSON), created_at, updated_at
- [ ] Set up migration system
- [ ] Create initial migrations for all tables

### 1.3 Redis Setup
- [ ] Install Redis locally or use Docker
- [ ] Configure Redis connection
- [ ] Set up Redis client with connection pooling
- [ ] Design Redis key structure:
  - `session:{sessionId}` - User sessions
  - `conversation:{sessionId}` - Conversation history cache
  - `dashboard:{userId}` - Dashboard data cache
  - `budget_calc:{userId}:{scenario}` - Calculation results cache

### 1.4 Environment Configuration
- [ ] Create .env.example file
- [ ] Set up environment variables:
  - Database credentials
  - Redis connection
  - n8n webhook URLs
  - OpenAI API key (for reference)
  - JWT secret
  - Port configurations
- [ ] Use dotenv for configuration management
- [ ] Set up different configs for dev/staging/production

---

## Phase 2: Core API Infrastructure

### 2.1 Express Server Setup
- [ ] Initialize Express app with TypeScript
- [ ] Configure middleware:
  - body-parser (JSON parsing)
  - cors (with proper configuration)
  - helmet (security headers)
  - morgan (HTTP logging)
  - compression (response compression)
- [ ] Set up error handling middleware
- [ ] Configure request logging
- [ ] Set up health check endpoint (GET /health)

### 2.2 Authentication & Authorization
- [ ] Implement JWT-based authentication
- [ ] Create auth middleware for protected routes
- [ ] Build endpoints:
  - POST /auth/register
  - POST /auth/login
  - POST /auth/logout
  - GET /auth/me (get current user)
  - POST /auth/refresh (refresh token)
- [ ] Implement session management with Redis
- [ ] Add rate limiting on auth endpoints

### 2.3 Input Validation & Security
- [ ] Set up validation library (Joi or express-validator)
- [ ] Create validation schemas for all endpoints
- [ ] Implement request sanitization
- [ ] Add rate limiting middleware (express-rate-limit)
- [ ] Configure CORS for frontend domain
- [ ] Set up API versioning (/api/v1/...)

### 2.4 Error Handling & Logging
- [ ] Create custom error classes (ValidationError, AuthError, etc.)
- [ ] Implement global error handler
- [ ] Set up logging system (Winston or Pino)
- [ ] Log levels: error, warn, info, debug
- [ ] Log to files and console
- [ ] Create error response format standard

---

## Phase 3: User Profile Management

### 3.1 User Profile Endpoints
- [ ] GET /api/v1/users/profile - Get current user profile
- [ ] PUT /api/v1/users/profile - Update user profile
- [ ] GET /api/v1/users/:id - Get user by ID (admin)
- [ ] DELETE /api/v1/users/:id - Delete user (soft delete)

### 3.2 User Profile Service
- [ ] Create UserService class
- [ ] Implement CRUD operations
- [ ] Add profile data validation
- [ ] Store financial profile data:
  - Bank account balance
  - Monthly income
  - Monthly expenses breakdown
  - Job status
  - Financial goals

### 3.3 Seed Test User Profile
- [ ] Create seed script for test user from initialPoC.json:
  - Balance: $5,250
  - Income: $4,500/month
  - Fixed expenses: $2,800/month (with breakdown)
  - Job: Software Developer
  - Goals: Emergency fund, house down payment, pay off loans
- [ ] Add command to seed database: `npm run seed`

---

## Phase 4: Category Management System

### 4.1 Category Endpoints
- [x] GET /api/v1/categories - Get all categories for user
- [x] POST /api/v1/categories - Create category
- [x] PUT /api/v1/categories/:id - Update category
- [x] DELETE /api/v1/categories/:id - Delete category
- [x] GET /api/v1/categories/:id/transactions - Get category transactions
- [x] GET /api/v1/categories/:id/summary - Get category spending summary

### 4.2 Category Service
- [x] Implement category CRUD operations
- [x] Create default categories on user signup:
  - Finance, Education, Family, Friends, Weekend Activities/Vacation
- [x] Calculate budget allocation per category
- [x] Track spending vs budget per category
- [x] Aggregate category data for dashboard

### 4.3 Transaction Management
- [x] POST /api/v1/transactions - Create transaction
- [x] GET /api/v1/transactions - List all transactions (with filters)
- [x] PUT /api/v1/transactions/:id - Update transaction
- [x] DELETE /api/v1/transactions/:id - Delete transaction
- [x] Implement transaction categorization logic

---

## Phase 5: Financial Calculation Engine

### 5.1 Budget Calculator Utility
- [x] Create BudgetCalculator class
- [x] Calculate monthly budget breakdown
- [x] Calculate disposable income (income - expenses)
- [x] Calculate savings rate
- [x] Project monthly savings
- [x] Calculate emergency fund progress

### 5.2 Scenario Testing Engine
- [x] Build ScenarioEngine class
- [x] Implement scenario: "Can I afford X with Y income?"
- [x] Expense projection over time (1 year, 5 years, 10 years)
- [x] Housing affordability calculator (based on income)
- [x] Debt payoff timeline calculator
- [x] Savings goal timeline calculator

### 5.3 Financial Projection Endpoints
- [x] POST /api/v1/calculations/budget - Calculate current budget status
- [x] POST /api/v1/calculations/scenario - Run "what-if" scenario
- [x] GET /api/v1/calculations/projections - Get expense projections
- [x] POST /api/v1/calculations/affordability - Check if expense is affordable
- [x] GET /api/v1/goals/progress - Calculate goal progress

### 5.4 Caching Strategy
- [x] Cache calculation results in Redis (TTL: 5 minutes)
- [x] Invalidate cache on profile/transaction updates
- [x] Implement cache warming for dashboard data

---

## Phase 6: n8n Integration (AI Chatbot)

### 6.1 n8n Workflow Enhancement
- [x] Export and version control n8n workflow
- [x] Provide structured payload (profile, budget, goals, categories) for Finny context
- [x] Add backend webhook with API key verification for n8n responses

### 6.2 Custom API ↔ n8n Integration
- [x] POST /api/v1/chat/message - Send message to AI
  - [x] Forward to n8n webhook
  - [x] Include user profile context
  - [x] Include session ID
- [x] POST /api/v1/webhooks/n8n/response - Receive AI response from n8n
  - [x] Store conversation in database
  - [x] Update Redis conversation cache
  - [x] Return response to frontend (JSON; SSE/WebSocket coming in Phase 7)

### 6.3 Conversation Management
- [x] Create ConversationService
- [x] Store conversations in PostgreSQL
- [x] Cache active conversations in Redis
- [x] GET /api/v1/chat/history/:sessionId - Get conversation history
- [x] DELETE /api/v1/chat/session/:sessionId - Clear conversation
- [x] Implement conversation context builder for AI

### 6.4 AI Agent Context Integration
- [x] Inject financial profile and budget summaries into AI requests
- [x] Format prompts with recent conversation history
- [x] Surface calculation results for LangChain tools
- [x] Provide GenZ-friendly fallbacks when n8n is unreachable

---

## Phase 7: Real-time Dashboard Data API

### 7.1 Dashboard Aggregation Endpoints
- [x] GET /api/v1/dashboard/overview - Main dashboard data
  - [x] Current budget status
  - [x] Monthly spending by category
  - [x] Savings progress
  - [x] Financial goals progress
  - [x] Recent transactions (last 10)
- [x] GET /api/v1/dashboard/charts/budget-breakdown - Pie chart data
- [x] GET /api/v1/dashboard/charts/spending-trends - Line chart data (monthly)
- [x] GET /api/v1/dashboard/charts/category-comparison - Bar chart data
- [x] GET /api/v1/dashboard/insights - AI-generated insights (from cached conversations)

### 7.2 Real-time Updates (WebSocket/SSE)
- [x] Set up WebSocket server (Socket.io)
- [x] Implement events:
  - [x] `dashboard:update` - Budget/category/transaction change notifications
  - [x] `chat:message` - New AI message
  - [x] `chat:session:cleared` - Session cleared signal
- [x] Create WebSocketService for managing connections
- [x] Implement room-based updates (per user via userId rooms)

### 7.3 Data Aggregation Service
- [x] Create DashboardService
- [x] Aggregate data from multiple sources:
  - [x] User profile
  - [x] Categories
  - [x] Transactions
  - [x] Financial goals
  - [x] Conversation insights
- [x] Implement caching with Redis (TTL: 1 minute)
- [ ] Optimize database queries with indexes

---

## Phase 8: Testing & Quality Assurance

### 8.1 Unit Tests
- [x] Set up Jest for testing
- [x] Test utilities (BudgetCalculator, ScenarioEngine)
- [ ] Test services (UserService, CategoryService, etc.)
- [x] Test middleware (auth, validation, error handling)
- [ ] Aim for 80%+ code coverage

### 8.2 Integration Tests
- [ ] Test API endpoints (Supertest)
- [ ] Test database operations
- [ ] Test Redis cache operations
- [ ] Test n8n webhook integration
- [ ] Test WebSocket/SSE functionality

### 8.3 API Testing
- [ ] Create Postman/Insomnia collection
- [ ] Test all endpoints with various payloads
- [ ] Test error scenarios
- [ ] Test rate limiting
- [ ] Test authentication flows

---

## Phase 9: Documentation & Developer Experience

### 9.1 API Documentation
- [x] Set up Swagger/OpenAPI (static spec in `docs/api/openapi.yaml`)
- [x] Document key endpoints with request/response schemas
- [ ] Generate interactive API docs (Swagger UI)
- [ ] Host docs at /api-docs

### 9.2 Code Documentation
- [x] Add JSDoc comments to functions (DashboardService, WebSocketService)
- [x] Document complex algorithms (architecture overview)
- [x] Create architecture diagrams (textual in docs/ARCHITECTURE.md)
- [x] Document database schema & Redis keys (docs/ARCHITECTURE.md)

### 9.3 Developer Scripts
- [x] Documented core scripts in docs/DEVELOPER_GUIDE.md

---

## Phase 10: Deployment & DevOps

### 10.1 Docker Configuration
- [x] Create Dockerfile for backend
- [ ] Create docker-compose.yml:
  - Backend service
  - PostgreSQL service
  - Redis service
  - n8n service
- [ ] Configure environment-specific compose files
- [ ] Add health checks to services

### 10.2 Environment Setup
- [x] Configure production environment variables
- [ ] Set up database connection pooling for production
- [ ] Configure Redis clustering (if needed)
- [ ] Set up logging aggregation
- [ ] Configure error monitoring (Sentry or similar)

### 10.3 CI/CD Pipeline
- [x] Set up GitHub Actions / GitLab CI
- [x] Automated testing on push
- [x] Automated linting
- [x] Build Docker image (via workflow)
- [ ] Deploy to staging on merge to develop
- [ ] Deploy to production on merge to main

### 10.4 Monitoring & Observability
- [x] Document monitoring & ops checklist
- [ ] Track API performance metrics
- [ ] Monitor database query performance
- [ ] Set up alerts for errors and downtime
- [ ] Track AI usage and costs

---

## Phase 11: Security & Performance Optimization

### 11.1 Security Hardening
- [ ] Implement SQL injection prevention (ORM parameterized queries)
- [ ] Add XSS protection
- [ ] Implement CSRF protection for state-changing operations
- [ ] Add security headers (helmet.js)
- [ ] Implement API key rotation mechanism
- [ ] Add request size limits
- [ ] Implement account lockout after failed login attempts

### 11.2 Performance Optimization
- [ ] Add database indexes on frequently queried fields
- [ ] Implement query result caching
- [ ] Optimize n+1 queries
- [ ] Add CDN for static assets (if any)
- [ ] Implement response compression
- [ ] Add database query logging and optimization
- [ ] Load test with Artillery or k6

### 11.3 Backup & Recovery
- [ ] Set up automated PostgreSQL backups
- [ ] Implement Redis persistence configuration
- [ ] Create backup restoration procedures
- [ ] Test disaster recovery plan

---

## Dependencies & Packages

### Core Dependencies
```json
{
  "express": "^4.18.0",
  "typescript": "^5.0.0",
  "dotenv": "^16.0.0",
  "cors": "^2.8.5",
  "helmet": "^7.0.0",
  "compression": "^1.7.4"
}
```

### Database
```json
{
  "pg": "^8.11.0",
  "sequelize": "^6.32.0" // or "typeorm": "^0.3.0",
  "sequelize-typescript": "^2.1.5",
  "redis": "^4.6.0"
}
```

### Authentication & Security
```json
{
  "jsonwebtoken": "^9.0.0",
  "bcrypt": "^5.1.0",
  "express-rate-limit": "^6.8.0",
  "joi": "^17.9.0" // or "express-validator": "^7.0.0"
}
```

### Utilities
```json
{
  "axios": "^1.4.0",
  "winston": "^3.10.0",
  "socket.io": "^4.7.0",
  "langchain": "^0.0.200"
}
```

### Development
```json
{
  "nodemon": "^3.0.0",
  "ts-node": "^10.9.0",
  "jest": "^29.6.0",
  "@types/jest": "^29.5.0",
  "supertest": "^6.3.0",
  "eslint": "^8.45.0",
  "prettier": "^3.0.0"
}
```

---

## Success Metrics

### Technical Metrics
- API response time < 200ms (p95)
- AI response time < 3 seconds
- 99.9% uptime
- Zero critical security vulnerabilities
- 80%+ test coverage

### Business Metrics (Focus Group Testing)
- AI agent responses feel friendly and helpful (user survey)
- Dashboard visualizations are clear and actionable
- User can complete key flows without confusion
- GenZ tone resonates with target audience

---

## Timeline Estimation (Backend Only)

- **Phase 1-2**: 3-4 days (Foundation & Infrastructure)
- **Phase 3-4**: 2-3 days (User & Category Management)
- **Phase 5**: 3-4 days (Financial Calculations)
- **Phase 6**: 2-3 days (n8n Integration)
- **Phase 7**: 2-3 days (Dashboard API)
- **Phase 8**: 2-3 days (Testing)
- **Phase 9-11**: 2-3 days (Docs, Deployment, Security)

**Total: ~16-23 days** (can be compressed with focused work)

---

## Notes for Implementation

1. **Start with Phase 1-2** to establish foundation
2. **Implement test user profile early** for realistic testing
3. **Test n8n integration frequently** to ensure hybrid architecture works
4. **Focus on API design first** before implementation (REST best practices)
5. **Use this document as checklist** - check off items as completed
6. **Prioritize features** based on grading criteria (focus on what scores points)
7. **Keep frontend requirements in mind** when designing endpoints
