# Audiobookshelf AI Recommender

An intelligent n8n workflow that analyzes your Audiobookshelf library and listening habits to generate personalized audiobook recommendations. It uses a Large Language Model (LLM) to understand your preferences based on your rated playlists and suggests unread books while respecting series order.

## 🚀 Overview

This workflow automates the discovery of new audiobooks. Instead of relying on generic algorithms, it looks at:

1. **Your Ratings:** It interprets your existing playlists (e.g., those named "⭐" through "⭐⭐⭐⭐⭐") as explicit ratings for books.
2. **Your Progress:** It identifies which books you have already started or finished to avoid redundant suggestions.
3. **Your Library:** It analyzes the metadata (authors, genres, series) of your entire collection.
4. **Series Logic:** It ensures it doesn't recommend a sequel unless you've completed the preceding books in that series.

The final result is a dynamically updated playlist in your Audiobookshelf instance named **"📖 Recommendations"**.

![AI-generated Recommendations Playlist](image.png)

## 📋 Prerequisites

To run this workflow successfully, you need:

1. **Audiobookshelf Instance:**
    * A running Audiobookshelf server.
    * An API Key with appropriate permissions.
    * The `Library ID` of the library you wish to scan.
    * **Rating Playlists:** To provide theAI with your preferences, you must create playlists in Audiobookshelf named with stars (e.g., `⭐`, `⭐⭐`, `⭐⭐⭐`, `⭐⭐⭐⭐`, `⭐⭐⭐⭐⭐`) and add books to them to represent how much you liked them.
2. **n8n Instance:**
    * A running n8n environment.
    * **Credentials:** You must create an n8n **Header Auth** credential to connect to Audiobookshelf.
        * **Header Name:** `Authorization`
        * **Header Value:** `Bearer <your_api_key>`
3. **LLM Backend:**
    * The workflow is currently configured to use a local `llama.cpp` instance (via the OpenAI node) running the `gemma-4-26B-A4B-it-UD-IQ4_XS.gguf` model.
    * *Note: You may need to update the credentials and model ID in the "Audiobook Recommendation Engine" node if using a different provider (like OpenAI, Anthropic, or a different local model).*
4. **Workflow Configuration:**
    * Update the **"Inject Secrets"** node with your specific `baseUrl`, `libraryId`, and `recommendationCount`.

## ✨ Expectations

After running the workflow (manually or via weekly schedule):

* **New Playlist:** A playlist titled **"📖 Recommendations"** will appear in your Audiobookshelf library.
* **Curated Content:** The playlist will contain a list of unread audiobooks (up to the count specified in "Inject Secrets") that match your taste.
* **Automatic Updates:** The workflow is set to run weekly, automatically deleting the old recommendation playlist and creating a fresh one.

## 🛠️ Workflow Design

The workflow is designed as a multi-stage pipeline:

### 1. Data Ingestion & Categorization

* **Secret Injection:** Loads configuration.
* **Parallel Fetching:** Simultaneously retrieves your playlists, full library items, and user media progress.
* **Playlist Parsing:** Uses a `Code` node to categorize playlists based on their name (e.g., "⭐⭐" becomes a 2-star rating).

### 2. Preference Modeling

* **Read/Rated Mapping:** Combines the parsed playlists to create a dataset of "Books I have read and how much I liked them."
* **Unread Identification:** Compares the full library against your "media progress" to isolate books you haven't touched yet.
* **Data Reduction:** A specialized Code node transforms the heavy library metadata into a lightweight, text-efficient summary. This minimizes LLM token usage and focuses only on what matters: Title, Author, Series, and Rating.

### 3. AI Reasoning

* **Recommendation Engine:** The summarized data is sent to the LLM. The prompt instructs the AI to:
  * Identify patterns in your preferred authors, genres, and themes.
  * Recommend only unread books.
  * Strictly adhere to series order (no "Book 2" if "Book 1" is unread).
  * Output a clean JSON object.

### 4. Playlist Management (Idempotency)

* **State Check:** The workflow checks if a "📖 Recommendations" playlist already exists.
* **Cleanup:** If it exists, the old playlist is deleted to prevent duplicates.
* **Deployment:** The new recommendations are parsed and sent to the Audiobookshelf API to create the new playlist.

## 🚧 Limitations

### Core Limitations

#### 1. Fragile & Improvised Rating System

The workflow does not utilize a native rating field from Audiobookshelf. Instead, it relies on a "proxy" system via the **Gather Read Books** node.

* **Emoji Dependency:** It identifies user preferences by looking for playlists named exactly with star emojis (e.g., `"⭐⭐"`).
* **User Friction:** For the recommender to work, the user must manually curate playlists using this specific naming convention. If a user renames a playlist or uses different emojis, the AI loses all context regarding their preferences.
* **Lack of Granularity:** Because it relies on playlist names, it is difficult to distinguish between subtle differences in preference beyond the number of stars present in a playlist name.

#### 2. Single-User Architecture

The workflow is designed as a personal automation tool rather than a multi-tenant service.

* **Hardcoded Context:** The workflow uses the `/api/me` endpoint to fetch user details, meaning it only ever retrieves the profile and media progress of the single authenticated user.
* **Configuration Constraints:** The `Inject Secrets` node is configured for a single `libraryId` and `baseUrl`. There is no mechanism to iterate through multiple users or handle a shared Audiobookshelf instance with multiple listeners.

#### 3. Reliance on Parametric Knowledge (No RAG)

The recommendation engine lacks a **Retrieval-Augmented Generation (RAG)** component, which significantly impacts accuracy.

* **Model Dependency:** The "Audiobook Recommendation Engine" node passes a JSON string of book metadata directly into the prompt. The AI must rely entirely on its internal training data (parametric knowledge) to understand the relationship between the books in your library.
* **Knowledge Gaps:** If the LLM has not been trained on specific niche titles, indie authors, or very recent releases present in your library, it will be unable to make informed connections or recognize the themes of those books.
* **No External Context:** The workflow cannot "look up" book descriptions, reviews, or genre taxonomies from external sources (like Google Books or OpenLibrary) to bolster its decision-making.

---

### Technical & Scalability Limitations

#### 4. Context Window & Token Limits

As a user's library grows, the workflow faces a high risk of failure due to LLM token limits.

* **Scaling Issues:** The `Reduce Items for AI` node aggregates the entire library summary into a single JSON object. For users with thousands of audiobooks, the resulting string will likely exceed the maximum context window of the LLM, leading to truncated data, lost information, or API errors.

#### 5. Lack of Explainability

The workflow is a "black box" for the end user.

* **Output Format:** The AI is strictly instructed to return only a JSON array of IDs. While this is necessary for the automation to work, it means the user receives a list of recommendations in their playlist without any explanation of *why* those books were chosen (e.g., "Because you liked [Author X]...").

#### 6. Error Handling & Playlist Collision

* **Destructive Updates:** The workflow handles updates by deleting the existing `"📖 Recommendations"` playlist and recreating it. While this ensures the list is fresh, it can be disruptive if a user is manually interacting with that playlist or if the "Create" step fails after the "Delete" step has already executed.
