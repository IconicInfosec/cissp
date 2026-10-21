# Playbook: building a self-study app with Claude

How this repo was built, written so you can do the same for another subject. Copy this file into the new
repo, change the subject, and give Claude the phase prompts one at a time.

The example throughout is **Business German, level C1**. The CISSP repo is the reference implementation: when a step
says "like CISSP", point Claude to the file named there.

---

## The approach in one paragraph

Start from a proven study method and check its facts against the **current** official exam or standard. Build a
small offline web app with no dependencies, where all content lives in plain data files. Add self-check tools
(flashcards, quizzes, exam simulator), then a beginner layer that explains things in plain words. Then add the
main study tool: one long **learning path** of short cards that goes from easy to hard and brings every idea back
2–4 times in new forms. Every content rule is enforced by a **validator script**. Work in small batches, and make one
commit per batch. `NEXT_STEPS.md` is the living plan, so any new session can pick up where the last one stopped.

## Principles (keep these in every project)

1. **A source method, with today's facts.** Base the plan on a method that worked for someone, then check every
   fact (format, timing, fees, syllabus) against the official source and write down the date you checked it.
2. **Content is data, the app is a thin shell.** Static HTML/CSS/JS, no build step, no npm packages. It opens from
   `index.html` or a tiny local server. Content goes in `app/data/*.js` files that call a registration function
   (`CISSP.stage(n, {...})`, `CISSP.explain(n, {...})`).
3. **Stable IDs.** Every concept, question and card has an ID that never changes (`d3-c9`, `s05-012`), because
   progress in localStorage is keyed by those IDs. Append new items; never reorder or renumber existing ones.
4. **The validator is the contract.** Each rule you care about ("every card has both languages", "every concept comes
   back twice", "no 3 cards of the same kind in a row") becomes a check in `tools/validate.js`. Nothing is
   committed unless it ends with `OK`.
5. **Two languages, one switch.** Content is in the target language (for CISSP: English, the exam language), and your
   native language is one tap away (RU). For German, see the language notes below.
6. **Pilot, review, then scale.** Build a small slice that uses every feature (stage 1 plus part of stage 2), look at
   it on desktop **and** on a phone, fix the format, and only then write the rest.
7. **Outline before bulk content.** Before writing stages 3–N, write `outline.md`: what each stage introduces, where
   each idea comes back, and which stories anchor it. Update it after each stage.
8. **One batch = one commit.** For example, "Add learning path stage 7: …". Run the validator before each commit.
   The git log then reads as the project's history.
9. **Original content, real facts.** Never copy official practice tests or paid question banks. Real cases and
   real examples must be accurate and well documented.
10. **`NEXT_STEPS.md` is always current.** Keep it in three parts: *Done* (what exists, with numbers), *Format reference*
    (data shapes), and *To do* (a prioritized plan, with finished items struck through). Update it in the same commit as the work.

## Repo layout (target)

```
README.md              what the app is, how to run it, current exam facts (with the date verified), repo layout
NEXT_STEPS.md          living plan: Done / Format reference / To do
PLAYBOOK.md            this file
Dockerfile             optional: nginx serving app/
.claude/launch.json    dev server entries so Claude can preview the app (node tools/serve.js <port>)
app/
  index.html
  css/style.css
  js/core.js           namespace, store (localStorage + export/import), i18n, Leitner INTERVALS
  js/views.js          dashboard, plan, topics, flashcards, quizzes
  js/exam.js           exam simulator
  js/learn.js          glossary pop-overs, diagram widget, basics page
  js/path.js           the learning path view
  data/meta.js         exam facts, study plan, strategy, resources
  data/d1.js … dN.js   per topic: concepts, glossary, questions
  data/x1.js … xN.js   beginner layer: plain-words explanation + example (+ diagram) per concept
  data/path/index.js   stage titles; data/path/s01.js … stage cards; data/path/outline.md
notes/                 your own free-form notes
practice/scores.md     log of external mock exams
tools/validate.js      content rules + coverage report
tools/check-diagrams.js  layout checks (if you use diagrams)
tools/serve.js         dependency-free static server
```

---

## Phases and the prompts to give Claude

Do the phases in order. Give one prompt per session or per batch, check the result, then commit.

### Phase 0 — Set the goal and verify facts

> I want a personal study repo for **[exam/goal]**. Base the approach on **[article/book/method link]**. Read it,
> summarize the method, then check the current official facts (format, sections, timing, scoring, fees,
> syllabus) on the official site, and note what changed since the source was written. Write a README with the plan and
> a "current exam facts (verified <month year>)" table. Remember the project goal in memory.

### Phase 1 — The study app skeleton

> Build an offline study app in `app/`: static HTML/CSS/JS, no dependencies, hash routing, progress saved in
> localStorage with JSON export/import. Use the same structure as the CISSP repo. Sections: dashboard (countdown,
> coverage, accuracy per section vs a target), study plan with checkboxes, topic pages (objectives, key concepts,
> glossary), flashcards with Leitner spaced repetition, quizzes with explanations in both languages, an exam simulator
> that matches the real format, exam strategy, and resources. Content goes in `app/data/*.js`. Add `tools/validate.js` to
> check the question bank, and `tools/serve.js` plus `.claude/launch.json` for previews.

### Phase 2 — Beginner layer

> Make every concept understandable for a beginner. Add a Basics primer page, glossary pop-overs (the first use of a
> term gets a dotted underline and shows its definition on hover or tap), and per concept: `plain` (an analogy in 2–4
> sentences), `ex` (a concrete real-world example) and an optional diagram. Do one topic per commit. Extend the
> validator to report "concepts explained N/M".

### Phase 3 — Polish

> Polish pass: the mobile layout (phone widths, swipe, full-screen diagrams), glossary density (link each term once per
> card), native-language matching for pop-overs, and a layout checker for diagrams. Update NEXT_STEPS.md.

### Phase 4 — Learning path pilot

> Add the learning path described in NEXT_STEPS.md item 0: a `#/path` view that resumes where I stopped, one card per
> screen, a stage list, ⭐ keep, and an optional refresh set using the Leitner intervals. Write stage 1 and the start of
> stage 2 (~60 cards) using all card kinds. Add the path rules to validate.js. Stop there so I can review it on desktop and phone.

### Phase 5 — Outline, then one stage per batch

> Write `app/data/path/outline.md`: every stage with its target card count, the concepts it introduces, its
> main comebacks and its anchor stories, plus a table of every concept with its intro stage → comeback stages.

> Write stage **N** following the outline. Run validate.js and check-diagrams.js, update the outline and NEXT_STEPS.md
> with what the stage actually contains, and commit as "Add learning path stage N: <title>".

Repeat until every stage exists. Then make the coverage rule an error: every concept is introduced once and revisited at least twice.

### Phase 6 — Review and grow

> Review the whole path on desktop and on a phone. Then grow it with more `back` and `think` cards where the outline
> shows the fewest comebacks.

---

## Learning path rules (subject-independent)

- **One continuous path**, ordered the way you would naturally learn the subject, not in syllabus order. Topic tags are
  kept only for filtering and progress.
- **Stages** of 35–90 cards. Each stage opens with a `story`/`idea` card and ends with a `recap` card.
- **Spiral repetition.** Each idea is introduced once and comes back 2–4 times in a new form (another case, a diagram,
  a comparison, a harder twist). Typical gaps: ~20 cards, then a stage or two later, then a late stage. A comeback card
  names the link ("Remember X? Here it is in…").
- **Rhythm.** Never 3 cards of the same kind in a row, and about 1 in 8 cards is `think`.
- **Short cards.** Body ≤ 120 words, one idea per card. Depth comes from more cards, not from longer ones.
- **Card kinds:** `story`, `idea`, `picture`, `vs`, `think`, `link`, `recap`, `back` (see NEXT_STEPS.md in the CISSP repo for the
  data format and examples).

Card format:

```js
APP.stage(1, { cards: [
  { id: 's01-001', kind: 'story', k: ['t1-c3'], lvl: 1,
    title: { de, ru }, body: { de: `<p>…</p>`, ru },
    reveal: { de, ru },        // think only
    take:   { de, ru },        // one-line takeaway
    back:   's01-004',         // back only: the earlier card it revisits
    items:  [['s01-002', { de, ru }], …] },   // recap only
] });
```

## Validator checklist (port these to every project)

- Unique IDs; IDs match their file (`s05-xxx` lives in stage 5)
- All language fields present
- Every `k` points to an existing concept; every `back` points to an **earlier** card
- `picture` has a diagram, `think` has a `reveal`, `recap` has 5–7 items that link to existing cards
- No 3 cards of the same kind in a row; each stage opens with `story`/`idea` and ends with `recap` (unless `partial`)
- Every concept is introduced once and revisited ≥ 2 times (a warning while stages are missing, an error once all exist)
- Prints coverage: cards per stage, concepts introduced N/M, comebacks per concept
- Project-specific checks (see the German section below)

---

## Applying it to Business German C1

### Phase 0 specifics

Pick **one** target exam first, because it decides the sections, task types and simulator. Candidates to check
on the official sites: **Goethe-Zertifikat C1** (modules Lesen, Hören, Schreiben, Sprechen), **telc Deutsch C1 Beruf**
(job-focused), and **TestDaF** or **telc C1 Hochschule** if the goal is academic. Ask Claude to verify the current format,
timing, scoring and fee and to note the date. The source method can be any well-documented C1 study plan or your
own plan. The CEFR C1 descriptors (Council of Europe) take the place of the "exam outline".

### What maps to what

| CISSP repo | Business German C1 |
|---|---|
| 8 domains with weights | Exam sections (Lesen, Hören, Schreiben, Sprechen) × business situations (meetings, negotiations, presentations, e-mail and letters, phone calls, reports, small talk, complaints, job interviews, workplace law and culture) |
| Concepts `d3-c9` | Grammar topics `g-c4` (Konjunktiv II, Passiv with modal verbs, Nominalisierung, Partizipialattribute, Konnektoren, indirekte Rede…) and communication functions `f-c7` (disagreeing politely, hedging, summarizing, escalating…) |
| Glossary terms | Vocabulary **chunks** and Redemittel, not single words: noun with article and plural, Nomen-Verb-Verbindungen (*eine Entscheidung treffen*), fixed phrases, register |
| Real incidents (Target 2013…) | Realistic business scenes and well-documented real cases (a merger, a product recall, a works-council dispute). Any statistics must be real |
| `think` card: "what would you do?" | "What would you say?": a situation, you form a reply, and the reveal shows 2–3 good C1 answers and why the literal translation from your native language sounds wrong |
| `vs` card | Confusable pairs: *bekommen/werden*, *scheinbar/anscheinend*, *Sie/du* at work, *seit/seitdem*, *bitten/beten/bieten*, formal vs neutral register |
| Diagrams | Sentence structure (Satzklammer, Mittelfeld order), e-mail structure, negotiation flow, register ladder |
| Exam simulator (CAT) | Timed mock per module: reading and listening auto-scored; writing and speaking with a self-assessment rubric based on the official criteria |
| "Think like a manager" | Exam strategy per module: time per task, the task types, what examiners score |

### Language-specific additions

- **Language fields:** `de` (target) plus `ru` and/or `en` support, toggled like EN/RU in CISSP. The German text is the
  main content; the translation is shown on tap.
- **Audio:** use the browser's `speechSynthesis` with a `de-DE` voice for every phrase and card (it needs no
  dependencies). Add a Hören practice mode: play the phrase, type what you heard, then compare.
- **Flashcards in both directions** (DE→RU and RU→DE). Show the article in colour (der/die/das) and the plural.
- **Writing practice:** prompts in the official task style, a checklist rubric (task fulfilment, coherence, range, accuracy,
  register), and model answers. Keep your own texts in `notes/writing/` and track mistakes in `practice/mistakes.md`.
- **Speaking practice:** prompts with a timer and a list of Redemittel to use. Recording is optional (MediaRecorder).

### Extra validator checks for German

- Every noun entry has an article (`der|die|das`) and a plural (or `—`)
- Every vocabulary entry has an example sentence and a register tag (`formell|neutral|umgangssprachlich`)
- Every `think` reveal has at least 2 model answers
- Every chunk is used in at least one card or quiz (otherwise it is never practised)
- Report: chunks per situation, grammar topics introduced N/M, comebacks per topic

### Suggested stage order (outline seed)

1. Why C1 is different: register, precision, nominal style (anchor: one e-mail rewritten from B1 to C1)
2. Self-introduction and small talk at work, *Sie/du* culture
3. E-mail and letters: structure, formulas, Konjunktiv II for politeness
4. Phone calls and video calls
5. Meetings: opinions, agreeing, disagreeing, interrupting politely
6. Presentations: structure, Konnektoren, describing charts and numbers
7. Reports and Nominalstil (Passiv, Partizipialattribute)
8. Negotiations: proposals, conditions, concessions
9. Complaints and conflict, indirekte Rede
10. Job interviews and HR topics
11. Workplace law and culture (Betriebsrat, Arbeitsvertrag, Datenschutz) as reading and listening material
12. Exam module training: Lesen and Hören task types
13. Exam module training: Schreiben and Sprechen, full mock exams

### First prompt for the German repo

> Read PLAYBOOK.md. This repo is for Business German at C1. My native language is Russian, and I also read English. Do
> Phase 0: help me choose between Goethe C1 and telc C1 Beruf (verify both on the official sites), then write the
> README with the plan and the verified facts, and save the project goal in memory. Then create NEXT_STEPS.md
> (Done / Format reference / To do) for Phases 1–6, adapted using the "Applying it to Business German C1" section.

---

## Session habits that worked

- **Start each session** with: "Read NEXT_STEPS.md and continue with the next item."
- **End each batch** with: validate, update NEXT_STEPS.md (and outline.md), commit, and `push` when you want it on GitHub.
- **Review on a phone** after every structural change; most of the polish fixes came from that.
- **Keep the scope of a batch** to one stage or one topic, so each commit can be reviewed and reverted on its own.
- **Ask for numbers** in NEXT_STEPS.md (cards, concepts covered, comebacks), so progress is measurable.
