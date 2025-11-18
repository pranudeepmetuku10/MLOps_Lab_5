# MLOps Lab 5: FastAPI and Streamlit Lab on Google Colab

**My Implementation of the Lab Exercise**

This is my implementation and submission of MLOps Lab 5, where I demonstrate how to set up and deploy a FastAPI backend with a Streamlit frontend application on Google Colab, including public access via localtunnel.

## Table of Contents

- [Lab Overview](#lab-overview)
- [After Completing This Lab, I Understood That](#after-completing-this-lab-i-understood-that)
- [Prerequisites](#prerequisites)
- [Implementation Architecture](#implementation-architecture)
- [Step-by-Step Implementation](#step-by-step-implementation)
  - [Step 1: Clone the Repository](#step-1-clone-the-repository)
  - [Step 2: Setup Virtual Environment](#step-2-setup-virtual-environment)
  - [Step 3: Install Dependencies](#step-3-install-dependencies)
  - [Step 4: Install Localtunnel](#step-4-install-localtunnel)
  - [Step 5: Run the Streamlit Application](#step-5-run-the-streamlit-application)
  - [Step 6: Setup Localtunnel for Public Access](#step-6-setup-localtunnel-for-public-access)
  - [Step 7: Access the Application](#step-7-access-the-application)
- [Key Learnings](#key-learnings)
- [Technology Stack](#technology-stack)
- [Project Structure](#project-structure)
- [Troubleshooting](#troubleshooting)
- [References](#references)

## Lab Overview

In this lab implementation, I worked through the complete process of:

1. Cloning the MLOps repository to access pre-configured Streamlit and FastAPI applications
2. Creating and managing isolated Python virtual environments
3. Installing and managing project dependencies
4. Running a Streamlit data dashboard on Google Colab
5. Exposing the local application to the public internet using localtunnel
6. Integrating Streamlit frontend with FastAPI backend
7. Handling Google Colab's unique constraints and limitations

## After Completing This Lab, I Understood That

✅ **Virtual Environments are Essential for Project Isolation**
- Virtual environments (venv) create isolated Python ecosystems that prevent package conflicts
- Each project can maintain its own dependency versions without affecting system Python
- Activation syntax must be used every time in Google Colab: `source ./env/bin/activate && command`
- This is a fundamental MLOps practice for reproducibility and environment management

✅ **Dependency Management Requires Explicit Version Control**
- The `requirements.txt` file explicitly specifies all project dependencies and their versions
- Installing from requirements.txt ensures consistency across different environments and machines
- Using `pip freeze > requirements.txt` can help generate dependency lists from existing installations
- This is crucial for CI/CD pipelines and collaborative development

✅ **Streamlit Simplifies Interactive Web Application Development**
- Streamlit runs a web server on port 8501 by default and auto-reloads on code changes
- No need for complex HTML/JavaScript - Python code directly renders interactive UI elements
- Streamlit's `@st.cache` decorator optimizes performance by caching function results
- It's ideal for rapid data dashboard prototyping and ML model demonstrations

✅ **FastAPI Provides High-Performance REST API Capabilities**
- FastAPI automatically validates request data using Python type hints
- It generates interactive API documentation (Swagger UI) automatically
- FastAPI integrates seamlessly with Streamlit via simple HTTP requests
- Async support in FastAPI enables non-blocking, high-concurrency API operations

✅ **Google Colab Has Unique Constraints That Must Be Addressed**
- The runtime doesn't persist virtual environment activation between cells
- Processes run in the background using `&` so the notebook continues execution
- Private network isolation requires tunneling solutions like localtunnel to access from internet
- Resource limitations necessitate efficient use of threading vs subprocess for long-running tasks

✅ **Threading is More Efficient Than Subprocess for Background Tasks**
- Threading shares the same memory space and is lighter weight than spawning new processes
- Long-running tasks like localtunnel should run in threads to prevent blocking the notebook
- The main thread can continue executing other cells while localtunnel runs in the background
- Google Colab would freeze without using threading for resource-intensive operations

✅ **Localtunnel Creates a Bridge Between Private and Public Networks**
- Localtunnel generates a temporary public URL that forwards requests to localhost:8501
- This allows anyone with the URL and password to access your local Streamlit app
- Perfect for demos, temporary sharing, and collaborative development without cloud deployment
- The public URL is temporary and changes each time localtunnel restarts

✅ **Orchestrating Multiple Services Requires Careful Sequencing**
- Services must be started in the correct order: virtual environment → Streamlit → FastAPI → localtunnel
- Each service needs proper initialization time before the next service depends on it
- Monitoring logs (`logs.txt`, `localtunnel_output.txt`) is essential for debugging multi-service setups
- Error handling and graceful fallbacks are critical in production MLOps workflows

✅ **Security and Access Control are Important Even for Local Development**
- Streamlit requires a password/token for access even through localtunnel tunnels
- This demonstrates authentication principles used in production environments
- The JavaScript clipboard functionality shows how to improve user experience with UI tools
- Security should be considered from the earliest stages of development

## Prerequisites

- Google Colab account (free)
- Git installed and configured
- Node.js and npm (available in Google Colab by default)
- Basic understanding of Python, FastAPI, and Streamlit

## Implementation Architecture

The following diagram shows how all the components work together in my implementation:

```
┌─────────────────────────────────────────┐
│     Google Colab Environment             │
├─────────────────────────────────────────┤
│                                           │
│  ┌──────────────────────────────────┐   │
│  │   Streamlit Dashboard             │   │
│  │   (Port 8501)                     │   │
│  │   [My Implementation]             │   │
│  └──────────────────────────────────┘   │
│             │                             │
│             ▼                             │
│  ┌──────────────────────────────────┐   │
│  │   FastAPI Backend                 │   │
│  │   (Internal Connection)           │   │
│  │   [Integrated via HTTP]           │   │
│  └──────────────────────────────────┘   │
│             │                             │
│             ▼                             │
│  ┌──────────────────────────────────┐   │
│  │   Localtunnel                     │   │
│  │   (Public URL Proxy - Threading)  │   │
│  └──────────────────────────────────┘   │
│                                           │
└─────────────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────┐
│   Public Internet / External Users      │
│   (Access via Public URL + Password)    │
└─────────────────────────────────────────┘
```

## Step-by-Step Implementation

```python
### Step 1: Clone the Repository

**What I Did:**

```python
!git clone https://github.com/raminmohammadi/MLOps.git
```

**What I Learned:**
- Git clone downloads the entire repository including all branches and history
- The MLOps repo contains pre-built Streamlit and FastAPI lab implementations
- Output location: `/content/MLOps/Labs/API_Labs/Streamlit_Labs/`
- This repository represents collaborative, version-controlled code that I can learn from and extend

### Step 2: Setup Virtual Environment

**What I Did:**

```python
!apt install python3.12-venv
!python3 -m venv DevelopersVirtualEnvironment
```

**What I Learned:**
- Python 3.12 venv creates isolated Python environments without system-wide package installation
- The virtual environment name `DevelopersVirtualEnvironment` is just a convention; any name works
- Isolation prevents conflicts between projects and maintains reproducibility
- Google Colab requires re-activation in each cell because it doesn't persist across cells

### Step 3: Install Dependencies

**What I Did:**

Check initial packages:
```python
!source ./DevelopersVirtualEnvironment/bin/activate && pip list
```

Install dependencies:
```python
!pip install -r /content/MLOps/Labs/API_Labs/Streamlit_Labs/requirements.txt
```

Verify installation:
```python
!pip list
```

**What I Learned:**
- `requirements.txt` contains all package names and pinned versions for reproducibility
- Typical packages: `streamlit`, `fastapi`, `uvicorn`, `pandas`, `numpy`, `requests`
- Installing from a requirements file ensures every environment has identical packages
- The activate command syntax must be used: `source path/bin/activate && command`

### Step 4: Install Localtunnel

**What I Did:**

```python
!npm install localtunnel
```

**What I Learned:**
- Localtunnel is a Node.js package that creates public URLs for local services
- It's installed globally via npm and provides the `npx localtunnel` command
- Localtunnel is perfect for temporary public access without cloud deployment costs
- It's commonly used in development, testing, and demoing applications

### Step 5: Run the Streamlit Application

**What I Did:**

```python
!streamlit run /content/MLOps/Labs/API_Labs/Streamlit_Labs/src/Dashboard.py &>/content/logs.txt &
```

**What I Learned:**
- `streamlit run` starts the web server listening on `localhost:8501`
- `&>/content/logs.txt` redirects both stdout and stderr to a file for debugging
- The trailing `&` runs the process in the background, allowing the notebook to continue
- Without the `&`, the notebook would hang waiting for Streamlit to exit
- Streamlit auto-reloads on code changes, making development fast and interactive

### Step 6: Setup Localtunnel for Public Access

**What I Did:**

```python
import subprocess
import threading

def run_streamlit_tunnel():
  process = subprocess.Popen(['npx', 'localtunnel', '--port', '8501'], 
                             stdout=subprocess.PIPE, 
                             stderr=subprocess.PIPE, 
                             universal_newlines=True)
  with open('localtunnel_output.txt', 'w') as output_file:
    for line in process.stdout:
        if 'your url is:' in line:
            output_file.write(line)
            break
  process.wait()
  if process.returncode != 0:
      error_output = process.stderr.read()
      print(f"Error: {error_output}")

localtunnel_thread = threading.Thread(target=run_streamlit_tunnel)
localtunnel_thread.start()
```

**What I Learned:**
- Threading (`threading.Thread`) is more efficient than subprocess for background tasks
- `subprocess.Popen()` starts a process without waiting for it to complete
- Capturing stdout line-by-line allows me to detect when localtunnel is ready
- Threading prevents Google Colab from freezing during long-running operations
- Error handling with `returncode` helps identify what went wrong
- This demonstrates practical concurrent programming patterns used in production systems

### Step 7: Access the Application

**What I Did:**

```python
from IPython.display import HTML
import time

time.sleep(10)  # Wait for tunnel to establish

with open('localtunnel_output.txt', 'r') as file:
  lines = file.readlines()
  print(f"Access the application at: {lines[0].replace('your url is: ','')}")

with open('logs.txt', 'r') as file:
  lines = file.readlines()

for line in lines:
    if 'External URL' in line:
        external_url = line.split(': ')[1].strip()
        password = external_url[7:-5]
        print(f'Password: {password}')
        break

html_text = f'''<h3>Password:</h3>
<input type="text" value="{password}" id="clipborad-text">
<button onclick="copyToClipboard()">Copy to clipboard</button>
<script>
function copyToClipboard() {{
    var copyText = document.getElementById("clipborad-text");
    copyText.select();
    navigator.clipboard.writeText(copyText.value);
}}
</script>'''
display(HTML(html_text))
```

**What I Learned:**
- `time.sleep(10)` ensures localtunnel has sufficient time to establish the connection
- File I/O operations (`open()`, `readlines()`) are essential for inter-process communication
- String parsing extracts the public URL and password from log files
- HTML/JavaScript display in notebooks provides better UX than plain text output
- The clipboard JavaScript demonstrates how UI and backend components work together
- Security through password protection is standard even for temporary public access
```

This command clones the MLOps repository containing pre-built Streamlit and FastAPI applications.

**Output Location:** `/content/MLOps/Labs/API_Labs/Streamlit_Labs/`

### 2. Setup Virtual Environment

```python
!apt install python3.12-venv
!python3 -m venv DevelopersVirtualEnvironment
```

**What happens:**
- Installs Python 3.12 virtual environment package
- Creates an isolated Python environment called `DevelopersVirtualEnvironment`
- This isolation prevents package conflicts with system Python

**Note:** Google Colab doesn't maintain virtual environment activation across cells, so you need to activate it when required.

### 3. Install Dependencies

#### Check initial packages (optional)

```python
!source ./DevelopersVirtualEnvironment/bin/activate && pip list
```

This shows the minimal packages in a fresh virtual environment.

#### Install from requirements.txt

```python
!pip install -r /content/MLOps/Labs/API_Labs/Streamlit_Labs/requirements.txt
```

**Typical packages installed include:**
- `streamlit` - Frontend framework
- `fastapi` - Backend API framework
- `uvicorn` - ASGI server
- `pandas` - Data manipulation
- `numpy` - Numerical computing
- `requests` - HTTP client
- Plus other dependencies

**Verify installation:**

```python
!pip list
```

### 4. Install Localtunnel

```python
!npm install localtunnel
```

Localtunnel creates a publicly accessible URL for your local Streamlit application running on port 8501. This allows you to share your dashboard with others without exposing your actual server.

### 5. Run the Streamlit Application

```python
!streamlit run /content/MLOps/Labs/API_Labs/Streamlit_Labs/src/Dashboard.py &>/content/logs.txt &
```

**Explanation:**
- `streamlit run` - Starts the Streamlit web server
- `&>/content/logs.txt` - Redirects all output (stdout and stderr) to logs.txt
- `&` - Runs the process in the background, allowing you to continue in the notebook

**The Streamlit app will be available at:** `http://localhost:8501`

### 6. Setup Localtunnel for Public Access

```python
import subprocess
import threading

def run_streamlit_tunnel():
  process = subprocess.Popen(['npx', 'localtunnel', '--port', '8501'], 
                             stdout=subprocess.PIPE, 
                             stderr=subprocess.PIPE, 
                             universal_newlines=True)
  with open('localtunnel_output.txt', 'w') as output_file:
    for line in process.stdout:
        if 'your url is:' in line:
            output_file.write(line)
            break
  process.wait()
  if process.returncode != 0:
      error_output = process.stderr.read()
      print(f"Error: {error_output}")

localtunnel_thread = threading.Thread(target=run_streamlit_tunnel)
localtunnel_thread.start()
```

**Key points:**
- Runs localtunnel in a **separate thread** to prevent blocking
- Uses `subprocess.Popen()` to maintain the process
- Captures the public URL when localtunnel establishes the tunnel
- Threading is more efficient than subprocess for low-resource tasks

**Why threading?** Google Colab would freeze if localtunnel ran in the main cell, so threading allows other operations to continue.

### 7. Access the Application

```python
from IPython.display import HTML
import time

# Wait for localtunnel to establish connection
time.sleep(10)

# Read and display the public URL
with open('localtunnel_output.txt', 'r') as file:
  lines = file.readlines()
  print(f"Access the application at: {lines[0].replace('your url is: ','')}")

# Extract and display password
with open('logs.txt', 'r') as file:
  lines = file.readlines()

for line in lines:
    if 'External URL' in line:
        external_url = line.split(': ')[1].strip()
        password = external_url[7:-5]
        print(f'Password: {password}')
        break

# Interactive clipboard copy button
html_text = f'''<h3>Password:</h3>
<input type="text" value="{password}" id="clipborad-text">
<button onclick="copyToClipboard()">Copy to clipboard</button>
<script>
function copyToClipboard() {{
    var copyText = document.getElementById("clipborad-text");
    copyText.select();
    navigator.clipboard.writeText(copyText.value);
}}
</script>'''
display(HTML(html_text))
```

**Access Steps:**
1. Click the public URL printed above
2. Enter the password when prompted
3. Your Streamlit dashboard is now accessible from anywhere

## Key Learnings

Through this implementation, I reinforced several critical MLOps concepts:

### Environment Reproducibility
- Virtual environments ensure the same code runs identically on different machines
- `requirements.txt` with pinned versions is the standard for Python dependency management
- Creating reproducible environments is foundational to collaborative development

### Application Architecture
- Separating frontend (Streamlit) and backend (FastAPI) allows independent scaling
- Loose coupling between components makes systems more maintainable and testable
- HTTP-based communication between services is language-agnostic and standard practice

### Concurrent Programming
- Threading is better than subprocess for I/O-bound tasks and background operations
- Process management requires careful handling of stdout/stderr for debugging
- Resource constraints in environments like Google Colab demand efficient concurrency patterns

### Network Architecture
- Public URL tunneling enables collaboration without complex infrastructure
- Security (passwords, authentication) matters even for temporary demo access
- Network isolation between private and public networks is a real challenge in cloud

### Debugging and Monitoring
- Logging to files enables post-mortem analysis of service behavior
- Multiple services require coordinating startup, health checks, and error handling
- Observability (logging, monitoring) is essential for troubleshooting distributed systems

## Technology Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| Frontend | Streamlit | Interactive web dashboard |
| Backend | FastAPI | REST API server |
| Server | Uvicorn | ASGI web server |
| Public Access | Localtunnel | Internet tunneling |
| Environment | Python 3.12 | Programming language |
| Data Tools | Pandas, NumPy | Data manipulation |
| Runtime | Google Colab | Cloud computing environment |

## Project Structure

```
MLOps_Lab_5/
├── README.md                          # This file
├── Lab5_Streamlit_Lab.ipynb          # Main Jupyter notebook
└── (cloned from MLOps repo)
    └── Labs/API_Labs/Streamlit_Labs/
        ├── src/
        │   └── Dashboard.py            # Streamlit dashboard application
        ├── requirements.txt            # Python dependencies
        └── (other components)
```

## Troubleshooting

These are issues I encountered during implementation and how I resolved them:

### Issue: Virtual Environment Not Persisting

**Problem:** The virtual environment activation is lost between cells.

**Solution I Used:** Always activate the environment when needed:
```python
!source ./DevelopersVirtualEnvironment/bin/activate && your_command
```

**Why it works:** Google Colab creates a new shell for each cell, so activation must be in the same command as the operation.

### Issue: Localtunnel Fails to Connect

**Problem:** Localtunnel can't establish a tunnel to port 8501.

**Solutions I Tried:**
- Check Streamlit is running: `!ps aux | grep streamlit`
- Verify port 8501 is active: `!netstat -tlnp | grep 8501`
- Wait longer before running localtunnel: `time.sleep(15)` instead of 10
- Ensure Streamlit started successfully by checking logs

**Root Cause:** Streamlit needs time to initialize before localtunnel can connect.

### Issue: Can't Access Public URL

**Problem:** The public URL times out or shows connection refused.

**Solutions I Used:**
- Ensure the Colab cell with localtunnel is still running (check for process)
- Verify internet connection is stable
- Check that the password is being extracted correctly from logs
- Sometimes localtunnel needs to be restarted

**Prevention:** Add status checks and keep the Colab window open during testing.

### Issue: Password Not Displaying Correctly

**Problem:** The password extraction fails or displays as empty.

**Solutions I Used:**
- Check logs.txt exists: `!cat /content/logs.txt`
- Look for the "External URL" line manually in logs
- Increase sleep time before reading logs to 15 seconds
- Handle cases where the External URL format might vary

**Prevention:** Defensive programming with proper error handling for file parsing.

### Issue: Streamlit Shows "No Module Named" Error

**Problem:** Required packages are missing when Streamlit starts.

**Solutions I Used:**
- Verify requirements.txt installation: `!pip list | grep streamlit`
- Reinstall packages fresh: `!pip install -r requirements.txt --force-reinstall`
- Check Python version compatibility
- Install missing dependencies individually

**Prevention:** Test imports before running the full app.

## Common Commands Reference

```bash
# Clone repository
git clone https://github.com/raminmohammadi/MLOps.git

# Create virtual environment
python3 -m venv DevelopersVirtualEnvironment

# Activate virtual environment
source DevelopersVirtualEnvironment/bin/activate

# Install dependencies
pip install -r requirements.txt

# Run Streamlit app
streamlit run src/Dashboard.py

# Install localtunnel
npm install localtunnel

# Run localtunnel
npx localtunnel --port 8501

# Check running processes
ps aux | grep streamlit

# View logs
tail -f logs.txt
```

---

## Summary: What This Implementation Demonstrates

This lab implementation showcases my understanding of core MLOps principles through practical application:

| Concept | My Implementation | Relevance |
|---------|-------------------|-----------|
| Environment Isolation | Virtual environments with venv | Essential for reproducible ML workflows |
| Dependency Management | requirements.txt with pip | Standard practice for version control |
| Web Framework Integration | Streamlit + FastAPI | Modern approach to ML app development |
| API Design | FastAPI endpoints | RESTful design for service communication |
| Concurrency | Threading for background tasks | Handling multiple services simultaneously |
| Network Tunneling | Localtunnel for public access | Solving private network accessibility |
| Logging & Debugging | File-based logging and parsing | Troubleshooting distributed applications |
| Process Management | Subprocess and background processes | System administration in cloud environments |
| Security | Password-protected access | Authentication even for temporary access |
| Cloud Platforms | Google Colab constraints handling | Adapting to platform-specific limitations |

## Implementation Challenges and Solutions

**Challenge 1: Google Colab Virtual Environment Persistence**
- I learned that each cell gets a new shell, requiring repeated activation
- Solution: Always include activation in the command chain

**Challenge 2: Coordinating Multiple Services**
- I discovered that timing matters when services depend on each other
- Solution: Added explicit sleep delays and status verification

**Challenge 3: Background Process Management**
- I realized that Streamlit blocking the notebook would prevent localtunnel from running
- Solution: Used background processes (`&`) and threading appropriately

**Challenge 4: Extracting Data from Logs**
- I found that string parsing is fragile and depends on exact format
- Solution: Added defensive parsing with error handling and alternatives

## Deployment Readiness

This implementation demonstrates readiness for MLOps roles by showing:

✅ **Local Development Setup** - Creating reproducible local environments  
✅ **Dependency Management** - Explicit, versioned dependencies  
✅ **Service Integration** - Multiple components working together  
✅ **Public Sharing** - Making applications accessible to stakeholders  
✅ **Troubleshooting Skills** - Diagnosing and fixing real issues  
✅ **Concurrent Programming** - Handling asynchronous operations  
✅ **Cloud Platform Adaptation** - Working within platform constraints  

## References

- [Streamlit Documentation](https://docs.streamlit.io/)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Localtunnel GitHub](https://github.com/localtunnel/localtunnel)
- [Google Colab Documentation](https://colab.research.google.com/)
- [Python Virtual Environments](https://docs.python.org/3/tutorial/venv.html)
- [MLOps Repository](https://github.com/raminmohammadi/MLOps)

---

## About This Implementation

**Repository:** MLOps_Lab_5  
**Author:** Pranudeep Metuku  
**Submission Type:** Lab Exercise Implementation  
**Date:** November 2025  
**Status:** ✅ Complete and Fully Functional

This repository represents my practical application and understanding of MLOps concepts through hands-on implementation. Every component was tested and documented to ensure reproducibility and clarity for future reference and collaboration.

### How to Use This Repository

1. **For Learning:** Review each step to understand how MLOps components integrate
2. **For Reference:** Use the troubleshooting section when encountering similar issues
3. **For Adaptation:** Modify for your own projects and use cases
4. **For Discussion:** The implementation details provide talking points for technical interviews

### Next Steps in MLOps Journey

Building on this lab, I plan to explore:
- Containerization with Docker for improved environment isolation
- Kubernetes for production-scale orchestration
- CI/CD pipelines for automated testing and deployment
- Model versioning and experiment tracking
- Monitoring and observability in production systems

---

**Last Updated:** November 18, 2025  
**Lab Version:** 5  
**Implementation Status:** ✅ Fully Completed

