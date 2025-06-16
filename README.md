This repository is the accompanying code for the "AI-based T-SQL Refactoring: an automatic intelligent code optimization with Azure OpenAI" article. Make sure that check that out.

# SQLAIRefactor

**SQLAIRefactor** is a Windows Forms application that leverages Azure OpenAI to analyze and optimize T-SQL queries. It connects to your SQL Server database, extracts schema metadata in JSON format, and uses prompt engineering and large language models to refactor queries and apply SQL Server best practices automatically.

## 🚀 Features

- 🧠 AI-powered SQL refactoring using GPT-4.1 or GPT-4o (Azure OpenAI)
- 📊 Retrieves and injects full table/column data types in JSON
- 🛠 Identifies inefficiencies (e.g., implicit conversions, index scan vs. seek)
- 🔁 Closed-loop optimization for second-pass improvements
- 🔐 Supports both Windows and SQL Authentication
- 🌐 Renders results in an HTML-based view with syntax highlighting

---

## 🖥️ Screenshots

| Input Query | Refactored Output |
|-------------|-------------------|
| ![input](docs/images/input-example.png) | ![output](docs/images/output-example.png) |

---

## 📦 Prerequisites

- Windows OS
- [.NET Framework 4.8+](https://dotnet.microsoft.com/en-us/download/dotnet-framework)
- Azure OpenAI Resource (with `gpt-4.1`, `gpt-4o`, or `o3-mini` deployments)
- SQL Server with accessible schema

---

## 🔧 Setup

1. Clone this repository:
   ```bash
   git clone https://github.com/yourusername/SQLAIRefactor.git
   cd SQLAIRefactor
