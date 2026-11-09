# Language Translator

A lightweight desktop language translator built with **Python** and **Tkinter**.

This project brings two related, but different, language tasks into one desktop application:

1. **Translation** — changes the meaning of text from one language to another.
2. **Script conversion (transliteration)** — changes how text is written, without intentionally translating its meaning.

The distinction matters when working with names and Romanized text. For example, translating `Hello` into Hindi is different from writing the name `Riya` in Devanagari script.

---

## Table of contents

- [The problem](#the-problem)
- [The solution](#the-solution)
- [Features](#features)
- [Screenshots](#screenshots)
- [Technology stack](#technology-stack)
- [Supported languages](#supported-languages)
- [How it works](#how-it-works)
- [Project structure](#project-structure)
- [Getting started](#getting-started)
- [How to use the application](#how-to-use-the-application)
- [Input examples](#input-examples)
- [Limitations](#limitations)
- [Troubleshooting](#troubleshooting)
- [Testing checklist](#testing-checklist)
- [Possible future improvements](#possible-future-improvements)
- [Contributing](#contributing)
- [License](#license)
- [Acknowledgements](#acknowledgements)

## The problem

A common issue with basic translation workflows is that users may want to convert text into another script rather than translate its meaning.

For example:

- `Hello` → `नमस्ते` is a meaning-based translation.
- `Riya` → `रिया` is a script conversion of a name.
- `kaise ho` → `कैसे हो` is Romanized Hindi converted into Devanagari.

These operations are not interchangeable. Automatic language detection can also be unreliable with short phrases, names, mixed-language text, and words typed in English letters.

## The solution

The application offers two modes with separate purposes.

### 1. Translate mode

Use this mode when you want the meaning of your text translated into another language.

It provides:

- Source-language selection, including Auto Detect.
- Target-language selection.
- A language-swap control for supported source selections.
- Hinglish (Hindi written in English letters) as a source option.
- Marathi written in English letters as a source option.

When Hinglish or Romanized Marathi is selected, the application first converts the text into the corresponding native script and then sends it for translation. This helps avoid treating Romanized input as an unrelated language.

### 2. Names & Hinglish mode

Use this mode when you want to write English-letter text in another supported script, rather than translate its meaning.

Examples:

| English-letter input | Example script output |
|---|---|
| `Riya` | `रिया` |
| `kaise ho` | `कैसे हो` |

Transliteration services may produce different spellings for names and regional pronunciations. Please review the result, especially before using it in official documents.

## Features

- Two distinct modes: **Translate** and **Names & Hinglish**.
- Language selectors with a broad selection of languages.
- Auto Detect option for translation source language.
- Romanized Hindi and Marathi input handling in Translate mode.
- Side-by-side input and output panels.
- Character counter and a maximum input length of 5,000 characters.
- Long text is split into smaller chunks before translation.
- Background processing to help prevent the interface from freezing during network requests.
- In-memory result caching for repeated requests during the current session.
- Clear button to reset the input, output, and cached results.
- Copy result button.
- `Ctrl + Enter` keyboard shortcut.
- Status messages for progress and common errors.
- Basic retry handling for rate-limited requests.

## Screenshots

Screenshots are not included in this repository yet. Once you capture the application window, you can add images in a `screenshots/` folder and display them here.

For example, after adding a screenshot named `translator-home.png`, use:

```markdown
![Language Translator application](screenshots/translator-home.png)
```

## Technology stack

| Technology / tool | Purpose |
|---|---|
| Python 3 | Main programming language |
| Tkinter | Desktop graphical user interface |
| `tkinter.ttk` | Themed controls such as buttons, radio buttons, and comboboxes |
| `threading` | Runs network work in a background thread |
| `requests` | Makes HTTP requests to translation and script-conversion endpoints |
| `deep-translator` | Provides an additional Google translation route |
| Google Translate web endpoint | Used by the direct translation route in the current implementation |
| Google Input Tools endpoint | Used for Romanized text-to-script conversion |
| Git and GitHub | Version control and project hosting |

### Important integration note

The current implementation communicates with Google web endpoints and uses the `deep-translator` Python package. It does **not** use an official Google Cloud Translation API integration, and this repository does not include an official Google Cloud API key.

These external endpoints and services may change, impose rate limits, or stop responding. Their continued availability is not guaranteed by this project.

## Supported languages

### Translate mode

The language choices currently defined in the code are:

- English
- Hindi
- Marathi
- Gujarati
- Bengali
- Tamil
- Telugu
- Kannada
- Malayalam
- Punjabi
- Urdu
- Nepali
- French
- Spanish
- German
- Italian
- Portuguese
- Russian
- Japanese
- Korean
- Chinese (Simplified)
- Arabic
- Turkish

The source selector additionally includes **Auto Detect**, **Hinglish (Hindi in English letters)**, and **Marathi in English letters**.

### Names & Hinglish mode

The current script-conversion choices are:

- Hindi
- Marathi
- Gujarati
- Bengali
- Tamil
- Telugu
- Kannada
- Malayalam
- Punjabi
- Urdu
- Nepali

The list reflects the language options present in the current source code. Actual support and output quality depend on the external service.

## How it works

At a high level, the application follows this process:

```text
                    User enters text
                           |
                           v
                    Select a mode
                     /         \
                    /           \
                   v             v
             Translate       Names & Hinglish
                  |                 |
                  v                 v
        Is input Romanized?   Select target script
             /       \               |
           Yes        No              v
            |          |       Convert text to
            v          |       target script
       Convert to      |              |
       native script   |              |
            |          |              |
            v          v              |
       Translate text  |              |
            \          /              |
             \        /               |
              v      v                v
                 Show result
                      |
             Copy or clear result
```

The main code responsibilities are:

- **Language configuration:** maps the language names shown in the interface to the language codes expected by the external services.
- **`classify_error()`:** categorizes common failures, such as rate limits, network errors, unsupported languages, and input-length errors.
- **`convert_chunk_once()`:** sends a text chunk to the script-conversion endpoint.
- **`run_conversion()`:** processes lines and word groups, then joins the converted output while retaining line breaks.
- **`translate_gtx()`:** attempts translation through the direct Google Translate web endpoint.
- **`translate_google_page()`:** provides an additional translation route through `deep-translator`.
- **`split_text()`:** divides longer input into manageable chunks while attempting to preserve whitespace.
- **`run_translation()`:** coordinates ordinary translation and the Romanized-input conversion step.
- **`TranslatorApp`:** creates the Tkinter interface and handles user interaction.
- **Worker thread:** performs network operations away from the main UI thread. UI updates are scheduled back onto Tkinter's main thread.

## Project structure

A simple repository layout for this project is:

```text
language-translator/
│
├── translator_app.py     # Main Python application
├── README.md              # Project documentation
├── requirements.txt       # Python dependencies
├── .gitignore             # Files Git should ignore
└── screenshots/           # Optional application screenshots
```

## Getting started

### Prerequisites

- Python 3.9 or later is recommended.
- A working Tkinter installation.
- An internet connection for translation and script conversion.
- Git, if you plan to clone the repository.

You can check your Python version with:

```bash
python --version
```

On some systems, the command may be:

```bash
python3 --version
```

### 1. Clone the repository

Replace `YOUR-USERNAME` with your GitHub username. The repository URL and folder name should match the repository you create.

```bash
git clone https://github.com/YOUR-USERNAME/language-translator.git
cd language-translator
```

Alternatively, download the repository as a ZIP file from GitHub and extract it.

### 2. Create a virtual environment

A virtual environment keeps this project's Python packages separate from other Python projects.

**Windows (PowerShell):**

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

**Windows (Command Prompt):**

```bat
python -m venv .venv
.venv\Scripts\activate.bat
```

**macOS / Linux:**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies

Install the packages listed in `requirements.txt`:

```bash
python -m pip install -r requirements.txt
```

### 4. Run the application

```bash
python translator_app.py
```

If your system uses `python3`, run:

```bash
python3 translator_app.py
```

## How to use the application

1. Start the application.
2. Choose **Translate** or **Names & Hinglish**.
3. Type or paste your text into the left-hand input panel.
4. Select the source and target languages as applicable.
5. Click **Translate** or **Convert**.
6. You can also use `Ctrl + Enter` to start the selected operation.
7. Read the result in the right-hand output panel.
8. Click **Copy result** to copy the result.
9. Click **Clear** to reset the fields and the current session's result cache.

### Language swap

The swap control is available in Translate mode when a regular source language is selected. It is not available when the source is Auto Detect or one of the Romanized-language options.

## Input examples

These examples illustrate the difference between translation and script conversion. The exact output can vary by context and by the external service.

| Mode | Input | Target | Expected kind of result |
|---|---|---|---|
| Translate | `Hello` | Hindi | Hindi translation, such as `नमस्ते` |
| Names & Hinglish | `Riya` | Hindi | Name written in Devanagari, such as `रिया` |
| Names & Hinglish | `kaise ho` | Hindi | Romanized Hindi written in Devanagari, such as `कैसे हो` |
| Translate | `Bonjour` | English | English translation |

## Limitations

This is a small desktop project, and there are some practical limitations:

- An internet connection is needed for the online language services.
- External endpoints may be unavailable, change their response format, or apply rate limits.
- Automatic language detection may be inaccurate for short phrases, names, and mixed-language text.
- Transliteration is not translation. It may produce a phonetic spelling that differs from a person's preferred spelling.
- The application currently limits input to 5,000 characters.
- Long input is processed in chunks, so sentence context across chunk boundaries may not always be preserved perfectly.
- The cache is stored in memory and does not persist after the app closes. Pressing **Clear** also clears it.
- The project currently has no automated test suite.
- The current code imports `deep-translator` as a required startup dependency, even though it also has a direct translation route using `requests`.

Do not rely on machine-generated translations alone for legal, medical, financial, immigration, or other high-stakes use. Have important translations reviewed by a qualified person.

## Troubleshooting

| Issue | Possible solution |
|---|---|
| `No module named deep_translator` | Activate the project's virtual environment and run `python -m pip install deep-translator`. |
| `No module named requests` | Run `python -m pip install requests`. |
| The application cannot connect | Check your internet connection and try again. |
| Google is busy or requests are limited | Wait a little before retrying. Avoid sending many requests in quick succession. |
| Language pair is not supported | Select another language pair available in the application. |
| The output for a name is unexpected | Try a different spelling or review the result with a native speaker. |
| Tkinter is unavailable | Install the Tkinter package appropriate for your operating system and Python distribution. |

## Testing checklist

The current repository does not include automated tests. Before publishing a change, you can manually check the following:

- [ ] The application opens without an import error.
- [ ] Translate mode works with a normal source and target language.
- [ ] Auto Detect works for a reasonably long sample.
- [ ] Hinglish input is converted to Hindi script before translation.
- [ ] Romanized Marathi input follows the conversion-and-translation path.
- [ ] Names & Hinglish mode converts a sample name into the selected script.
- [ ] Empty input displays a helpful status message.
- [ ] Input longer than 5,000 characters is rejected.
- [ ] Copy result copies the displayed result.
- [ ] Clear removes input and output and clears the session cache.
- [ ] `Ctrl + Enter` starts the selected operation.
- [ ] The window remains responsive while a network request is running.
- [ ] Network and rate-limit failures show understandable messages.

## Possible future improvements

The following are ideas for future versions, not features that are already implemented:

- Add automated unit and integration tests.
- Add a provider abstraction so translation services can be switched more easily.
- Integrate an official translation API where appropriate.
- Improve timeout, retry, and provider-failure handling.
- Add a settings page for preferred languages and font sizes.
- Add a translation history option with clear privacy controls.
- Add optional offline phrase support.
- Improve accessibility and keyboard navigation.
- Package the application for Windows, macOS, and Linux.
- Add screenshots and a short demo GIF to the repository.
- Add support for more languages and regional transliteration preferences.

## Contributing

Contributions, bug reports, and suggestions are welcome.

1. Fork this repository.
2. Create a branch for your change:

   ```bash
   git checkout -b feature/your-feature-name
   ```

3. Make your changes.
4. Test the application manually.
5. Commit your work:

   ```bash
   git add .
   git commit -m "Add: describe your change"
   ```

6. Push your branch and open a Pull Request.

Please do not commit virtual environments, temporary files, personal input text, credentials, or other private information.

## License

No license has been selected for this project yet. Until a license is added, do not assume that others have permission to reuse, modify, or distribute the code.

If you want to make the project open source, choose a license and add its full text in a `LICENSE` file. For example, the MIT License is a common permissive option, but you should choose it only if its terms match your intentions.

## Acknowledgements

- The Python community and the Python standard library.
- The Tkinter toolkit for the desktop interface.
- The maintainers of the `deep-translator` and `requests` packages.
- Google language services used by the current implementation.

---

**A learning-focused desktop project built to make the difference between translating meaning and converting scripts clear.**
