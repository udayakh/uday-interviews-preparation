# Uday – Interview Preparation

A personal tracker for technical interviews — capturing company, date/time, and the questions asked, along with concise, interview-ready answers.

## 📋 Interview Trackers

| Company       | Date       | Time    | Notes                                   |
| ------------- | ---------- | ------- | --------------------------------------- |
| GlobalLogics  | 2026-06-17 | 3:00 PM | [View](GlobalLogics_2026-06-17.md)      |

## 📁 Structure

Each interview is a separate markdown file named `<Company>_<YYYY-MM-DD>.md` containing:

- **Header** — company, date, and time
- **Index** — quick links to every question
- **Questions & Answers** — grouped by topic with explanations and code snippets
- **Notes** — space for follow-ups and interviewer feedback

## 🗂️ Topics Covered So Far

- Design Patterns (Builder)
- Search Algorithms (BFS, DFS)
- Distributed Systems (SAGA pattern, handling third-party outages)
- Databases (SQL vs NoSQL, Redis)
- Kubernetes (HPA)
- Spring (Circular Dependency)
- Java Collections (HashMap internals)

## ✍️ Adding a New Interview

1. Create a file: `<Company>_<YYYY-MM-DD>.md`
2. Add the header, index, and questions with answers
3. Add a row to the table above
4. Commit and push:
   ```bash
   git add .
   git commit -m "Add <Company> interview tracker"
   git push
   ```
