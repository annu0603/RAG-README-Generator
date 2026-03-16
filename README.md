# RAG-README-Generator

## Description
The RAG-README-Generator is an automated system designed to generate and continuously update `README.md` files for GitHub repositories. It leverages Retrieval Augmented Generation (RAG) principles to intelligently extract relevant code context from a repository and feed it to a Large Language Model (LLM) for high-quality documentation generation.

This project streamlines the process of keeping project documentation current by integrating with GitHub Actions, ensuring that your `README.md` is always up-to-date with your latest code changes. It's particularly useful for open-source projects or rapidly evolving private repositories where manual README maintenance can be a burden.

## Tech Stack
*   **Python**: The primary programming language used for all scripts.
*   **Google Gemini API**: Utilized for generating the README content based on the provided code context.
*   **`python-dotenv`**: For loading environment variables, such as API keys, from `.env` files.
*   **GitHub Actions**: Orchestrates the automated README generation and update workflow on every push to the `main` branch.
*   **Git**: For repository management and commit operations within the GitHub Actions workflow.

## Project Structure
The repository is structured into several Python modules and a GitHub Actions workflow, each with a specific role in the README generation process:

*   `generate_readme.py`:
    This is the main entry point script. It orchestrates the entire README generation process by initializing the `RAGChunker` and `ReadmeGenerator`, reading repository files, building the context, generating the README content using the LLM, and finally saving the output to `README.md`.

*   `init_models.py`:
    This module handles the initialization and interaction with the Google Gemini API. It loads the `GEMINI_API_KEY` from environment variables and provides a method to send prompts to the specified Gemini model (`gemini-2.5-flash`).

*   `rag_chunker.py`:
    The core component responsible for the Retrieval Augmented Generation (RAG) aspect.
    *   It scans the repository directory, filtering for relevant code files based on defined extensions and skipping common non-code folders (e.g., `node_modules`, `.git`).
    *   It reads file content, truncating excessively large files.
    *   Content is split into manageable chunks with a defined `CHUNK_SIZE` and `CHUNK_OVERLAP`.
    *   It prioritizes important files (e.g., `main.py`, `requirements.txt`) when building the final context, ensuring critical information is included.
    *   It formats selected code chunks with file path, language, and chunk information for clarity.
    *   It also provides functionality to list the overall file structure.

*   `readme_generator.py`:
    This module is responsible for crafting the detailed prompt sent to the Gemini model and processing its response.
    *   It defines a comprehensive `PROMPT` template that guides the LLM to generate a professional README with specific sections (Description, Tech Stack, Installation, Usage, etc.) and formatting rules.
    *   It calls the `init_models` to interact with the Gemini API.
    *   It includes a cleaning function (`_clean`) to remove any potential preambles or extraneous text from the LLM's raw output, ensuring the README starts directly with the project title.
    *   It handles saving the generated README content to a specified file.

*   `update_readme.yml`:
    This YAML file defines a GitHub Actions workflow.
    *   It triggers on every `push` to the `main` branch.
    *   It checks out the code, sets up Python, installs dependencies from `requirements.txt`.
    *   It then runs `generate_readme.py`, providing the `GEMINI_API_KEY` securely from GitHub Secrets.
    *   Finally, it commits any changes to `README.md` back to the repository using a dedicated bot user, ensuring the documentation remains synchronized. The commit message includes `[skip ci]` to prevent infinite workflow loops.

## Installation

To set up and run this project locally, follow these steps:

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/annu0603/RAG-README-Generator.git
    cd RAG-README-Generator
    ```

2.  **Create a virtual environment (recommended):**
    ```bash
    python -m venv .venv
    source .venv/bin/activate # On Windows use `.\.venv\Scripts\activate`
    ```

3.  **Install dependencies:**
    ```bash
    pip install -r requirements.txt
    ```

4.  **Set up your Google Gemini API Key:**
    You need a `GEMINI_API_KEY` from Google AI Studio.
    Create a `.env` file in the root of the repository and add your API key:
    ```
    GEMINI_API_KEY="YOUR_GEMINI_API_KEY_HERE"
    ```
    For GitHub Actions, you will configure this as a repository secret (see Configuration).

## Usage

### Manual Generation
To generate the `README.md` for the current repository manually (or any repository you point it to):

1.  Ensure you have followed the installation steps, including setting up your `.env` file.
2.  Run the `generate_readme.py` script:
    ```bash
    python generate_readme.py
    ```
    This will generate or update the `README.md` file in the current directory.

### Automated Generation via GitHub Actions
The project includes a GitHub Actions workflow (`update_readme.yml`) that automates the README generation process.

1.  **Configure GitHub Secret**:
    Add your `GEMINI_API_KEY` as a repository secret in your GitHub repository:
    *   Go to your repository on GitHub.
    *   Navigate to `Settings` > `Secrets and variables` > `Actions`.
    *   Click `New repository secret`.
    *   Name it `GEMINI_API_KEY` and paste your Google Gemini API key as the value.

2.  **Push to `main` branch**:
    Once the secret is configured, every time you push changes to the `main` branch, the `Auto-Generate README` workflow will run.
    It will:
    *   Checkout your code.
    *   Install dependencies.
    *   Run `generate_readme.py`.
    *   Commit any changes made to `README.md` back to your `main` branch, automatically updating your documentation.

## Configuration

The following environment variable is crucial for the project's operation:

*   `GEMINI_API_KEY`:
    *   **Purpose**: Authenticates your requests with the Google Gemini API.
    *   **Local Usage**: Set this in a `.env` file in the root of your project (e.g., `GEMINI_API_KEY="your_api_key"`).
    *   **GitHub Actions Usage**: Must be configured as a repository secret named `GEMINI_API_KEY` under your GitHub repository's settings (`Settings` > `Secrets and variables` > `Actions`). The workflow then securely accesses this key via `secrets.GEMINI_API_KEY`.

No other configuration files or environment variables are strictly required beyond the API key.

## Contributing

Contributions are welcome! If you have suggestions for improving the README generation, chunking logic, or prompt engineering, please feel free to:

1.  Fork the repository.
2.  Create a new branch for your feature or bug fix.
3.  Make your changes.
4.  Commit your changes with a clear commit message.
5.  Push your branch and open a pull request.

## License
MIT License.