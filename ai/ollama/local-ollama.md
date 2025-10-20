<div style="text-align: right; color: #6c757d; font-size: 12px; margin-bottom: 20px;">
Updated at 16/10/2025
</div>

# Ollama

Ollama is a free, open-source tool that lets you run powerful AI models (like ChatGPT, Claude, etc.) **directly on your own computer** - no internet connection required once installed.

--------------------------------------------------------------------------------

## ⚙️ Installing Ollama

### 1\. Download and Install

- Visit <https://ollama.com/download>
- Select the appropriate version for your operating system (macOS, Linux, Windows)
- Download and follow the standard installation process

### 2\. Verify It's Working

Open your Terminal/Command Prompt and run:

```sh
ollama --version
```

If installation was successful, it will display the Ollama version

### 3\. Try Your First Model

Let's download and chat with an AI model:

```sh
# Download a small, fast model
ollama pull llama3.2

# Start chatting
ollama run llama3.2
```

Type a message, press Enter, and watch the AI respond! Type `/bye` to exit.

--------------------------------------------------------------------------------

## 💬 Using the Chat Interface (GUI)

Ollama provides a user-friendly Desktop Application for chatting with AI without using CLI commands

### 1\. Open the App

Find Ollama in your Applications folder and open it. You'll see a clean, simple interface

![Ollama App](https://firebasestorage.googleapis.com/v0/b/a6dd-1e710cb4332d.firebasestorage.app/o/ai%2Follama%2Flocal-ollama%2Fapplication.png?alt=media&token=c39eeaa5-e8a1-4086-a5ad-3e1f086e2538)

### 2\. Pick Your AI Model

- Click on the model selector to choose which AI you want to chat with
- If the model is not available locally, the system will automatically download it (this may take a moment)

![Pick AI Model](https://firebasestorage.googleapis.com/v0/b/a6dd-1e710cb4332d.firebasestorage.app/o/ai%2Follama%2Flocal-ollama%2FSCR-20251013-lfrs.png?alt=media&token=3aa6ae13-fa55-4a11-b1ea-422e61ba1d8b)

### 3\. Chat Away

Now just type your questions

**Examples to try:**

- "Explain quantum physics in simple terms"
- "Write a Python script to rename files"
- "What's the difference between React and Vue?"
- "Create a workout plan for beginners"

--------------------------------------------------------------------------------

## 💻 Basic CLI Commands

### Pull a Model

Download a model from Ollama library:

```sh
ollama pull <model-name>

# Example
ollama pull llama3.2
ollama pull deepseek-r1
```

### List Models

Show all downloaded models:

```sh
ollama list
```

### Run a Model

Start chatting with a model:

```sh
ollama run <model-name>

# Example
ollama run llama3.2
```

### Show Model Information

Display details about a specific model:

```sh
ollama show <model-name>
```

### List Running Models

See currently running models:

```sh
ollama ps
```

### Remove a Model

Delete a downloaded model:

```sh
ollama rm <model-name>

# Example
ollama rm llama3.2
```

--------------------------------------------------------------------------------

## 🌐 Using Ollama API

Ollama automatically starts a REST API server in the background, allowing you to integrate AI into your applications.

### Check if API is Running

```sh
curl http://localhost:11434
```

### Chat with AI

Send a prompt and get AI response

```sh
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.2",
  "prompt": "Why is the sky blue?"
}'
```

> **API Endpoint:** `http://localhost:11434`<br>
> **Full API Docs:** <https://github.com/ollama/ollama/blob/main/docs/api.md>
