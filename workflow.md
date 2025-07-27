# AI Lead Management System Workflow

This document describes the comprehensive workflow for an AI-powered lead management system with conversational AI, automated follow-ups, and SDR handoff capabilities.

## Workflow Diagram

```mermaid
graph TB
    subgraph "Workflow 1: Lead Ingestion & State Management"
        A1[Webhook - New Lead Trigger] --> B1[Set - Extract Lead Variables]
        B1 --> C1[Airtable - Create AI Lead Log Record]
        C1 --> D1[Code - Data Validation Check]
        D1 --> E1{IF - Is Data Complete?}
        E1 -->|Yes| F1[HTTP Request - Trigger Handoff Workflow]
        E1 -->|No| G1[HTTP Request - Trigger AI Engagement]
        F1 --> H1[End - Complete Lead]
        G1 --> I1[End - AI Engagement Started]
    end

    subgraph "Workflow 2: Conversational AI Logic & Context"
        A2[Webhook - AI Engage / Inbound Reply] --> B2[Airtable - Get Lead Record]
        B2 --> C2[Code - Check for STOP Command]
        C2 --> D2{IF - Is STOP Command?}
        D2 -->|Yes| E2[Airtable - Update Status to Opt-Out]
        E2 --> F2[GoHighLevel - Add AI Opt-Out Tag]
        F2 --> G2[End - Opt-Out Processed]
        
        D2 -->|No| H2[Code - Parse Reply & Determine Field]
        H2 --> I2[Airtable - Update Lead Data Field]
        I2 --> J2[GoHighLevel - Update Custom Field]
        J2 --> K2[Code - Determine Next Question]
        K2 --> L2{IF - All Data Collected?}
        L2 -->|Yes| M2[HTTP Request - Trigger Handoff]
        L2 -->|No| N2[OpenAI - Generate AI Response]
        N2 --> O2[GoHighLevel - Send SMS]
        O2 --> P2[Airtable - Update Last Message & History]
        P2 --> Q2[End - Message Sent]
        M2 --> R2[End - Ready for Handoff]
    end

    subgraph "Workflow 3: Automated Follow-ups & Maintenance"
        A3[Schedule Trigger - Hourly] --> B3[Airtable - List AI Working Records]
        B3 --> C3[Code - Filter for Follow-up Candidates]
        C3 --> D3[Split In Batches - Process One by One]
        D3 --> E3{Switch - Follow-up Type}
        
        E3 -->|First Follow-up| F3[OpenAI - Generate First Follow-up]
        F3 --> G3[GoHighLevel - Send Follow-up SMS]
        G3 --> H3[Airtable - Update Follow-up Count to 1]
        H3 --> I3[Airtable - Update Last Message Timestamp]
        I3 --> J3[End - First Follow-up Sent]
        
        E3 -->|Second Follow-up| K3[OpenAI - Generate Second Follow-up]
        K3 --> L3[GoHighLevel - Send Follow-up SMS]
        L3 --> M3[Airtable - Update Follow-up Count to 2]
        M3 --> N3[Airtable - Update Last Message Timestamp]
        N3 --> O3[End - Second Follow-up Sent]
        
        E3 -->|Mark Unresponsive| P3[Airtable - Update Status to Unresponsive]
        P3 --> Q3[GoHighLevel - Add AI Unresponsive Tag]
        Q3 --> R3[GoHighLevel - Remove AI Working Tag]
        R3 --> S3[End - Marked Unresponsive]
    end

    subgraph "Workflow 4: Process Completion & SDR Handoff"
        A4[Webhook - Handoff Trigger] --> B4[Airtable - Get Complete Lead Record]
        B4 --> C4[Code - Generate Lead Summary]
        C4 --> D4[OpenAI - Generate Final Confirmation Message]
        D4 --> E4[GoHighLevel - Send Final SMS]
        E4 --> F4[Airtable - Update Status to AI Complete]
        F4 --> G4[Airtable - Set AI Completion Timestamp]
        G4 --> H4[GoHighLevel - Add AI Complete Tag]
        H4 --> I4[GoHighLevel - Remove AI Working Tag]
        I4 --> J4[GoHighLevel - Update All Custom Fields]
        J4 --> K4[GoHighLevel - Create Opportunity in Sales Pipeline]
        K4 --> L4[Slack - Send Qualified Lead Notification]
        L4 --> M4[Airtable - Update Notes with Conversation Summary]
        M4 --> N4[End - Lead Handed to SDR]
    end

    subgraph "Workflow 5: System Maintenance & SDR Overrides"
        subgraph "SDR Override Branch"
            A5[Webhook - SDR Called Tag Added] --> B5[Airtable - Update Status to SDR Override]
            B5 --> C5[Airtable - Set Last SDR Action Timestamp]
            C5 --> D5[GoHighLevel - Remove All AI Tags]
            D5 --> E5[End - AI Activity Stopped]
        end
        
        subgraph "Daily Maintenance Branch"
            F5[Schedule Trigger - Daily 10 PM] --> G5[Airtable - List All Active Records]
            G5 --> H5[Code - Identify System Issues]
            H5 --> I5[Split In Batches - Process Issues]
            I5 --> J5{Switch - Issue Type}
            
            J5 -->|Stuck Records| K5[Airtable - Reset Stuck Lead Status]
            K5 --> L5[GoHighLevel - Clean Up Tags]
            L5 --> M5[End - Stuck Records Fixed]
            
            J5 -->|Data Inconsistencies| N5[Code - Fix Data Mismatches]
            N5 --> O5[Airtable - Update Corrected Data]
            O5 --> P5[GoHighLevel - Sync Corrected Fields]
            P5 --> Q5[End - Data Synchronized]
            
            J5 -->|Performance Report| R5[Code - Generate Daily Report]
            R5 --> S5[Slack - Send Performance Summary]
            S5 --> T5[End - Report Sent]
        end
    end

    %% Workflow Connections
    F1 -.-> A4
    G1 -.-> A2
    M2 -.-> A4
    
    %% Styling
    classDef webhookNode fill:#e74c3c,stroke:#c0392b,stroke-width:2px,color:#fff
    classDef processNode fill:#3498db,stroke:#2980b9,stroke-width:2px,color:#fff
    classDef decisionNode fill:#f39c12,stroke:#e67e22,stroke-width:2px,color:#fff
    classDef actionNode fill:#27ae60,stroke:#229954,stroke-width:2px,color:#fff
    classDef endNode fill:#95a5a6,stroke:#7f8c8d,stroke-width:2px,color:#fff
    classDef scheduleNode fill:#9b59b6,stroke:#8e44ad,stroke-width:2px,color:#fff
    
    class A1,A2,A5 webhookNode
    class B1,C2,D1,H2,K2,C3,C4,H5,N5,R5 processNode
    class E1,D2,L2,E3,J5 decisionNode
    class C1,E2,F2,I2,J2,G3,H3,I3,L3,M3,N3,P3,Q3,R3,B4,F4,G4,H4,I4,J4,K4,M4,B5,C5,D5,K5,L5,O5,P5 actionNode
    class H1,I1,G2,Q2,R2,J3,O3,S3,N4,E5,M5,Q5,T5 endNode
    class A3,F5 scheduleNode
```

## Workflow Overview

This system consists of five main workflows that work together to create a comprehensive AI-powered lead management solution:

### Workflow 1: Lead Ingestion & State Management
- Handles new lead intake and initial data validation
- Creates lead records and determines next steps
- Routes leads to either immediate handoff or AI engagement

### Workflow 2: Conversational AI Logic & Context
- Manages real-time conversations with leads
- Handles opt-out requests and data collection
- Determines conversation flow and next questions
- Triggers handoff when all required data is collected

### Workflow 3: Automated Follow-ups & Maintenance
- Runs hourly to check for leads needing follow-up
- Sends automated follow-up messages based on engagement level
- Marks leads as unresponsive after multiple attempts

### Workflow 4: Process Completion & SDR Handoff
- Handles the transition from AI to human SDR
- Generates lead summaries and final confirmation messages
- Creates opportunities in the sales pipeline
- Sends notifications to the sales team

### Workflow 5: System Maintenance & SDR Overrides
- Handles SDR override scenarios when humans take control
- Performs daily system maintenance and cleanup
- Generates performance reports and fixes data inconsistencies

## Key Features

- **Hybrid AI-Human Approach**: Seamless transition between AI and human interaction
- **Automated Follow-ups**: Intelligent follow-up system with multiple touchpoints
- **Data Synchronization**: Real-time sync between Airtable and GoHighLevel
- **Opt-out Management**: Respectful handling of opt-out requests
- **Performance Monitoring**: Daily reports and system health checks
- **Flexible Override System**: Allows human intervention when needed 