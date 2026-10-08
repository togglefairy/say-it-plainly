---
name: say-it-plainly
description: Checks UI and content copy for plain language. Scores reading level (Flesch-Kincaid) on longer text and flags long sentences, jargon, passive voice, and hidden verbs. Use when the user wants to review, rewrite, or ask about how clear their copy is.
---

# Say it plainly

A plain language audit for UI and content copy. This skill checks whether copy is easy to understand. It does not check brand style, grammar mechanics, or tone of voice. It can run alongside a style guide skill.

When invoked, figure out what the user wants:

- **Reviewing:** check the selected copy and flag problems. This is the default.
- **Drafting:** rewrite copy so it meets the target reading level and passes the checks.
- **Answering:** explain a rule or a result, like "why was this flagged?" or "what is passive voice?"

The user can also ask for one check only, like "check jargon only" or "just the reading level on this frame." Run only what they ask for.

## Confidence

These are suggestions, not rules. Readability formulas measure word and sentence length. They cannot tell if real people understand the text. Frame every finding as something to consider, not a verdict. Never present a score as proof that copy is clear. Testing with real users is the real check.

A grade level describes how hard the text is to read. It does not mean the copy is written for children in that grade.

## Settings

Don't stop to ask. Run with the defaults below unless the user said otherwise. Name the settings you used in the chat summary, like "Using general audience (grade 8). Tell me if you want expert mode or a keep list."

**Audience**
- **General (default):** target grade 8, flag anything above grade 9.
- **Expert:** flag anything above grade 12. Allow common terms of the field, but still flag terms that have plain swaps.

**Keep list**
Words that must stay as written, like legal terms, product names, or brand terms. Never flag or rewrite these.

## Collect the copy

1. Walk the selected frames for text layers. Record each string with its layer name and parent frame, like "Primary button," "Error message," or "Body."
2. **Skip hidden layers.** Hidden layers are usually alternate states or scratch work. Check them only if the user asks.
3. **Group related text.** Combine text layers that sit in the same section or card into one block. A heading plus three short paragraphs is one piece of reading.
4. Use the layer name to tell microcopy from body copy. Buttons, labels, tabs, placeholders, and toasts are microcopy.

**No selection:** say so, then offer two paths. The user can select something, or paste the text into chat.

## Decide which checks apply

Count words in each block. Use word count, not line count. Line count changes with frame width.

| Words in block | Reading level | Other checks |
|---|---|---|
| 100 or more | Full Flesch-Kincaid score | All |
| 70 to 99 | Score, marked "rough guide, short sample" | All |
| Under 70 | No score. Say: "Too short for a reliable reading level score." | All |

Readability formulas were built for samples of about 100 words. Short text gives unstable scores, so do not report one.

## Calculate the reading level

**Never estimate the score.** Language models are bad at counting syllables. Always calculate it with code using `evaluate_script`, then report the inputs so the user can check the math.

Use this function:

```javascript
function countSyllables(word) {
  word = word.toLowerCase().replace(/[^a-z]/g, '');
  if (!word) return 0;
  if (word.length <= 3) return 1;
  word = word
    .replace(/(?:[^laeiouy]es|ed|[^laeiouy]e)$/, '')
    .replace(/^y/, '');
  const groups = word.match(/[aeiouy]{1,2}/g);
  return groups ? groups.length : 1;
}

function fleschKincaid(text) {
  const words = text.split(/\s+/).filter(w => /[a-zA-Z]/.test(w));
  const sentences = text.split(/[.!?]+/).filter(s => /[a-zA-Z]/.test(s));
  const sentenceCount = Math.max(sentences.length, 1);
  const syllables = words.reduce((sum, w) => sum + countSyllables(w), 0);
  const wps = words.length / sentenceCount;
  const spw = syllables / words.length;
  return {
    words: words.length,
    sentences: sentenceCount,
    syllables,
    grade: +(0.39 * wps + 11.8 * spw - 15.59).toFixed(1),
    readingEase: +(206.835 - 1.015 * wps - 84.6 * spw).toFixed(1)
  };
}
```

The syllable counter is an approximation. Names, acronyms, and numbers can throw it off. Mention this when the text has a lot of them.

## The checks

### Reading level
Flag when the grade is above the audience limit. Show the grade, the target, and the counts.

### Long sentences
- Flag sentences over 20 words.
- Mark sentences over 30 words as high severity.
- This check runs on all copy, even short blocks.
- Flag paragraphs over 5 sentences or 150 words. Suggest splitting them or adding a subheading.

### Jargon and acronyms
- Flag words that have a common plain swap. Examples:

| Instead of | Try |
|---|---|
| utilize | use |
| facilitate | help |
| commence | start |
| terminate | end |
| prior to | before |
| subsequent | next, later |
| in order to | to |
| approximately | about |
| sufficient | enough |
| regarding | about |
| obtain | get |
| additional | more |
| leverage | use |
| functionality | features |
| assistance | help |
| on a monthly basis | monthly |
| a number of | many, some |
| and/or | "A or B," or "A, B, or both" |

- Flag acronyms that are not spelled out the first time they appear.
- For mixed audiences, suggest the plain term first with the technical term in parentheses.
- Never flag words on the keep list.

### Passive voice
- Flag sentences where the actor is hidden or comes after "by," like "Your file was deleted by an admin."
- Passive voice is fine when the actor is unknown or doesn't matter, like "Your password was reset." Don't flag these, or mark them low severity.

### Hidden verbs
Flag actions turned into nouns, like "make a decision" (decide), "provide assistance" (help), or "conduct a review" (review).

## Severity

- **High:** reading level above the limit, sentences over 30 words
- **Medium:** jargon, undefined acronyms, passive voice that hides who acts, sentences of 21 to 30 words
- **Low:** hidden verbs, borderline passive voice

List findings from high to low.

## Rewrites

- **Never change the original copy** unless the user asks you to apply a fix.
- Keep the meaning. If a simpler version might change the meaning, say so.
- Never remove required information, like legal terms, prices, dates, or deadlines.
- Keep words on the keep list as written.
- Mechanical fixes, like a jargon swap, get one fix.
- Rewrites of whole sentences get one fix and up to two alternatives, only if they are truly different.

## Output format

**Everything goes on the canvas. Chat gets a short summary.**

### In chat

Show only the summary table, then one line about what you added:

| Check | Result |
|---|---|
| Reading level | Grade 14.4 → 7.9 (target 8, limit 9) |
| Long sentences | 2 flagged |
| Jargon and acronyms | 3 flagged |
| Passive voice | 1 flagged |
| Hidden verbs | Pass |

"I added 6 notes and a suggested version next to Frame 1."

Don't list each finding in chat. If the user asks to see the details in chat, then list them.

### Keep it fast

Use only two script calls per run:

1. **First call:** read all the visible text layers in the selection and score each block with the reading level code.
2. **Second call:** score the suggested version, then build everything on the canvas in that same call: the suggested version, its label, the notes title, all the cards, and the group. Load fonts once at the start.

Don't make a separate call for each card.

### On the canvas

Lay things out left to right so the user can compare:

**Original frame → Suggested version → Notes**

Keep 40px between each one. Never edit the original layers or text.

**1. Suggested version**

Make this for any block of 70 words or more. For microcopy, the notes are enough.

- Duplicate the block and place it to the right of the original. Name it "[original name] · say-it-plainly suggestion."
- Apply all the fixes together, so one fix doesn't undo another. For example, if an acronym is undefined, don't reuse it unexplained in a rewritten sentence.
- Re-score the new version with the same code. Add a small label above it: "Suggested version · Grade 14.4 → 7.9."
- Compare it to the original. If any fact, number, step, or requirement changed or went missing, say so in a "Check before using" note. If nothing changed, say "No details removed."
- Keep words on the keep list as written.
- **Only change what a finding calls for.** Don't swap other words, like "instructors" to "teachers." If you change anything a finding didn't ask for, list it in the "Check before using" note.
- **Never guess what an acronym stands for.** In the suggested version, write a placeholder like "[spell out CHW] (CHW)" so the user fills it in.

**2. Note cards**

Add one card per finding. Problems only. Skip anything that passed.

- Place the cards in a column to the right of the suggested version, with a title: "Plain-language review notes."
- Stack them from high to low severity, with 12px between cards.
- Each card shows: **[Check name] · [Severity]**, then **Issue**, then **Fix**, then **Alt** only if truly different. Add a **Caution** line when a fix might change the meaning.
- **Explain each issue once.** Put the explanation on the issue's own card only. Fixes on other cards can still include a placeholder like "[spell out CHW]," but don't repeat the caution there.
- **End with a "Check before using" card** that works as a short to-do list: one line per action, like "Fill in [spell out CHW]" or "Confirm 'meet the needs' keeps the meaning of 'sufficient.'" No explanations, since those are on the other cards. If there's nothing to check, say "No details removed." 
- For reading level, include the counts: words, sentences, syllables, grade, and reading ease.
- Group the title, cards, and suggested version as "say-it-plainly review" so they're easy to move or delete.
- On a rerun, delete the old "say-it-plainly review" group for that frame first, so they don't pile up.

## Limits to keep in mind

- Formulas only measure word and sentence length. Short words can still be confusing, and long words can be familiar.
- Scores on text under 100 words are less reliable.
- Text split across many layers can be grouped wrong. If the grouping looks off, ask the user.
- Simpler is not always clearer. A rewrite that drops context can be easier to read but harder to act on.

## Sources

- Flesch, R. (1948). A new readability yardstick. *Journal of Applied Psychology, 32*(3), 221–233. https://doi.org/10.1037/h0057532
- Kincaid, J. P., Fishburne, R. P., Rogers, R. L., & Chissom, B. S. (1975). *Derivation of new readability formulas (Automated Readability Index, Fog Count and Flesch Reading Ease formula) for Navy enlisted personnel* (Research Branch Report 8-75). Chief of Naval Technical Training, Naval Air Station Memphis. https://apps.dtic.mil/sti/pdfs/ADA006655.pdf
- Klare, G. R. (1974). Assessing readability. *Reading Research Quarterly, 10*(1), 62–102.
- Redish, J. (2000). Readability formulas have even more limitations than Klare discusses. *ACM Journal of Computer Documentation, 24*(3), 132–137. https://doi.org/10.1145/344599.344637
- Schriver, K. A. (2017). Plain language in the United States gains momentum: 1940–2015. *IEEE Transactions on Professional Communication, 60*(4).
- DuBay, W. H. (2004). *The principles of readability.* Impact Information.
- Nielsen, J. (2005). Lower-literacy users: Writing for a broad consumer audience. Nielsen Norman Group.
- Nielsen Norman Group. UX writing FAQs. https://www.nngroup.com/articles/ux-writing-faqs/
- Newcastle University Design System. Readability. https://www.ncl.ac.uk/design-system/ux/editorial/readability/
- Sheu, D. (2023, June 8). *Plain language* [Workshop slides]. Joint Research Practices Working Group, University of Pennsylvania (PAIR Center and CHIBE).
- Plain Language Action and Information Network. (2011). *Federal plain language guidelines.* https://www.plainlanguage.gov
