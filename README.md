# Integrating Docker + CodeLlama + Python for Code Analysis

This repository shows how to leverage [Code Llama](https://ollama.com/library/codellama), a cutting-edge AI model for code analysis, in conjunction with Docker to create an efficient and automated code review workflow.


## How it works?

This project sets up an Ollama Docker container and integrates a "pre-commit" hook. Whenever someone modifies or commits a Python file, the hook triggers a code review using the codellama model. The review is then saved into a `review.md` file, allowing developers to compare their code against the review feedback. It's important to note that AI models can take varying amounts of time to process, depending on the computer's CPU and memory. In my experience, the review process typically takes between 2 to 5 minutes, which is comparable to the time a human might spend conducting a thorough review.

# Getting Started

## Step 1. Clone the repository

```
git clone https://github.com/dockersamples/codellama-python
```



## Step 2: Start the Ollama container

Change directory to `codellama-python` and run the following command:

```
sh start-ollama.sh
```


## Step 3. Create the pre-commit file

Create the following file `.git/hooks/pre-commit`

```
  #!/bin/sh

# Find all modified Python files
FILES=$(git diff --cached --name-only --diff-filter=ACM | grep '\.py$')

if [ -z "$FILES" ]; then
  echo "No Python files to check."
  exit 0
fi

# Clean the review.md file before starting
> review.md

# Review each modified Python file
for FILE in $FILES; do
  content=$(cat "$FILE")
  prompt="\n Review this code, provide suggestions for improvement, coding best practices, improve readability, and maintainability. Remove any code smells and anti-patterns. Provide code examples for your suggestion. Respond in markdown format. If the file does not have any code or does not need any changes, say 'No changes needed'."
  
  # get model review suggestions
  suggestions=$(docker exec ollama ollama run codellama "Code: $content $prompt")
  
  # Añadir el prefijo del nombre del archivo y las sugerencias al archivo review.md
  echo "## Review for $FILE" >> review.md
  echo "" >> review.md
  echo "$suggestions" >> review.md
  echo "" >> review.md
  echo "---" >> review.md
  echo "" >> review.md
done

echo "All Python files were applied the code review."
exit 0
```

The provided shell script automates the code review process by finding all modified Python files, cleaning the review.md file, and iterating through each file to generate suggestions using the Ollama model. The suggestions are then appended to the `review.md` file, providing developers with immediate feedback on their code changes.


## Step 4. Proper permission to execute

```
chmod +x .git/hooks/pre-commit
```

## Step 5. Add or modify the python files

Go ahead and make changes to the python files.


## Step 6. Apply the changes

```
git add .
git commit -m "Test code review in pre-commit hook"
````

## Results

Apply a commit message and wait for the review. You might see something like


```
pulling manifest
pulling 3a43f93b78ec... 100% ▕████████████████▏ 3.8 GB
pulling 8c17c2ebb0ea...   0% ▕                ▏    0 B
pulling 590d74a5569b...   0% ▕                ▏    0 B
pulling 2e0493f67d0c... 100% ▕████████████████▏   59 B
pulling 7f6a57943a88... 100% ▕████████████████▏  120 B
pulling 316526ac7323...   0% ▕                ▏    0 B
verifying sha256 digest
writing manifest
removing any unused layers
success
```

It might take some minutes, but the final result is a `review.md` with all suggestions.


## How can I customize the prompts provided to the CodeLLama model for generating suggestions?

To customize the prompts provided to the CodeLLama model for generating suggestions, you can modify the prompt variable within the shell script.

