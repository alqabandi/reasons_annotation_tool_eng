# Text Annotation Tool (English)

This tool lets research annotators read short text responses about political topics and label them for emotion, sentiment, moral foundations, and political leaning. It runs in the browser, saves progress automatically, and gives each annotator their own output file.

## What Annotators Do

Each annotator reads a participant's response to a political statement -- along with the context of what statement they responded to and whether they agreed or disagreed -- and answers four sets of questions:

- **Emotions** -- Pick the primary emotion (Anger, Joy, Sadness, or Optimism) and rate the intensity of each on a 1--7 scale.
- **Sentiment** -- Classify the text as Positive, Neutral, or Negative, then rate it on a 1--7 scale.
- **Moral Foundations** -- Mark Yes or No for each of eight foundations: Care, Fairness, Loyalty, Authority, Purity, Liberty, Honesty, and Self-discipline.
- **Political Inference** -- Guess the writer's political affiliation: Democrat, Republican, Independent/Neither, or Can't tell.

## Project Structure

```
.
├── server.py              # Python server that loads data and saves annotations
├── annotation_tool.html   # The annotation interface (single-page app)
├── INSTRUCTIONS.html      # Step-by-step setup guide for annotators
├── annotations_empty.csv  # Template CSV with texts to annotate (not tracked in git)
└── .gitignore
```

## Requirements

- **Python 3.6+** -- uses only the standard library, so you don't need to install any packages.
- A modern web browser (Chrome, Firefox, Safari, or Edge).

## Getting Started

### 1. Prepare your data

Drop a CSV file named `annotations_empty.csv` into the project root. It needs these columns:

| Column | What it contains |
|---|---|
| `ResponseId` | A unique participant ID |
| `X` | A unique row ID (the tool uses this as its annotation key) |
| `statement` | A code for the political statement (e.g., `climate_change`, `gun_laws`) |
| `agree` | Whether the participant agreed or disagreed |
| `X_describe` | The participant's free-text response -- this is what annotators read and label |

### 2. Read the instructions

Before anything else, open `INSTRUCTIONS.html` in your browser. It explains the full annotation task -- what you'll read, what you'll label, and how to use the tool. It also walks you through installing Python and starting the server on both Mac and Windows.

### 3. Start the server

```bash
python3 server.py        # Mac / Linux
python server.py         # Windows
```

You'll see a banner confirming the server is running at **http://localhost:8000**.

### 4. Open the tool and annotate

Go to [http://localhost:8000](http://localhost:8000) in your browser. Type in a username and click **Start Annotating**. The tool creates a personal output file for that username (`annotations_[username].csv`) and you're off.

## How It Works

### The server (`server.py`)

A small Python HTTP server -- no frameworks, no dependencies -- that does three things:

1. **Serves the frontend** files (HTML, CSS, JS) from the project directory.
2. **Loads the template data** from `annotations_empty.csv` and sends it to the browser.
3. **Reads and writes each annotator's CSV** so progress persists across sessions.

Here are the API endpoints it exposes:

| Method | Endpoint | What it does |
|---|---|---|
| `GET` | `/api/template` | Sends all rows from the template CSV |
| `GET` | `/api/annotations?username=X` | Fetches a user's saved annotations |
| `GET` | `/api/check-user?username=X` | Checks whether a user file already exists |
| `POST` | `/api/init-user` | Creates a fresh `annotations_[username].csv` from the template |
| `POST` | `/api/save` | Writes the current annotations to the user's CSV |

### The frontend (`annotation_tool.html`)

Everything lives in a single HTML file -- styles, markup, and logic all in one place. Here's what it gives you:

- **Setup screen** -- Enter a username; the tool detects whether you have a previous session.
- **Annotation interface** -- The text stays pinned at the top while you scroll through questions. A progress bar tracks how far along you are.
- **Validation** -- You must answer every question before moving on.
- **Dual saving** -- The tool writes to the server CSV *and* stashes a backup in the browser's `localStorage`, so you won't lose work even if the server hiccups.
- **Session resumption** -- Come back later, type the same username, and pick up right where you left off.
- **Navigation** -- Go forward, go back, or jump to any item by number.
- **Completion screen** -- When you finish, download your annotations as a CSV or review them.

### Output format

The tool saves each annotator's work to `annotations_[username].csv`. This file contains the original data columns plus all the annotation columns:

```
annotator_id, emotion_categorical, emotion_anger_likert, emotion_joy_likert,
emotion_sadness_likert, emotion_optimism_likert, sentiment_categorical,
sentiment_likert, mf_care, mf_fairness, mf_loyalty, mf_authority,
mf_purity, mf_liberty, mf_honesty, mf_selfdiscipline, political_guess
```

## For Annotators

If you're an annotator, open `INSTRUCTIONS.html` in your browser. It walks you through everything:

- How to install Python (Mac and Windows)
- How to start and stop the server
- How to use the annotation interface
- How to take breaks and resume later
- What to do when something goes wrong

## Troubleshooting

**"Address already in use"** -- A previous server instance is still running. On Mac, kill it with `lsof -ti:8000 | xargs kill -9`. On Windows, close all terminal windows and try again.

**"python not found"** -- You need to install Python 3 from [python.org](https://www.python.org/downloads/). On Windows, make sure you check "Add Python to PATH" during installation.

**Page won't load** -- Double-check that the server is running in your terminal and that you're using `http://` (not `https://`).

**Progress not restored** -- Make sure you type the exact same username you used before, then click **Resume Session**.
