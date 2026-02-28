# Task Manager - Complete Project Analysis & Feature Investigation Guide

---

# Part 1: Project Understanding & Architecture

## Your Initial Understanding

You correctly identified this as:
- A Task Manager CLI application for storing and tracking to-do tasks
- User can manage tasks and their statuses throughout the day
- App tracks whether tasks were completed on time
- Stores all data persistently

### Project Structure
```
app/      → Application business logic
cli/      → Command-line interface
model/    → Domain objects (Task, TaskStatus, TaskPriority)
storage/  → Persistence layer (JSON file storage)
```

---

## Validation & Corrections

### ✅ Correct Assumptions
1. **Core Purpose**: Yes, it's a task tracking application with persistence
2. **CLI-Based**: Correct - command-line interface for user interaction
3. **Layered Architecture**: Yes - follows logical separation of concerns (CLI → App → Model/Storage)
4. **Data Persistence**: Correct - stores tasks in JSON format

### ❌ Misconceptions to Correct

#### Misconception #1: Spring Framework
**Your assumption**: "Appears to use Java and Spring framework"

**Reality**: **NO Spring used**. This is a plain Java application.
- No Spring annotations (`@SpringBootApplication`, `@Component`, etc.)
- No dependency injection framework
- No Spring configuration files
- Pure object-oriented Java with manual dependency management

**Evidence**: 
- Look at `cli/TaskManagerCli.java` - directly instantiates TaskManager
- No `application.properties` or `application.yml` files
- No Spring Boot starter dependencies in Maven/Gradle

#### Misconception #2: Domain-Driven Design (DDD)
**Your assumption**: "Folder structure seems to follow domain driven design"

**Reality**: **Simple package-based organization, NOT full DDD**
- Uses basic layering (presentation, business, persistence)
- No bounded contexts, aggregates, value objects, or repositories (DDD patterns)
- Model layer is simple POJOs (Plain Old Java Objects) with getters/setters
- Not a complex domain model with ubiquitous language

**What it actually is**: Layered architecture (3-tier)
```
Presentation Layer: cli/
Business Logic Layer: app/
Data Layer: model/ + storage/
```

---

## Key Technologies & Libraries Identified

### 1. **Apache Commons CLI** 🔧
**Purpose**: Command-line argument parsing  
**Location**: Used in `cli/TaskManagerCli.java`  
**Classes Used**: `Options`, `CommandLineParser`, `DefaultParser`, `HelpFormatter`, `CommandLine`  
**Version**: Typically 1.5.0 or 1.6.0  

**Example Usage**:
```java
Options options = new Options();
options.addOption("h", "help", false, "Show help");
CommandLineParser parser = new DefaultParser();
CommandLine cmd = parser.parse(options, args);
```

### 2. **Google Gson** 📦
**Purpose**: JSON serialization/deserialization  
**Location**: Used in `storage/TaskStorage.java`  
**Classes Used**: `Gson`, `GsonBuilder`, `JsonSerializer`, `JsonDeserializer`  
**Version**: Typically 2.8.9 or 2.10.1  

**Key Feature**: Custom serializers/deserializers for `LocalDateTime` (Java 8+ time API)

**Example Usage**:
```java
Gson gson = new GsonBuilder()
    .registerTypeAdapter(LocalDateTime.class, new LocalDateTimeSerializer())
    .create();
Task[] tasks = gson.fromJson(reader, Task[].class);
```

### 3. **Java 8+ Time API** ⏰
**Classes Used**: `LocalDateTime`, `LocalDate`, `LocalTime`, `DateTimeFormatter`  
**Purpose**: Track task creation, due dates, and completion times  
**Location**: `model/Task.java`, `app/TaskManager.java`  

### 4. **Java Built-in APIs** 🔨
- **UUID**: Unique task identifiers (`java.util.UUID`)
- **File I/O**: Reading/writing tasks.json (`java.io.*`, `java.nio.file.*`)
- **Collections**: Maps, Lists for in-memory task storage
- **Streams API**: Filtering, sorting tasks

### 5. **Target JDK Version** ☕
**Java Version**: JDK 22 (from IntelliJ config `.idea/misc.xml`)  
**Indication**: Modern Java features supported, Java 8+ compatibility

---

## What Each Main Folder Contains & Purpose

### 📁 `model/` - Domain Model Layer
**Purpose**: Defines the core data structures and business rules

**Files**:
- `Task.java` - Main domain object
  - Properties: id, title, description, status, priority, dueDate, tags, createdAt, completedAt
  - Getters/setters for all properties
  - Validation logic for task fields
  
- `TaskStatus.java` - Enum for task states
  - Constants: TODO, IN_PROGRESS, REVIEW, DONE
  - String mapping for JSON persistence: `getValue()`, `fromValue(String)`
  
- `TaskPriority.java` - Enum for priority levels
  - Constants: LOW, MEDIUM, HIGH
  - Similar pattern to TaskStatus

**Responsibilities**:
- Represent task data structure
- Enforce constraints (valid statuses, priorities)
- Provide serialization mapping (enum ↔ string)

---

### 📁 `storage/` - Persistence Layer
**Purpose**: Handle reading/writing tasks to disk

**Files**:
- `TaskStorage.java` - Main storage implementation
  - Reads tasks from `tasks.json`
  - Writes tasks to `tasks.json`
  - Custom Gson serializers/deserializers for LocalDateTime
  - In-memory Map<UUID, Task> cache
  - File I/O error handling

**Responsibilities**:
- Load tasks from disk into memory
- Save tasks from memory to disk
- Handle JSON format conversion
- Manage file paths and I/O errors
- Adapt Java 8 time types to JSON-compatible formats

**File Location**: 
- Default path: `tasks.json` (in project root or app working directory)
- See `app/TaskManager.java` line ~30 for actual path configuration

---

### 📁 `app/` - Business Logic Layer
**Purpose**: Orchestrate application behavior and task operations

**Files**:
- `TaskManager.java` - Main application controller
  - Instantiates TaskStorage
  - Implements task CRUD operations: `createTask()`, `updateTask()`, `deleteTask()`, `getTask()`
  - Filtering: `listTasks(TaskStatus, TaskPriority, boolean overdue)`
  - Statistics: `getTaskStats()`
  - Time tracking: check if tasks are overdue based on dueDate and current time

**Responsibilities**:
- Coordinate between CLI and storage layers
- Implement business rules (e.g., task validation, overdue detection)
- Manage task state transitions (status changes)
- Provide query/filtering capabilities
- Calculate statistics

---

### 📁 `cli/` - User Interface Layer (Presentation)
**Purpose**: Interact with users via command-line interface

**Files**:
- `TaskManagerCli.java` - Command-line interface handler
  - **Entry Point**: Contains `main(String[] args)` method
  - Parses CLI commands: `create`, `list`, `update`, `delete`, `mark-done`, etc.
  - Uses Apache Commons CLI for argument parsing
  - Formats and displays output to console
  - Shows help messages and error messages
  - Calls TaskManager methods based on user commands

**Command Examples** (inferred from code):
```bash
java taskmanager.cli.TaskManagerCli --help
java taskmanager.cli.TaskManagerCli create "Buy groceries" "Shopping list" 2
java taskmanager.cli.TaskManagerCli list --status todo
java taskmanager.cli.TaskManagerCli list --priority HIGH --overdue
```

**Responsibilities**:
- Parse user input from command line
- Validate CLI arguments
- Call appropriate TaskManager methods
- Format and display results
- Provide user-friendly error messages
- Show help/usage information

---

## Application Entry Points

### Primary Entry Point: `TaskManagerCli.main()`
**Location**: `cli/TaskManagerCli.java` - line with `public static void main(String[] args)`

**Execution Flow**:
```
1. JVM executes TaskManagerCli.main(args)
   ↓
2. CLI parses command-line arguments using Apache Commons CLI
   ↓
3. Instantiates TaskManager (if not already created)
   ↓
4. TaskManager constructor loads tasks from storage
   ↓
5. Based on command, calls appropriate TaskManager method
   ↓
6. Formats and displays results to console
   ↓
7. Program exits
```

### Secondary Entry Point: `TaskManager` Constructor
**Location**: `app/TaskManager.java`

**What happens on instantiation**:
```java
public TaskManager(String filePath) {
    this.storage = new TaskStorage(filePath);
    // Loads existing tasks from JSON file
    // Creates new file if doesn't exist
}
```

### Data Flow Diagram
```
User Types Command in Terminal
         ↓
   TaskManagerCli.main()
         ↓
   Parse args with Apache Commons CLI
         ↓
   Create/Get TaskManager instance
         ↓
   TaskManager instantiates TaskStorage
         ↓
   TaskStorage reads tasks.json
         ↓
   Tasks loaded into Map<UUID, Task>
         ↓
   TaskManager executes business logic
         ↓
   TaskStorage saves to tasks.json
         ↓
   CLI formats and displays results
         ↓
   User sees output in terminal
```

---

## Suggested Questions to Ask Your Team

### Question 1: File Storage & Environment Configuration
**What I want to understand**: Where should `tasks.json` be stored in production vs development?

**Specific Questions**:
- Should `tasks.json` live in repo root, user home directory, or a configurable path?
- Are different environments (dev, test, prod) expected to have separate task files?
- How should the application locate the file if the user runs it from different directories?

**Relevant Code**: See `app/TaskManager.java` - constructor parameter for file path

**Why it matters**: Affects portability, testing, and multi-user scenarios

---

### Question 2: Concurrency & File Locking
**What I want to understand**: What happens if multiple processes access the same task file simultaneously?

**Specific Questions**:
- Is the application expected to support concurrent access (e.g., multiple users/processes)?
- Should we implement file locking to prevent corruption?
- What's the expected behavior if file is modified externally while app is running?
- Should in-memory cache be flushed periodically or only on explicit save?

**Relevant Code**: See `storage/TaskStorage.java` - load/save methods

**Why it matters**: Prevents data loss, race conditions, and corrupted JSON files

---

### Question 3: Build Tool & Dependency Management
**What I want to understand**: Should we use Maven or Gradle, and what are the exact dependency versions?

**Specific Questions**:
- Should we use Maven (pom.xml) or Gradle (build.gradle) for this project?
- What are the confirmed versions for:
  - Apache Commons CLI
  - Google Gson
  - JUnit (if testing is required)
- Should dependencies be managed by the build tool or added manually to classpath?
- Any CI/CD pipeline expectations (GitHub Actions, Jenkins, etc.)?

**Why it matters**: Proper dependency management, reproducible builds, team consistency

---

### Question 4: Testing & Code Quality
**What I want to understand**: What are the testing and code quality standards?

**Specific Questions**:
- Should unit tests be written for TaskManager, TaskStorage, and CLI?
- What's the expected code coverage percentage?
- Are there linting rules (e.g., Google Java Style, Checkstyle)?
- Should we have integration tests for the full CLI-to-file flow?
- Any automated testing in CI/CD?

**Why it matters**: Ensures code maintainability and prevents regressions

---

### Question 5: Feature Scope & Domain Expansion
**What I want to understand**: Will task statuses/priorities change, and what's the product roadmap?

**Specific Questions**:
- Are the current statuses (TODO, IN_PROGRESS, REVIEW, DONE) stable or will they change?
- Should priorities (LOW, MEDIUM, HIGH) be numeric, or could they be text?
- Are tags/categories planned for tasks?
- Should we support recurring tasks or task dependencies?
- Do we need due time (not just due date)?
- Any internationalization (i18n) requirements for status/priority labels?

**Relevant Code**: See `model/TaskStatus.java` and `model/TaskPriority.java`

**Why it matters**: Affects enum design and serialization strategy

---

## Small Exploration Exercise

### Goal
Verify your understanding by manually creating a task and confirming it persists.

### Exercise Steps

#### Step 1: Run the CLI to see help
```bash
cd c:\Users\user11\Documents\projects\ai-code-exercises\use-cases\task-manager\java
java -cp ".:lib/*" taskmanager.cli.TaskManagerCli --help
```

**Expected Output**: Show available commands (create, list, update, delete, etc.)

#### Step 2: Create a test task via CLI
```bash
java -cp ".:lib/*" taskmanager.cli.TaskManagerCli create "Buy groceries" "Shopping list items" 2
```

**Expected Output**: Confirmation message with task ID, or list of all tasks

#### Step 3: List tasks
```bash
java -cp ".:lib/*" taskmanager.cli.TaskManagerCli list
```

**Expected Output**: Shows all tasks including the one you just created

#### Step 4: Inspect the tasks.json file
```bash
# View the file content
cat tasks.json
# Or on Windows:
type tasks.json
```

**Expected Output**: JSON array with task objects, including:
```json
[
  {
    "id": "uuid-here",
    "title": "Buy groceries",
    "description": "Shopping list items",
    "status": "todo",
    "priority": 2,
    "dueDate": null,
    "tags": [],
    "createdAt": "2026-02-28T...",
    "completedAt": null
  }
]
```

#### Step 5: Update task status
```bash
java -cp ".:lib/*" taskmanager.cli.TaskManagerCli mark-done <TASK_ID>
```

**Expected Output**: Task status changes to "done" in memory and file

#### Step 6: Re-verify persistence
```bash
# Check tasks.json again - should show status: "done"
type tasks.json
```

### What This Exercise Validates
- ✅ CLI argument parsing works
- ✅ TaskManager logic executes
- ✅ TaskStorage reads/writes JSON
- ✅ Data persists across program runs
- ✅ File format and encoding correct
- ✅ Your understanding of the architecture

### Troubleshooting
- **"Class not found"**: Classpath issue - ensure Gson and Commons CLI JARs are in lib/ or classpath
- **"File not created"**: Check write permissions in project directory
- **"JSON parse error"**: Check tasks.json format, may be corrupted
- **"UUID error"**: Ensure task ID format matches what CLI expects

---

## Summary Table: Project at a Glance

| Aspect | Detail |
|--------|--------|
| **Type** | CLI Application (Java) |
| **Framework** | None (plain Java) |
| **Architecture** | Layered (CLI → App → Model/Storage) |
| **Primary Libraries** | Apache Commons CLI, Gson |
| **Data Format** | JSON (tasks.json) |
| **Java Version** | JDK 22 |
| **Entry Point** | `taskmanager.cli.TaskManagerCli.main()` |
| **Persistence** | File-based (tasks.json) |
| **Concurrency** | Not currently supported |
| **Testing** | Unknown (check with team) |
| **Build Tool** | Unknown (Maven or Gradle recommended) |

---

# Part 2: CSV Export Feature Investigation

## Prompt 1: Evaluation of Your Search Approach

Your search approach is **sound but incomplete**. Here's the assessment:

### What You Did Right
- ✅ Keyword search for "CSV" is a good starting point
- ✅ Checking app/cli/model/storage folders is the correct strategy
- ✅ Identifying `TaskStorage.java` as potentially relevant

### What You Missed
The feature might not be found because:
- Named differently (e.g., "export", "report", "download")
- In comments or configuration files
- Incomplete or not yet implemented
- Missing dedicated classes (e.g., `CSVExporter.java`)

### Improved Search Strategy
1. **Expand keyword search** to catch variations:
   - "CSV", "export", "report", "download", "serialize", "write", "file"
   - "format", "delimiter", "column", "header"

2. **Search in specific file types**:
   - `.java` files in all packages
   - `pom.xml` or `build.gradle` (dependency hints for CSV libraries)
   - `README.md` or `.md` files (feature documentation)

3. **Check for library dependencies**:
   - Apache Commons CSV
   - OpenCSV
   - Jackson CSV
   - Custom implementation

---

## Prompt 2: Files and Directories Most Likely Containing the Feature

Files likely involved in CSV export feature:

| Layer | File | Purpose |
|-------|------|---------|
| **CLI** | `cli/TaskManagerCli.java` | Command parsing for "export" or "csv" command |
| **App Logic** | `app/TaskManager.java` | Core export method (e.g., `exportToCSV()`) |
| **Storage** | `storage/TaskStorage.java` | May handle file I/O for CSV |
| **Model** | `model/Task.java` | Data source; defines what fields to export |
| **New File** | `storage/CSVExporter.java` (or similar) | Dedicated CSV formatting logic (likely missing) |

**My assessment**: The feature is **likely incomplete or not yet implemented**. The storage layer handles JSON; CSV export would be a new responsibility.

---

## Prompt 3: Effective Search Terms and Patterns

Use these in VS Code's **Find in Files** (`Ctrl+Shift+F`):

### Regex Patterns (enable "Use Regular Expression")
```
Pattern 1: export.*csv (case-insensitive)
Pattern 2: csv.*export
Pattern 3: \bexport\b (standalone word "export")
Pattern 4: \.csv (file extension references)
Pattern 5: CSVExporter|CsvExporter (class names)
Pattern 6: toCSV|toCsv|exportCSV (method names)
Pattern 7: "csv"|'csv' (string literals)
Pattern 8: FileWriter.*csv|PrintWriter.*csv (file I/O patterns for CSV)
```

### How to Use in VS Code
- Press `Ctrl+Shift+F` → enable "Use Regular Expression" (.*) toggle → search `export|csv` across all files

---

## Prompt 4: Feature Component Breakdown by Architecture Layer

```
CLI Layer (cli/TaskManagerCli.java)
    ↓
    Responsibilities:
    - Parse "export --format csv --output tasks.csv" command
    - Validate user input (file path, format)
    - Call app layer with parameters
    ↓
    
App Layer (app/TaskManager.java)
    ↓
    Responsibilities:
    - Method: exportToCSV(String filePath) or exportTasks(ExportFormat format)
    - Orchestrate the export process
    - Retrieve tasks from storage
    - Delegate CSV formatting to storage/exporter
    ↓
    
Model Layer (model/Task.java, TaskStatus.java, TaskPriority.java)
    ↓
    Responsibilities:
    - Provide getters for all exportable fields
    - Fields to export: id, title, description, status, priority, dueDate, tags, createdAt, completedAt
    - Handle proper string representations (e.g., status.getValue())
    ↓
    
Storage/Export Layer (storage/TaskStorage.java or NEW: storage/CSVExporter.java)
    ↓
    Responsibilities:
    - CSV header generation
    - CSV row formatting with proper escaping
    - Handle special characters (commas, quotes, newlines)
    - Optional fields (nulls) handling
    ↓
    
File I/O (Java FileWriter, PrintWriter)
    ↓
    Responsibilities:
    - Write CSV content to file
    - Handle file creation and overwrite logic
    - Error handling (file permissions, disk space)
    ↓
    Output: tasks.csv file on disk
```

---

## Prompt 5: Step-by-Step Investigation Process

### Step 1: Check CLI for Command
- Open `cli/TaskManagerCli.java`
- Search for: "export", "csv", "format", "output"
- Look for command definitions

### Step 2: Check App Logic
- Open `app/TaskManager.java`
- Search for export-related methods
- Look for `exportToCSV()`, `exportTasks()`, `export()`

### Step 3: Check Storage Layer
- Open `storage/TaskStorage.java`
- Check if CSV logic exists

### Step 4: Check for Dedicated Export Classes
- Look in `storage/` folder for `CSVExporter.java`, `Exporter.java`, etc.

### Step 5: Check Dependencies
- Look for `pom.xml` or `build.gradle`
- Search for CSV library imports

### Step 6: Check Test Files
- Look in `src/test/` for CSV/export tests

---

## Prompt 6: Questions to Ask Yourself While Exploring

- [ ] Does the feature exist? (search for "export" or "csv")
- [ ] Is there a test file testing CSV export?
- [ ] Which CLI command triggers export?
- [ ] What fields should be exported from Task?
- [ ] How is the CSV file created and where is it saved?
- [ ] Are there edge cases to handle? (special characters, nulls, etc.)
- [ ] Does the project use a CSV library or manual implementation?

---

## Summary: CSV Export Feature Status

| Aspect | Status |
|--------|--------|
| **Feature Existence** | Not yet implemented |
| **Missing Components** | CSVExporter.java, CLI export command, app export method |
| **Required Libraries** | Apache Commons CSV or manual implementation |
| **Next Step** | Create CSVExporter class and integrate into app/cli layers |

