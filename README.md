# Alzheimer’s Disease Risk Assessment Web Application

Use of a Self-report Alzheimer’s Disease Risk Index for Population Brain Health Prevention  
COMP491 – Computer Engineering Design Project (Spring 2022)

**Author**
- Sedat Çoban 


---

## Overview

This project is a web application that implements a self-report Alzheimer’s Disease Risk Index for Turkish adults aged 65 and above. It allows users to:

- Complete a validated risk-assessment questionnaire,
- Receive personalized, evidence-based recommendations to reduce their dementia risk,
- Be monitored longitudinally through user accounts.

Collected data is intended to support **population-level brain health prevention research** and to help identify common risk patterns in the Turkish population. :contentReference[oaicite:1]{index=1}  

---

## Problem & Goals

### Problem

Although there are online dementia / Alzheimer’s risk calculators, there was no Turkish-language, locally deployed tool targeting older adults and their healthcare providers. Early detection and continuous monitoring of modifiable risk factors is crucial to delay or prevent Alzheimer’s disease progression. :contentReference[oaicite:2]{index=2}  

### Goals

- Provide an **online risk assessment test** (based on the ANU-ADRI methodology) to Turkish users over 65.
- Compute a **personalized risk profile** from self-reported lifestyle and health data.
- Generate **tailored suggestion reports** for modifiable risk and protective factors.
- Allow **admins** and **super-admins** to:
  - View statistics on responses,
  - Inspect individual assessment sessions,
  - Export data for research (CSV/XLSX),
  - Manage questions and educational content. :contentReference[oaicite:3]{index=3}  

---

## Features

### For Test-Takers (Patients)

- Web-based risk assessment test covering:
  - Health status (diabetes, depression, BMI, traumatic brain injury, smoking, cholesterol, alcohol, pesticide exposure),
  - Protective factors (physical activity, fish intake, social engagement, cognitive activity),
  - Basic demographics and lifestyle factors. :contentReference[oaicite:4]{index=4}  
- Automatic computation of risk scores per domain using a scoring algorithm derived from the Australian National University Alzheimer’s Disease Risk Index (ANU-ADRI). :contentReference[oaicite:5]{index=5}  
- A **personal suggestion report** per assessment, with targeted recommendations for each risk / protective factor.
- User accounts to support **repeated assessments** over time.

### For Admins (Doctors / Nurses)

- Dashboard with **aggregated statistics** for each question:
  - Answer distributions (percentages),
  - Averages and binned statistics.
- Ability to **view population-level trends** in responses.

### For Super-Admins (Project Owners – Nursing Faculty)

- Access to **individual assessment sessions** including:
  - All given answers,
  - Generated suggestions for that session.
- Ability to:
  - Add / remove **questions** from the assessment,
  - Manage **static content** (education pages, risk factor information),
  - Add / remove **images and videos** shown on the website,
  - Download collected data in **CSV or XLSX** format for research. :contentReference[oaicite:6]{index=6}  

---

## System Design

The system follows a **Model–View–Controller (MVC)** architecture. :contentReference[oaicite:7]{index=7}  

### Model

- Implemented using **Microsoft SQL Server (MSSQL)** with ~12 tables, including:
  - `User`, `UserType`,
  - `AssessmentSession`,
  - `Question`, `Answer`,
  - `Suggestions`, and other supporting entities.
- A **Python Data Access Layer (DAL)** handles all communication with the database (via `pyodbc` connection).
- Each entity (e.g., `Answer`) has:
  - Table mappings,
  - Helper methods (e.g., `has_item_by_column`, counting routines, averages, binned statistics). :contentReference[oaicite:8]{index=8}  

#### Scoring Algorithm

- Domain-specific scoring functions (e.g. **Physical_Activity**) compute risk / protective scores from grouped question responses.
- Risk values per factor (e.g., physical activity, alcohol, BMI, etc.) are mapped to **suggestion IDs**.
- An assessment session yields:
  - Total / per-domain scores,
  - A list of suggestion IDs,
  - Textual recommendation content returned to the frontend. :contentReference[oaicite:9]{index=9}  

### View

- Implemented as a **React** front-end.
- Uses an **Action → Service → API** pattern:
  - Pages compose request payloads,
  - Actions are async functions,
  - Services call REST APIs exposed by the backend.
- UI includes:
  - Assessment pages (questionnaire),
  - Suggestion result page,
  - Statistics pages,
  - Administrative content management,
  - Elderly-friendly interface design (fonts, layout). :contentReference[oaicite:10]{index=10}  

### Controller

- Implemented as a **Python web backend** (Flask in the report) that exposes REST endpoints for the React frontend. :contentReference[oaicite:11]{index=11}  
- Responsibilities:
  - Input validation and parsing,
  - Calling DAL methods,
  - Running scoring functions,
  - Returning JSON responses.

Examples of controller functionality:

- `getSuggestionsByAssessmentId`  
  - Takes an assessment session ID,  
  - Retrieves associated suggestion IDs from `AssessmentSession`,  
  - Resolves them into suggestion texts,  
  - Returns the recommendation list as JSON.

- `getAnswerPercentage`  
  - Accepts a question–answer mapping,  
  - Computes answer distributions and averages using `Answer` DAL methods,  
  - Returns statistics used for admin charts. :contentReference[oaicite:12]{index=12}  

---

## Typical Workflows

### 1. Taking the Assessment & Getting Recommendations

1. User logs in / registers.
2. User fills out all questionnaire pages.
3. Client sends responses to backend.
4. Backend:
   - Stores answers in `Answer` table,
   - Runs scoring algorithm,
   - Stores scores and suggestion IDs in `AssessmentSession`,
   - Returns suggestion texts.
5. Frontend displays a **personalized suggestion report** per risk factor. :contentReference[oaicite:13]{index=13}  

### 2. Viewing Statistics (Admin)

1. Admin selects a question or a set of questions.
2. Frontend requests aggregated statistics via API.
3. Backend computes:
   - Counts per answer option,
   - Percentages, averages, binned values.
4. Frontend visualizes results as tables / charts for research use. :contentReference[oaicite:14]{index=14}  

### 3. Super-Admin Review

1. Super-admin browses all assessment sessions.
2. For a given session, backend returns:
   - Full answer set,
   - Generated suggestions.
3. Super-admin can export data and manage content (questions, static pages, media). :contentReference[oaicite:15]{index=15}  

---

## Getting Started (High-Level)

> **Note:** Exact file names and scripts may differ; please adapt paths/commands to your repository layout.

### Prerequisites

- Python 3.x
- Node.js & npm
- Microsoft SQL Server (MSSQL) instance
- Python packages (Flask, pyodbc, etc.)

### Backend (Flask)

1. Configure the MSSQL connection string in the DAL.
2. Install dependencies:
   ```bash
   pip install -r requirements.txt
