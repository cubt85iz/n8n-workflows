# n8n Workflow Collection

A curated collection of powerful, ready-to-use n8n workflows designed to automate personal media management, development maintenance, and more.

## 🚀 Overview

This repository serves as a library of automation templates. Instead of building complex logic from scratch, you can leverage these pre-built workflows to enhance your digital ecosystem.

### Featured Workflows

#### 🎧 [Audiobookshelf AI Recommender](./audiobookshelf/ai_recommendations.json)

Transform your audiobook listening experience with artificial intelligence. This workflow:

- Fetches your entire library and listening progress from **Audiobookshelf**.
- Summarizes your preferences based on books you have already rated.
- Uses a **Large Language Model (LLM)** (via OpenAI-compatible endpoints like llama.cpp) to analyze your tastes and identify unread books that match your patterns.
- Automatically manages a dedicated **"📖 Recommendations"** playlist in your Audiobookshelf instance, ensuring you always have something great to listen to next.

#### 🛠️ [GitHub Workflow Monitor](./github/workflow_monitor.json)

Ensure your CI/CD pipelines and automation never sleep. This workflow:

- Periodically scans a specified list of **GitHub repositories**.
- Checks the status of your GitHub Actions workflows.
- Automatically **re-enables** any workflows that have been disabled, preventing silent failures in your development lifecycle.

---

## 📥 How to Import Workflows

You can quickly bring these workflows into your n8n instance using the raw JSON files hosted on GitHub.

### Method 1: Copy & Paste (Easiest)

1. Navigate to the folder containing the workflow you want (e.g., `audiobookshelf/`).
2. Click on the `.json` file.
3. Click the **"Raw"** button at the top right of the file view.
4. Press `Ctrl+A` (or `Cmd+A`) to select all the text, then `Ctrl+C` (or `Cmd+C`) to copy it.
5. Open your **n8n editor** and click anywhere on the empty canvas.
6. Press `Ctrl+V` (or `Cmd+V`) to paste. The workflow will appear instantly.

### Method 2: Import via URL

If your n8n version supports importing via URL:

1. Click the **"Raw"** button on the GitHub file page.
2. Copy the URL from your browser's address bar.
3. In n8n, use the **"Import from URL"** option and paste the link.

---

## ⚙️ Configuration Note

Every workflow requires specific configuration to work in your environment:

- **Credentials**: You will need to set up your own credentials (e.g., GitHub Personal Access Tokens, Audiobookshelf Bearer Tokens, or OpenAI API keys) within n8n.
- **Environment Variables**: Some workflows use a "Set" node (like `Inject Secrets` in the Audiobookshelf workflow) to define base URLs and IDs. Ensure these are updated to match your specific setup.
