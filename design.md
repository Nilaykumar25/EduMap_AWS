# Design Document - EduMap Platform

## 1. Executive Summary

**Team**: Serendipity  
**Team Leader**: Nilay Kumar  
**Platform**: EduMap - AI-Powered Exam Intelligence Platform

EduMap is the first predictive exam intelligence platform built specifically for Indian universities. It transforms exam preparation from exhaustive content consumption to strategic, data-driven mastery by combining PYQ pattern analysis, AI-powered pedagogical feedback, and real-time readiness tracking.

### 1.1 Core Innovation

Unlike generic learning platforms, EduMap provides:
- **Predictive Intelligence**: Analyze years of PYQs to identify 8-10 topics yielding 70% marks
- **University-Specific**: Tailored to each university's exam patterns and syllabus
- **Semantic Grading**: Rubric-based evaluation with specific feedback (not just "wrong")
- **Strategic Focus**: "Study what matters most" vs "study everything"
- **Accessible Scale**: Scalable to 40 million students at <₹10/user/month

### 1.2 Design Principles

- **Data-Driven Prioritization**: Every recommendation backed by PYQ analysis
- **Pedagogical Rigor**: AI feedback matches academic evaluation standards
- **Localization-First**: Built for Indian universities, languages, and context
- **Measurable Progress**: Real-time readiness scores reduce exam anxiety
- **Serverless Architecture**: AWS Lambda + AppSync for infinite scale
- **Cost-Optimized**: Target ₹3,900-9,800/month for 1000+ users


## 2. System Architecture

### 2.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                     Indian College Students                          │
│              (Web: React 18 | Mobile: React Native)                 │
└─────────────────────────────────────────────────────────────────────┘
                                  │
                                  │ HTTPS
                                  ↓
┌─────────────────────────────────────────────────────────────────────┐
│              Amazon CloudFront (India-Optimized CDN)                 │
│  - ap-south-1 (Mumbai) edge locations                               │
│  - SSL/TLS termination                                              │
│  - DDoS protection (AWS Shield)                                     │
└─────────────────────────────────────────────────────────────────────┘
                    │                              │
        ┌───────────┴──────────┐      ┌──────────┴──────────┐
        ↓                      ↓      ↓                      ↓
┌──────────────┐      ┌──────────────────────┐      ┌──────────────┐
│ AWS Amplify  │      │   AWS AppSync        │      │   S3 Bucket  │
│ (Frontend)   │      │   (GraphQL API)      │      │   (Assets)   │
└──────────────┘      └──────────────────────┘      └──────────────┘
                                  │
                                  │ Real-Time Subscriptions
                                  ↓
                      ┌───────────────────────┐
                      │   AWS Lambda          │
                      │   (Python 3.11)       │
                      │   - Answer Grading    │
                      │   - Question Gen      │
                      │   - Material Process  │
                      └───────────────────────┘
                                  │
                ┌─────────────────┼─────────────────┐
                │                 │                 │
                ↓                 ↓                 ↓
    ┌──────────────────┐  ┌──────────────┐  ┌──────────────┐
    │  AWS Cognito     │  │  DynamoDB    │  │  Amazon S3   │
    │  (Auth)          │  │  (Data)      │  │  (Storage)   │
    └──────────────────┘  └──────────────┘  └──────────────┘
                │                 │                 │
                └─────────────────┼─────────────────┘
                                  │
                ┌─────────────────┼─────────────────┐
                │                 │                 │
                ↓                 ↓                 ↓
    ┌──────────────────┐  ┌──────────────┐  ┌──────────────┐
    │  AWS Bedrock     │  │  Bedrock KB  │  │  Kendra      │
    │  (Claude 3.5)    │  │  (RAG)       │  │  (Search)    │
    └──────────────────┘  └──────────────┘  └──────────────┘
                │                 │                 │
                └─────────────────┼─────────────────┘
                                  │
                                  ↓
                      ┌───────────────────────┐
                      │   AWS Textract        │
                      │   (OCR for Notes)     │
                      └───────────────────────┘
```

### 2.2 AI Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Student Interaction                           │
│  - Ask Question | Submit Answer | Upload Material | Request Revision│
└─────────────────────────────────────────────────────────────────────┘
                                  │
                                  ↓
┌─────────────────────────────────────────────────────────────────────┐
│                         AWS AppSync (GraphQL)                        │
│  - Real-time subscriptions for AI streaming                         │
│  - Mutation for user actions                                        │
│  - Query for data retrieval                                         │
└─────────────────────────────────────────────────────────────────────┘
                                  │
                                  ↓
┌─────────────────────────────────────────────────────────────────────┐
│                         AWS Lambda Functions                         │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  Answer Grading Pipeline:                                   │   │
│  │  1. Receive answer → 2. Load rubric → 3. Invoke Bedrock    │   │
│  │  4. Parse feedback → 5. Save to DynamoDB → 6. Return score │   │
│  └─────────────────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  AI Tutor Pipeline:                                         │   │
│  │  1. Receive query → 2. Search Kendra (PYQ) → 3. Query KB   │   │
│  │  4. Invoke Bedrock with context → 5. Stream response       │   │
│  └─────────────────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  Material Processing Pipeline:                              │   │
│  │  1. Upload to S3 → 2. Textract OCR → 3. Extract concepts   │   │
│  │  4. Index in Kendra → 5. Add to Knowledge Base             │   │
│  └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
                                  │
                ┌─────────────────┼─────────────────┐
                │                 │                 │
                ↓                 ↓                 ↓
    ┌──────────────────┐  ┌──────────────┐  ┌──────────────┐
    │  Bedrock KB      │  │  Kendra      │  │  Bedrock     │
    │  (RAG Context)   │  │  (PYQ Search)│  │  (Claude 3.5)│
    │                  │  │              │  │              │
    │  - User Materials│  │  - PYQ DB    │  │  - Grading   │
    │  - Syllabus Docs │  │  - Syllabus  │  │  - Tutoring  │
    │  - PYQ Archive   │  │  - Materials │  │  - Generation│
    └──────────────────┘  └──────────────┘  └──────────────┘
                                  │
                                  ↓
                      ┌───────────────────────┐
                      │   Response to Student │
                      │   - Graded Answer     │
                      │   - AI Explanation    │
                      │   - Generated Content │
                      └───────────────────────┘
```

### 2.3 Data Flow: Answer Evaluation

```
Student submits answer
    ↓
AppSync Mutation (submitAnswer)
    ↓
Lambda: answer_grading_handler
    ↓
1. Fetch question + rubric from DynamoDB
    ↓
2. Prepare grading prompt:
   "Question: {question}
    Student Answer: {answer}
    Rubric: {rubric_json}
    Evaluate semantically and provide:
    - Score per criterion
    - Specific missing components
    - Strengths and improvements"
    ↓
3. Invoke Bedrock (Claude 3.5 Sonnet)
   Model: anthropic.claude-3-5-sonnet-20241022-v2:0
   Temperature: 0.3 (for consistency)
    ↓
4. Parse JSON response:
   {
     "total_score": 85,
     "criteria_scores": [...],
     "missing_components": ["time complexity analysis"],
     "strengths": [...],
     "improvements": [...]
   }
    ↓
5. Save grading result to DynamoDB
   PK: USER#{uid}
   SK: GRADE#{timestamp}#{question_id}
    ↓
6. Update user analytics:
   - Topic accuracy
   - Readiness score
   - Skill assessment
    ↓
7. Publish to AppSync subscription
    ↓
Student receives real-time feedback
```

### 2.4 Data Flow: AI Tutor with RAG

```
Student asks question
    ↓
AppSync Mutation (askTutor)
    ↓
Lambda: ai_tutor_handler
    ↓
1. Search Kendra for relevant PYQs
   Query: student's question
   Filters: university, subject, topic
    ↓
2. Retrieve top 5 PYQ results with excerpts
    ↓
3. Query Bedrock Knowledge Base
   - Searches user's uploaded materials
   - Searches syllabus documents
   - Returns relevant context chunks
    ↓
4. Prepare enhanced prompt:
   "You are an expert tutor for {university} {subject}.
    
    Student Question: {question}
    
    Relevant PYQs:
    {pyq_excerpts}
    
    Relevant Study Materials:
    {kb_context}
    
    Provide a detailed, step-by-step explanation.
    Cite sources using [PYQ-2023] or [Material: Notes.pdf]"
    ↓
5. Invoke Bedrock with streaming
   Model: Claude 3.5 Sonnet
   Stream: true
    ↓
6. Stream response chunks via AppSync
    ↓
7. Save complete conversation to DynamoDB
   PK: USER#{uid}
   SK: CHAT#{timestamp}#{session_id}
    ↓
Student sees response streaming in real-time
```


## 3. Technology Stack

### 3.1 Core AI Services

| Service | Purpose | Cost (Monthly) | Key Features |
|---------|---------|----------------|--------------|
| **Amazon Bedrock (Claude 3.5 Sonnet)** | AI tutor, answer grading, content generation | $20-50 | Rubric-based grading, multilingual support, 200K context window |
| **Bedrock Knowledge Bases** | RAG pipeline for contextual responses | $15-30 | Automated indexing, vector search, source citations |
| **Amazon Kendra** | Intelligent PYQ search | $0 (Free tier) | Semantic search, faceted filtering, relevance ranking |
| **Amazon Textract** | OCR for handwritten notes | $10-20 | Handwriting recognition, PDF processing, 90%+ accuracy |

**AI Subtotal**: $45-100/month (₹3,700-8,300)

### 3.2 Backend & Data

| Service | Purpose | Cost (Monthly) | Key Features |
|---------|---------|----------------|--------------|
| **AWS Lambda** | Serverless compute | $0 (Free tier) | Auto-scaling, pay-per-use, 1M free requests |
| **Amazon DynamoDB** | Real-time data storage | $0 (Free tier) | Single-table design, on-demand billing, 25GB free |
| **Amazon S3** | Document storage | $2-5 | PYQ archive, user materials, cheat sheets |

**Backend Subtotal**: $2-5/month (₹150-400)

### 3.3 Frontend & Delivery

| Service | Purpose | Cost (Monthly) | Key Features |
|---------|---------|----------------|--------------|
| **AWS Amplify** | Frontend hosting + CI/CD | $0-5 | Auto-deployment, Cognito integration, 50K users free |
| **AWS AppSync** | GraphQL real-time API | $0-8 | Real-time subscriptions, AI streaming, offline sync |
| **Amazon CloudFront** | CDN (India-optimized) | $0 (Free tier) | ap-south-1 edge, 1TB free transfer |
| **Amazon Cognito** | User authentication | Included | JWT tokens, OAuth, MFA support |

**Frontend Subtotal**: $0-13/month (₹0-1,100)

### 3.4 Total AWS Infrastructure Cost

**Monthly**: $47-118 (~₹3,900-9,800)  
**Per User** (1000 users): <₹10/month

### 3.5 Additional Costs

- **Data Collection**: ₹15,000 (one-time) - PYQ scanning, student contributor incentives
- **Legal & Compliance**: ₹8,000 (one-time) - Privacy policy, terms of service
- **Domain & Operations**: ₹800/year - Domain registration

**Total Implementation Cost**: ~₹24,000 + monthly AWS costs


## 4. Data Model Design

### 4.1 DynamoDB Single-Table Design

**Table Name**: `edumap-data`

**Primary Key**:
- Partition Key (PK): Entity identifier (USER#{uid}, UNIVERSITY#{id})
- Sort Key (SK): Entity type + timestamp + ID

**Global Secondary Index (GSI1)**:
- GSI1PK: Subject/Topic/University identifier
- GSI1SK: Timestamp or score for sorting

#### 4.1.1 Key Access Patterns

| Pattern | PK | SK | Description |
|---------|----|----|-------------|
| User profile | USER#{uid} | PROFILE | User account details |
| Study sessions | USER#{uid} | SESSION#{timestamp} | Pomodoro sessions |
| Grading results | USER#{uid} | GRADE#{timestamp}#{qid} | Answer evaluations |
| Chat history | USER#{uid} | CHAT#{timestamp}#{sid} | AI tutor conversations |
| User materials | USER#{uid} | MATERIAL#{timestamp}#{id} | Uploaded documents |
| Cheat sheets | USER#{uid} | CHEAT#{timestamp}#{id} | Generated cheat sheets |
| PYQ questions | PYQ#{university}#{subject} | Q#{year}#{id} | Question database |
| Topic analytics | USER#{uid} | TOPIC#{subject}#{topic} | Topic-wise performance |

#### 4.1.2 Entity Schemas

**User Profile**:
```json
{
  "PK": "USER#{uid}",
  "SK": "PROFILE",
  "uid": "string",
  "email": "string",
  "name": "string",
  "university": "string",
  "course": "string",
  "year": "number",
  "semester": "number",
  "preferred_language": "en|hi|ta|bn|mr",
  "exam_date": "ISO8601",
  "study_goal_minutes": "number",
  "current_streak": "number",
  "readiness_score": "number",
  "topics_mastered": "number",
  "created_at": "ISO8601",
  "last_active": "ISO8601"
}
```

**Grading Result**:
```json
{
  "PK": "USER#{uid}",
  "SK": "GRADE#{timestamp}#{question_id}",
  "question_id": "string",
  "question_text": "string",
  "student_answer": "string",
  "total_score": "number",
  "max_score": "number",
  "percentage": "number",
  "criteria_scores": [
    {
      "criterion": "Conceptual Understanding",
      "score": 8,
      "max_score": 10,
      "feedback": "Good grasp of core concepts..."
    }
  ],
  "missing_components": ["time complexity analysis", "edge cases"],
  "strengths": ["Clear explanation", "Good examples"],
  "improvements": ["Add complexity analysis", "Consider edge cases"],
  "overall_feedback": "Strong answer with room for improvement...",
  "topic": "string",
  "subject": "string",
  "graded_at": "ISO8601"
}
```

**PYQ Question**:
```json
{
  "PK": "PYQ#{university}#{subject}",
  "SK": "Q#{year}#{id}",
  "id": "string",
  "university": "string",
  "subject": "string",
  "topic": "string",
  "year": "number",
  "exam_type": "string",
  "question_text": "string",
  "marks": "number",
  "difficulty": "easy|medium|hard",
  "frequency": "number",
  "weightage": "number",
  "model_answer": "string",
  "rubric": {...},
  "s3_key": "string",
  "GSI1PK": "TOPIC#{subject}#{topic}",
  "GSI1SK": "FREQ#{frequency}"
}
```

**Study Session (Pomodoro)**:
```json
{
  "PK": "USER#{uid}",
  "SK": "SESSION#{timestamp}#{id}",
  "id": "string",
  "duration_minutes": "number",
  "topic": "string",
  "subject": "string",
  "questions_attempted": "number",
  "questions_correct": "number",
  "session_type": "practice|revision|material_review",
  "started_at": "ISO8601",
  "ended_at": "ISO8601"
}
```

**Topic Analytics**:
```json
{
  "PK": "USER#{uid}",
  "SK": "TOPIC#{subject}#{topic}",
  "subject": "string",
  "topic": "string",
  "mastery_level": "number",
  "questions_attempted": "number",
  "questions_correct": "number",
  "accuracy": "number",
  "time_spent_minutes": "number",
  "last_practiced": "ISO8601",
  "priority": "high|medium|low",
  "exam_weightage": "number",
  "status": "not_started|in_progress|mastered"
}
```

### 4.2 S3 Bucket Structure

**Bucket**: `edumap-uploads`
```
{university}/
  pyq/
    {year}/
      {subject}/
        {exam_type}/
          {question_id}.pdf
          metadata.json
  
{user_uid}/
  materials/
    {material_id}/
      original.pdf
      extracted_text.txt
  cheat_sheets/
    {cheat_sheet_id}.pdf
```

**Bucket**: `edumap-processed`
```
{user_uid}/
  textract_output/
    {material_id}/
      blocks.json
      text.txt
  generated_content/
    {content_id}.pdf
```


## 5. API Design

### 5.1 GraphQL Schema (AWS AppSync)

```graphql
# Core Types
type User {
  uid: ID!
  email: String!
  name: String!
  university: String!
  course: String!
  readinessScore: Float!
  currentStreak: Int!
  topicsMastered: Int!
}

type Question {
  id: ID!
  questionText: String!
  topic: String!
  subject: String!
  marks: Int!
  difficulty: Difficulty!
  year: Int!
  frequency: Int!
  weightage: Float!
}

type GradingResult {
  questionId: ID!
  totalScore: Float!
  percentage: Float!
  criteriaScores: [CriterionScore!]!
  missingComponents: [String!]!
  strengths: [String!]!
  improvements: [String!]!
  overallFeedback: String!
}

type CriterionScore {
  criterion: String!
  score: Float!
  maxScore: Float!
  feedback: String!
}

type ChatMessage {
  id: ID!
  sessionId: ID!
  userMessage: String!
  aiResponse: String!
  citations: [Citation!]
  timestamp: AWSDateTime!
}

type Citation {
  source: String!
  excerpt: String!
  type: CitationType!
  relevanceScore: Float!
}

enum CitationType {
  PYQ
  MATERIAL
  SYLLABUS
}

enum Difficulty {
  EASY
  MEDIUM
  HARD
}

# Queries
type Query {
  getUser(uid: ID!): User
  getReadinessScore(uid: ID!): Float!
  getPriorityTopics(uid: ID!, limit: Int): [TopicPriority!]!
  getTopicHeatmap(uid: ID!): [TopicStatus!]!
  getQuestions(filters: QuestionFilters!): [Question!]!
  searchPYQ(query: String!, filters: PYQFilters): [Question!]!
  getChatHistory(sessionId: ID!): [ChatMessage!]!
  getGradingHistory(uid: ID!, limit: Int): [GradingResult!]!
}

# Mutations
type Mutation {
  submitAnswer(input: SubmitAnswerInput!): GradingResult!
  askTutor(input: AskTutorInput!): ChatMessage!
  uploadMaterial(input: UploadMaterialInput!): Material!
  generateCheatSheet(input: CheatSheetInput!): CheatSheet!
  startPomodoroSession(input: SessionInput!): StudySession!
  endPomodoroSession(sessionId: ID!): StudySession!
  updateUserProfile(input: UpdateProfileInput!): User!
}

# Subscriptions
type Subscription {
  onGradingComplete(uid: ID!): GradingResult
    @aws_subscribe(mutations: ["submitAnswer"])
  
  onTutorResponse(sessionId: ID!): ChatMessage
    @aws_subscribe(mutations: ["askTutor"])
  
  onMaterialProcessed(uid: ID!): Material
    @aws_subscribe(mutations: ["uploadMaterial"])
  
  onReadinessUpdated(uid: ID!): Float
    @aws_subscribe(mutations: ["submitAnswer", "endPomodoroSession"])
}

# Input Types
input SubmitAnswerInput {
  questionId: ID!
  answer: String!
  language: String
}

input AskTutorInput {
  sessionId: ID!
  message: String!
  language: String
}

input QuestionFilters {
  subject: String
  topic: String
  difficulty: Difficulty
  year: Int
  university: String
}
```

### 5.2 Key API Endpoints (Lambda Functions)

#### 5.2.1 Answer Grading
```python
# Lambda: answer_grading_handler
def handler(event, context):
    """
    Grade student answer using rubric-based evaluation
    
    Input:
    - question_id: str
    - student_answer: str
    - language: str (optional)
    
    Process:
    1. Fetch question + rubric from DynamoDB
    2. Prepare grading prompt for Bedrock
    3. Invoke Claude 3.5 Sonnet (temp=0.3)
    4. Parse JSON response
    5. Save to DynamoDB
    6. Update user analytics
    7. Publish to AppSync
    
    Output:
    - GradingResult with detailed feedback
    """
```

#### 5.2.2 AI Tutor
```python
# Lambda: ai_tutor_handler
def handler(event, context):
    """
    Provide AI tutoring with RAG-powered responses
    
    Input:
    - session_id: str
    - message: str
    - language: str (optional)
    
    Process:
    1. Search Kendra for relevant PYQs
    2. Query Bedrock Knowledge Base for context
    3. Prepare enhanced prompt with citations
    4. Stream response from Bedrock
    5. Save conversation to DynamoDB
    6. Publish chunks to AppSync subscription
    
    Output:
    - Streaming ChatMessage with citations
    """
```

#### 5.2.3 Material Processing
```python
# Lambda: material_processing_handler
def handler(event, context):
    """
    Process uploaded study materials
    
    Input:
    - s3_key: str
    - material_type: str
    
    Process:
    1. Trigger Textract for OCR
    2. Extract text and concepts
    3. Index in Kendra
    4. Add to Bedrock Knowledge Base
    5. Generate metadata
    6. Update DynamoDB
    7. Publish to AppSync
    
    Output:
    - Material with extracted content
    """
```

#### 5.2.4 Priority Topic Calculation
```python
# Lambda: priority_calculator_handler
def handler(event, context):
    """
    Calculate AI-prioritized topics based on PYQ analysis
    
    Input:
    - user_id: str
    - university: str
    - subject: str
    
    Process:
    1. Fetch PYQ frequency data
    2. Calculate exam weightage per topic
    3. Analyze user's current mastery
    4. Apply probability-weighted importance algorithm
    5. Rank topics by priority
    6. Return top 8-10 topics
    
    Algorithm:
    priority_score = (exam_weightage * 0.4) + 
                     (frequency * 0.3) + 
                     ((1 - user_mastery) * 0.3)
    
    Output:
    - Ranked list of priority topics
    """
```

#### 5.2.5 Readiness Score Calculation
```python
# Lambda: readiness_calculator_handler
def handler(event, context):
    """
    Calculate real-time exam readiness score
    
    Input:
    - user_id: str
    
    Process:
    1. Fetch user's topic mastery data
    2. Fetch study consistency (streak, time)
    3. Fetch practice accuracy
    4. Fetch PYQ pattern alignment
    5. Calculate weighted score
    
    Algorithm:
    readiness = (topic_coverage * 0.35) +
                (practice_accuracy * 0.30) +
                (study_consistency * 0.20) +
                (pyq_alignment * 0.15)
    
    Where:
    - topic_coverage = mastered_topics / priority_topics
    - practice_accuracy = correct / attempted
    - study_consistency = streak_factor * time_factor
    - pyq_alignment = similarity to past exam patterns
    
    Output:
    - Readiness score (0-100%)
    """
```


## 6. Frontend Design

### 6.1 Component Architecture

```
src/
├── app/
│   ├── App.tsx                          # Main app component
│   └── components/
│       ├── Dashboard/
│       │   ├── ExamDashboard.tsx        # AI Exam Dashboard (readiness, heatmap)
│       │   ├── ReadinessScore.tsx       # Real-time readiness display
│       │   ├── PriorityTopics.tsx       # AI-prioritized topic list
│       │   └── TopicHeatmap.tsx         # Color-coded mastery heatmap
│       ├── Analytics/
│       │   ├── ProgressAnalytics.tsx    # Performance cards, charts
│       │   ├── PerformanceCards.tsx     # Study streak, topics mastered
│       │   ├── WeeklyTimeChart.tsx      # Daily study time visualization
│       │   └── TopicAccuracyTrends.tsx  # Topic-wise accuracy graphs
│       ├── Practice/
│       │   ├── FastRevisionMode.tsx     # Last-minute preparation
│       │   ├── SmartQuestionPractice.tsx # PYQ pattern-based questions
│       │   ├── AnswerWorkspace.tsx      # Split-pane evaluation interface
│       │   └── PracticeModeSelector.tsx # Chapter/Difficulty/Topic modes
│       ├── AITutor/
│       │   ├── TutorChat.tsx            # 24/7 conversational interface
│       │   ├── MessageStream.tsx        # Real-time streaming responses
│       │   └── CitationPanel.tsx        # Source citations display
│       ├── Materials/
│       │   ├── MaterialUpload.tsx       # Upload notes/assignments
│       │   ├── ConceptExtraction.tsx    # Extracted concepts view
│       │   └── CheatSheetGenerator.tsx  # AI cheat sheet creation
│       ├── Pomodoro/
│       │   ├── PomodoroTimer.tsx        # Integrated timer
│       │   ├── SessionLogger.tsx        # Session tracking
│       │   └── StreakTracker.tsx        # Daily streak display
│       ├── Auth/
│       │   ├── LoginPage.tsx            # User login
│       │   ├── SignupPage.tsx           # User registration
│       │   └── OnboardingFlow.tsx       # University profile setup
│       └── Shared/
│           ├── Sidebar.tsx              # Navigation
│           ├── Header.tsx               # Top bar with user menu
│           ├── GlassCard.tsx            # Glassmorphism card component
│           └── LoadingStates.tsx        # Skeleton loaders
├── config/
│   └── aws.ts                           # AWS Amplify configuration
├── services/
│   ├── graphql/
│   │   ├── queries.ts                   # GraphQL queries
│   │   ├── mutations.ts                 # GraphQL mutations
│   │   └── subscriptions.ts             # Real-time subscriptions
│   └── api/
│       ├── grading.ts                   # Answer grading service
│       ├── tutor.ts                     # AI tutor service
│       └── analytics.ts                 # Analytics service
└── hooks/
    ├── useAuth.ts                       # Authentication hook
    ├── useReadinessScore.ts             # Real-time readiness
    ├── usePriorityTopics.ts             # Priority recommendations
    ├── useGrading.ts                    # Answer grading
    └── useTutorChat.ts                  # AI tutor chat
```

### 6.2 Key UI Screens

#### 6.2.1 AI Exam Dashboard
```typescript
// Central command center showing:
// - Real-time readiness score (0-100%)
// - AI-prioritized topics (top 8-10)
// - Color-coded heatmap (red/yellow/green)
// - Quick actions (Practice, Revise, Ask Tutor)

interface DashboardProps {
  readinessScore: number;
  priorityTopics: TopicPriority[];
  heatmapData: TopicStatus[];
  studyStreak: number;
}
```

#### 6.2.2 Answer Evaluation Workspace
```typescript
// Split-pane interface:
// Left: Question display with metadata
// Right: Rich-text editor for answer
// Bottom: Hints system (3 levels)
// Modal: Post-submission feedback panel

interface WorkspaceProps {
  question: Question;
  onSubmit: (answer: string) => Promise<GradingResult>;
  hints: Hint[];
  rubric: Rubric;
}
```

#### 6.2.3 AI Tutor Chat
```typescript
// Conversational interface with:
// - Message history
// - Streaming responses
// - Citation panels
// - Quick actions (Explain, Example, Practice)

interface TutorChatProps {
  sessionId: string;
  messages: ChatMessage[];
  onSendMessage: (message: string) => void;
  isStreaming: boolean;
}
```

### 6.3 Real-Time Features (AppSync Subscriptions)

```typescript
// Subscribe to grading completion
const { data } = useSubscription(ON_GRADING_COMPLETE, {
  variables: { uid: user.uid }
});

// Subscribe to tutor responses (streaming)
const { data } = useSubscription(ON_TUTOR_RESPONSE, {
  variables: { sessionId }
});

// Subscribe to readiness score updates
const { data } = useSubscription(ON_READINESS_UPDATED, {
  variables: { uid: user.uid }
});
```

### 6.4 State Management

```typescript
// Global state using React Context
interface AppState {
  user: User | null;
  readinessScore: number;
  priorityTopics: TopicPriority[];
  studyStreak: number;
  currentSession: StudySession | null;
}

// Local state for components
// Apollo Client cache for GraphQL data
// Session storage for temporary data
```


## 7. Security & Compliance

### 7.1 Authentication Flow

```
1. User visits EduMap
2. Redirected to AWS Cognito Hosted UI
3. User signs in (email/password or Google OAuth)
4. Cognito returns JWT tokens (ID, Access, Refresh)
5. Frontend stores tokens securely
6. All API requests include ID token in Authorization header
7. AppSync validates token with Cognito
8. Lambda functions receive authenticated user context
```

### 7.2 Authorization Model

**User Roles**:
- **Student**: Default role, access to all learning features
- **Contributor**: Can upload PYQs to database
- **Admin**: Platform management (future)

**IAM Policies**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::edumap-uploads/${cognito-identity.amazonaws.com:sub}/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:Query"
      ],
      "Resource": "arn:aws:dynamodb:*:*:table/edumap-data",
      "Condition": {
        "ForAllValues:StringEquals": {
          "dynamodb:LeadingKeys": ["USER#${cognito-identity.amazonaws.com:sub}"]
        }
      }
    }
  ]
}
```

### 7.3 Data Protection

**Encryption**:
- At Rest: S3 (SSE-S3), DynamoDB (AWS KMS)
- In Transit: HTTPS/TLS 1.2+
- Presigned URLs: 24-hour expiration

**Data Isolation**:
- S3 paths: `{user_uid}/...`
- DynamoDB rows: `PK = USER#{user_uid}`
- User can only access own data

**Privacy**:
- GDPR compliant (data deletion, portability)
- User materials are private
- PYQ contributions are anonymized
- No data sharing with third parties

## 8. Deployment & Operations

### 8.1 CI/CD Pipeline

```
GitHub Repository
    ↓
Push to main branch
    ↓
GitHub Actions Workflow
    ↓
┌─────────────────────────────────┐
│  Frontend Pipeline              │
│  1. npm install                 │
│  2. npm run build               │
│  3. Deploy to AWS Amplify       │
│  4. Invalidate CloudFront cache │
└─────────────────────────────────┘
    ↓
┌─────────────────────────────────┐
│  Backend Pipeline               │
│  1. pip install -r requirements │
│  2. Run tests (pytest)          │
│  3. Package Lambda functions    │
│  4. Deploy via AWS SAM          │
│  5. Update AppSync schema       │
└─────────────────────────────────┘
    ↓
Production Deployment Complete
```

### 8.2 Monitoring & Alerting

**CloudWatch Dashboards**:
- API response times (p50, p95, p99)
- Lambda invocations and errors
- DynamoDB capacity and throttling
- Bedrock token usage and costs
- User activity metrics

**Alarms**:
- High error rate (>1%)
- High latency (>5s for grading)
- DynamoDB throttling
- Budget threshold (80% of ₹10,000)
- Low readiness score trends

**Logging**:
```json
{
  "timestamp": "ISO8601",
  "level": "INFO|ERROR",
  "service": "edumap-backend",
  "function": "answer_grading_handler",
  "user_id": "uid",
  "request_id": "uuid",
  "message": "Answer graded successfully",
  "metadata": {
    "question_id": "q123",
    "score": 85,
    "duration_ms": 4500
  }
}
```

### 8.3 Disaster Recovery

**Backup Strategy**:
- DynamoDB: Point-in-time recovery (35 days)
- S3: Versioning enabled
- Lambda: Code in GitHub
- Infrastructure: CloudFormation templates

**Recovery Objectives**:
- RTO (Recovery Time): <1 hour
- RPO (Recovery Point): <5 minutes

## 9. Cost Optimization

### 9.1 Strategies

1. **Free Tier Maximization**:
   - Lambda: 1M requests/month free
   - DynamoDB: 25GB storage free
   - CloudFront: 1TB transfer free
   - Kendra: 750 hours free (hackathon phase)

2. **AI Token Optimization**:
   - Cache common responses
   - Limit context window to relevant chunks
   - Use temperature=0.3 for grading (consistency)
   - Batch similar requests

3. **Storage Optimization**:
   - S3 Intelligent-Tiering for old PYQs
   - Compress uploaded materials
   - Delete temporary files after processing

4. **Compute Optimization**:
   - Right-size Lambda memory (1024MB default)
   - Use Lambda container reuse
   - Async processing for non-critical tasks

### 9.2 Cost Monitoring

```python
# Daily cost tracking Lambda
def track_costs():
    costs = {
        'bedrock': get_bedrock_usage(),
        'lambda': get_lambda_invocations(),
        'dynamodb': get_dynamodb_capacity(),
        's3': get_s3_storage()
    }
    
    total = sum(costs.values())
    
    if total > BUDGET_THRESHOLD:
        send_alert(f"Cost alert: ₹{total}/month")
    
    log_to_cloudwatch(costs)
```

## 10. Future Enhancements

### Phase 2: Mobile & Offline (3-6 months)
- React Native apps for iOS and Android
- Offline mode with local caching
- Push notifications for streaks and reminders
- Mobile-optimized UI

### Phase 3: Advanced AI (6-12 months)
- Adaptive learning paths
- Predictive exam score analytics
- Automated syllabus mapping
- Multi-modal content (diagrams, videos)

### Phase 4: Institutional (12+ months)
- College partnerships
- Faculty dashboards
- Bulk PYQ upload
- Custom branding for universities

## 11. Success Metrics

### Technical KPIs
- API p99 latency: <500ms
- Grading accuracy: >90% match with human graders
- System uptime: 99.9%+
- Cost per user: <₹10/month

### User KPIs
- DAU: 1000+ within 3 months
- Readiness score improvement: +30% average
- Study streak retention: 40%+ maintain 7-day
- User satisfaction: 4.5+ stars

### Business KPIs
- Universities covered: 50+ in 6 months
- PYQ database: 10,000+ questions
- Student reach: 10,000+ in first year
- Cost efficiency: <₹10,000/month AWS spend

---

**Document Version**: 2.0  
**Last Updated**: February 15, 2024  
**Status**: Final - Ready for Implementation  
**Team**: Serendipity  
**Team Leader**: Nilay Kumar
