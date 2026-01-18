# 🤖 WhatsApp AI Interview Automation

Automate your recruitment process with AI-powered WhatsApp interviews using n8n, Google Gemini, and Supabase.

![n8n](https://img.shields.io/badge/n8n-Workflow-red?style=flat-square)
![WhatsApp](https://img.shields.io/badge/WhatsApp-Business%20API-25D366?style=flat-square)
![Google Gemini](https://img.shields.io/badge/Google-Gemini%202.0-4285F4?style=flat-square)
![Supabase](https://img.shields.io/badge/Supabase-Database-3ECF8E?style=flat-square)

## 📋 Table of Contents

- [About](#about)
- [Features](#features)
- [How It Works](#how-it-works)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Database Schema](#database-schema)
- [Customization](#customization)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

## About

This n8n workflow automates the entire interview process via WhatsApp. Candidates submit their CVs (PDF, text, or voice), and an AI interviewer named **Sarah** conducts professional, conversational interviews in real-time.

**Perfect for:**

- HR departments handling high-volume recruitment
- Startups automating initial screening
- Educational institutions conducting student interviews
- Recruitment agencies streamlining candidate evaluation

## Features

- **Multi-format CV Processing** - Accept PDFs, text messages, or voice recordings
- **AI-Powered Data Extraction** - Automatically parse candidate information
- **Conversational AI Interviewer** - Natural, human-like interview conversations
- **Audio Transcription** - Voice-to-text using Google Gemini 2.0 Flash
- **Persistent Memory** - Context-aware conversations across sessions
- **Database Integration** - Store candidates and interview data in Supabase
- **Dynamic Question Generation** - Personalized questions based on skills
- **Interview Tracking** - Monitor progress and store complete histories

## How It Works

### Workflow Overview

```
WhatsApp Message → Message Type Detection → Processing Path
                                          ├── PDF: Extract text → CV Processor
                                          ├── Audio: Transcribe → CV Processor
                                          └── Text: Direct → CV Processor
                                                              ↓
                                          CV Data Extracted & Stored in Supabase
                                                              ↓
                                          AI Interviewer (Sarah) Conducts Interview
                                                              ↓
                                          Responses Sent Back via WhatsApp
```

### Interview Process

**Phase 1: CV Submission**

- Candidate sends CV via WhatsApp (PDF/text/audio)
- System extracts information and generates interview questions
- Data stored in Supabase database

**Phase 2: Interview Introduction**

- Sarah greets the candidate
- Explains the interview process
- Asks if they're ready to begin

**Phase 3: Question & Answer**

- Asks ONE question at a time
- Candidate responds (text or voice)
- Sarah provides encouraging feedback
- Continues until all questions covered

**Phase 4: Conclusion**

- Thanks the candidate
- Provides timeline for next steps
- Ends professionally

## Prerequisites

Before you begin, you need:

- **n8n** - Workflow automation platform ([Install Guide](https://docs.n8n.io/hosting/installation/))
- **WhatsApp Business API** - For messaging ([Sign Up](https://business.whatsapp.com/))
- **Google Gemini API Key** - For AI processing ([Get API Key](https://ai.google.dev/))
- **Supabase Account** - For database ([Sign Up](https://supabase.com/))
- **PostgreSQL** - For chat memory (included with Supabase)

## Installation

### Step 1: Clone or Download

Download the workflow file:

```bash
# Option 1: Clone the repository
git clone https://github.com/HAQ9827/n8n-whatsapp-job-alert-automation.git
cd n8n-whatsapp-job-alert-automation

# Option 2: Download the JSON file directly
# Download: job-alert-automation-n8n.json
```

### Step 2: Import to n8n

1. Open your n8n instance
2. Click **Workflows** → **Import from File**
3. Select `job-alert-automation-n8n.json`
4. Click **Import**

### Step 3: Add Credentials

#### WhatsApp Business API

1. Go to **Credentials** in n8n
2. Click **Add Credential** → **WhatsApp**
3. Enter:
   - **Phone Number ID**: Your WhatsApp Business phone number ID
   - **Access Token**: Your WhatsApp API access token

#### Google Gemini API

1. Click **Add Credential** → **Google PaLM API**
2. Enter:
   - **API Key**: Your Gemini API key

#### Supabase

1. Click **Add Credential** → **Supabase**
2. Enter:
   - **Host**: `https://your-project.supabase.co`
   - **Service Role Secret**: Your Supabase service role key

#### PostgreSQL

1. Click **Add Credential** → **PostgreSQL**
2. Enter your database connection details:
   - **Host**: Your Supabase or PostgreSQL host
   - **Database**: Database name
   - **User**: Database user
   - **Password**: Database password
   - **Port**: 5432 (default)

### Step 4: Create Database Tables

Go to your Supabase SQL Editor and run:

```sql
-- Create cv_data table
CREATE TABLE cv_data (
  id BIGSERIAL PRIMARY KEY,
  session_id TEXT UNIQUE NOT NULL,
  name TEXT,
  email TEXT,
  phone TEXT,
  skills TEXT[],
  interview_questions TEXT[],
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Create index for faster lookups
CREATE INDEX idx_session_id ON cv_data(session_id);
```

The `n8n_chat_histories` table will be created automatically by the Postgres Chat Memory node.

## Configuration

### Update Phone Number

Open the workflow and find the **"interview send msg"** node:

```json
{
  "phoneNumberId": "REPLACE_WITH_YOUR_PHONE_NUMBER_ID"
}
```

Replace with your actual WhatsApp Business phone number ID.

### Test the Workflow

1. Click **Execute Workflow** in n8n
2. Send a test message to your WhatsApp number
3. Check execution logs for errors
4. Verify data appears in Supabase

## Usage

### For Candidates

**Send Your CV in Any Format:**

1. **PDF Document** - Attach your CV PDF
2. **Text Message** - Type your CV details
3. **Voice Message** - Record your CV information

**Interview Process:**

1. Send your CV to the WhatsApp number
2. Wait for Sarah to introduce herself
3. Answer questions one at a time
4. Use text or voice for responses
5. Complete the interview
6. Receive confirmation and next steps

### For Recruiters

**View Candidate Data:**

Go to Supabase → Table Editor → `cv_data`

```sql
-- View all candidates
SELECT * FROM cv_data ORDER BY created_at DESC;

-- View specific candidate
SELECT * FROM cv_data WHERE email = 'candidate@example.com';

-- View interview history
SELECT * FROM n8n_chat_histories WHERE session_id = 'PHONE_NUMBER';
```

**Monitor Interviews:**

Check the n8n execution history to see:

- Successful/failed interviews
- Processing times
- Error logs

## Database Schema

### cv_data Table

| Column              | Type        | Description                               |
| ------------------- | ----------- | ----------------------------------------- |
| id                  | BIGSERIAL   | Primary key                               |
| session_id          | TEXT        | WhatsApp phone number (unique identifier) |
| name                | TEXT        | Candidate full name                       |
| email               | TEXT        | Email address                             |
| phone               | TEXT        | Phone number                              |
| skills              | TEXT[]      | Array of candidate skills                 |
| interview_questions | TEXT[]      | Generated interview questions             |
| created_at          | TIMESTAMPTZ | Record creation timestamp                 |

### n8n_chat_histories Table

Auto-created by n8n. Stores conversation history for context persistence.

## Customization

### Change Sarah's Personality

Edit the **Interview Conductor** node system message:

```
Current: "You are Sarah, a professional and friendly AI interviewer..."

Options:
- Casual: "You are Sarah, a laid-back and conversational interviewer..."
- Formal: "You are Sarah, a formal corporate interviewer..."
- Technical: "You are Sarah, a technical interviewer focused on coding..."
```

### Modify Question Generation

Edit the **CV Processor Agent** prompt to change question types:

```
Current: Generates 5 general interview questions

Customize:
- "Generate 3 technical and 2 behavioral questions"
- "Focus questions on leadership experience"
- "Create beginner-friendly questions for entry-level"
```

### Add More CV Fields

Update both the CV Processor prompt and database schema:

```sql
ALTER TABLE cv_data ADD COLUMN linkedin TEXT;
ALTER TABLE cv_data ADD COLUMN years_experience INTEGER;
ALTER TABLE cv_data ADD COLUMN location TEXT;
```

Then update the **Create a row** node to include these fields.

### Change Language

Update all prompts to your language:

```javascript
// Spanish
"Eres Sarah, una entrevistadora profesional y amigable...";

// French
"Vous êtes Sarah, une intervieweuse professionnelle et amicale...";
```

## Troubleshooting

### CV Not Processing

**Problem:** PDF CV not being processed

**Solutions:**

- Ensure PDF contains actual text (not just images)
- Check Gemini API quota in Google Cloud Console
- Test with a simple text CV first
- Review CV Processor Agent execution logs

### Interview Not Starting

**Problem:** Sarah doesn't respond after CV submission

**Solutions:**

- Verify CV data was saved to Supabase:
  ```sql
  SELECT * FROM cv_data ORDER BY created_at DESC LIMIT 1;
  ```
- Check PostgreSQL connection for chat memory
- Ensure Interview Conductor node is connected properly
- Review execution logs for errors

### Messages Not Sending

**Problem:** WhatsApp messages not being delivered

**Solutions:**

- Verify WhatsApp Business API credentials
- Check phone number format (use international format: +1234567890)
- Ensure WhatsApp number is verified
- Check WhatsApp API rate limits in dashboard

### Audio Transcription Failing

**Problem:** Voice messages not being transcribed

**Solutions:**

- Verify Gemini API key is valid
- Check audio file format compatibility
- Ensure audio quality is clear
- Test with shorter audio clips first

### Memory Not Persisting

**Problem:** Sarah doesn't remember previous conversation

**Solutions:**

- Verify PostgreSQL connection:
  ```sql
  SELECT * FROM n8n_chat_histories LIMIT 5;
  ```
- Check session_id consistency (phone number format)
- Ensure Postgres Chat Memory node is configured
- Review database permissions

### Enable Debug Logging

Add this to your n8n environment:

```bash
export N8N_LOG_LEVEL=debug
```

Or in Docker:

```yaml
environment:
  - N8N_LOG_LEVEL=debug
```

## Contributing

Contributions are welcome! Here's how to contribute:

### Report Bugs

1. Check if issue already exists
2. Create new issue with:
   - Clear description
   - Steps to reproduce
   - Expected vs actual behavior
   - Screenshots/logs if applicable

### Suggest Features

1. Open issue with `enhancement` label
2. Describe the feature and benefits
3. Provide examples if possible

### Submit Pull Requests

1. Fork the repository
2. Create feature branch: `git checkout -b feature/my-feature`
3. Make your changes
4. Test thoroughly
5. Commit: `git commit -m 'Add new feature'`
6. Push: `git push origin feature/my-feature`
7. Create Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments

- **n8n** - Workflow automation platform
- **Google Gemini** - AI language model
- **Supabase** - Database and backend services
- **WhatsApp Business API** - Messaging infrastructure

## Support

- **Issues**: [GitHub Issues](https://github.com/yourusername/whatsapp-ai-interview/issues)
- **Documentation**: [n8n Docs](https://docs.n8n.io/)
- **Community**: [n8n Community](https://community.n8n.io/)

## Roadmap

- [ ] Video interview support
- [ ] Multi-language support
- [ ] Interview performance scoring
- [ ] Integration with ATS systems
- [ ] Automated interview reports
- [ ] Sentiment analysis
- [ ] Email notifications for recruiters

---

**Made with ❤️ for automated recruiting**

_Note: This is an AI-powered system. Human oversight is recommended for final hiring decisions._
