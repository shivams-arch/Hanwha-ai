# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PROJECT AQUA THISTLE is a personalized mentorship application for GenZ users to make better life decisions. The project consists of:
- **AI Agent**: A conversational chatbot optimized for long-term financial well-being
- **Mobile-First App/Platform**: Visual dashboard with interactive charts and categorization

## Project Requirements

### Core Features
1. **Visual Dashboard**
   - GenZ-friendly design language with interactive visualizations (bar graphs, pie charts)
   - Charts must react/update to user inputs and queries in real-time
   - All visual elements should respond to changes in budget, expenses, and projections

2. **Category Organization**
   - Finance, Education, Family, Friends, Weekend Activities/Vacation
   - Each category must be reflected in visual language with interactive graphical aids

3. **AI Chatbot Assistant**
   - Preferably using ChatGPT-4 or comparable LLM
   - Pre-prompted to focus on long-term user well-being
   - Answers based on fabricated test user profile (bank status, job status, etc.)
   - Provides budget-conscious decision guidance
   - GenZ tone: friendly, concise, like talking to a friend

### Test User Profile (from initialPoC.json)
- Bank Account Balance: $5,250
- Monthly Income: $4,500
- Monthly Fixed Expenses: $2,800 (Rent: $1,200, Utilities: $200, Insurance: $300, Car Payment: $400, Student Loan: $350, Subscriptions: $150, Phone: $100, Internet: $100)
- Job: Full-time Software Developer
- Financial Goals: Build $10,000 emergency fund, save for house down payment, pay off student loans early

## AI Agent Guidelines

When working with the AI agent prompts:
- Character name: "Finny"
- Response style: Concise, GenZ tone, friendly
- Always consider user's complete financial picture
- Prioritize long-term financial health over short-term gratification
- Encourage emergency fund building and debt reduction
- Provide specific, actionable budget recommendations
- Use calculator tool for financial calculations
- Tie current events back to user's financial situation when relevant
- Suggest budget-conscious alternatives for purchases/expenses

## Grading Criteria Reference

### AI Agent (30 points)
- User-Centric Response (10pts): Friendly, contextual, GenZ tone
- Performance (5pts): Timely responses
- Logic (5pts): Functions according to scenario
- Technical (5pts): Integration with app/platform
- Information Hierarchy/Clarity (5pts): Clear formatting with spacing, bullets, emphasis

### App/Platform (30 points)
- Performance (10pts): Functions as PoC for focus groups
- User Flow Logic (10pts): Clear journey from question to insight to answers
- Technical (5pts): Technical implementation
- Information Visualization (5pts): Dashboard clearly communicates through visuals

## Architecture Notes

### Initial PoC Structure (n8n workflow)
The `initialPoC.json` file contains an n8n workflow with:
1. **Webhook**: Receives POST requests from React app at `/finance-assistant`
2. **User Profile Configuration**: Sets up user context with profile data
3. **AI Agent**: LangChain agent with system prompt
4. **OpenAI GPT-4o Model**: LLM integration (temperature: 0.7)
5. **Conversation Memory**: Buffer window with 10-message context
6. **Calculator Tool**: For financial calculations
7. **Response Handler**: Returns AI output to React app

This workflow demonstrates the backend integration pattern for the AI agent.

## Backend Architecture (Chosen Stack)

**IMPORTANT**: Refer to `BACKEND_IMPLEMENTATION_PLAN.md` for detailed implementation phases and checklist.

### Tech Stack
- **API Framework**: Node.js + Express.js with TypeScript
- **Database**: PostgreSQL (user profiles, categories, financial data)
- **Cache/Sessions**: Redis (session management, conversation cache, real-time data)
- **AI Integration**: LangChain (via hybrid n8n + custom API architecture)
- **Real-time**: WebSocket/Server-Sent Events for live dashboard updates

### Hybrid Architecture
- **n8n workflows**: Handle AI chatbot (LangChain + OpenAI integration)
- **Custom Express API**: User management, financial calculations, dashboard data, real-time updates
- **Integration**: Custom API triggers n8n workflows and receives responses via webhooks

### Database Schema Overview
- **Users**: Profile data, financial information
- **Categories**: Finance, Education, Family, Friends, Vacation
- **Transactions**: Income/expense tracking
- **Financial_Goals**: Goal tracking with progress
- **Conversations**: Chat history with AI

## Development Context

- **Deadline**: Friday, October 31st 2025 at 11:59PM PST
- **Delivery Format**: Must be accessible via web link (or iOS download link)
- **Design Reference**: Figma design at https://www.figma.com/design/IeEPiyMsAjNupnVT7UhVA8/PROJECT-AQUA-THISTLE
- **Platform**: Mobile-first (can be web or iOS app)

## Key Implementation Considerations

1. **Interactive Visualizations**: Charts and numbers must update dynamically based on:
   - User queries (e.g., "How do life expenses increase over time?")
   - Test scenarios (e.g., "How could I afford housing in San Francisco with X income?")

2. **Category-Visual Integration**: Visual dashboard should reflect category-specific data with appropriate chart types

3. **AI-Dashboard Sync**: AI responses should potentially trigger dashboard updates and vice versa

4. **Focus on UX**: This is a focus group prototype, so emphasis on visual appeal and intuitive flow is critical
