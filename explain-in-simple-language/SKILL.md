---
name: explain-in-simple-language
description: Explain code, a decision, a recap, or a question to the user, or write a document a human reads, so a fluent but often non-native reader understands it on the first read. Use when explaining or asking the user anything, when writing a review, report, or brief a human reads, or when the user asks for simpler words or says they did not understand.
---

# Explain in simple language

This skill owns only the wording: how sentences and paragraphs are built, and what you add on your
own initiative. What to say (the evidence, the required parts) and the voice stay with whatever
asked for the text.

## The reader

A competent developer, fluent in English but often not native, who did not see what you just saw:
the files, the tool output, your own reasoning. When whatever asked for the text names other
readers, write for the least technical of them: a code name comes with the plain name of what you
saw it do. Test: would a colleague who knows the domain but just walked up to your desk understand
this?

## Steps

1. **Pick the reader.** Decide who reads the text, using [The reader](#the-reader). Continue once
   you know the least technical reader.
2. **Draft.** Write the content the caller asked for, applying the [Rules](#rules) as you go.
3. **Check each sentence.** Re-read it as that reader. Rewrite a sentence only if it breaks a rule
   and another wording would raise the chance it is understood; leave every other sentence alone.
4. **Stop.** Send the text once no sentence fails the check in step 3. Don't polish further.
5. **If the user says they don't understand,** apply the "I don't understand" rule, then go back to
   step 3.

## Rules

- **Name things concretely.** Never a stand-in for something you know and the reader doesn't: a
  reference ("both directions", "the fallback", "the second approach"), a term you coined where
  the subject already has a name ("the baseline" for the saved count), or shorthand for a case the
  reader didn't watch you work out ("a pick closes its finding"); say what it is, in their words.
  Lost the reader: "the class serves both directions, and the index only matters in one of them."
  Understood on the first read: "the class does two jobs, reading the file the user uploads and
  writing the one they download. The index only matters for the reading job." Longer, and better.
- **Never echo the wording of whatever instructions requested the text**: the reader never saw
  them ("the part worth a close look", not the calling skill's "files where judgment matters").
- **Only what you saw.** Context, names and reasons come from what you verified, never from what
  would make the sentence clearer. Clarity never beats accuracy.
- **Words.** Plainer words preferred; a fancy or technical term is fine unless it confuses. Keep a
  term the user already used or that is the subject; introduce a term before leaning on it; reuse
  the user's own words where you can.
- **Sentences.** One idea per sentence, one point per paragraph. Concrete case before general
  mechanism.
- **Depth.** Answer what was asked, at the depth it was asked, then stop: no history, alternatives,
  or adjacent facts unless the answer can't stand without them or whatever asked for the text
  requires them. When the user asks what to do or which option to take, open with your
  recommendation, then the why; a list of the options only when the user or whatever asked for the
  text asks for one. Prefer brevity, never at the expense of clarity:
  - if the same thing fits in fewer words without losing a detail that matters, use fewer words;
  - if the text is long anyway, lead with a simple and short TLDR, then the details;
  - if a few more words make it clearer, spend them.
- **Questions.** Before asking, give just enough context to answer, in concrete terms: what the
  choice is between and why it matters.
- **"I don't understand".** Don't restate the same sentence with more words: rebuild it from a
  concrete case and drop the term or reference that lost them.

## When to stop and ask

- If clear wording needs a fact you haven't verified, say it is unchecked, or check it first.
- If the user still doesn't understand after one rebuild, ask which part lost them instead of
  rewording again.
- If the user misunderstands twice because of the wording, after this skill was applied or when it
  should have been, the skill is the problem: hand it to `self-improve`.
