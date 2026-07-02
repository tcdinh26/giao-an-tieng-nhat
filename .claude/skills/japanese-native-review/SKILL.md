---
name: japanese-native-review
description: Review Japanese-language teaching content (giáo án, kanji explanations, vocabulary lists, example sentences, grammar points) for accuracy at a native-speaker level, and drive a generate-check-fix loop until the content is clean or a retry limit is hit. Use every time a lesson plan / giáo án about Japanese language is produced, right before presenting it to the user.
---

# Japanese Native-Level Review

Acts as a native Japanese speaker and qualified language teacher checking someone else's teaching material before it goes in front of students. Catches wrong readings, wrong meanings, wrong JLPT level, unnatural example sentences, and cultural/usage mistakes — the kind of errors a non-native content generator can introduce confidently and incorrectly.

## When to use

Trigger this immediately after producing (or regenerating) any Japanese-language lesson plan, kanji lesson, vocabulary list, or curriculum content — before showing the final version to the user. This includes output from the `teaching-lesson-plan` and `japanese-lesson` skills when the topic is Japanese.

## Review Checklist

For every kanji, word, grammar point, and example sentence in the content, check:

1. **Readings** — Are 音読み (on'yomi) and 訓読み (kun'yomi) correct and correctly labeled? Are readings shown in hiragana/katakana, never romaji (unless the user explicitly asked for romaji)?
2. **Meaning** — Is the stated meaning/translation accurate and is it the primary meaning (not an obscure secondary one) unless context calls for the secondary one?
3. **JLPT level** — Is the kanji/vocab/grammar point actually classified at the stated N5–N1 level? Flag if a word is presented as N5 but is actually N3+ or vice versa.
4. **Grammar correctness** — Are grammar patterns, conjugations, and particle usage in example sentences grammatically correct?
5. **Naturalness** — Would a native speaker actually say the example sentences this way, or are they technically correct but stilted/unnatural machine-Japanese?
6. **Register and social context** — Politeness level (敬語/謙譲語/丁寧語), uchi-soto (内・外) distinctions, and any situation where using the wrong register would be a real mistake a native speaker would notice (e.g. 母 vs お母さん, casual vs formal verb forms).
7. **Internal consistency** — Do stated stroke counts, radicals, and example words match what's taught elsewhere in the same document? Does timing in the lesson structure add up to the stated duration?

## Workflow (generate → check → fix loop)

1. Generate the giáo án / lesson content as normal.
2. Run this checklist against it. Produce a findings list: each issue gets a one-line description, the location in the content, and severity (`blocking` = factually wrong, would teach students something incorrect; `minor` = stylistic/could be more natural).
3. If there are zero `blocking` findings, stop — present the lesson plan to the user, and note it passed native-level review.
4. If there are `blocking` findings, regenerate only the affected parts of the lesson plan to fix them (do not regenerate from scratch), then go back to step 2.
5. **Hard cap: 3 total review passes.** If blocking issues remain after the 3rd pass, stop looping. Present the lesson plan anyway, but list the unresolved issues explicitly so the user can judge whether to use it — do not silently ship unverified content, and do not loop forever.

## Output

After the loop ends, report to the user in this format:

```
Kiểm tra bản địa: [X/3 vòng] — [Sạch lỗi | Còn N vấn đề chưa tự sửa được]

[Nếu còn vấn đề, liệt kê]:
- [Vị trí trong giáo án] — [Vấn đề] — [Mức độ: blocking/minor]
```

## Anti-patterns

- Do not mark content as passed if you are not actually confident in a reading/meaning — say so and flag it as unresolved rather than guessing.
- Do not regenerate the entire lesson plan from scratch on every fix pass — only touch the specific flagged items, to avoid introducing new errors elsewhere.
- Do not loop more than 3 times even if issues remain — report and let the user decide.
- Do not skip this review for content that "looks simple" (e.g. a single kanji) — simple content is exactly where overconfident wrong answers slip through.
