# Task 1 – Vibe Coding: Company Leaderboard

## Approach

I used **GitHub Copilot CLI** (AI-enhanced IDE assistant) as the primary tool for this task, working entirely through prompting rather than writing code manually.

### Tools & Techniques

| Tool | Purpose |
|------|---------|
| GitHub Copilot CLI | Main coding assistant – generated all HTML/CSS/JS from natural-language prompts |
| Screenshots + visual analysis | Reconstructed the original UI layout, colours, and behaviour from screenshots |
| Iterative prompting | Refined the implementation through back-and-forth dialogue describing each UI section in detail |

### How I analysed the original

Because the original leaderboard lives behind corporate SSO (SharePoint), the AI assistant could not access it directly. I provided a series of annotated screenshots covering:

- The page header, breadcrumb, and title
- The filter bar (All Years / All Quarters / All Categories / Search)
- The top-3 podium with gold/silver/bronze styling
- A full list row (position number, avatar, name, title+dept code, activity-type icons with counts, total score, chevron)
- An expanded list row showing the **Recent Activity** table (Activity · Category · Date · Points)
- All three dropdown option sets

The assistant analysed the screenshots and recreated every UI element pixel-accurately as a single self-contained `index.html` with embedded CSS and JavaScript – no build tools required.

### Data replacement strategy

The original contains real employee names, photos, job titles, and internal department codes. To comply with responsible AI usage policy:

- **Names** → replaced with entirely fictional names (e.g. *Alex Morgan*, *Jordan Blake*)
- **Photos** → replaced with coloured initials avatars generated purely in CSS/JS (no external requests, no real photos)
- **Department codes** → replaced with structurally identical but fictional codes using invented location prefixes (`NA`, `EU`, `AS`) and generic unit/group notation (`U1.D1.G1`)
- **Activity names** → replaced with generic software-industry topics (e.g. *"[REG] Tech Summit: Cloud Architecture Masterclass"*, *"[LAB] Mentoring of Alex Turner"*)
- **Real scores and dates** → replaced with independently generated fictional values; no original numbers were carried over

No corporate data was entered into any AI tool at any point.

### Functionality implemented

- ✅ Podium display (ranks 1–3, gold/silver/bronze styling, position badges)
- ✅ Ranked list (all employees, position numbers, activity-type icons, totals)
- ✅ Expandable rows with Recent Activity table
- ✅ **All Years** and **2025** year filter
- ✅ **Q1–Q4** quarter filter (recalculates scores & ranking dynamically)
- ✅ **Education / Public Speaking / University Partners** category filter
- ✅ **Search employee** text filter
- ✅ All filters combine correctly; podium and list update in real time

### Deployment

The application is deployed to **GitHub Pages** from the `/docs` folder on the `main` branch.

Live URL: https://86tokar.github.io/AI-challenge/
