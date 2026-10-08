# Say It Plainly

A plain language audit skill for Figma's agent. It checks whether your copy is easy to understand.

Most copy tools check style: capitalization, punctuation, brand voice. This one checks clarity. It scores reading level on longer text and flags long sentences, jargon, passive voice, and hidden verbs.

## Why I made this

Clear copy helps everyone, not just people with lower reading levels. Research from Nielsen Norman Group shows even experts prefer plain language. In civic tech and government work, it's often required by law.

Readability formulas are useful but easy to misuse. They give precise-looking scores even when the text is too short to measure, and AI tools tend to guess at those scores instead of calculating them. I wanted a skill that's honest about what it can and can't tell you.

## What it checks

| Check | What it looks for |
|---|---|
| Reading level | Flesch-Kincaid grade, calculated in code |
| Long sentences | Sentences over 20 words |
| Jargon and acronyms | Words with plain swaps, acronyms not spelled out |
| Passive voice | Sentences that hide who is acting |
| Hidden verbs | Actions turned into nouns, like "make a decision" |

## How it works

- **Reading level only runs on longer text.** Readability formulas were built for samples of about 100 words. Text with 70 to 99 words gets a score marked as a rough guide. Text under 70 words gets no score, just the other checks.
- **Scores are calculated, never guessed.** The agent runs the formula in code and shows the word, sentence, and syllable counts so you can check the math.
- **Your copy is never changed.** The skill places a suggested version next to your original, re-scores it, and points out any details the rewrite changed. Notes go in a column beside it, and chat just gets a short summary.
- **Hidden layers are skipped**, since they're usually alternate states or scratch work.

## How to use it

1. Save the skill in Figma, or copy `SKILL.md` into your own skills.
2. Select a frame and run the skill with `/say-it-plainly`.
3. Or ask for something specific:
   - "Audit this frame"
   - "Check jargon only"
   - "Rewrite this at a grade 8 level"
   - "Why was this flagged?"

**Settings you can change:**
- **Audience:** General (target grade 8, limit 9) or Expert (limit 12)
- **Keep list:** words that must stay as written, like legal or brand terms

## Limits

- Formulas only measure word and sentence length. They can't tell if people actually understand the text.
- The syllable counter is an estimate. Names, acronyms, and numbers can throw it off.
- Simpler isn't always clearer. A rewrite can be easier to read but drop context people need.

Treat the results as suggestions. The real test is watching real people use your product.

## Research behind it

The thresholds and checks are based on readability research and plain language guidelines, including Flesch (1948), Kincaid et al. (1975), Redish (2000), Schriver (2017), Nielsen Norman Group, Newcastle University's design system, the University of Pennsylvania's Joint Research Practices plain language workshop (2023), and the Federal Plain Language Guidelines. Full citations are at the end of `SKILL.md`.

## How this was made

I designed this skill and made the key decisions: what to check, when a score is reliable enough to show, and how results should be reported. I used Claude, an AI assistant, to help research sources and draft the skill file, then reviewed and edited it myself.

That fits the point of this project. AI is useful, but its output should be checked, not trusted blindly.

## Version history

- **v0.2** Tested in Figma over 3 rounds. Results now go on the canvas: original, a re-scored suggested version, and note cards. Runs with default settings instead of asking. Never guesses what an acronym means. Only changes what a finding calls for, and lists anything else. Explains each issue once and ends with a short to-do list. Builds everything in two script calls, cutting run time from about 4 minutes to 1.5.
- **v0.1** First draft. Not yet tested on real Figma files.

## Author

Made by [togglefairy](https://github.com/togglefairy)

## License

MIT. Free to use and adapt with credit.
