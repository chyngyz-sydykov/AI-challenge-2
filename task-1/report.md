# Implementation steps
 1) open internal dashboard
 2) copy over all the filters to AI IDE prompt
 3) ask chrome ai to anonymize the data
 4) make 4 screenshots. top, middle, bottom, and staff details
 5) copy screenshots to AI IDE prompt. put first prompt and put to planning mode. then execute
 6) after implementation check the result and give the IDE second prompt. then execute

# how you handled the data replacement from the original leaderboard.

I opened chrome devtools and asked AI do following:
- "replace all personal info with dummy data: fullname, position, location"
- "also replace the thumnails"

# First prompt

```
Create a single-page leaderboard website in one index.html (HTML + CSS + vanilla JS in the same file)
that visually matches the provided leaderboard screenshots and supports the requested filters/interactions.

UI:
- Header area with breadcrumb + title (similar to screenshots)
- Filter bar:
  - All Years: All Years, 2026
  - All Quarters: All Quarters, Q1
  - All Categories dropdown
  - Search input (Search employee...)
- Podium / top-3 summary
- Ranked list (avatar, name/position, icons, total points, chevron)
- Expandable row details panel with “Recent Activity” table

Behavior:
- Filters combine (AND): year + quarter + category + search
- Changing filters re-renders podium + ranked list
- Search filters by name (case-insensitive)
- Clicking a row expands details (only one expanded at a time)

Data:
- Inlined dummy employees + activities with year/quarter/category/date/points
## Categories: "Communication", "Collaboration", "Innovation", "Leadership", "Problem Solving"
categories will have exercises with points. each exercise will have a title and points.
```

# Second prompt
 refactor: 
 - the design is different from I asked. it should be 1 to 1 copy of the screenshots.
 - use fake user names: current user name 1---n does not make sense
 - profile pics: https://www.w3schools.com/howto/img_avatar.png
 - podium does not look like an attached image. redo it.
 - font: Aeonik Pro