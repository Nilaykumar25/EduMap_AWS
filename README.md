# EduMap – AI Exam Tutor  
AWS-Native Generative AI Powered Learning Platform

---

## 📌 Project Overview

EduMap is a cloud-native, AI-driven exam preparation platform that provides personalized learning using Generative AI and intelligent content processing. It enables students to upload study materials, interact with an AI tutor, generate practice questions, receive automated grading, and track performance through real-time analytics.

The system is fully serverless, scalable, and built entirely on AWS using a Retrieval-Augmented Generation (RAG) architecture.

---

## 🎯 Core Features

- AI Tutor (Claude 3.5 Sonnet via Amazon Bedrock)  
- Handwritten & PDF material processing  
- Intelligent question generation  
- Rubric-based answer grading  
- Multilingual tutoring (Hindi, Tamil, Bengali, Marathi)  
- Personalized dashboard  
- Study analytics & readiness scoring  
- Pomodoro focus timer  
- Automated revision & cheat sheets  

---

## 🧭 User Flow

Login / Signup  
→ Dashboard  
→ Add Materials  
→ My Library  
→ Practice  
→ Analytics  
→ Revision  
→ Pomodoro Timer  
→ Cheat Sheets  
→ Settings  

---

## 🏗️ System Architecture

Layered AWS-native architecture:

User Layer  
↓  
Frontend & Delivery  
↓  
API & Authentication  
↓  
Content Processing & RAG  
↓  
GenAI Layer  
↓  
Backend & Data  
↓  
Analytics & Monitoring  

---

## 🖥️ Frontend & Delivery Layer

### Client Applications
- React 18 (Web)  
- React Native (Mobile)

### Hosting & CDN
- AWS Amplify – Frontend hosting & CI/CD  
- Amazon CloudFront – Global content delivery  

### Realtime Communication
- AWS AppSync (GraphQL)  
  - Streaming AI tutor responses  
  - Live dashboard updates  

---

## 🔐 API & Authentication Layer

All requests pass through:

AWS Amplify  
→ Amazon Cognito (Authentication)  
→ AWS AppSync / REST APIs  
→ AWS Lambda  

---

## 🧠 Content Processing & RAG Pipeline

Amazon S3 (Raw Files)  
→ AWS Textract (OCR)  
→ AWS Lambda (Cleaning & Chunking)  
→ Amazon Bedrock Knowledge Bases  
→ Amazon OpenSearch (Vector Store)  
→ AWS Bedrock (Claude 3.5 Sonnet)  
→ AI Tutor Responses  

---

## 🤖 GenAI Layer

### Core AI Services

Amazon Bedrock (Claude 3.5 Sonnet)  
- AI tutor chat  
- Answer grading  
- Question generation  
- Multilingual support  

Amazon Bedrock Knowledge Bases  
- RAG pipeline: S3 → OpenSearch → Bedrock  

Amazon Kendra  
- Semantic indexing of PYQs and syllabus  
- Contextual document retrieval  

---

## ⚙️ Backend & Data Layer

### Serverless Microservices
AWS Lambda  
- Material processing  
- Answer evaluation  
- Question generation  
- Analytics aggregation  

### Datastores

Amazon DynamoDB  
- User progress  
- Readiness scores  
- Study streaks  
- Session analytics  

Amazon S3  
- Uploaded materials  
- PYQ archives  
- Generated cheat sheets  

---

## 📊 Analytics Layer

Amazon DynamoDB + AWS Bedrock Outputs  
→ Analytics Service  
→ Analytics Dashboard  

Provides:
- Performance insights  
- Study trends  
- AI effectiveness metrics  

---

## 🔍 Monitoring & Security

Amazon Cognito – Secure authentication  
AWS CloudWatch – Logs, metrics, alarms  
AWS IAM – Fine-grained access control  

---

## 🧰 Complete AWS Tech Stack

| Service | Purpose |
|--------|---------|
| Amazon S3 | Object storage |
| Amazon OpenSearch | Vector search |
| Amazon Kendra | Semantic search |
| AWS Lambda | Serverless compute |
| Amazon DynamoDB | NoSQL database |
| Amazon Cognito | User authentication |
| AWS Bedrock | Foundation models |
| AWS AppSync | Real-time GraphQL |
| AWS Amplify | Frontend hosting & CI/CD |
| Amazon CloudFront | Global CDN |
| AWS Textract | OCR |
| AWS CloudWatch | Monitoring |

---

## 🧩 Design Principles

- Fully serverless  
- Horizontally scalable  
- Cost-efficient (pay-per-use)  
- AI-first architecture  
- Secure by design  
- Modular microservices  

---

## 🚀 Future Enhancements

- Voice-based AI tutor  
- Adaptive learning paths  
- Gamified challenges  
- Peer study rooms  
- Offline mobile support  
- AI-powered mock exams  

---

## 🏁 Final Note

EduMap demonstrates how modern cloud architecture combined with Generative AI can transform education. It provides a scalable, intelligent, and personalized learning ecosystem using 100% AWS-native services.
