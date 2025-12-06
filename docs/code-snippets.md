# Code Walkthrough

Complete code for all three Python files with detailed explanations.

---

## 1. Overview

Our project consists of three main Python files:

1. **`main.py`** - Application controller and user interface
2. **`gemini_parser.py`** - Gemini AI integration for command parsing
3. **`whatsapp_automation.py`** - Selenium browser automation

Let's dive into each file!

---

## 2. File 1: `main.py`

**Purpose:** Main application controller that coordinates everything.

```python
import os
from gemini_parser import parse_command
from whatsapp_automation import WhatsAppAutomation


def main():
    """Main application loop"""
    # Display welcome banner
    print("=" * 60)
    print("WhatsApp AI Agent - Powered by Gemini & Selenium")
    print("=" * 60)
    print()
    
    # Check if API key is configured
    if not os.getenv('GEMINI_API_KEY'):
        print("ERROR: GEMINI_API_KEY not found in environment variables!")
        print("Please create a .env file with your API key.")
        print("See .env.example for reference.")
        return
    
    # Initialize WhatsApp automation
    print("Initializing WhatsApp automation...")
    wa = WhatsAppAutomation()
    wa.initialize_driver()
    
    # Open WhatsApp and wait for login
    if not wa.open_whatsapp():
        print("Failed to load WhatsApp Web. Exiting...")
        wa.close()
        return
    
    print()
    print("=" * 60)
    print("Agent is ready! You can now send messages.")
    print("Type your command in natural language.")
    print("Type 'quit' or 'exit' to stop the agent.")
    print("=" * 60)
    print()
    
    # Main command loop
    try:
        while True:
            # Get user command
            user_command = input("\nYou: ").strip()
            
            # Check for exit commands
            if user_command.lower() in ['quit', 'exit', 'q']:
                print("Shutting down agent...")
                break
            
            if not user_command:
                continue
            
            # Parse the command using Gemini
            print("Processing command...")
            parsed = parse_command(user_command)
            
            if not parsed:
                print("Could not understand the command. Please try again.")
                continue
            
            # Display what was understood
            print(f"\n→ Contact: {parsed['contact']}")
            print(f"→ Message: {parsed['message']}")
            
            # Ask for confirmation
            confirm = input("\nSend this message? (y/n): ").strip().lower()
            
            if confirm == 'y':
                # Send the message
                success = wa.send_message(parsed['contact'], parsed['message'])
                
                if success:
                    print("✓ Message sent successfully!")
                else:
                    print("✗ Failed to send message. Please check if the contact exists.")
            else:
                print("Message cancelled.")
    
    except KeyboardInterrupt:
        print("\n\nInterrupted by user. Shutting down...")
    
    finally:
        # Clean up
        wa.close()
        print("Agent stopped. Goodbye!")


if __name__ == "__main__":
    main()
```

### 2.1 Key Sections Explained

#### 2.1.1 Imports
```python
import os
from gemini_parser import parse_command
from whatsapp_automation import WhatsAppAutomation
```
- `os` - Access environment variables (API key)
- `gemini_parser` - Our AI command parsing module
- `whatsapp_automation` - Our browser automation module

---

#### 2.1.2 API Key Check
```python
if not os.getenv('GEMINI_API_KEY'):
    print("ERROR: GEMINI_API_KEY not found in environment variables!")
    return
```
**Why?** Prevents the app from running without proper configuration.

---

#### 2.1.3 WhatsApp Initialization
```python
wa = WhatsAppAutomation()
wa.initialize_driver()
```
- Creates automation object
- Opens Chrome browser with WhatsApp Web

---

#### 2.1.4 Main Command Loop
```python
while True:
    user_command = input("\nYou: ").strip()
    
    # Exit check
    if user_command.lower() in ['quit', 'exit', 'q']:
        break
    
    # Parse command
    parsed = parse_command(user_command)
    
    # Send message
    wa.send_message(parsed['contact'], parsed['message'])
```

**Flow:**
1. Get user input
2. Check for exit command
3. Parse command with Gemini
4. Show confirmation
5. Send message via Selenium

---

#### 2.1.5 Error Handling
```python
except KeyboardInterrupt:
    print("\n\nInterrupted by user. Shutting down...")

finally:
    wa.close()
```
- Catches `Ctrl+C` interruption
- Always closes browser properly

---

## 3. File 2: `gemini_parser.py`

**Purpose:** Uses Gemini AI to understand natural language commands.

```python
import os
import json
import google.generativeai as genai
from dotenv import load_dotenv

# Load environment variables
load_dotenv()

# Configure Gemini API
genai.configure(api_key=os.getenv('GEMINI_API_KEY'))

# System prompt for the agent
SYSTEM_PROMPT = """You are a WhatsApp message-sending agent.

Your job:
1. Read the user's natural language command.
2. Identify:
   - The contact name (the person to message)
   - The final message content that should be sent
3. Return your answer ONLY in the following JSON format:

{
  "contact": "<name of recipient>",
  "message": "<message the agent must send>"
}

Rules:
- Do not add emojis unless the user explicitly says to.
- If the user gives multiple sentences, join them into one message unless otherwise instructed.
- If the user does not specify the message content, politely ask for clarification.
- Never invent a contact name that the user has not provided.
- Preserve the user's original phrasing for the message exactly as said.

Examples:
User: "hey gemini send message to mom saying hi i am out"
Output: {"contact": "mom", "message": "hi i am out"}

User: "tell John I'll meet him at 6 near the metro station"
Output: {"contact": "John", "message": "I'll meet him at 6 near the metro station"}
"""


def parse_command(user_command):
    """
    Uses Gemini API to parse the user command and extract contact and message.
    
    Args:
        user_command (str): The natural language command from the user
        
    Returns:
        dict: Dictionary with 'contact' and 'message' keys, or None if parsing fails
    """
    try:
        # Initialize the model
        model = genai.GenerativeModel('gemini-2.5-flash')
        
        # Create the full prompt
        full_prompt = f"{SYSTEM_PROMPT}\n\nUser command: {user_command}\n\nReturn only the JSON response:"
        
        # Generate response
        response = model.generate_content(full_prompt)
        response_text = response.text.strip()
        
        # Extract JSON from response (remove markdown code blocks if present)
        if '```json' in response_text:
            response_text = response_text.split('```json')[1].split('```')[0].strip()
        elif '```' in response_text:
            response_text = response_text.split('```')[1].split('```')[0].strip()
        
        # Parse JSON
        parsed_data = json.loads(response_text)
        
        # Validate required fields
        if 'contact' in parsed_data and 'message' in parsed_data:
            return parsed_data
        else:
            print("Error: Missing required fields in response")
            return None
            
    except json.JSONDecodeError as e:
        print(f"Error parsing JSON: {e}")
        print(f"Response received: {response_text}")
        return None
    except Exception as e:
        print(f"Error calling Gemini API: {e}")
        return None


if __name__ == "__main__":
    # Test the parser
    test_command = "send message to mom saying hi i am out"
    result = parse_command(test_command)
    if result:
        print(f"Parsed command: {json.dumps(result, indent=2)}")
```

### 3.1 Key Sections Explained

#### 3.1.1 Environment Setup
```python
load_dotenv()
genai.configure(api_key=os.getenv('GEMINI_API_KEY'))
```
- Loads `.env` file
- Configures Gemini with API key

---

#### 3.1.2 System Prompt
```python
SYSTEM_PROMPT = """You are a WhatsApp message-sending agent.
...
"""
```
**Purpose:** Instructs Gemini how to parse commands.

**Key Instructions:**
- Extract contact name and message
- Return JSON format
- Preserve user's exact wording

---

#### 3.1.3 Parse Function
```python
def parse_command(user_command):
    model = genai.GenerativeModel('gemini-2.5-flash')
    full_prompt = f"{SYSTEM_PROMPT}\n\nUser command: {user_command}"
    response = model.generate_content(full_prompt)
    parsed_data = json.loads(response_text)
    return parsed_data
```

**Steps:**
1. Initialize Gemini model
2. Combine system prompt + user command
3. Get AI response
4. Parse JSON from response
5. Return structured data

---

#### 3.1.4 JSON Extraction
```python
if '```json' in response_text:
    response_text = response_text.split('```json')[1].split('```')[0].strip()
```
**Why?** Gemini sometimes wraps JSON in markdown code blocks. This extracts the actual JSON.

---

#### 3.1.5 Error Handling
```python
except json.JSONDecodeError as e:
    print(f"Error parsing JSON: {e}")
    return None
```
Catches and reports parsing errors gracefully.

---

## 4. File 3: `whatsapp_automation.py`

**Purpose:** Controls Chrome browser to interact with WhatsApp Web.

```python
import time
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options


class WhatsAppAutomation:
    """Handles WhatsApp Web automation using Selenium"""
    
    def __init__(self):
        """Initialize the Chrome WebDriver"""
        self.driver = None
        
    def initialize_driver(self):
        """Set up Chrome WebDriver with options"""
        chrome_options = Options()
        
        # Create absolute path for user data directory
        import os
        user_data_dir = os.path.abspath("./chrome_profile")
        chrome_options.add_argument(f"--user-data-dir={user_data_dir}")
        
        # Additional options to prevent crashes
        chrome_options.add_argument("--no-sandbox")
        chrome_options.add_argument("--disable-dev-shm-usage")
        chrome_options.add_argument("--disable-blink-features=AutomationControlled")
        chrome_options.add_argument("--remote-debugging-port=9222")
        chrome_options.add_experimental_option("excludeSwitches", ["enable-automation"])
        chrome_options.add_experimental_option('useAutomationExtension', False)
        
        self.driver = webdriver.Chrome(options=chrome_options)
        self.driver.maximize_window()
        
    def open_whatsapp(self):
        """Open WhatsApp Web and wait for QR scan"""
        print("Opening WhatsApp Web...")
        self.driver.get("https://web.whatsapp.com/")
        
        print("Please scan the QR code to log in...")
        print("Waiting for WhatsApp to load...")
        
        try:
            # Wait for the main chat interface to load (indicating successful login)
            WebDriverWait(self.driver, 60).until(
                EC.presence_of_element_located((By.XPATH, '//div[@contenteditable="true"][@data-tab="3"]'))
            )
            print("WhatsApp loaded successfully!")
            return True
        except Exception as e:
            print(f"Error loading WhatsApp: {e}")
            return False
    
    def send_message(self, contact_name, message):
        """
        Send a message to a specific contact on WhatsApp
        
        Args:
            contact_name (str): Name of the contact
            message (str): Message to send
            
        Returns:
            bool: True if message sent successfully, False otherwise
        """
        try:
            # Search for contact
            print(f"Searching for contact: {contact_name}")
            
            # Click on search box
            search_box = WebDriverWait(self.driver, 10).until(
                EC.presence_of_element_located((By.XPATH, '//div[@contenteditable="true"][@data-tab="3"]'))
            )
            search_box.click()
            time.sleep(0.5)
            
            # Type contact name
            search_box.send_keys(contact_name)
            time.sleep(1)
            
            # Click on the contact
            contact = WebDriverWait(self.driver, 10).until(
                EC.presence_of_element_located((By.XPATH, f'//span[@title="{contact_name}"]'))
            )
            contact.click()
            time.sleep(1)
            
            # Find message input box
            message_box = WebDriverWait(self.driver, 10).until(
                EC.presence_of_element_located((By.XPATH, '//div[@contenteditable="true"][@data-tab="10"]'))
            )
            
            # Type and send message
            print(f"Sending message: {message}")
            message_box.click()
            message_box.send_keys(message)
            time.sleep(0.5)
            message_box.send_keys(Keys.RETURN)
            
            print("Message sent successfully!")
            return True
            
        except Exception as e:
            print(f"Error sending message: {e}")
            return False
    
    def close(self):
        """Close the browser"""
        if self.driver:
            self.driver.quit()
            print("Browser closed.")


if __name__ == "__main__":
    # Test the automation
    wa = WhatsAppAutomation()
    wa.initialize_driver()
    
    if wa.open_whatsapp():
        input("\nPress Enter after WhatsApp is loaded to test sending a message...")
        
        # Test sending a message
        contact = input("Enter contact name: ")
        message = input("Enter message: ")
        
        wa.send_message(contact, message)
        
        input("\nPress Enter to close the browser...")
    
    wa.close()
```

### 4.1 Key Sections Explained

#### 4.1.1 Class Structure
```python
class WhatsAppAutomation:
    def __init__(self):
        self.driver = None
```
**Object-oriented approach:** Encapsulates all automation logic in one class.

---

#### 4.1.2 Chrome Options
```python
chrome_options.add_argument("--user-data-dir=./chrome_profile")
chrome_options.add_argument("--no-sandbox")
chrome_options.add_argument("--disable-dev-shm-usage")
```

**Why these options?**
- `--user-data-dir` → Saves login session (no QR scan every time)
- `--no-sandbox` → Prevents crashes in some environments
- `--disable-dev-shm-usage` → Prevents memory issues

---

#### 4.1.3 WebDriverWait
```python
WebDriverWait(self.driver, 60).until(
    EC.presence_of_element_located((By.XPATH, '//div[@contenteditable="true"]'))
)
```

**Purpose:** Waits for elements to load before interacting.

**Parameters:**
- `60` → Maximum wait time (seconds)
- `EC.presence_of_element_located` → Wait condition

---

#### 4.1.4 XPath Selectors
```python
search_box = driver.find_element(By.XPATH, '//div[@contenteditable="true"][@data-tab="3"]')
```

**What is XPath?** 
A way to locate elements on a webpage.

**This XPath finds:**
- A `<div>` element
- That is `contenteditable="true"`
- With attribute `data-tab="3"` (WhatsApp's search box)

---

#### 4.1.5 Sending Keys
```python
search_box.send_keys(contact_name)
message_box.send_keys(message)
message_box.send_keys(Keys.RETURN)
```

**Methods:**
- `send_keys(text)` → Types text
- `Keys.RETURN` → Presses Enter key

---

## 5. Running the Application

### 5.1 Set up environment
```bash
# Create virtual environment
python -m venv .venv

# Activate (Windows)
.venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

---

### 5.2 Configure API key
Create `.env` file:
```env
GEMINI_API_KEY=your_api_key_here
```

---

### 5.3 Run the application
```bash
python main.py
```

---

### 5.4 Use the agent
```
You: send message to John saying meeting at 3pm

Processing command...

→ Contact: John
→ Message: meeting at 3pm

Send this message? (y/n): y
✓ Message sent successfully!
```

---

## 6. Testing Individual Files

### 6.1 Test `gemini_parser.py`
```bash
python gemini_parser.py
```
Runs the test code at the bottom of the file.

---

### 6.2 Test `whatsapp_automation.py`
```bash
python whatsapp_automation.py
```
Opens WhatsApp and lets you manually test sending a message.

---

## 7. Common Code Patterns

### 7.1 Try-Except Blocks
```python
try:
    # Risky operation
    result = parse_command(user_command)
except Exception as e:
    print(f"Error: {e}")
    return None
```
**Why?** Prevents crashes, provides error messages.

---

### 7.2 Context Managers (implicit)
```python
finally:
    wa.close()
```
**Why?** Ensures cleanup (closing browser) even if errors occur.

---

### 7.3 String Formatting
```python
print(f"→ Contact: {parsed['contact']}")
print(f"→ Message: {parsed['message']}")
```
**f-strings:** Modern Python string formatting.

---

### 7.4 Conditional Returns
```python
if not parsed:
    return
```
**Early returns:** Exit function early if conditions aren't met.

---

## 8. Dependencies Explained

From `requirements.txt`:

```txt
google-generativeai>=0.3.0
selenium>=4.0.0
python-dotenv>=1.0.0
```

### `google-generativeai`
- Google's official Gemini API client
- Provides `genai.GenerativeModel()`
- Handles API calls and authentication

### `selenium`
- Browser automation framework
- Controls Chrome like a human would
- Finds elements, clicks, types, etc.

### `python-dotenv`
- Loads environment variables from `.env` file
- Keeps secrets (API keys) out of code
- `load_dotenv()` reads `.env` file

---

## Next Steps

Now that you understand the code, try:

1. **Run the application** - Follow the instructions above
2. **Modify the system prompt** - Change how Gemini parses commands
3. **Add new features** - Schedule messages, send media files
4. **Customize the UI** - Add colors, better formatting

---

## Troubleshooting

### "Module not found" error?
```bash
pip install -r requirements.txt
```

### "API key not found" error?
Check `.env` file exists and contains:
```env
GEMINI_API_KEY="your_key_here"
```

### Chrome won't open?
Make sure Chrome browser is installed.

### WhatsApp elements not found?
WhatsApp Web updates their HTML. You may need to update XPaths.

---

**Congratulations!** You now understand how the entire WhatsApp AI Agent works! 🎉

[Back to Overview](index.md){ .md-button }
