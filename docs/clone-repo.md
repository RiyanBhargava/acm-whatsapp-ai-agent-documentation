# Clone the Repository

Let's get the code on your computer!

## What is Cloning?

**Cloning** means downloading a copy of the project from GitHub to your computer.

Think of it like downloading a ZIP file, but with version control superpowers!

---

## Prerequisites

Before cloning, make sure you have:

- ✅ **Git installed** - [Download Git](https://git-scm.com/downloads)
- ✅ **Terminal/Command Prompt** access
- ✅ **Internet connection**

---

## Step-by-Step Guide

### Step 1: Open Terminal

**Windows:**
- Press `Win + R`
- Type `cmd` or `powershell`
- Press Enter

**Mac/Linux:**
- Press `Cmd + Space` (Mac) or `Ctrl + Alt + T` (Linux)
- Type `terminal`
- Press Enter

---

### Step 2: Navigate to Desired Folder

Choose where you want to save the project.

**Example - Desktop:**
```bash
cd Desktop
```

**Example - Documents:**
```bash
cd Documents
```

**Example - Custom Folder:**
```bash
cd C:\Users\YourName\Projects
```

---

### Step 3: Clone the Repository

Copy and paste this command:

```bash
git clone https://github.com/RiyanBhargava/acm-whatsapp-ai-agent.git
```

**Press Enter!**

You'll see output like:
```
Cloning into 'acm-whatsapp-ai-agent'...
remote: Enumerating objects: 25, done.
remote: Counting objects: 100% (25/25), done.
remote: Compressing objects: 100% (20/20), done.
remote: Total 25 (delta 5), reused 25 (delta 5), pack-reused 0
Receiving objects: 100% (25/25), done.
Resolving deltas: 100% (5/5), done.
```

✅ **Success!** The repository has been cloned.

---

### Step 4: Navigate into Project

```bash
cd acm-whatsapp-ai-agent
```

Now you're inside the project folder!

---

### Step 5: Verify Files

List all files to make sure everything cloned correctly:

**Windows (PowerShell/CMD):**
```bash
dir
```

**Mac/Linux:**
```bash
ls
```

You should see:
```
.gitignore
main.py
gemini_parser.py
whatsapp_automation.py
requirements.txt
README.md
```

✅ If you see these files, you're all set!

---

## Project Structure

Here's what each file does:

```
acm-whatsapp-ai-agent/
│
├── main.py                      # Main application (run this!)
├── gemini_parser.py              # Gemini AI integration
├── whatsapp_automation.py        # Selenium automation
│
├── requirements.txt              # Python dependencies
├── .env.example                  # API key template
├── .gitignore                    # Files to ignore in Git
│
├── README.md                     # Project documentation
```

---

## Alternative: Download ZIP

Don't have Git? You can download a ZIP file instead:

1. Go to: [https://github.com/RiyanBhargava/acm-whatsapp-ai-agent](https://github.com/RiyanBhargava/acm-whatsapp-ai-agent)
2. Click the green **"Code"** button
3. Select **"Download ZIP"**
4. Extract the ZIP file
5. Navigate to the extracted folder in terminal

---

## Next: Install Dependencies

Now that you have the code, let's install the required Python packages:

### Step 1: Create Virtual Environment

```bash
python -m venv .venv
```

### Step 2: Activate Virtual Environment

**Windows:**
```bash
.venv\Scripts\activate
```

**Mac/Linux:**
```bash
source .venv/bin/activate
```

You'll see `(.venv)` appear at the start of your command prompt.

---

### Step 3: Install Requirements

```bash
pip install -r requirements.txt
```

This installs:
- `google-generativeai` - For Gemini AI
- `selenium` - For browser automation
- `python-dotenv` - For environment variables

**Wait for installation to complete...**

✅ **Done!** All dependencies installed.

---

## Verify Installation

Check if packages are installed:

```bash
pip list
```

You should be able to see all of these:
```
google-generativeai    0.3.0
selenium               4.16.0
python-dotenv          1.0.0
```

---

## Quick Troubleshooting

### "git is not recognized" error?

**Solution:** Install Git from [git-scm.com](https://git-scm.com/downloads)

After installation, close and reopen your terminal.

---

### "python is not recognized" error?

**Solution:** Install Python from [python.org](https://www.python.org/downloads/)

**Important:** During installation, check **"Add Python to PATH"**

---

### Virtual environment activation issues?

**Windows PowerShell Error:**
```
execution of scripts is disabled on this system
```

**Solution:**
```bash
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

Then try activating again.

---

## File Checklist

Before moving forward, make sure you have:

- ✅ Cloned/downloaded the repository
- ✅ Navigated into project folder
- ✅ Created virtual environment (`.venv`)
- ✅ Activated virtual environment
- ✅ Installed requirements (`pip install -r requirements.txt`)
- ✅ See all Python files (`main.py`, `gemini_parser.py`, `whatsapp_automation.py`)

---

## What's Next?

You have the code and dependencies installed. Now let's look at the actual Python code!

[View Code Snippets →](code-snippets.md){ .md-button .md-button--primary }

---
