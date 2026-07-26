# 🚀 ResumeForge AI

**AI-powered, end-to-end job application automation** — built on [n8n](https://n8n.io/), Google Workspace, and the Google Gemini API.

> 💡 **Suggested name:** `ResumeForge AI`
> *(Alternatives if you want a different vibe: `ApplyFlow AI`, `JobPilot Resume`, `AutoApply AI`)*

ResumeForge AI automatically searches LinkedIn for jobs matching your preferences, extracts the full job description, and uses Google Gemini to rewrite your resume so it's tailored and ATS-optimized for that exact posting — then saves everything to Google Docs and logs it in a Google Sheet. No manual copy-pasting, no generic one-size-fits-all resume.

---

## ✨ Features

- 🔍 **Automated job discovery** — searches LinkedIn based on keyword, location, experience level, remote/on-site preference, job type, and Easy Apply filters pulled from a Google Sheet
- 🧠 **AI-powered resume tailoring** — a Gemini-backed AI Agent rewrites your resume to match each job's specific keywords and requirements
- 📄 **Automatic document generation** — creates a new Google Docs file per application, fully written and formatted
- ☁️ **Cloud-native storage** — every generated resume is saved directly to Google Drive
- 📊 **Centralized tracking** — a live Google Sheet logs every job applied to, along with the company, description, and resume link
- 🔁 **Batch processing** — loops through every matching job posting in a single run
- ✅ **ATS-safe output** — plain-text formatting, standard section headers, and honest keyword matching (no fabricated skills or experience)

---

## 🧭 Workflow Overview

| Step | Action |
|------|--------|
| 1️⃣ | Read job search preferences from a Google Sheet |
| 2️⃣ | Build a LinkedIn job search URL from those preferences |
| 3️⃣ | Fetch LinkedIn search results and extract job posting links |
| 4️⃣ | Loop over each job link |
| 5️⃣ | Fetch and parse job title, company, location, and full description |
| 6️⃣ | Send job details + resume template to the AI Agent |
| 7️⃣ | Gemini generates a tailored, ATS-friendly resume |
| 8️⃣ | Create a new Google Doc and write the resume into it |
| 9️⃣ | Save the document to Google Drive |
| 🔟 | Append job URL, company info, and resume link to the tracking sheet |

---

## 🏗️ Architecture

```
┌─────────────────────┐
│  Google Sheet        │  ← Job search preferences
│  (Preferences)        │
└──────────┬───────────┘
           │
           ▼
┌─────────────────────┐
│  Build LinkedIn URL   │  (JavaScript Code node)
└──────────┬───────────┘
           │
           ▼
┌─────────────────────┐
│  HTTP Request         │  → LinkedIn Search Results Page
└──────────┬───────────┘
           │
           ▼
┌─────────────────────┐
│  HTML Extraction      │  → Job posting links
└──────────┬───────────┘
           │
           ▼
┌─────────────────────┐
│  Loop Over Items      │  (Split in Batches)
└──────────┬───────────┘
           │
           ▼
┌─────────────────────┐
│  HTTP Request         │  → Individual Job Posting Page
└──────────┬───────────┘
           │
           ▼
┌─────────────────────┐
│  HTML Extraction      │  → Title, Company, Location, Description
└──────────┬───────────┘
           │
           ▼
┌─────────────────────┐        ┌───────────────────────┐
│  AI Agent             │ ───▶ │  Google Gemini Chat Model │
│  (Resume Rewriter)     │  ◀── │                        │
└──────────┬───────────┘        └───────────────────────┘
           │
           ▼
┌─────────────────────┐
│  Google Docs          │  → Create + write resume
└──────────┬───────────┘
           │
           ▼
┌─────────────────────┐
│  Google Drive         │  → Store final resume
└──────────┬───────────┘
           │
           ▼
┌─────────────────────┐
│  Google Sheet         │  → Log application details
│  (Tracking)            │
└─────────────────────┘
```

---

## 🛠️ Tech Stack

| Category | Technology |
|---|---|
| **Automation Engine** | [n8n](https://n8n.io/) |
| **AI Model** | Google Gemini API (via LangChain Chat Model node) |
| **AI Orchestration** | n8n AI Agent |
| **Data Source** | Google Sheets |
| **Job Source** | LinkedIn (public search & job pages) |
| **Document Generation** | Google Docs |
| **Storage** | Google Drive |
| **Web Scraping** | HTTP Request + HTML Extraction nodes |
| **Scripting** | JavaScript (Code node) |
| **Data Format** | JSON |

---

## ✅ Prerequisites

Before setting this up, make sure you have:

- An [n8n](https://n8n.io/) instance (self-hosted or n8n Cloud)
- A **Google Cloud project** with the following APIs enabled:
  - Google Sheets API
  - Google Docs API
  - Google Drive API
- A **Google Gemini API key** ([Google AI Studio](https://aistudio.google.com/))
- A Google Sheet set up with two tabs:
  - `Preference` — job search filters
  - A tracking tab for logged applications
- Valid Google OAuth2 credentials configured in n8n

---

## 📦 Installation

1. **Clone this repository**
   ```bash
   git clone https://github.com/<your-username>/resumeforge-ai.git
   cd resumeforge-ai
   ```

2. **Import the workflow into n8n**
   - Open your n8n instance
   - Go to **Workflows → Import from File**
   - Select `My_workflow_3.json` from this repo

3. **Install/verify required n8n nodes**
   - `n8n-nodes-base` (Google Sheets, Google Docs, HTTP Request, HTML, Code, Split Out, Split In Batches)
   - `@n8n/n8n-nodes-langchain` (AI Agent, Google Gemini Chat Model)

---

## ⚙️ Configuration

### 1. Google Sheet — Preferences Tab

Create a sheet named **`Job finder`** with a **`Preference`** tab containing these columns:

| Column | Example Value |
|---|---|
| `Keyword` | `Software Engineer` |
| `Location` | `Bangalore, India` |
| `Experience Level` | `Entry level, Associate` |
| `Remote` | `Remote, Hybrid` |
| `job type` | `Full-time` |
| `Easy Apply` | `true` |

### 2. Credentials in n8n

| Credential | Used By |
|---|---|
| Google Sheets OAuth2 | Read preferences / write tracking log |
| Google Docs OAuth2 | Create & update resume documents |
| Google Drive OAuth2 | Store generated resumes |
| Google Gemini API Key | AI Agent's language model |

### 3. Resume Template

Add your base resume as plain text inside the AI Agent's prompt (or connect it as a separate input). The AI Agent uses this as the **source of truth** — it will only reorganize, rephrase, and emphasize real content from your resume, never invent new skills or experience.

---

## 🔑 Environment Variables

While most credentials are managed through n8n's credential manager, if you deploy this via Docker/CLI you may want to configure:

```env
GEMINI_API_KEY=your_google_gemini_api_key
GOOGLE_CLIENT_ID=your_google_oauth_client_id
GOOGLE_CLIENT_SECRET=your_google_oauth_client_secret
N8N_ENCRYPTION_KEY=your_n8n_encryption_key
```

> ⚠️ Never commit real API keys or credentials to version control. Use `.env` files and add them to `.gitignore`.

---

## ⚡ How It Works

1. The workflow reads your job preferences from Google Sheets and dynamically constructs a LinkedIn search URL with the correct filter codes (experience level, work type, job type, Easy Apply).
2. It scrapes the search results page and extracts every job posting link.
3. For each job link, it visits the page and extracts the title, company, location, and full job description.
4. This data — plus your base resume — is passed to an **AI Agent** powered by **Google Gemini**, which is prompted to act as an ATS resume optimization specialist.
5. The AI rewrites your resume to mirror the job's keywords and requirements *without fabricating any experience*, output as clean, ATS-safe plain text.
6. A new Google Doc is created and the tailored resume is written into it, then saved to Google Drive.
7. Finally, the job URL, company name, company description, and resume link are appended to your tracking sheet — giving you a running log of every tailored application.

---

## 📁 Folder Structure (Example)

```
resumeforge-ai/
├── My_workflow_3.json        # n8n workflow export
├── README.md                 # Project documentation
└── screenshots/
    ├── workflow-overview.png
    ├── generated-resume.png
    └── tracking-sheet.png
```

---

## 🔮 Future Improvements

- [ ] Add support for additional job boards (Indeed, Glassdoor, Naukri)
- [ ] Auto-generate a tailored **cover letter** alongside the resume
- [ ] Add a scoring system to rank jobs by ATS match percentage
- [ ] Slack/Email notifications when a new resume is generated
- [ ] Convert output to PDF automatically
- [ ] Add deduplication to avoid processing the same job posting twice
- [ ] Build a simple dashboard to visualize application history

---

## 📄 License

This project is licensed under the **MIT License** — feel free to use, modify, and distribute it with attribution.

```
MIT License

Copyright (c) 2026

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files, to deal in the Software
without restriction, including without limitation the rights to use, copy,
modify, merge, publish, distribute, sublicense, and/or sell copies of the
Software, subject to the inclusion of the above copyright notice in all
copies or substantial portions of the Software.
```

---

## 👤 Author

**Debayani Das**

🔗 [LinkedIn](https://www.linkedin.com/in/debayani-das12/) · [GitHub](https://github.com/Debayani13)

---

<p align="center">⭐ If you found this project useful, consider giving it a star!</p>
