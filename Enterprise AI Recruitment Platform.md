# Enterprise AI Recruitment Platform

## Master-Level Technical Architecture & Codebase Masterclass

---

# 1. Executive Overview

## 1.1 What is this project?

This repository implements a **full-stack AI-powered recruitment and interview platform**.

At its core, it combines:

- Candidate management
- Resume upload and AI parsing
- Job-description intelligence
- AI-generated interview questions
- Adaptive interview questioning
- Voice-based interviews
- Speech-to-text
- Text-to-speech
- AI answer evaluation
- Deterministic communication/speech analysis
- Competency scoring
- Job-fit analysis
- AI hiring recommendations
- Candidate ranking
- Candidate comparison
- Recruiter notes
- Hiring-decision overrides
- Browser-based proctoring
- Company-level recruitment analytics

The system is therefore much more than an "AI interview app".

Conceptually, it is evolving toward:

```text
                    ENTERPRISE RECRUITMENT PLATFORM

        ┌───────────────────────────────────────────┐
        │               Recruiter                    │
        │ Candidate management / analytics          │
        └─────────────────────┬─────────────────────┘
                              │
                              ▼
        ┌───────────────────────────────────────────┐
        │          Recruitment Intelligence          │
        │                                             │
        │ Resume AI                                   │
        │ Job Intelligence                            │
        │ Interview Intelligence                      │
        │ Answer Evaluation                           │
        │ Job Fit                                     │
        │ Hiring Decision                             │
        │ Candidate Ranking                           │
        └─────────────────────┬─────────────────────┘
                              │
                              ▼
        ┌───────────────────────────────────────────┐
        │             Interview Engine               │
        │                                             │
        │ REST API                                    │
        │ WebSocket session                           │
        │ Voice                                       │
        │ Adaptive questioning                        │
        │ Real-time feedback                          │
        └─────────────────────┬─────────────────────┘
                              │
                              ▼
        ┌───────────────────────────────────────────┐
        │                Data Layer                  │
        │                                             │
        │ PostgreSQL                                  │
        │ Prisma                                      │
        │ Redis                                       │
        │ Local resume storage                        │
        └───────────────────────────────────────────┘
```

---

# 2. The Most Important Architectural Insight

The repository looks like a conventional Express + React application at first glance.

It is actually composed of **four different architectural systems**:

### System A — Enterprise CRUD Platform

Handles:

```text
Companies
Users
Candidates
Resumes
Recruiter Notes
Hiring Decisions
```

### System B — AI Interview Engine

Handles:

```text
Job Description
       ↓
Job Intelligence
       ↓
Question Generation
       ↓
Interview
       ↓
Answer
       ↓
AI Evaluation
       ↓
Interview Context
       ↓
Next Question
```

### System C — Real-Time Voice System

Handles:

```text
Browser Microphone
       ↓
MediaRecorder
       ↓
Base64
       ↓
WebSocket
       ↓
Deepgram
       ↓
Transcript
       ↓
AI Evaluation
       ↓
WebSocket
       ↓
Browser
```

### System D — Recruitment Intelligence / Decision Support

Handles:

```text
Interview Data
     │
     ├── Competency Analysis
     ├── Job Fit
     ├── Hiring Decision
     ├── Candidate Ranking
     ├── Candidate Comparison
     └── Skill Gap Analytics
```

That decomposition is extremely important for understanding the project.

---

# 3. Repository Structure

The project contains approximately **162 files**.

The major structure is:

```text
enterprise-ai-recruitment-platform-main/
│
├── README.md
│
├── backend/
│   ├── prisma/
│   │   ├── schema.prisma
│   │   └── migrations/
│   │
│   ├── src/
│   │   ├── analysis/
│   │   ├── config/
│   │   ├── controllers/
│   │   ├── middleware/
│   │   ├── models/
│   │   ├── routes/
│   │   ├── scripts/
│   │   ├── services/
│   │   ├── types/
│   │   ├── utils/
│   │   ├── validators/
│   │   ├── websocket/
│   │   ├── app.ts
│   │   └── server.ts
│   │
│   ├── uploads/
│   ├── package.json
│   └── tsconfig.json
│
└── frontend/
    ├── src/
    │   ├── api/
    │   ├── components/
    │   ├── hooks/
    │   ├── pages/
    │   ├── proctoring/
    │   ├── store/
    │   ├── types/
    │   ├── App.tsx
    │   └── main.tsx
    │
    ├── package.json
    ├── vite.config.ts
    └── tailwind.config.js
```

The backend is clearly the more architecturally complex side.

---

# 4. Architectural Pattern

The backend primarily follows a:

# Layered / Controller-Service-Repository-like Architecture

Although there isn't a formal repository layer, the practical structure is:

```text
HTTP Request
     │
     ▼
Route
     │
     ▼
Middleware
     │
     ▼
Controller
     │
     ▼
Service
     │
     ├──────────────► AI Services
     │
     ├──────────────► Redis
     │
     ├──────────────► Analysis
     │
     └──────────────► Prisma
                         │
                         ▼
                     PostgreSQL
```

For example:

```text
POST /api/v1/interviews

        ↓

interview.routes.ts

        ↓

authenticate middleware

        ↓

validation middleware

        ↓

interview.controller.ts

        ↓

interview.service.ts

        ↓
 ┌──────┴───────────────┐
 │                      │
 ▼                      ▼
job-intelligence      ai.service
 │                      │
 ▼                      ▼
Gemini              Gemini
 │
 ▼
Prisma
 │
 ▼
PostgreSQL
```

This is a good architecture for a project of this scale because it prevents controllers from becoming giant business-logic files.

---

# 5. Technology Stack

## Backend

| Technology     | Purpose                       |
| -------------- | ----------------------------- |
| Node.js        | Runtime                       |
| TypeScript     | Type safety                   |
| Express        | HTTP API                      |
| Prisma         | ORM                           |
| PostgreSQL     | Primary relational database   |
| Redis          | Real-time session state/cache |
| JWT            | Authentication                |
| bcryptjs       | Password hashing              |
| Zod            | Request validation            |
| Multer         | File upload                   |
| pdf-parse      | PDF extraction                |
| Mammoth        | DOCX extraction               |
| Google Gemini  | LLM                           |
| Deepgram       | Speech-to-text                |
| WebSocket / ws | Real-time communication       |
| Helmet         | Security headers              |
| CORS           | Cross-origin control          |
| Morgan         | HTTP logging                  |

## Frontend

| Technology      | Purpose                              |
| --------------- | ------------------------------------ |
| React           | UI                                   |
| TypeScript      | Type safety                          |
| Vite            | Build/dev server                     |
| Tailwind CSS    | Styling                              |
| React Router    | Routing                              |
| Axios           | HTTP client                          |
| Zustand         | State management                     |
| React Hook Form | Form management                      |
| Zod             | Validation                           |
| React Query     | Dependency included for server state |
| MediaPipe       | Face detection                       |
| Web APIs        | Camera/microphone/screen             |
| Lucide React    | Icons                                |

---

# 6. Why These Technologies?

## 6.1 Why React?

React is appropriate because the interview experience is highly interactive.

The UI needs to respond to:

```text
WebSocket events
Microphone state
Recording state
Question changes
AI scores
Transcript updates
Proctoring events
Connection state
```

A server-rendered architecture would be significantly more cumbersome.

---

# 7. Why TypeScript?

This project has a very large number of moving parts.

For example:

```text
Frontend
     ↓
REST
     ↓
Backend
     ↓
WebSocket
     ↓
Redis
     ↓
AI
     ↓
Database
```

Without static typing, message mismatches would become much easier.

TypeScript provides:

```text
compile-time contracts
        +
IDE assistance
        +
safer refactoring
```

The backend uses strict TypeScript configuration.

The frontend also enables:

```json
"strict": true
```

which is appropriate for production-style code.

---

# 8. Why Express?

Express is intentionally lightweight.

The project doesn't require:

- NestJS dependency injection
- Spring-style enterprise framework
- Django ORM
- Rails conventions

The developers instead implemented their own:

```text
routes
controllers
middleware
services
```

This gives direct control.

Trade-off:

### Advantage

Simple mental model.

### Disadvantage

As the project grows, architecture enforcement becomes the responsibility of developers.

For a large enterprise system, NestJS or another strongly opinionated architecture could eventually become attractive.

---

# 9. Why PostgreSQL?

PostgreSQL is an excellent fit.

The data has strong relationships:

```text
Company
   │
   ├── Users
   │
   └── Candidates
          │
          └── Interviews
                 │
                 ├── Questions
                 ├── Answers
                 ├── Competencies
                 ├── Job Fit
                 ├── Hiring Decision
                 └── Feedback Report
```

This is relational data.

Using MongoDB would make relationship management more complicated.

PostgreSQL also provides:

- transactions
- constraints
- indexes
- referential integrity
- JSONB
- enum types

The project uses both relational columns and JSONB strategically.

---

# 10. Why Prisma?

Prisma gives:

```text
schema.prisma
      ↓
generated TypeScript client
      ↓
type-safe database operations
```

For example:

```typescript
await prisma.interview.findUnique({
    where: { id: interviewId }
});
```

Instead of writing raw SQL everywhere.

The biggest advantage is type safety.

The trade-off is less direct control than writing SQL manually.

For this project, Prisma is a sensible choice.

---

# 11. Why Redis?

Redis is not being used as the primary database.

Its most important purpose here is:

# Real-time interview session state

The WebSocket handler stores:

```text
voice_session:[interviewId]
```

containing:

```text
interviewId
userId
currentQuestionIndex
questions
startedAt
isProcessing
```

with a TTL of:

```text
2 hours
```

This is an excellent conceptual use of Redis.

PostgreSQL is persistent state.

Redis is ephemeral session state.

---

# 12. Authentication Architecture

The system uses:

```text
Access Token
+
Refresh Token
```

## Login flow

```text
User
 │
 │ email + password
 ▼
POST /auth/login
 │
 ▼
auth.service
 │
 ├── find user
 ├── bcrypt.compare()
 │
 ▼
token.service
 │
 ├── access JWT
 └── refresh JWT
       │
       ▼
   PostgreSQL
```

The refresh token is persisted in:

```text
refresh_tokens
```

This is better than using completely stateless refresh tokens because the server can revoke stored tokens.

---

# 13. Password Security

Passwords are hashed using:

```typescript
bcrypt.hash(password, 12)
```

A cost factor of 12 is reasonable.

The login implementation also uses a dummy bcrypt hash when the user does not exist:

```typescript
const dummyHash = ...
```

This attempts to reduce timing differences between:

```text
existing user
```

and

```text
nonexistent user
```

That is a security-conscious design.

---

# 14. Role-Based Access Control

Roles include:

```text
SUPER_ADMIN
COMPANY_ADMIN
RECRUITER
INTERVIEWER
CANDIDATE
```

Authorization is implemented using middleware.

Example:

```typescript
authorize(
    UserRole.COMPANY_ADMIN,
    UserRole.RECRUITER
)
```

The architecture is:

```text
Authentication
      ↓
Who are you?
      ↓
Authorization
      ↓
What are you allowed to do?
```

These are two different concepts and the project correctly separates them.

---

# 15. Multi-Tenant Architecture

This is one of the most important enterprise concepts in the repository.

Companies represent tenants.

```text
Company
   │
   ├── Users
   │
   └── Candidates
```

Users contain:

```text
companyId
```

Candidates contain:

```text
companyId
```

This allows queries like:

```typescript
where: {
    companyId
}
```

The system therefore follows a:

# Shared database / shared schema multi-tenancy model

rather than:

```text
database per company
```

or:

```text
schema per company
```

This is much cheaper and simpler operationally.

---

# 16. Critical Multi-Tenancy Principle

Every tenant-owned query should follow:

```text
WHERE resource.id = [ID]
AND resource.companyId = currentUser.companyId
```

The repository does this correctly in several places.

For example candidate lookup:

```typescript
where: {
    id: candidateId,
    companyId,
}
```

This is good.

However, this rule is **not consistently applied everywhere**, which becomes one of the project's important security weaknesses.

More on that later.

---

# 17. Database Architecture

The Prisma schema contains the following major entities:

```text
User
RefreshToken

Company
Candidate
CandidateResumeAnalysis

Interview
InterviewContext
Question
Answer

FeedbackReport
InterviewCompetency
JobFitAnalysis
HiringDecision

RankingConfig
RecruiterNote
HiringDecisionOverride
```

---

# 18. Entity Relationship Model

Conceptually:

```text
                    ┌──────────────┐
                    │   Company    │
                    └──────┬───────┘
                           │
                ┌──────────┴─────────┐
                │                    │
                ▼                    ▼
          ┌──────────┐          ┌───────────┐
          │   User   │          │ Candidate │
          └────┬─────┘          └─────┬─────┘
               │                      │
               │                      │
               └──────────┬───────────┘
                          ▼
                    ┌───────────┐
                    │ Interview │
                    └─────┬─────┘
                          │
          ┌───────────────┼──────────────────┐
          │               │                  │
          ▼               ▼                  ▼
      Questions       Answers          InterviewContext
          │
          ▼
       Analysis
          │
     ┌────┼───────────────┐
     │    │               │
     ▼    ▼               ▼
 Feedback JobFit     HiringDecision
 Report   Analysis
```

---

# 19. Interview as the Central Aggregate

The most important domain object is:

```text
Interview
```

Almost everything revolves around it.

An interview contains:

```text
job role
job description
candidate
questions
answers
context
competencies
feedback
job fit
hiring decision
recruiter notes
overrides
```

This makes `Interview` the effective aggregate root of the AI interview domain.

---

# 20. Interview Lifecycle

The state machine is:

```text
PENDING
   │
   │ start
   ▼
ACTIVE
   │
   │ complete
   ▼
COMPLETED
```

There is also:

```text
ABANDONED
```

although the current implementation doesn't appear to have a complete abandonment workflow.

---

# 21. Interview Creation

The important method is:

```text
interviewService.createInterview()
```

The flow is:

```text
Create Interview Request
        │
        ▼
Optional Job Description
        │
        ▼
Job Intelligence
        │
        ├── summary
        ├── required skills
        ├── preferred skills
        ├── competencies
        └── seniority
        │
        ▼
Create Interview
        │
        ▼
Create InterviewContext
        │
        ▼
Optional Resume Context
        │
        ▼
Gemini Question Generation
        │
        ▼
Create Question #1
```

This is an excellent example of domain orchestration.

---

# 22. Job Intelligence

`job-intelligence.service.ts` converts unstructured job descriptions into structured data.

Input:

```text
Software Engineer

We need someone experienced with:
PostgreSQL
Redis
REST APIs
Docker
System Design
...
```

Output:

```json
{
  "summary": "...",
  "requiredSkills": [],
  "preferredSkills": [],
  "competencies": [],
  "seniorityLevel": ""
}
```

This creates a structured job representation.

That representation is then reused throughout the interview.

---

# 23. Why This Is Important

Without Job Intelligence:

```text
Job description
       ↓
LLM directly generates random questions
```

With Job Intelligence:

```text
Job description
       ↓
Structured job profile
       ↓
Required skills
       ↓
Interview strategy
       ↓
Questions
       ↓
Job fit
       ↓
Hiring decision
```

The second architecture is much more deterministic.

---

# 24. Resume Pipeline

Resume processing is divided into three layers.

### Layer 1 — File Upload

Multer handles:

```text
PDF
DOC
DOCX
```

with a 10 MB limit.

Files are stored in:

```text
backend/uploads/resumes/
```

UUID filenames are used.

---

# 25. Resume Extraction

`resume-extractor.service.ts` determines the file extension.

```text
.pdf
  ↓
pdf-parse

.docx
  ↓
mammoth
```

Legacy `.doc` is deliberately rejected for AI parsing.

---

# 26. Resume AI Parsing

After text extraction:

```text
Resume
 ↓
Text extraction
 ↓
Normalization
 ↓
Gemini
 ↓
Structured JSON
```

The structured resume includes:

```text
summary
skills
education
experience
projects
certifications
languages
totalExperienceYears
```

This is stored in:

```text
CandidateResumeAnalysis
```

---

# 27. Resume Context

The parsed resume isn't simply displayed.

It becomes input to interview generation.

Conceptually:

```text
Resume
   ↓
Parsed Resume
   ↓
Resume Context Builder
   ↓
Question Generator
```

This enables personalized interviews.

Example:

Candidate says:

```text
"I worked extensively with Redis."
```

The AI can ask:

```text
How did you handle cache invalidation?
```

instead of asking:

```text
What is Redis?
```

---

# 28. AI Service

`ai.service.ts` is the largest and most important AI integration.

It centralizes Gemini interaction.

Major responsibilities:

```text
generateQuestions()
evaluateAnswer()
summarizeAnswer()
generateFeedbackReport()
generateStructuredJSON()
streamResponse()
```

This is a good architectural decision.

Instead of:

```text
service A → Gemini
service B → Gemini
service C → Gemini
```

the project uses:

```text
            Gemini
              ▲
              │
        ai.service
       ▲     ▲     ▲
       │     │     │
    Resume Job  Interview
```

This creates a single integration boundary.

---

# 29. Prompt Engineering Architecture

Question generation incorporates:

```text
Job Role
Interview Mode
Resume
Job Context
Required Skills
Preferred Skills
Competencies
Existing Topics
Strengths
Weaknesses
Difficulty
Conversation Summary
Last Answer
Interview Strategy
```

This is effectively a context assembly system.

The model isn't simply prompted with:

```text
Generate interview questions.
```

It receives a state representation.

---

# 30. Adaptive Interview Engine

This is one of the most interesting parts of the project.

`interview-intelligence.service.ts` determines the next questioning strategy.

Possible strategies:

```text
FOLLOW_UP
NEW_TOPIC
DEEP_DIVE
WEAKNESS_CHECK
STRENGTH_CHALLENGE
WRAP_UP
```

This creates a state-machine-like interview strategy.

---

# 31. Decision Algorithm

The current logic is approximately:

```text
if question limit reached
    → WRAP_UP

else if required skills remain
    → NEW_TOPIC

else if weaknesses exist
    → WEAKNESS_CHECK

else if average score >= 90
    → STRENGTH_CHALLENGE

else
    → FOLLOW_UP
```

This is a deterministic policy layer sitting in front of the LLM.

That's an important architectural decision.

The LLM is not given unlimited control.

---

# 32. Why Use a Deterministic Strategy Layer?

Pure LLM:

```text
Candidate answer
      ↓
LLM
      ↓
Whatever question it thinks is interesting
```

Problems:

- topic drift
- inconsistent difficulty
- repeated questions
- missed required skills
- unpredictable interview structure

Current architecture:

```text
Candidate state
      ↓
Deterministic strategy engine
      ↓
Target topic
      ↓
Difficulty
      ↓
LLM question generation
```

This is substantially better.

---

# 33. Interview Context

`InterviewContext` acts as the interview's working memory.

It stores:

```text
currentQuestionIndex
topicsCovered
strengths
weaknesses
technologiesCovered
averageScore
currentDifficulty
conversationSummary
lastAnswerSummary
```

This is effectively:

# Short-term memory for the interview agent

---

# 34. Context Update

After every evaluated answer:

```text
Answer
  ↓
Score
  ↓
Topic
  ↓
Update context
```

For example:

```text
score >= 80
    → strength

score < 60
    → weakness
```

Difficulty is also adjusted.

```text
average >= 85
    → HARD

average >= 65
    → MEDIUM

otherwise
    → EASY
```

---

# 35. Competency Engine

Topics are mapped to competencies.

For example:

```text
JWT
 ↓
SECURITY
API_DESIGN
```

Redis:

```text
Redis
 ↓
PERFORMANCE
BACKEND
```

System design:

```text
system architecture
 ↓
SYSTEM_DESIGN
BACKEND
```

Debugging:

```text
debug/error/bug
 ↓
DEBUGGING
PROBLEM_SOLVING
```

This is implemented in:

```text
competency-mapper.ts
```

---

# 36. Competency Scoring

Each competency stores:

```text
averageScore
totalScore
questionCount
evidence
```

The score is updated incrementally:

```text
newTotal = oldTotal + currentScore

newCount = oldCount + 1

newAverage = newTotal / newCount
```

This is mathematically straightforward and efficient.

---

# 37. Answer Analysis

The system doesn't rely exclusively on Gemini.

This is a very important design decision.

It also has deterministic analysis modules:

```text
Speech
Fillers
Readability
Confidence
Completeness
Sentiment
```

---

# 38. Speech Analysis

Speech rate is calculated as:

```text
WPM = words / durationSeconds × 60
```

Classification:

```text
< 80       VERY_SLOW
80–109     SLOW
110–160    NORMAL
161–190    FAST
> 190      VERY_FAST
```

This is simple and explainable.

---

# 39. Filler Analysis

Recognized fillers include:

```text
um
uh
like
actually
basically
literally
you know
kind of
sort of
i mean
```

The system calculates:

```text
total fillers
filler percentage
individual filler frequencies
```

This is deterministic and cheap.

---

# 40. Confidence Analysis

Confidence combines:

```text
certainty words
-
uncertainty words
-
filler usage
+
speech pace
```

For example:

```text
"definitely"
"certainly"
"absolutely"
```

increase confidence.

Words like:

```text
maybe
perhaps
probably
guess
unsure
```

decrease it.

This is a heuristic rather than a scientifically validated confidence model.

That distinction matters in an interview.

Do **not** claim this system performs psychological confidence detection.

It performs:

> lexical and speech-pattern-based confidence estimation.

---

# 41. Sentiment Analysis

The system uses a manually maintained positive/negative word list.

Example:

```text
positive:
excellent
successful
efficient
confident
resolved

negative:
failed
difficult
problem
error
confused
```

The algorithm counts occurrences.

Again, this is:

```text
lexicon-based sentiment
```

not transformer sentiment classification.

---

# 42. Readability Analysis

The system computes:

```text
average sentence length
average word length
unique word ratio
```

Then uses a heuristic score.

The source itself acknowledges:

```text
Simple readability heuristic.
We'll replace this later with
a better communication model.
```

This is a useful example of engineering honesty in the codebase.

---

# 43. Completeness Analysis

Expected key points are generated with questions.

Example:

```text
Question:
Why did you choose PostgreSQL?

Expected key points:
- ACID
- relational structure
- transactions
- indexing
```

The current implementation checks:

```typescript
answer.includes(normalizedKeyPoint)
```

Therefore completeness is essentially:

```text
covered key points
------------------ × 100
total key points
```

This is computationally cheap.

But it has a major limitation:

Semantic equivalence is not recognized.

For example:

```text
"transactional consistency"
```

may fail to match:

```text
"ACID"
```

even though conceptually they may be equivalent.

---

# 44. Voice Architecture

Voice interaction follows:

```text
Browser
 │
 │ MediaRecorder
 ▼
Audio Blob
 │
 │ ArrayBuffer
 ▼
Base64
 │
 │ WebSocket
 ▼
Node WebSocket Handler
 │
 ▼
Deepgram
 │
 ▼
Transcript
 │
 ▼
Interview Service
 │
 ▼
AI Evaluation
 │
 ▼
WebSocket
 │
 ▼
Browser
```

---

# 45. Why WebSockets?

Normal HTTP would require:

```text
POST answer
GET status
GET transcript
GET evaluation
GET next question
```

WebSockets provide:

```text
persistent connection
        +
server push
```

This is ideal for an interactive interview.

---

# 46. WebSocket Session State

Redis stores:

```json
{
  "interviewId": "...",
  "userId": "...",
  "currentQuestionIndex": 0,
  "questions": [],
  "startedAt": 123456,
  "isProcessing": false
}
```

The `isProcessing` flag is effectively a concurrency guard.

It prevents:

```text
Answer A
Answer B
Answer C
```

from being submitted simultaneously.

---

# 47. Text Mode

The interview also supports:

```text
Type mode
```

This is useful when:

- microphone isn't available
- browser permissions fail
- user wants deterministic testing
- development is performed without audio hardware

It is an excellent development fallback.

---

# 48. Text-to-Speech

Supported providers include:

```text
gTTS
Google Cloud
ElevenLabs
```

The provider is selected using:

```text
TTS_PROVIDER
```

There is also fallback logic.

For example:

```text
ElevenLabs
   ↓ failure
gTTS
```

This is a form of graceful degradation.

---

# 49. Resume Storage Architecture

Currently:

```text
Browser
 ↓
Multer
 ↓
Local filesystem
 ↓
backend/uploads/resumes
```

The database stores metadata:

```text
filename
path
mime type
size
upload timestamp
```

This is acceptable for development.

It is not ideal for production-scale distributed deployment.

---

# 50. Frontend Architecture

The frontend uses:

```text
Pages
Components
Hooks
Stores
API layer
Proctoring engine
```

This is a sensible React architecture.

---

# 51. React Pages

Major pages:

```text
LoginPage
RegisterPage
DashboardPage
NewInterviewPage
InterviewSessionPage
ResultsPage
```

Flow:

```text
Login
  ↓
Dashboard
  ↓
New Interview
  ↓
Interview Details
  ↓
Interview Session
  ↓
Results
```

---

# 52. Zustand

Two major stores exist:

```text
auth.store.ts
interview.store.ts
```

Authentication state contains:

```text
user
tokens
isAuthenticated
```

Interview state contains:

```text
interviews
currentInterview
voiceSession
```

This avoids prop drilling.

---

# 53. Axios Token Refresh

The Axios client automatically attaches:

```text
Authorization: Bearer [ACCESS_TOKEN]
```

When the backend returns:

```text
401
TOKEN_EXPIRED
```

the client attempts refresh.

Importantly, it prevents multiple simultaneous refreshes.

The mechanism is:

```text
Request A ─┐
Request B ─┼── expired
Request C ─┘

       ↓

one refresh request

       ↓

all requests reuse result
```

This is a strong implementation detail.

---

# 54. Browser Proctoring Architecture

The project includes a client-side proctoring subsystem.

Components:

```text
camera-manager
microphone-manager
screen-manager
permission-manager
media-manager
face-detection-engine
behaviour-engine
risk-engine
```

---

# 55. Proctoring Flow

```text
Interview starts
      │
      ├── Camera
      ├── Microphone
      └── Screen Share
              │
              ▼
        Media Manager
              │
              ▼
       Face Detection
              │
              ▼
       Behaviour Engine
              │
              ▼
         Risk Engine
```

---

# 56. Face Detection

MediaPipe performs local browser-side face detection.

The system detects:

```text
0 faces
1 face
multiple faces
```

Events include:

```text
FACE_LOST
SINGLE_FACE
MULTIPLE_FACES
```

This is a good architectural choice because raw camera frames don't need to be continuously uploaded to the backend simply to detect faces.

---

# 57. Risk Engine

Risk starts at:

```text
0
```

Risk increases when:

```text
face not visible
multiple faces
warnings
```

Example:

```text
face invisible      +30
multiple faces      +40
each warning         +5
```

Risk is capped:

```text
100
```

Levels:

```text
0–19     LOW
20–49    MEDIUM
50–74    HIGH
75–100   CRITICAL
```

---

# 58. Important Proctoring Limitation

This is **not secure server-side proctoring**.

The entire logic executes in the browser.

A malicious candidate could potentially manipulate client-side state.

For high-integrity assessments, proctoring should become:

```text
Client detection
       +
server event ingestion
       +
tamper-resistant telemetry
       +
audit logs
       +
possibly video evidence
```

---

# 59. Full End-to-End Interview Flow

The intended flow is:

```text
                    CREATE INTERVIEW
                          │
                          ▼
                 Analyze Job Description
                          │
                          ▼
                   Parse Resume
                          │
                          ▼
                  Generate Q1
                          │
                          ▼
                 Start Interview
                          │
                          ▼
              Establish WebSocket
                          │
                          ▼
                   Ask Question
                          │
                          ▼
                 Candidate Answer
                          │
             ┌────────────┴────────────┐
             │                         │
           Voice                      Text
             │                         │
             ▼                         │
         Deepgram                      │
             │                         │
             └────────────┬────────────┘
                          ▼
                      Transcript
                          │
                          ▼
                   AI Evaluation
                          │
             ┌────────────┼─────────────┐
             │            │             │
             ▼            ▼             ▼
          Score       Summary       Linguistic
                                      Analysis
             │
             ▼
             Interview Context
             │
             ▼
          Competencies
             │
             ▼
       Interview Intelligence
             │
             ▼
          Next Question
             │
             ▼
           Repeat
             │
             ▼
          Completion
             │
             ▼
       Feedback Report
             │
             ▼
          Job Fit
             │
             ▼
       Hiring Decision
             │
             ▼
       Ranking / Analytics
```

---

# 60. Critical Current Implementation Issue

There is an important architectural mismatch in the current repository.

The intended system supports **dynamic question generation**.

But the WebSocket session currently loads:

```typescript
interview.questions
```

when the session begins.

At interview creation, only **one question** is initially generated:

```text
Question #1
```

After the answer is evaluated, `_generateNextQuestion()` can create Question #2.

However, the WebSocket session's Redis state was initially populated with the original question list.

Therefore:

```text
SESSION_START
     ↓
questions = [Q1]
     ↓
candidate answers Q1
     ↓
nextIndex = 1
     ↓
nextIndex >= questions.length
     ↓
SESSION_COMPLETE
```

The architecture intends:

```text
Q1
 ↓
generate Q2
 ↓
Q2
 ↓
generate Q3
 ↓
Q3
```

but the current session state behaves closer to:

```text
Q1
 ↓
complete
```

This is one of the most important things to understand before claiming that the adaptive interview engine is production-ready.

---

# 61. Another Related Issue: NEXT\_QUESTION

The backend's interview service contains:

```text
sendToInterview(...)
```

and emits:

```text
NEXT_QUESTION
```

But the frontend WebSocket message types do not properly model `NEXT_QUESTION`.

The frontend's `WSServerMessage` union contains:

```text
SESSION_READY
QUESTION
TRANSCRIPT
EVALUATION
SESSION_COMPLETE
ERROR
```

but not:

```text
NEXT_QUESTION
```

Therefore the adaptive-question architecture is incomplete at the protocol level as well.

---

# 62. AI Model Configuration Inconsistency

The README says:

```text
gemini-1.5-flash
```

is the intended model.

However, the actual `ai.service.ts` selects:

```typescript
gemini-2.5-flash
```

and:

```typescript
gemini-2.5-pro
```

The resume AI service also records:

```text
gemini-2.5-flash
```

This means the documentation and implementation are inconsistent.

For an interview, always distinguish:

```text
documented architecture
```

from:

```text
actual implementation
```

The code wins.

---

# 63. Answer Duration Bug

The WebSocket handler calculates audio duration approximately using:

```text
audioBuffer.length / 16000
```

This is not a reliable duration calculation for WebM/Opus audio.

Why?

Because:

```text
bytes != PCM samples
```

The input is compressed WebM audio.

Correct duration should be derived from:

```text
MediaRecorder timestamps
```

or decoded media metadata.

This directly affects:

```text
WPM
speech pace
confidence
communication score
```

Therefore this is not merely a cosmetic issue.

It can affect analytics.

---

# 64. Resume Processing Bottleneck

`triggerResumeAnalysis()` sounds asynchronous but internally performs:

```text
extract
 ↓
Gemini
 ↓
save
```

before returning.

Therefore uploading a resume can potentially wait on an external AI request.

At scale:

```text
100 resume uploads
       ↓
100 Gemini requests
       ↓
request latency
       ↓
backend worker exhaustion
```

A production architecture should move this to a background queue.

---

# 65. Background AI Architecture

The project already uses fire-and-forget patterns such as:

```typescript
void this._evaluateAnswerAsync(...)
```

and:

```typescript
void this._generateReportAsync(...)
```

This improves request latency.

But it is not a true job queue.

If the Node process crashes:

```text
background task disappears
```

There is no durable job record.

---

# 66. Correct Production Architecture

Introduce:

```text
BullMQ
Redis
or another durable queue
```

Then:

```text
HTTP Request
    │
    ▼
Create Answer
    │
    ▼
Queue Evaluation Job
    │
    ▼
HTTP response immediately
```

Worker:

```text
Worker
 │
 ▼
Deepgram/Gemini
 │
 ▼
PostgreSQL
 │
 ▼
WebSocket notification
```

This makes the system resilient.

---

# 67. Feedback Report Pipeline

After completion:

```text
Interview
   │
   ▼
Feedback Report
   │
   ▼
Job Fit
   │
   ▼
Hiring Decision
```

The report generation has a timeout:

```text
60 seconds
```

and a fallback report.

That is good resilience engineering.

---

# 68. Graceful AI Failure

If Gemini fails:

```text
AI report unavailable
```

the system creates a fallback report using average scores.

This demonstrates a good principle:

# AI should enhance the system, not become the single point of failure.

---

# 69. Hiring Decision

The hiring decision AI receives:

```text
Candidate
Resume
Experience
Job role
Interview mode
Overall score
Strengths
Weaknesses
Competencies
Feedback report
```

It outputs:

```text
STRONG_HIRE
HIRE
MAYBE
NO_HIRE
```

plus:

```text
confidence
summary
strengths
concerns
hiring reasons
rejection reasons
suggested role
suggested seniority
```

This is a decision-support system.

It should **not** be presented as an autonomous hiring authority.

---

# 70. Human Override Architecture

The repository correctly recognizes that AI decisions should be overridable.

Recruiters can create:

```text
HiringDecisionOverride
```

with:

```text
previousRecommendation
newRecommendation
reason
recruiterId
```

This establishes:

```text
AI recommendation
       ↓
Human review
       ↓
Human override
```

That is much more appropriate for enterprise hiring.

---

# 71. Critical Security Issue: Override Tenant Isolation

The override service verifies:

```text
interview exists
interview completed
AI decision exists
```

but does not sufficiently establish:

```text
interview belongs to recruiter's company
```

The same concern exists for effective-decision retrieval.

Therefore an attacker who has a valid recruiter account and obtains another interview ID could potentially interact with another tenant's decision data.

The fix should be:

```text
recruiter.companyId
        ==
interview.user.companyId
```

and this check must happen server-side.

---

# 72. Critical Security Issue: Recruiter Notes

`recruiter-note.service.ts` checks ownership of the note for update/delete.

But note creation and retrieval don't fully enforce company ownership of the underlying interview.

The correct rule is:

```text
note.interview.user.companyId
    ==
currentUser.companyId
```

This should be enforced for:

```text
create
read
update
delete
```

---

# 73. Interview Authorization Limitation

The core interview service frequently checks:

```text
interview.userId === userId
```

This works for the original interview creator.

But enterprise recruitment introduces a different model:

```text
Company
  ├── Recruiter A
  ├── Recruiter B
  └── Interviewer C
```

Recruiter B may legitimately need access to an interview created by Recruiter A.

The current ownership model is therefore more like:

```text
user-owned interviews
```

rather than fully:

```text
company-owned interviews
```

This should be redesigned for the enterprise use case.

---

# 74. Dashboard Performance Problem

The dashboard service loads all company interviews:

```text
findMany()
```

and then performs aggregation in Node.js.

Conceptually:

```text
PostgreSQL
    ↓
potentially thousands of rows
    ↓
Node.js
    ↓
filter/map/reduce
```

This is acceptable for a prototype.

At scale, use:

```text
SQL aggregation
```

such as:

```text
COUNT
AVG
GROUP BY
```

instead.

---

# 75. Ranking Complexity

Candidate ranking fetches all completed interviews for a company/job role.

Then:

```text
O(N)
```

calculation occurs in application memory.

For thousands of candidates, this becomes manageable but increasingly expensive.

For millions:

```text
database-side materialization
```

or:

```text
precomputed ranking tables
```

would be more appropriate.

---

# 76. Skill Gap Approximation

The skill-gap system currently treats:

```text
matched skill
```

as:

```text
strong
```

and:

```text
missing skill
```

as:

```text
weak
```

This is not truly measuring proficiency.

For example:

```text
Candidate mentions Redis
```

does not necessarily mean:

```text
Candidate is strong in Redis
```

This should eventually incorporate:

```text
competency score
+
question evidence
+
answer quality
+
skill frequency
```

---

# 77. Ranking Formula

The current ranking formula is:

```text
Ranking Score =
    competencyAverage × competencyWeight
  + jobMatch × jobFitWeight
  + confidence × confidenceWeight
  + recommendationWeight × recommendationWeight
```

Default weights:

```text
Competency       40%
Job Fit          30%
Confidence       20%
Recommendation   10%
```

Recommendation is mapped:

```text
STRONG_HIRE = 100
HIRE        = 85
MAYBE       = 60
NO_HIRE      = 30
```

This is a weighted scoring model.

---

# 78. Important Mathematical Concern

The system mixes:

```text
continuous values
```

and:

```text
LLM-generated categorical judgments
```

For example:

```text
confidence = 92
```

is multiplied directly into the ranking formula.

But AI confidence is not necessarily statistically calibrated.

A model saying:

```text
confidence = 92
```

does not mean:

```text
92% probability that recommendation is correct
```

For a serious hiring system, calibration should be studied.

---

# 79. Database Migration Evolution

The migration history reveals the project's evolution.

The system started relatively simply:

```text
Users
Interviews
Questions
Answers
Feedback
```

Then progressively added:

```text
Answer analysis
Expected key points
Companies
Performance indexes
Candidates
Resume metadata
Resume AI
Candidate ↔ Interview
Interview context
Question limits
Interview memory
Competencies
Job intelligence
Job fit
Hiring decisions
Ranking configuration
Recruiter notes
Human overrides
```

This is extremely useful for understanding the developer's thinking.

The system evolved from:

# AI Interview App

toward:

# Enterprise Recruitment Intelligence Platform

---

# 80. Migration History as Architectural Evidence

The migration sequence tells a story:

```text
Phase 1
Basic interview system

        ↓

Phase 2
Answer intelligence

        ↓

Phase 3
Enterprise company model

        ↓

Phase 4
Candidate management

        ↓

Phase 5
Resume intelligence

        ↓

Phase 6
Adaptive interviews

        ↓

Phase 7
Job intelligence

        ↓

Phase 8
Hiring intelligence

        ↓

Phase 9
Recruiter workflow
```

This is important during an interview because it demonstrates how the architecture expanded organically.

---

# 81. Configuration

Required environment variables include:

```env
DATABASE_URL=[POSTGRESQL_CONNECTION_STRING]

REDIS_URL=[REDIS_CONNECTION_STRING]

JWT_SECRET=[STRONG_ACCESS_TOKEN_SECRET]

JWT_REFRESH_SECRET=[STRONG_REFRESH_TOKEN_SECRET]

GEMINI_API_KEY=[GEMINI_API_KEY]

DEEPGRAM_API_KEY=[DEEPGRAM_API_KEY]
```

Optional TTS:

```env
TTS_PROVIDER=[gtts|google|elevenlabs]

GOOGLE_TTS_API_KEY=[GOOGLE_TTS_API_KEY]

ELEVENLABS_API_KEY=[ELEVENLABS_API_KEY]
```

AWS values exist for future storage integration.

---

# 82. Local Startup

## Step 1 — PostgreSQL

Example:

```bash
docker run --name interview-postgres \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=[POSTGRES_PASSWORD] \
  -e POSTGRES_DB=ai_interview_db \
  -p 5432:5432 \
  -d postgres:15
```

## Step 2 — Redis

```bash
docker run --name interview-redis \
  -p 6379:6379 \
  -d redis:7
```

## Step 3 — Backend

```bash
cd backend

cp .env.example .env

npm install

npx prisma generate

npx prisma migrate dev
```

Then:

```bash
npm run dev
```

---

# 83. Frontend Startup

```bash
cd frontend

npm install

npm run dev
```

Expected:

```text
http://localhost:5173
```

Backend:

```text
http://localhost:3001
```

WebSocket:

```text
ws://localhost:3001/ws/interview
```

---

# 84. Production Architecture Recommended

The current architecture is suitable for development and early-stage deployment.

For production:

```text
                    Internet
                       │
                       ▼
                 Load Balancer
                       │
              ┌────────┴────────┐
              ▼                 ▼
        API Server A       API Server B
              │                 │
              └────────┬────────┘
                       │
          ┌────────────┼─────────────┐
          ▼            ▼             ▼
      PostgreSQL      Redis       Object Store
                                      │
                                      ▼
                                    S3
                                     
                       │
                       ▼
                  Job Queue
                       │
             ┌─────────┼─────────┐
             ▼         ▼         ▼
          AI Worker AI Worker AI Worker
```

---

# 85. Production Storage

Replace:

```text
local uploads/
```

with:

```text
Amazon S3
Cloudflare R2
Google Cloud Storage
Azure Blob Storage
```

Database stores:

```text
object key
```

rather than local filesystem path.

---

# 86. Production AI Workers

Use a queue:

```text
Redis
+
BullMQ
```

Jobs:

```text
RESUME_PARSE
ANSWER_EVALUATION
ANSWER_SUMMARY
REPORT_GENERATION
JOB_ANALYSIS
HIRING_DECISION
```

This provides:

```text
retry
backoff
dead-letter handling
job visibility
durability
concurrency limits
```

---

# 87. AI Rate Limiting

AI providers have quotas.

Therefore the system should introduce:

```text
AI Gateway
```

with:

```text
rate limiting
retry
exponential backoff
provider fallback
circuit breaker
token accounting
request tracing
```

Example:

```text
Application
     ↓
AI Gateway
     ↓
Gemini
```

rather than allowing every service to directly consume the provider.

---

# 88. AI Prompt Versioning

Currently prompts are embedded directly inside TypeScript files.

For production:

```text
Prompt Registry
```

should track:

```text
prompt name
version
model
temperature
schema
createdAt
```

Example:

```text
answer-evaluation-v3
question-generation-v8
hiring-decision-v2
```

This enables reproducibility.

---

# 89. Structured AI Output

The project already uses JSON output.

That is good.

But production should additionally validate every LLM response with Zod.

Instead of trusting:

```text
Gemini → JSON.parse()
```

use:

```text
Gemini
 ↓
JSON
 ↓
Zod schema
 ↓
validated domain object
```

This is essential.

---

# 90. AI Hallucination Risk

The project has some safeguards such as:

```text
Do NOT invent information.
Use only information present.
Return only JSON.
```

But prompts alone are not sufficient.

Production architecture should add:

```text
schema validation
+
grounding
+
evidence references
+
confidence thresholds
+
human review
```

---

# 91. Hiring Bias Risk

This system makes hiring recommendations.

That makes fairness extremely important.

Current inputs include:

```text
resume
communication
confidence
sentiment
experience
```

Some of these may unintentionally correlate with:

```text
accent
language fluency
disability
culture
communication style
```

Therefore:

# These scores should never automatically determine hiring outcomes.

The safest architecture is:

```text
AI recommendation
       ↓
Evidence
       ↓
Human review
       ↓
Final decision
```

The existing override model is a step in the right direction.

---

# 92. Proctoring Privacy Risk

The browser requests:

```text
camera
microphone
screen
```

This introduces privacy concerns.

Production implementation should include:

```text
explicit consent
privacy policy
data retention policy
regional compliance
audit logs
secure media handling
```

Also clearly communicate:

```text
what is captured
why it is captured
how long it is retained
who can access it
```

---

# 93. API Validation

Zod is used for many endpoints.

Good examples include:

```text
auth
candidate
company
company user
interview
```

But not every newer endpoint has equally strong validation.

The following should be validated explicitly:

```text
ranking query
comparison body
override body
recruiter notes
analytics query
```

Do not rely on TypeScript for runtime input validation.

---

# 94. API Rate Limiting

`express-rate-limit` exists in dependencies.

However, the examined route architecture does not show a comprehensive application-level rate-limiting strategy.

Recommended:

```text
/login
/register
/refresh
/interviews
/AI endpoints
/resume upload
```

should have targeted limits.

Especially:

```text
AI-consuming operations
```

because they cost money and consume quota.

---

# 95. File Upload Security

Current upload restrictions include:

```text
PDF
DOC
DOCX
10 MB
```

This is good but insufficient for production.

Add:

```text
magic-byte validation
virus scanning
content inspection
safe filename handling
object-storage isolation
download authorization
signed URLs
retention policies
```

Checking MIME type alone can be spoofed.

---

# 96. Error Handling

The project has:

```text
AppError
Errors helper
errorMiddleware
```

This is good.

The architecture is:

```text
Service
 ↓
throw AppError
 ↓
Controller next(error)
 ↓
errorMiddleware
 ↓
HTTP response
```

This centralizes error formatting.

---

# 97. Logging

Morgan provides HTTP logs.

The backend also uses:

```text
console.error
console.warn
console.log
```

For production, replace this with structured logging:

```text
Pino
Winston
OpenTelemetry
```

Log fields should include:

```text
requestId
userId
companyId
interviewId
jobId
AI request ID
latency
error code
```

---

# 98. Observability

A production system should measure:

```text
API latency
WebSocket connections
WebSocket disconnects
Gemini latency
Gemini errors
Deepgram latency
resume parsing duration
question generation latency
answer evaluation latency
queue depth
Redis latency
database latency
```

This project currently has limited observability.

---

# 99. Concurrency Problems

Several areas need careful concurrency handling.

## Answer submission

Potential race:

```text
Answer submitted twice
```

The `isProcessing` Redis flag helps, but it is not a full distributed lock.

Better:

```text
Redis SET NX
```

with lock expiry.

Or use database state transitions.

---

# 100. Duplicate AI Reports

`FeedbackReport.interviewId` is unique.

That protects the database from multiple successful reports.

However, multiple workers could still perform duplicate expensive AI calls before one insert succeeds.

A distributed job-idempotency strategy would be better.

---

# 101. Database Transactions

Company creation correctly uses:

```typescript
prisma.$transaction(...)
```

because:

```text
Company
+
Admin User
```

must be created atomically.

This is exactly where transactions should be used.

---

# 102. Candidate Creation

Candidate duplicate detection uses:

```text
findFirst
```

followed by:

```text
create
```

This creates a race:

```text
Request A → check → none
Request B → check → none

Request A → create
Request B → create
```

For true uniqueness, use a database constraint such as:

```text
UNIQUE(companyId, email)
```

where appropriate.

---

# 103. Data Modeling Improvement

Some business decisions are stored as strings:

```text
hiringDecision.recommendation
hiringDecision.suggestedRole
```

Recommendation should ideally be a PostgreSQL enum.

This prevents:

```text
STRONG_HIRE
strong-hire
Strong Hire
HIRE
```

from becoming inconsistent.

---

# 104. JSONB Usage

JSONB is used for:

```text
requiredSkills
preferredSkills
competencies
expectedKeyPoints
analysis
evidence
matchedSkills
missingSkills
strengths
concerns
```

This provides flexibility.

But too much JSONB can make querying difficult.

Rule of thumb:

Use relational columns for:

```text
important searchable business state
```

Use JSONB for:

```text
semi-structured evidence
```

The project currently mixes both.

---

# 105. Recommended Domain Boundaries

As the project grows, split the backend conceptually into:

```text
Identity
├── Authentication
├── Users
└── Authorization

Organization
├── Companies
├── Candidates
└── Recruiters

Interview
├── Interview lifecycle
├── Questions
├── Answers
├── Context
└── Competencies

AI
├── Resume AI
├── Job AI
├── Interview AI
└── Hiring AI

Media
├── Voice
├── STT
├── TTS
└── Storage

Analytics
├── Ranking
├── Comparison
├── Dashboard
└── Skill gaps

Compliance
├── Proctoring
├── Audit logs
├── Overrides
└── Privacy
```

---

# 106. Should This Become Microservices?

Not immediately.

The current codebase is better described as a:

# Modular monolith

That is actually a good thing.

Do not prematurely split it into:

```text
10 microservices
```

The better evolution is:

```text
Modular monolith
      ↓
Background workers
      ↓
Scale bottleneck modules
      ↓
Extract services only when necessary
```

---

# 107. Which Components Would Eventually Become Services?

Potential candidates:

```text
AI Worker
Voice Processing Service
Resume Processing Service
Notification Service
Analytics Service
Proctoring Service
```

But keep:

```text
Auth
Candidates
Interviews
Recruiter workflow
```

together initially.

---

# 108. Testing Strategy

The repository has limited visible automated test coverage.

A production-grade system needs:

## Unit tests

Test:

```text
scoring
topic extraction
competency mapping
risk engine
validators
token logic
```

## Integration tests

Test:

```text
database
auth
candidate CRUD
interview lifecycle
resume processing
```

## WebSocket tests

Test:

```text
SESSION_START
AUDIO_CHUNK
TEXT_ANSWER
SESSION_END
reconnect
timeout
duplicate answer
```

## AI contract tests

Mock Gemini and validate:

```text
valid JSON
invalid JSON
missing field
hallucinated field
timeout
quota error
```

---

# 109. Testing the Adaptive Interview

This deserves dedicated tests.

Given:

```text
requiredSkills = [Redis, PostgreSQL]
coveredSkills = [Redis]
```

expected:

```text
NEW_TOPIC
target = PostgreSQL
```

Given:

```text
weaknesses = [Redis]
```

expected:

```text
WEAKNESS_CHECK
```

Given:

```text
averageScore = 95
strengths = [System Design]
```

expected:

```text
STRENGTH_CHALLENGE
difficulty = HARD
```

This is deterministic logic and therefore highly testable.

---

# 110. Recommended Refactoring Priority

## P0 — Critical

Fix:

```text
dynamic WebSocket question lifecycle
```

Fix:

```text
tenant isolation
```

Fix:

```text
AI model/documentation mismatch
```

Fix:

```text
audio duration calculation
```

---

# 111. P1 — High Priority

Introduce:

```text
background job queue
```

Move:

```text
resume parsing
answer evaluation
report generation
hiring decision
```

to workers.

Also add:

```text
structured logging
rate limiting
runtime AI schema validation
```

---

# 112. P2 — Medium Priority

Improve:

```text
dashboard SQL aggregation
candidate ranking
skill-gap accuracy
database constraints
```

Add:

```text
audit logs
```

for:

```text
AI decisions
human overrides
candidate modifications
```

---

# 113. P3 — Long Term

Add:

```text
S3
OpenTelemetry
Redis distributed locks
AI gateway
prompt versioning
model evaluation
bias monitoring
feature flags
event-driven architecture
```

---

# 114. Ideal Future Architecture

```text
                         ┌─────────────────────┐
                         │      Frontend       │
                         │ React + TypeScript  │
                         └──────────┬──────────┘
                                    │
                       ┌────────────┴────────────┐
                       │                         │
                     HTTPS                    WebSocket
                       │                         │
                       ▼                         ▼
               ┌──────────────┐          ┌──────────────┐
               │ API Gateway  │          │ WS Gateway   │
               └──────┬───────┘          └──────┬───────┘
                      │                         │
                      └────────────┬────────────┘
                                   ▼
                        ┌────────────────────┐
                        │ Modular Application │
                        │                    │
                        │ Auth               │
                        │ Company            │
                        │ Candidate          │
                        │ Interview          │
                        │ Recruitment        │
                        └─────────┬──────────┘
                                  │
                 ┌────────────────┼────────────────┐
                 │                │                │
                 ▼                ▼                ▼
           PostgreSQL          Redis           Object Store
                                               S3/R2
                 │
                 │
                 ▼
             Job Queue
                 │
        ┌────────┼────────────┐
        ▼        ▼            ▼
    Resume     Interview      Hiring
     AI          AI             AI
    Worker      Worker         Worker
        │        │              │
        └────────┼──────────────┘
                 ▼
             AI Gateway
                 │
          ┌──────┴─────────┐
          ▼                ▼
       Gemini           Deepgram
```

---

# 115. How to Explain This Project in an Interview

A strong explanation would be:

> "This is a modular monolithic AI recruitment platform. The frontend is React and TypeScript, while the backend is Express and TypeScript with PostgreSQL through Prisma and Redis for ephemeral interview session state. The core domain is the interview aggregate, which connects candidates, questions, answers, competencies, job-fit analysis, feedback reports and hiring decisions.
>
> The interesting part is the adaptive interview engine. Job descriptions are first converted into structured job intelligence, resumes are parsed into structured candidate context, and a deterministic interview intelligence layer decides whether the next question should cover a missing skill, investigate a weakness, challenge a strength, or naturally follow the previous topic. Gemini is then responsible for generating the actual question.
>
> For voice interviews, the browser records audio using MediaRecorder, sends it over WebSockets, Deepgram converts it into text, and the backend evaluates the transcript. Redis stores the live interview session state, while PostgreSQL stores durable results.
>
> After completion, the system generates a feedback report, job-fit analysis and hiring recommendation. Recruiters can compare and rank candidates and can override AI recommendations with an auditable reason.
>
> The current implementation is best described as a strong prototype or modular monolith rather than a production-scale distributed system. The major next steps would be fixing the dynamic WebSocket question lifecycle, strengthening tenant isolation, moving AI processing into durable background workers, replacing local file storage with object storage, and adding comprehensive observability and auditability."

---

# 116. Questions an Interviewer Could Ask

## Architecture

### "Why PostgreSQL?"

Answer:

> Because the domain is highly relational and requires transactional consistency across companies, users, candidates, interviews, questions, answers and hiring decisions. PostgreSQL also gives us JSONB for flexible AI-generated structures.

---

### "Why Redis?"

Answer:

> Redis stores ephemeral interview session state such as the current question index and processing status. PostgreSQL remains the durable source of truth.

---

### "Why WebSockets?"

Answer:

> The interview is interactive and requires server-to-client events such as transcripts, evaluations and next questions. WebSockets avoid constant polling and provide low-latency bidirectional communication.

---

### "Why not microservices?"

Answer:

> The current system is better suited to a modular monolith. Splitting it too early would introduce operational complexity. I would first isolate background AI workers and extract services only where independent scaling or reliability requirements justify it.

---

# 117. AI Questions

### "Why not let Gemini decide everything?"

Answer:

> Because LLMs are probabilistic. I want deterministic business constraints around them. The interview intelligence service controls required skill coverage, weakness handling, strength challenges and difficulty. Gemini generates the natural-language question within those constraints.

---

### "How do you prevent hallucinations?"

Answer:

> The prompts instruct the model not to invent information and return structured JSON, but prompt constraints alone aren't sufficient. A production system should validate the output with schemas, maintain evidence, version prompts, and apply human review to hiring decisions.

---

# 118. Database Questions

### "Why JSONB?"

Answer:

> AI-generated evidence has semi-structured data. JSONB lets us evolve the schema without creating a relational column for every possible AI output. However, important searchable business attributes should remain relational.

---

# 119. Security Questions

### "How does authentication work?"

Answer:

```text
password
 ↓
bcrypt
 ↓
JWT access token
 +
persistent refresh token
```

The access token is short-lived and the refresh token can be revoked server-side.

---

### "How is multi-tenancy enforced?"

Answer:

> Company membership is attached to users and candidates. Tenant-scoped queries should always constrain data using the authenticated user's company ID.

Then mention honestly:

> The current implementation does this inconsistently for some recruiter workflow endpoints, so tenant isolation is an area I would strengthen before production.

That is a much stronger interview answer than pretending the implementation is perfect.

---

# 120. Important Engineering Lessons From This Codebase

This project teaches several senior-level concepts.

## Lesson 1

# AI systems should be controlled by deterministic business logic.

Not:

```text
LLM controls everything
```

but:

```text
Business policy
      ↓
LLM
```

---

## Lesson 2

# Persistent state and session state are different.

PostgreSQL:

```text
durable truth
```

Redis:

```text
temporary state
```

---

## Lesson 3

# Async does not automatically mean reliable.

This:

```typescript
void someAsyncFunction()
```

reduces latency but does not provide:

```text
durability
retry
dead-letter queues
```

A job queue is required.

---

## Lesson 4

# Multi-tenancy is a security property.

Every query must ask:

```text
Does this object belong to the authenticated tenant?
```

not merely:

```text
Does this object exist?
```

---

## Lesson 5

# AI output is untrusted input.

Even if Gemini returns:

```json
{
  "score": 95
}
```

the application should validate it.

---

# 121. Recommended Learning Order

If your goal is to **master this codebase rather than merely run it**, study it in this order:

## Phase 1 — Foundation

Learn:

```text
React
TypeScript
Express
HTTP
REST
JWT
PostgreSQL
Prisma
Redis
```

---

## Phase 2 — Backend

Read:

```text
server.ts
app.ts

environment.ts
database.ts
redis.ts

auth.middleware.ts
authorize.middleware.ts
error.middleware.ts
```

Then:

```text
routes
 ↓
controllers
 ↓
services
```

---

## Phase 3 — Database

Study:

```text
schema.prisma
```

Then study migrations chronologically.

Do not skip migrations.

They explain how the architecture evolved.

---

# 122. Phase 4 — Interview Engine

Study in this order:

```text
interview.service.ts
        ↓
ai.service.ts
        ↓
interview-context.service.ts
        ↓
interview-intelligence.service.ts
        ↓
interview-competency.service.ts
```

This is the heart of the application.

---

# 123. Phase 5 — Voice

Study:

```text
useAudioRecorder.ts
        ↓
useVoiceSession.ts
        ↓
interview.handler.ts
        ↓
voice.service.ts
        ↓
Deepgram
```

Understand:

```text
MediaRecorder
Blob
ArrayBuffer
Base64
WebSocket
STT
Transcript
```

---

# 124. Phase 6 — Recruitment Intelligence

Study:

```text
resume-extractor.service.ts
resume-ai.service.ts
job-intelligence.service.ts
job-fit.service.ts
hiring-decision.service.ts
candidate-ranking.service.ts
candidate-comparison.service.ts
skill-gap.service.ts
```

At this point you understand the full business domain.

---

# 125. Phase 7 — Proctoring

Finally study:

```text
media-manager
camera-manager
microphone-manager
screen-manager
face-detection-engine
behaviour-engine
risk-engine
```

Understand the difference between:

```text
media capture
face detection
behaviour interpretation
risk calculation
```

These are four separate responsibilities.

---

# 126. Final Architectural Assessment

## Overall Design

**Strong prototype / advanced modular-monolith architecture.**

### Strengths

- Clear controller/service separation
- TypeScript throughout
- PostgreSQL relational model
- Prisma type safety
- Redis session state
- JWT authentication
- Role-based authorization
- AI integration abstraction
- Structured AI outputs
- Adaptive interview strategy
- Deterministic analysis alongside AI
- Resume intelligence
- Job intelligence
- Hiring intelligence
- Human override capability
- Browser-side face detection
- Graceful AI fallbacks

### Weaknesses

- Dynamic WebSocket interview lifecycle is incomplete
- AI model documentation conflicts with implementation
- Audio duration calculation is unreliable
- AI work isn't backed by a durable queue
- Tenant isolation isn't consistently enforced
- Recruiter workflow authorization needs strengthening
- Dashboard aggregation is application-side
- Local filesystem storage isn't production-ready
- Proctoring is client-trusted
- AI confidence isn't calibrated
- Skill-gap analysis is approximate
- Limited visible automated testing
- Limited structured observability
- Several newer APIs lack robust runtime validation

---

# 127. The Most Important Mental Model

If you remember only one diagram from this entire project, remember this:

```text
                   ┌─────────────────┐
                   │      COMPANY    │
                   └────────┬────────┘
                            │
                       Candidates
                            │
                            ▼
                    ┌───────────────┐
                    │   INTERVIEW   │
                    └───────┬───────┘
                            │
             ┌──────────────┼──────────────┐
             │              │              │
             ▼              ▼              ▼
           Resume          Job          Interview
           Context       Context        Context
             │              │              │
             └──────────────┼──────────────┘
                            ▼
                     AI Question Engine
                            │
                            ▼
                         Question
                            │
                            ▼
                          Answer
                            │
                 ┌──────────┴─────────┐
                 │                    │
                 ▼                    ▼
              Gemini            Deterministic
             Evaluation            Analysis
                 │                    │
                 └──────────┬─────────┘
                            ▼
                       Competencies
                            │
                            ▼
                    Interview Intelligence
                            │
                            ▼
                      Next Question
                            │
                         repeat
                            │
                            ▼
                       Final Report
                            │
              ┌─────────────┼──────────────┐
              ▼             ▼              ▼
           Job Fit      Hiring Decision   Ranking
              │             │              │
              └─────────────┼──────────────┘
                            ▼
                    Recruiter Decision
                            │
                            ▼
                     Human Override
```

That is the architecture.

---

# 128. Bottom Line

The project demonstrates a sophisticated progression from a basic AI interview application into an **enterprise recruitment intelligence platform**.

The strongest architectural idea is the combination of:

```text
Structured data
+
deterministic algorithms
+
LLM reasoning
+
real-time communication
+
human oversight
```

rather than treating the LLM as the entire application.

For senior-level mastery, however, the most valuable exercise is not simply memorizing the existing implementation.

You should be able to answer:

```text
Why PostgreSQL?
Why Redis?
Why WebSockets?
Why JWT?
Why Prisma?
Why a modular monolith?
Why deterministic interview strategy?
Why use AI only after policy selection?
Why store interview context?
Why use JSONB?
Why background processing?
Why human overrides?
Where does the system break under load?
Where can a tenant escape isolation?
What happens if Gemini fails?
What happens if Redis fails?
What happens if the WebSocket disconnects?
What happens if the process crashes during AI evaluation?
What happens if two workers evaluate the same answer?
What happens if 10,000 candidates upload resumes?
What happens if 100,000 interviews are ranked?
How do we prove an AI hiring decision is auditable?
```

Those are the questions that move your understanding from **"I know this project"** to **"I can architect and defend this project as a senior engineer."**

## Priority Roadmap

```text
CURRENT
  │
  ▼
Fix WebSocket dynamic questions
  │
  ▼
Fix tenant isolation
  │
  ▼
Add durable AI job queue
  │
  ▼
Move files → object storage
  │
  ▼
Add AI schema validation
  │
  ▼
Add structured observability
  │
  ▼
Add audit/event logging
  │
  ▼
Improve scoring/calibration
  │
  ▼
Harden proctoring
  │
  ▼
Load test
  │
  ▼
Extract independently scalable workers/services
```

**Final assessment:** this is a strong learning codebase for understanding **AI application architecture, real-time systems, relational modeling, multi-tenancy, LLM orchestration, speech processing, adaptive agents, and enterprise recruitment workflows**. The most important caveat is that several parts are still prototype-grade and should not be described as production-hardened without addressing the issues above.
