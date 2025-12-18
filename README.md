# Bash Task Runner (btr)

A lightweight bash script for running tasks and managing actions without requiring additional tools or build system dependencies. Perfect for simple automation, build tasks, and project workflows.

## Table of Contents
- [Installation](#installation)
- [Usage](#usage)
- [Exposed Methods](#exposed-methods)
- [Examples](#examples)
- [Developer Guide](#developer-guide)

## Installation

### Option 1: Local Usage
Simply run `btr` from your project directory:
```bash
./btr <command> <action> [arguments]
```

### Option 2: Global Installation
Copy the `btr` script to a directory in your PATH:
```bash
cp btr /usr/local/bin/btr
chmod +x /usr/local/bin/btr
```

Once installed globally, you can run `btr` from anywhere within your project:
```bash
btr <command> <action> [arguments]
```

## Usage

### Basic Syntax
```bash
btr <command> <action> [arguments]
```

### Structure

Create a `.btr` directory in your project root with the following structure:
```
project-root/
├── btr
├── .btr/
│   ├── docker/
│   │   ├── install
│   │   ├── build
│   │   └── clean
│   ├── dev/
│   │   ├── start
│   │   ├── stop
│   │   └── logs
│   └── build/
│       ├── release
│       └── debug
```

### Examples

List all available commands:
```bash
btr
```

List actions for a specific command:
```bash
btr docker
```

Run a specific action:
```bash
btr docker install
btr dev start
btr build release arg1 arg2
```

## Exposed Methods

The following functions and variables are exported from `btr` and available in all action scripts:

### Colored Print Functions

#### `printError <message>`
Prints an error message in red to stderr.
```bash
printError "Something went wrong!"
```

#### `printWarning <message>`
Prints a warning message in yellow.
```bash
printWarning "This is a warning"
```

#### `printInfo <message>`
Prints an informational message in blue.
```bash
printInfo "Processing files..."
```

#### `printSuccess <message>`
Prints a success message in green.
```bash
printSuccess "Task completed successfully!"
```

#### `print <message>`
Prints a plain message with support for formatting.
```bash
print "Usage: $0 <option>"
```

### Docker Helper Function

#### `runInDocker <image> [command...]`
Runs a command inside a Docker container with the following features:
- Mounts the repository root at `/workspace`
- Preserves current user ID and GID to avoid permission issues
- Supports passing multiple arguments
```bash
runInDocker "node:18" npm install
runInDocker "python:3.9" python script.py arg1 arg2
```

### Environment Variables

The following color variables are available for custom formatting:
- `$RED` - Red color code
- `$GREEN` - Green color code
- `$YELLOW` - Yellow color code
- `$BLUE` - Blue color code
- `$NC` - No color (reset)

Example:
```bash
echo -e "${GREEN}Success!${NC}"
```

## Examples

### Simple Action Script

Create `.btr/dev/start`:
```bash
#!/bin/bash

printInfo "Starting development server..."
npm start
printSuccess "Server started!"
```

Make it executable:
```bash
chmod +x .btr/dev/start
```

Run it:
```bash
btr dev start
```

### Docker Action Script

Create `.btr/docker/build`:
```bash
#!/bin/bash

printInfo "Building project in Docker..."
runInDocker "node:18" bash -c "npm install && npm run build"

if [[ $? -eq 0 ]]; then
    printSuccess "Build completed!"
else
    printError "Build failed!"
    exit 1
fi
```

### Nested Commands

You can organize commands in subfolders:
```
.btr/
├── docker/
│   ├── dev/
│   │   ├── start
│   │   └── stop
│   └── prod/
│       ├── build
│       └── deploy
```

Usage:
```bash
btr docker dev start
btr docker prod deploy
```

## Developer Guide

### How It Works

1. **Directory Discovery**: When `btr` is executed, it searches upward from the current working directory to find the `.btr` directory in the repository root.

2. **Command Resolution**: The first argument to `btr` is treated as a command, which maps to a directory in `.btr/`.

3. **Action Resolution**: Subsequent arguments are resolved in order:
   - Directories are treated as sub-commands (allowing nested organization)
   - The first executable file encountered is treated as the action
   - Remaining arguments are passed to the action script

4. **Function Export**: Before executing an action, `btr` exports all helper functions and color variables so they're available in child processes.

5. **Repository Context**: Actions have access to `$ROOT_REPO_DIR` which points to the repository root (the directory containing `.btr`).

### Script Template

Here's a template for creating new action scripts:

```bash
#!/bin/bash

# Your action logic here
printInfo "Starting action..."

# Do work
# ...

# Handle success/failure
if [[ $? -eq 0 ]]; then
    printSuccess "Action completed!"
    exit 0
else
    printError "Action failed!"
    exit 1
fi
```

### Key Features

- **No Dependencies**: Pure bash, no external tools required
- **Hierarchical Commands**: Support for nested sub-commands
- **User-Friendly**: Automatic help when no action is specified
- **Docker Integration**: Built-in function for running commands in Docker containers
- **Color Output**: Consistent colored output for better readability
- **Permission Preservation**: Docker containers run with your user's UID/GID
- **Repository-Aware**: Works from any directory within the project

### Environment Available to Actions

Each action script has access to:
- All exported functions: `printError`, `printWarning`, `printInfo`, `printSuccess`, `print`, `runInDocker`
- Color variables: `$RED`, `$GREEN`, `$YELLOW`, `$BLUE`, `$NC`
- `$ROOT_REPO_DIR` - Path to the repository root
- Standard bash environment and any variables exported by the parent shell
