# CSV Export Feature Investigation Guide

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

## Prompt 2: Files and Directories Most Likely Containing the Implementation

Based on typical architecture, the CSV export feature would span multiple layers:

| Layer | File | Purpose |
|-------|------|---------|
| **CLI** | `cli/TaskManagerCli.java` | Command parsing for "export" or "csv" command |
| **App Logic** | `app/TaskManager.java` | Core export method (e.g., `exportToCSV()`) |
| **Storage** | `storage/TaskStorage.java` | File I/O for CSV (may be new) |
| **Model** | `model/Task.java` | Data source; defines what fields to export |
| **Utilities** | `storage/CSVExporter.java` (likely missing) | Dedicated CSV formatting logic |

### Key Finding
**The feature is likely incomplete or not yet implemented.** The storage layer currently handles JSON; CSV export would be a new responsibility requiring a new class.

### Investigation Priority
1. **Primary**: `cli/TaskManagerCli.java` (does the command exist?)
2. **Secondary**: `app/TaskManager.java` (is there an export method?)
3. **Tertiary**: `storage/TaskStorage.java` (is CSV logic mixed in?)
4. **Confirm**: Look for `storage/CSVExporter.java` or similar

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

### Simple Text Searches
```
export
csv
CSV
format
delimiter
column
header
report
download
serialize
```

### How to Use in VS Code
- Press `Ctrl+Shift+F` (Find in Files)
- Enable **"Use Regular Expression"** toggle (.*) if using regex
- Type pattern: `export|csv` to find either term
- Scope: Search all files in workspace

---

## Prompt 4: Feature Component Breakdown by Location

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

### Field Mapping for CSV Export
Expected CSV columns (based on Task model):
```
ID | Title | Description | Status | Priority | Due Date | Tags | Created At | Completed At
```

### Edge Cases to Handle
- Special characters in task titles (commas, quotes, newlines)
- Missing/null optional fields (dueDate, completedAt)
- Tag lists (how to represent comma-separated tags in CSV)
- Date/time formatting (ISO 8601 or human-readable?)

---

## Prompt 5: Step-by-Step Investigation Process

### Step 1: Check CLI for Command
**File**: `cli/TaskManagerCli.java`
**Action**:
- Open the file
- Search for: "export", "csv", "format", "output"
- Look for command definitions (e.g., `addOption("format")`)
- Check if there's a command handler for export

**Expected Findings**:
- Command name (e.g., `export`, `csv-export`)
- Required/optional parameters
- Help text or examples

### Step 2: Check App Logic
**File**: `app/TaskManager.java`
**Action**:
- Open the file
- Search for export-related methods
- Look for method signatures like `exportToCSV()`, `exportTasks()`, `export()`
- Check method parameters (filePath, format, filter options)

**Expected Findings**:
- Export method implementation
- How it retrieves tasks
- How it calls the storage/exporter layer

### Step 3: Check Storage Layer
**File**: `storage/TaskStorage.java`
**Action**:
- Open the file
- Search for CSV-related methods or helper classes
- Check if CSV writing is mixed with JSON logic
- Look for any file writing methods for formats other than JSON

**Expected Findings**:
- Whether CSV logic exists (likely not)
- How JSON export is handled (patterns to reuse)
- Whether a separate CSVExporter class is referenced

### Step 4: Check for Dedicated Export Classes
**Directory**: `storage/`
**Action**:
- List all files in the storage directory
- Look for files like: `CSVExporter.java`, `Exporter.java`, `CsvFormatter.java`, `ExportFormat.java`
- Check if abstract/interface patterns exist for multiple export formats

**Expected Findings**:
- Dedicated export classes (if feature is implemented)
- Or confirmation that feature is incomplete

### Step 5: Check Dependencies
**Files**: `pom.xml`, `build.gradle`, or any `.java` file
**Action**:
- Look for `pom.xml` (Maven) or `build.gradle` (Gradle)
- Search for CSV library imports in `.java` files
- Check Maven Central or Gradle plugins

**Expected Findings**:
```
Dependencies to look for:
- commons-csv (Apache Commons CSV)
- opencsv (OpenCSV)
- jackson-dataformat-csv (Jackson CSV)
```

### Step 6: Check Test Files
**Directory**: Any `src/test/` or `*Test.java` files
**Action**:
- Search for "CSV", "export" in test files
- Look for test cases covering export functionality
- Check expected output formats

**Expected Findings**:
- Test cases for export feature
- Sample CSV output
- Edge case handling tests

### Step 7: Check Configuration/Documentation
**Files**: `README.md`, `.md` files, `pom.xml`, `build.gradle`
**Action**:
- Search for "export", "csv" in documentation
- Look for feature descriptions or roadmap
- Check build configuration for clues

---

## Questions to Ask Yourself While Exploring

### Does the Feature Exist?
- [ ] Can I find any mention of "export" or "csv" in the codebase?
- [ ] Is there a test file testing CSV export?
- [ ] Does the CLI help text mention export functionality?
- [ ] Are there any comments referencing CSV export?

### Where is the Entry Point?
- [ ] Which CLI command triggers export? (e.g., `taskmanager export --format csv`)
- [ ] How is it parsed in `TaskManagerCli.java`?
- [ ] What parameters are required vs optional?
- [ ] Is there error handling for invalid formats?

### What Fields are Exported?
- [ ] Looking at `Task.java`, which fields should appear in the CSV?
  - id, title, description, status, priority, dueDate, tags, createdAt, completedAt?
- [ ] Are all fields exported or only a subset?
- [ ] How are complex types represented? (e.g., TaskStatus enum, List<String> for tags)

### How is the File Created?
- [ ] Does it use `FileWriter`, `PrintWriter`, or a CSV library?
- [ ] Where is the output file saved (current directory, user home, specified path)?
- [ ] What happens if the file already exists (overwrite, error, append)?
- [ ] Is there progress feedback for large exports?

### Is There Existing Similar Code?
- [ ] How does JSON export work in `TaskStorage.java`?
- [ ] Can I reuse patterns from JSON serialization?
- [ ] Is there a base exporter interface or abstract class?
- [ ] How are multiple export formats handled?

### Are There Edge Cases?
- [ ] How are special characters (commas, quotes, newlines) in task titles handled?
- [ ] What if a task has no dueDate or other optional fields?
- [ ] How are tags (list) represented in CSV?
- [ ] What date/time format is used (ISO 8601 vs human-readable)?
- [ ] What about Unicode characters in task descriptions?

### Dependencies
- [ ] Does the project use a CSV library, or is CSV formatting done manually?
- [ ] What version of the CSV library? Is it compatible with the Java version?

---

## Investigation Challenge: Test Your Understanding

### Your Task
**Find or infer the complete export feature flow.**

### Steps to Complete
1. **Open `cli/TaskManagerCli.java`** and search for "export" or "csv"
   - If found: Note the command name and parameters
   - If not found: This confirms the feature is missing

2. **Open `app/TaskManager.java`** and search for "export" methods
   - Document the method name and signature
   - Note what it delegates to

3. **Open `storage/TaskStorage.java`** and check for CSV logic
   - Is any CSV formatting present?
   - Is it mixed with JSON or separated?

4. **List files in `storage/` directory**
   - Look for export-related classes
   - Document what you find

5. **Check for dependencies**
   - Look for pom.xml or build.gradle
   - Search for CSV library mentions

### Expected Deliverable
Create a summary document with:
- **CLI Command Existence**: Yes/No
  - Location: `cli/TaskManagerCli.java` (line number, if found)
  - Command syntax: (e.g., `export --format csv --output file.csv`)
  
- **App Logic**: Yes/No
  - Location: `app/TaskManager.java` (method name, if found)
  - Method signature: (e.g., `public void exportToCSV(String filePath)`)
  
- **Dedicated CSVExporter Class**: Yes/No
  - Location: (if it exists)
  - Class name: (if it exists)
  
- **CSV Library Used**: 
  - Library name and version (if any)
  - Or: "Manual implementation"
  
- **Feature Status**:
  - ✅ Complete (fully implemented)
  - 🟡 Partial (partially implemented)
  - ❌ Missing (not yet implemented)

### My Prediction
**The export-to-CSV feature is NOT yet implemented in the codebase.** 

Evidence:
- It's listed as a task for you to work on
- The storage layer currently only handles JSON
- You'd likely need to create a new class like `storage/CSVExporter.java`

---

## Next Steps

Once you've completed the investigation challenge:

1. **If the feature exists**: Locate all relevant code and understand the current implementation
2. **If the feature is missing**: Plan the implementation:
   - Create `storage/CSVExporter.java`
   - Add export method to `app/TaskManager.java`
   - Add CLI command to `cli/TaskManagerCli.java`
3. **If partially implemented**: Identify what's missing and complete it

Would you like guidance on implementing the CSV export feature?
