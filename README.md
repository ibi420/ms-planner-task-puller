# ms-planner-task-puller — Microsoft Planner Task Extraction & Export Toolkit

> An interactive PowerShell toolkit that connects to Microsoft Graph to retrieve, filter, and export Microsoft Planner and Teams tasks into CSV, JSON, and formatted Markdown tables with automated GitHub PR link resolution.

---

## Table of Contents

- [ms-planner-task-puller — Microsoft Planner Task Extraction \& Export Toolkit](#ms-planner-task-puller--microsoft-planner-task-extraction--export-toolkit)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Key Features](#key-features)
  - [Prerequisites](#prerequisites)
  - [Setup \& Installation](#setup--installation)
  - [Finding Your Planner Plan ID](#finding-your-planner-plan-id)
    - [Method 1: Scraping from the URL](#method-1-scraping-from-the-url)
    - [Method 2: Using Microsoft Graph Explorer](#method-2-using-microsoft-graph-explorer)
      - [Option A: If you have a task assigned to the plan](#option-a-if-you-have-a-task-assigned-to-the-plan)
      - [Option B: If you do not have a task assigned to the plan](#option-b-if-you-do-not-have-a-task-assigned-to-the-plan)
  - [How to Run the Script](#how-to-run-the-script)
    - [Main Menu Options](#main-menu-options)
    - [Data Export Workflow](#data-export-workflow)
    - [CSV to Markdown Conversion](#csv-to-markdown-conversion)
  - [How It Works](#how-it-works)
  - [Output Columns](#output-columns)

---

## Overview

`ms-planner-task-puller` is designed to interact with the Microsoft Graph API to pull task information from Microsoft Planner. It provides a flexible, menu-driven CLI that allows for filtering and multiple export formats.

The script stores connection details for multiple plans in a `config.json` file, allowing seamless switching between boards. It fetches task details including bucket names, completion status, attachments (specifically scanning references for GitHub pull request links), and assigned users with their roles (Main Contributor, Reviewer).

Collected data can be exported to **CSV**, **JSON**, or report-ready **Markdown** tables with selectable columns and automatic GitHub hyperlink formatting.

---

## Key Features

- **Multiple Filtering Options:** Pull tasks by assignment, date range, completion status, or specific bucket.
- **Multiple Export Formats:** Export data to CSV, JSON, or a formatted Markdown table.
- **Automatic Link Conversion:** For Markdown exports, automatically converts raw GitHub pull request URLs into formatted, clickable hyperlinks (e.g. `[PR#123](...)`).
- **CSV to Markdown Utility:** Dedicated interactive tool to convert existing CSV files into clean Markdown tables with a built-in file picker.
- **Customizable Columns:** Interactively select precisely which columns to include in your CSV or Markdown export.
- **Plan Management:** Utility menu to add, delete, and view saved Planner plans stored in `config.json`.
- **Robust Logging:** Records detailed debug logs in the `./logs` directory with configurable verbosity.
- **Environment Checks:** Automatically validates PowerShell version (7+) and the `Microsoft.Graph` module before execution.

---

## Prerequisites

- **PowerShell 7 or higher (`pwsh`):** Required for the main script. The script is not supported on Windows PowerShell 5.1. Download PowerShell 7 from the [Microsoft Installation Guide](https://learn.microsoft.com/powershell/scripting/install/installing-powershell) or [PowerShell GitHub Releases](https://github.com/PowerShell/PowerShell).
- **Microsoft 365 Account:** Must have permissions to read the targeted Microsoft Planner boards.
- **`Microsoft.Graph` PowerShell Module:** Required to communicate with the Graph endpoint.
- **Internet connection.**

---

## Setup & Installation

Before running the script for the first time, install the `Microsoft.Graph` module in PowerShell 7:

```powershell
Install-Module Microsoft.Graph -Scope CurrentUser -AllowClobber -Force
```

To configure predefined plans, copy `config.example.json` to `config.json`:

```powershell
Copy-Item config.example.json config.json
```

---

## Finding Your Planner Plan ID

The Plan ID is required to fetch tasks from a specific board.

### Method 1: Scraping from the URL

1. Open [Microsoft Planner](https://tasks.office.com/) in your browser.
2. Navigate to your desired plan.
3. Look at the URL in your browser's address bar:
   ```text
   https://tasks.office.com/your-tenant.com/en-US/Home/Plan?planId=YOUR_PLAN_ID&ownerId=...
   ```
4. The `planId` is the alphanumeric string immediately following `planId=`. Copy this value.

> [!TIP]
> A step-by-step visual walkthrough is available in this [Plan ID Video Tutorial](https://youtu.be/KdOdRppqxCk).

### Method 2: Using Microsoft Graph Explorer

If the URL method is not available, use the [Microsoft Graph Explorer](https://developer.microsoft.com/graph/graph-explorer):

#### Option A: If you have a task assigned to the plan
1. In the left sidebar, expand **Users**.
2. Select **all my planner tasks** and click **Run query**.
3. In the JSON response, locate the `planId` field.

#### Option B: If you do not have a task assigned to the plan
1. In the left sidebar, expand **Groups** and select **all groups I belong to**.
2. Run the query and locate the `id` field corresponding to your group name (this is the **Group ID**).
3. Under **Planner**, select **all Planner plans associated with a group**.
4. Replace the query placeholder with your Group ID and click **Run query**.
5. Find your plan by title and copy its `id` value.

---

## How to Run the Script

1. Open a **PowerShell 7** terminal (`pwsh`).
2. Navigate to the repository directory.
3. Launch the script:
   ```powershell
   .\Planner_pull.ps1
   ```
4. Follow the on-screen prompts to sign in via device code authentication.

### Main Menu Options

* `1-5` — **Data Pulling Options:**
  1. Pull all tasks (including unassigned)
  2. Pull only tasks with assigned users
  3. Pull tasks of assigned users based on a date range
  4. Pull tasks from a specific bucket
  5. Pull tasks by completion status (Not Started, In Progress, Completed)
* `6` — **Manage saved plans:** Add, view, or remove plan shortcuts from `config.json`.
* `7` — **Convert CSV to Markdown:** Utility to transform any local CSV file into a Markdown table with optional PR hyperlink formatting.
* `Q` — **Quit:** Exit the application.

### Data Export Workflow

1. **Preview:** Review a summary of retrieved tasks in the console.
2. **Select Format:** Choose between **CSV**, **JSON**, or **Markdown**.
3. **Select Columns:** For CSV and Markdown, choose exactly which columns to retain (e.g. entering `1,3,5`).
4. **File Name:** Specify your desired destination file name.
5. **Convert Links:** When exporting to Markdown, confirm whether to format raw GitHub URLs into clickable `[PR#...](...)` links.

### CSV to Markdown Conversion

Option 7 lets you import existing CSV files (such as previously exported tasks or manually edited sheets) and generate clean Markdown tables. It includes a built-in file picker for local `.csv` files and offers automatic GitHub PR link conversion.

---

## How It Works

1. **Pre-flight Validation:** Checks for PowerShell 7+ and verifies that the `Microsoft.Graph` module is loaded.
2. **Device Code Authentication:** Authenticates against Microsoft Graph via a device code login flow.
3. **Configuration Storage:** Automatically manages plan shortcuts in `config.json`.
4. **Data Aggregation:** Queries tasks, buckets, user assignments, and task reference links.
5. **Role Detection:** Distinguishes between `Main Contributor` and `Reviewer` based on task assignment order.
6. **Link Parsing:** Extracts GitHub pull request references and formats them for reporting.
7. **Logging:** Writes execution trails and debug messages to `./logs/script.log`.

---

## Output Columns

| Column          | Description                                                                                |
| --------------- | ------------------------------------------------------------------------------------------ |
| **Name**        | Display name of the user assigned to the task                                              |
| **Role**        | Inferred role (`Main Contributor`, `Reviewer`, or unassigned)                              |
| **Task**        | Title of the Planner task                                                                  |
| **Bucket**      | Bucket name where the task resides                                                         |
| **Attachments** | Semicolon-separated list of GitHub URLs, with optional `[PR#123](...)` Markdown formatting |
| **Status**      | Task completion state (`Not Started`, `In Progress`, or `Completed`)                       |
