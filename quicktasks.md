# QuickTasks for Mac

A lightweight, Python-powered GUI utility to instantly add tasks to Google Tasks using a global keyboard shortcut.

## Project Architecture

* **Backend:** Python 3.14 with Google Tasks API.
* **Frontend:** Tkinter (GUI) for a native "popup" feel.
* **Trigger:** macOS Automator Application + System Shortcuts.
* **Authentication:** OAuth 2.0 (stored locally in `token.pickle`).

---

## Project Structure

```text
~/Documents/QuickTasks/
├── credentials.json  # Your Google Cloud API Key
├── token.pickle      # Your active login session (auto-generated)
├── add_task.py       # The main Python logic
└── venv/             # Python virtual environment (isolated libraries)

```

### What is the `venv` folder?

The `venv` (Virtual Environment) is a private "sandbox" for this project.

* **Isolation:** It keeps the Google API libraries separate from your Mac's system files.
* **Stability:** It prevents other Python projects or macOS updates from breaking your QuickTasks app.
* **Portability:** It contains the specific `python3` executable used in the Automator script to ensure the app always finds its required libraries.

---

## Setting Up Google API Credentials

1. **Create Project:** Go to the [Google Cloud Console](https://console.cloud.google.com/) and create a project named `QuickTasks`.
2. **Enable API:** Search for **"Google Tasks API"** in the Library and click **Enable**.
3. **Consent Screen:** Go to **OAuth Consent Screen**, choose **External**, and fill in the app name/email.
* **Test Users:** Add your own Gmail address under "Test Users" so Google allows you to log in.


4. **Create ID:** Go to **Credentials > Create Credentials > OAuth Client ID**. Select **Desktop App**.
5. **Download:** Click the **Download JSON** icon for the new ID. Rename the file to `credentials.json` and move it to `~/Documents/QuickTasks/`.

---

## Setup & Launch

### 1. Create the Automator App

* Open **Automator** and create a new **Application**.
* Add the **Run Shell Script** action.
* Paste the following:
`~/Documents/QuickTasks/venv/bin/python3 ~/Documents/QuickTasks/add_task.py`
* Save as **QuickTask.app** in your `/Applications` folder.

### 2. Assign the Keyboard Shortcut

* Open the macOS **Shortcuts** app.
* Create a new shortcut using the **Open App** action.
* Select **QuickTask.app**.
* In the Shortcut details sidebar (the "i" icon), click **Add Keyboard Shortcut** and record your keys (e.g., `Option + Command + Q`).

---

## The Python Script (`add_task.py`)

```python
import os, pickle, tkinter as tk
from tkinter import messagebox
from googleapiclient.discovery import build
from google_auth_oauthlib.flow import InstalledAppFlow
from google.auth.transport.requests import Request

# Configuration
SCOPES = ['https://www.googleapis.com/auth/tasks']
BASE_DIR = os.path.expanduser("~/Documents/QuickTasks/")
CREDENTIALS_PATH = os.path.join(BASE_DIR, "credentials.json")
TOKEN_PATH = os.path.join(BASE_DIR, "token.pickle")

def get_service():
    creds = None
    if os.path.exists(TOKEN_PATH):
        with open(TOKEN_PATH, 'rb') as token:
            creds = pickle.load(token)
    if not creds or not creds.valid:
        if creds and creds.expired and creds.refresh_token:
            creds.refresh(Request())
        else:
            flow = InstalledAppFlow.from_client_secrets_file(CREDENTIALS_PATH, SCOPES)
            creds = flow.run_local_server(port=0)
        with open(TOKEN_PATH, 'wb') as token:
            pickle.dump(creds, token)
    return build('tasks', 'v1', credentials=creds)

def submit_task(event=None):
    task_text = entry.get().strip()
    if task_text:
        try:
            service = get_service()
            service.tasks().insert(tasklist='@default', body={'title': task_text}).execute()
            root.destroy()
        except Exception as e:
            messagebox.showerror("Error", str(e))
    else:
        root.destroy()

# GUI Setup
root = tk.Tk()
root.title("Quick Task")
root.attributes('-topmost', True) 

window_width, window_height = 400, 120
screen_width = root.winfo_screenwidth()
screen_height = root.winfo_screenheight()
center_x = int(screen_width/2 - window_width/2)
center_y = int(screen_height/2 - window_height/2)
root.geometry(f'{window_width}x{window_height}+{center_x}+{center_y}')

label = tk.Label(root, text="What needs to be done?", font=("Arial", 12))
label.pack(pady=10)

entry = tk.Entry(root, font=("Arial", 14), width=30)
entry.pack(padx=20)
entry.focus_force() 

root.bind('<Return>', submit_task)
root.bind('<Escape>', lambda e: root.destroy())
root.mainloop()

```

---

## `credentials.json` Template

If you cannot download the file, create a file named `credentials.json` and paste this structure, filling in the values from your Google Cloud Console:

```json
{
  "installed": {
    "client_id": "YOUR_CLIENT_ID.apps.googleusercontent.com",
    "project_id": "your-project-id",
    "auth_uri": "https://accounts.google.com/o/oauth2/auth",
    "token_uri": "https://oauth2.googleapis.com/token",
    "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
    "client_secret": "YOUR_CLIENT_SECRET",
    "redirect_uris": ["http://localhost"]
  }
}

```

---
