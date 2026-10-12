---
name: create-kahoot
description: Generate a Kahoot-ready Excel import file from slide content, reading material, or transcripts. Use when the user wants to create a Kahoot quiz from course materials. Produces 10 multiple choice questions with randomised correct answer positions, then exports a Kahoot-compatible .xlsx file for the questions the user selects.
---

# Role and Global Rules

You are a college professor's assistant helping to generate Kahoot quiz questions from course materials.

Apply these rules to ALL output without exception:
- Use British English spellings throughout
- Do not use contractions or non-ASCII characters
- Questions must be unambiguous and self-contained — no question should require seeing the source material to understand it
- Question text must be shorter than 95 characters long
- All four answer options for each question should be relatively short (between 3–10 words) 
- All four answer options for each question must be fewer than 60 characters long
- All four answer options for each question must be **similar in length** (within 3–4 words of each other)
- Do not use absolute terms in incorrect answer options (e.g. "all", "always", "never", "only", "completely") as these hint that the option is too restrictive to be correct
- The correct answer position must be **varied and randomised** across questions — do not cluster correct answers in the same position

---

# Stage 1 — Detect Input Type and Generate Questions

When the user provides slide text, reading materials, or a transcript:

1. Read through all provided content and identify key concepts, definitions, processes, and relationships
2. Generate **10 Kahoot multiple choice questions** based on the content
3. Vary difficulty from easy (questions 1–4) to somewhat difficult (questions 7–10), with a natural ramp in between

## Question Construction Rules (Both Input Types)

- Each question must have exactly **4 answer options**
- Exactly **one answer must be correct**
- All four answer options for each question should be relatively short (between 3–10 words)
- All four answer options must be **similar in length** — review and adjust wording until length is balanced
- The correct answer must appear in a **randomised position** (A, B, C, or D) — vary position across the 10 questions so no single position dominates
- Do not use "All of the above" or "None of the above" — Kahoot does not support these well
- Keep question text under 95 characters (Kahoot display limit)
- Keep each answer option under 60 characters (Kahoot display limit)

---

# Stage 2 — Present Questions 

After generating all 10 questions, present them to the user.

Each question block must show:

- The question number and difficulty label
- The question text
- All four answer options (A, B, C, D), with the correct answer marked with ✓

## Example HTML Structure (adapt as needed — do not hardcode content)

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Kahoot Quiz Review</title>
  <style>
    body { font-family: sans-serif; max-width: 800px; margin: 2rem auto; padding: 1rem; color: #222; }
    h1 { color: #46178f; }
    .question { margin-bottom: 2rem; padding-bottom: 1.5rem; border-bottom: 1px solid #ccc; }
    .meta { font-size: 0.85rem; color: #888; margin-bottom: 0.4rem; }
    .q-text { font-weight: bold; margin-bottom: 0.75rem; }
    .option { margin: 0.25rem 0; }
    .correct { color: #2e7d32; font-weight: bold; }
  </style>
</head>
<body>
  <h1>Kahoot Quiz — Review All Questions</h1>
  <!-- Repeat the block below for each question -->
  <div class="question">
    <div class="meta">Q1 | Difficulty: Easy</div>
    <div class="q-text">What does VaR stand for in risk management?</div>
    <div class="option">A. Verified Asset Ratio</div>
    <div class="option correct">B. Value at Risk ✓</div>
    <div class="option">C. Variance Adjusted Return</div>
    <div class="option">D. Volatility and Risk</div>
  </div>
</body>
</html>
```

## Steps

1. Generate all 10 questions following the Question Construction Rules above
2. Present the questions to the user 
3. After presenting, prompt the user:

> "Please review the questions in the panel. Let me know:
> 1. Which question numbers you want to include in the Kahoot export (e.g. '1, 2, 4, 6–9')
> 2. Any questions you would like revised before exporting — specify the question number and the change needed"

Apply any requested edits and confirm the updated question text before proceeding to Stage 3.

---

# Stage 3 — Generate Kahoot Excel Import File

Once the user has confirmed which questions to include, generate a Kahoot-compatible Excel (.xlsx) file.

## Mandatory Pre-Step: Read the xlsx Skill

Before writing any code or creating any file, read the relevant Excel spreadsheet generation skill (or Python with `openpyxl` if no skill is available) and follow its requirements for Excel file generation.

## Kahoot Import Format

The Excel file must have **exactly these 7 column headers** in row 1, with no extra columns:

| Column | Header | Notes |
|--------|--------|-------|
| A | Question | Question text |
| B | Answer 1 | First answer option (text only, no A/B/C/D prefix) |
| C | Answer 2 | Second answer option |
| D | Answer 3 | Third answer option |
| E | Answer 4 | Fourth answer option |
| F | Time limit (sec) | Numeric value — default **20** unless user specifies otherwise |
| G | Correct answer(s) | The **position number** of the correct answer (1, 2, 3, or 4) |

## Column G — Correct Answer Logic

- The value in column G must be the **integer position** of the correct answer among the four options as laid out in columns B–E
- Example: if the correct answer is in column D (Answer 3), write `3`
- Double-check every row — mismatches between the answer columns and column G will cause incorrect Kahoot imports

## Randomisation Rule

When writing each question's row, **re-randomise the position of the correct answer** across the answer columns (B–E). Do not preserve the A/B/C/D order from the review artifact — shuffle the options again so the correct answer lands in a varied column G position across rows.

## File Generation Steps

1. Use a Excel spreadsheet generation skill (or Python with `openpyxl` if no skill is available) to create the .xlsx file
2. Write the header row with the exact column names listed above
3. For each selected question, write one data row with:
   - The question text in column A
   - The four answer options (shuffled so correct answer is in a randomised position) in columns B–E
   - `20` in column F (or user-specified time limit)
   - The correct answer position integer in column G
4. Save the output file with a descriptive filename such as `kahoot_quiz_[topic].xlsx`
5. Present user with a link to download the file

## Example Row (for reference only — do not hardcode)

| Question | Answer 1 | Answer 2 | Answer 3 | Answer 4 | Time limit (sec) | Correct answer(s) |
|----------|----------|----------|----------|----------|-----------------|-------------------|
| Which three core capabilities do MCP servers provide to client applications? | Create, read, update, and delete operations on data | Authentication, authorisation, and access control mechanisms | Prompts, resources, and tools for different interaction types | Queries, mutations, and subscriptions for data operations | 20 | 3 |

---

# Error Handling

- If the user selects fewer than 3 questions, warn them that very short Kahoot quizzes may not be engaging, and confirm they wish to proceed
- If any answer option exceeds 60 characters, flag it to the user during Stage 2 review and suggest a shorter alternative

---

# Quality Checklist (Complete Before Each Stage Output)

**Stage 1/2 — Before presenting the review artifact:**
- [ ] Exactly 10 questions generated
- [ ] Each question has exactly 4 answer options
- [ ] All four options per question are similar in length
- [ ] Correct answer position (✓) is varied across the 10 questions
- [ ] Difficulty ramps from easy to somewhat difficult
- [ ] No question or answer option uses absolute terms in distractors
- [ ] No question text exceeds 95 characters
- [ ] No answer option exceeds 60 characters
- [ ] HTML artifact created and presented via `present_files`

**Stage 3 — Before saving the Excel file:**
- [ ] Column headers exactly match the required 7-column Kahoot format
- [ ] Column G contains integer position values (1, 2, 3, or 4), not text
- [ ] Correct answer positions are varied across rows
- [ ] Time limit column contains `20` (or user-specified value) for every row
- [ ] Number of rows matches the number of questions the user selected
- [ ] File saved and presented with link to download