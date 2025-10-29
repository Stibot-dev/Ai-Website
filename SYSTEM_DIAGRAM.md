# System Architecture Diagrams

## 🎯 High-Level Overview

```
┌─────────────────────────────────────────────────────────────┐
│                         USER                                 │
│                    (Web Browser)                             │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                   FRONTEND LAYER                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │ Landing Page │→ │ Chat Interface│→ │Results Page  │     │
│  │  (index.html)│  │(ai_consult..)│  │(results.html)│     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│                                                              │
│  ┌──────────────────────────────────────────────────┐      │
│  │         localStorage (Client-Side Storage)        │      │
│  │  - Conversation history                           │      │
│  │  - Consultation data                              │      │
│  │  - 2-hour expiration                              │      │
│  └──────────────────────────────────────────────────┘      │
└────────────────────────┬────────────────────────────────────┘
                         │ HTTPS
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                   BACKEND LAYER                              │
│  ┌──────────────────────────────────────────────────┐      │
│  │         Flask Web Server (web_app.py)            │      │
│  │  Routes:                                          │      │
│  │  - GET  /                                         │      │
│  │  - GET  /consultation                             │      │
│  │  - POST /start_consultation                       │      │
│  │  - POST /send_message                             │      │
│  │  - GET  /results                                  │      │
│  │  - GET  /api/get_recommendations                  │      │
│  └──────────────────────────────────────────────────┘      │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                   AI LAYER                                   │
│  ┌──────────────────────────────────────────────────┐      │
│  │      AI Consultant (ai_consultant.py)            │      │
│  │  - Conversation management                        │      │
│  │  - Context understanding                          │      │
│  │  - Data extraction                                │      │
│  └──────────────────────────────────────────────────┘      │
│                         │                                    │
│                         ↓                                    │
│  ┌──────────────────────────────────────────────────┐      │
│  │         Groq API (Llama 3.3 70B)                 │      │
│  │  - Natural language processing                    │      │
│  │  - Conversation generation                        │      │
│  │  - Tool recommendation                            │      │
│  └──────────────────────────────────────────────────┘      │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                   DATA LAYER                                 │
│  ┌──────────────────────────────────────────────────┐      │
│  │    Tools Database (tools_database.py)            │      │
│  │  - 127 curated AI tools                          │      │
│  │  - Name, URL, category                           │      │
│  └──────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔄 User Journey Flow

```
START
  │
  ↓
┌─────────────────────┐
│   Landing Page      │
│                     │
│  - See features     │
│  - Read "How It     │
│    Works"           │
│  - Click "Start"    │
└──────────┬──────────┘
           │
           ↓
┌─────────────────────┐
│  Chat Interface     │
│                     │
│  User: "I want to   │
│  start a business"  │
│                     │
│  AI: "What type?"   │
│                     │
│  User: "E-commerce" │
│                     │
│  AI: "Budget?"      │
│                     │
│  User: "$50/month"  │
│                     │
│  AI: "Skills?"      │
│                     │
│  User: "Beginner"   │
│                     │
│  AI: "Main need?"   │
│                     │
│  User: "Marketing"  │
│                     │
│  AI: ✅ Complete!   │
└──────────┬──────────┘
           │
           ↓
┌─────────────────────┐
│  "View              │
│  Recommendations"   │
│  Button Appears     │
└──────────┬──────────┘
           │
           ↓
┌─────────────────────┐
│  Results Page       │
│                     │
│  📊 Summary         │
│                     │
│  🎯 Curated Tools   │
│  1. Tool A          │
│  2. Tool B          │
│  3. Tool C          │
│                     │
│  🌐 Additional      │
│  1. Tool X          │
│  2. Tool Y          │
└─────────────────────┘
  │
  ↓
END
```

---

## 💬 Conversation Flow Detail

```
┌──────────────────────────────────────────────────────────┐
│                    USER SENDS MESSAGE                     │
└────────────────────────┬─────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────┐
│              SAVED TO localStorage                        │
│  messages.push({role: 'user', content: message})         │
└────────────────────────┬─────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────┐
│              SENT TO SERVER                               │
│  POST /send_message                                       │
│  Body: {message, messages: [...]}                        │
└────────────────────────┬─────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────┐
│         SERVER PROCESSES                                  │
│  1. Get session messages OR reconstruct from client      │
│  2. Call Groq API with conversation history              │
│  3. Get AI response                                       │
│  4. Check if CONSULTATION_COMPLETE                       │
└────────────────────────┬───��─────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────┐
│              RESPONSE TO CLIENT                           │
│  {message: "AI response", complete: true/false}          │
└────────────────────────┬─────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────┐
│         SAVED TO localStorage                             │
│  messages.push({role: 'assistant', content: response})   │
└────────────────────────┬─────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────┐
│              DISPLAYED TO USER                            │
│  If complete: Show "View Recommendations" button         │
│  If not: Continue conversation                           │
└──────────────────────────────────────────────────────────┘
```

---

## 🎯 Recommendation Generation

```
┌──────────────────────────────────────────────────────────┐
│         USER CLICKS "VIEW RECOMMENDATIONS"                │
└────────────────────────┬─────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────┐
│              NAVIGATE TO /results                         │
└────────────────────────┬─────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────┐
│         FETCH /api/get_recommendations                    │
└────────────────────────┬─────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────┐
│              SERVER PROCESSES                             │
│                                                           │
│  ┌─────────────────────────────────────────────┐        │
│  │  PHASE 1: Curated Recommendations           │        │
│  │                                              │        │
│  │  1. Get consultation data from session      │        │
│  │  2. Load 127-tool database                  │        │
│  │  3. Send to Groq API:                       │        │
│  │     "Select 3-5 tools from this list        │        │
│  │      that match these needs..."             │        │
│  │  4. AI analyzes and selects tools           │        │
│  │  5. Returns with ratings                    │        │
│  └─────────────────────────────────────────────┘        │
│                         │                                 │
│                         ↓                                 │
│  ┌─────────────────────────────────────────────┐        │
│  │  PHASE 2: Additional Recommendations        │        │
│  │                                              │        │
│  │  1. Use same consultation data              │        │
│  │  2. Send to Groq API:                       │        │
│  │     "Find 2-3 additional tools NOT in       │        │
│  │      the database that would help..."       │        │
│  │  3. AI searches its knowledge               │        │
│  │  4. Returns emerging/specialized tools      │        │
│  └─────────────────────────────────────────────┘        │
└────────────────────────┬─────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────┐
│              RETURN TO CLIENT                             │
│  {                                                        │
│    curated_tools: [...],                                 │
│    additional_tools: [...]                               │
│  }                                                        │
└────────────────────────┬─────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────┐
│              RENDER ON PAGE                               │
│                                                           │
│  Section 1: 🎯 Curated Database Tools                    │
│  - Tool 1 (complexity, cost, scalability, impact)       │
│  - Tool 2 (complexity, cost, scalability, impact)       │
│  - Tool 3 (complexity, cost, scalability, impact)       │
│                                                           │
│  Section 2: 🌐 Additional Web-Discovered Tools           │
│  - Tool X (complexity, cost, scalability, impact)       │
│  - Tool Y (complexity, cost, scalability, impact)       │
└──────────────────────────────────────────────────────────┘
```

---

## 🔄 Auto-Retry Mechanism

```
┌──────────────────────────────────────────────────────────┐
│         USER SENDS MESSAGE                                │
└────────────────────────┬─────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────┐
│         TRY TO SEND TO SERVER                             │
└────────────────────────┬─────────────────────────────────┘
                         │
                    ┌────┴────┐
                    │         │
              SUCCESS?    NETWORK ERROR
                    │         │
                    ↓         ↓
            ┌───────────┐  ┌──────────────────────────┐
            │  SHOW AI  │  │  ATTEMPT 1/3             │
            │  RESPONSE │  │  Wait 10 seconds         │
            └───────────┘  │  Show: "Server waking up"│
                           └────────┬─────────────────┘
                                    │
                                    ↓
                           ┌──────────────────────────┐
                           │  RETRY AUTOMATICALLY     │
                           └────────┬─────────────────┘
                                    │
                               ┌────┴────┐
                               │         │
                         SUCCESS?    STILL FAILING
                               │         │
                               ↓         ↓
                       ┌───────────┐  ┌──────────────────────────┐
                       │  SHOW AI  │  │  ATTEMPT 2/3             │
                       │  RESPONSE │  │  Wait 20 seconds         │
                       └───────────┘  │  Show: "Retrying..."     │
                                      └────────┬─────────────────┘
                                               │
                                               ↓
                                      ┌──────────────────────────┐
                                      │  RETRY AUTOMATICALLY     │
                                      └────────┬─────────────────┘
                                               │
                                          ┌────┴────┐
                                          │         │
                                    SUCCESS?    STILL FAILING
                                          │         │
                                          ↓         ↓
                                  ┌───────────┐  ┌──────────────────────────┐
                                  │  SHOW AI  │  │  ATTEMPT 3/3             │
                                  │  RESPONSE │  │  Wait 30 seconds         │
                                  └───────────┘  │  Show: "Final retry..."  │
                                                 └────────┬─────────────────┘
                                                          │
                                                          ↓
                                                 ┌──────────────────────────┐
                                                 │  RETRY AUTOMATICALLY     │
                                                 └────────┬─────────────────┘
                                                          │
                                                     ┌────┴────┐
                                                     │         │
                                               SUCCESS?    FAILED
                                                     │         │
                                                     ↓         ↓
                                             ┌───────────┐  ┌──────────────────┐
                                             │  SHOW AI  │  │  SHOW ERROR:     │
                                             │  RESPONSE │  │  "Please start   │
                                             └───────────┘  │   new consult"   │
                                                            └──────────────────┘
```

---

## 📊 Data Flow

```
┌─────────────────────────────────────────────────────────────┐
│                    USER INPUT                                │
│  "I want to start an e-commerce business selling jewelry"   │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                 AI PROCESSING                                │
│  Extracts:                                                   │
│  - business_type: "e-commerce jewelry store"                │
│  - business_functions: "sales, marketing, operations"       │
│  - business_activities: "product photos, descriptions"      │
│  - budget: "under $50/month"                                │
│  - technical_skills: "beginner"                             │
│  - scalability: "start small, grow later"                   │
│  - impact_potential: "reach online customers"               │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ↓
┌─────────────────────────────────────────────────────────────┐
│              RECOMMENDATION ENGINE                           │
│                                                              │
│  Curated Selection:                                          │
│  ✓ Canva AI (design) - Low complexity, Free                │
│  ✓ ChatGPT (content) - Low complexity, Free                │
│  ✓ Shopify (platform) - Medium complexity, $29/mo          │
│                                                              │
│  Additional Discovery:                                       │
│  ✓ Looka (logo design) - Low complexity, $20/mo            │
│  ✓ Printful (print-on-demand) - Low complexity, Free       │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                    OUTPUT TO USER                            │
│  Each tool with:                                             │
│  - Name & URL                                                │
│  - Description                                               │
│  - Why perfect for jewelry e-commerce                       │
│  - Complexity: Low/Medium/High                              │
│  - Cost: Free/Freemium/Paid                                 │
│  - Scalability: ⭐⭐⭐⭐⭐                                      │
│  - Impact: ⭐⭐⭐⭐⭐                                           │
└─────────────────────────────────────────────────────────────┘
```

---

**End of System Diagrams**
