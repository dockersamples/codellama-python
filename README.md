# Integrating Docker + CodeLlama + Python for Code Analysis

This repository shows how to leverage Code Llama, a cutting-edge AI model for code analysis, in conjunction with Docker to create an efficient and automated code review workflow

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


## Step 3. Create the following file `.git/hooks/pre-commit`

Copy the content below and put it in the right directory:

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

Apply a commit message and wait for the review. It could take some minutes, the final result is a `review.md` with all suggestions.



## How can I customize the prompts provided to the CodeLLama model for generating suggestions?

To customize the prompts provided to the CodeLLama model for generating suggestions, you can modify the prompt variable within the shell script.

