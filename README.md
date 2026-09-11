# hanzigen

A tiny command-line tool that turns a daily list of Chinese words into
clean Anki cards - adding pinyin for me, skipping anything I already have.

## 1. The demo

I am learning Chinese and English. Every day I meet a few new Chinese
words, and I drop them into a plain text file, `today.txt`, one per line,
just the character and its English meaning separated by a semicolon:

```
狗;dog
猫 ; cat
狗;dog
ház;house

你好;hello
```

It is messy: a duplicate (`狗`), stray spaces, a blank line, and one bad
line (`haus` - not Chinese). I already have `known.txt` from earlier days,
which contains `你好;hello`. I type:

```
hanzigen today.txt --against known.txt --deck new.apkg --report clean.html
```

In under a second it prints:

```
Read 5 lines. 1 blank skipped, 1 not-Chinese skipped,
1 duplicate removed, 1 already-known skipped.
Wrote 1 new card to new.apkg.
```

Only 狗 is new and valid, so one card is written. The tool has filled in
the pinyin for me: front `狗`, back `gǒu — dog`. I open `clean.html` in a
browser and see a table of every line it skipped and exactly why, so I can
trust what went in and fix my source file next time. I import `new.apkg`
into Anki, and the single new card is there, pinyin and all.

## 2. The shape

```
in       today.txt   a text file, one entry per line, Chinese
                      characters and an English meaning separated
                      by a delimiter (default ';')
         --against    an optional file of entries I already have,
                      same format, used only to skip repeats
out      new.apkg     an Anki-importable deck, one card per NEW,
                      valid, unique entry; each card's back is the
                      auto-generated pinyin plus my meaning
         stdout       one summary line: read, skipped (blank,
                      not-Chinese, duplicate, already-known), written
         clean.html   optional table of every skipped or changed
                      line with its line number and the reason
on disk  the deck and report sit where I asked; today.txt,
         known.txt, and any prior deck are never modified
exit     0 when at least one card is written; 2 when the input file
         is missing, or nothing valid and new remains, with a
         message naming the problem
```

## 3. The size

**First useful version**

- read a `.txt`/`.csv` list, one entry per line, `characters<delim>meaning`
- clean: trim whitespace, drop blank lines, remove exact duplicates
- validate each entry as *well-formed*: the word field must be actual
  Chinese characters (Han script); flag and skip anything that is not,
  and flag lines missing a meaning
- generate pinyin from the characters automatically (offline, via the
  `pypinyin` library) and put `pinyin — meaning` on the card back
- with `--against FILE`, skip any entry already present there, so a
  daily list only adds genuinely new words
- build a valid `.apkg` with one basic (front/back) note per new entry
- print a one-line summary counting each category of skip and the number
  written; write an optional `clean.html` listing each skipped line and why
- never touch the input or the --against file; refuse to overwrite an
  existing `.apkg` unless `--force` is given

**Not this term**

- checking whether a meaning is *correct* (does 狗 really mean "dog") —
  that needs a dictionary lookup online or an AI, and this tool stays
  offline; I take responsibility for the meanings I type
- reading, merging, or repairing existing `.apkg` decks
- extra card types (cloze, reverse, character→pinyin→meaning splits) —
  first version is one front/back note per entry
- audio, stroke order, images, or example sentences
- fuzzy or tone-insensitive duplicate detection beyond exact match
- handling traditional/simplified conversion or multi-character ambiguity
  in pinyin beyond what `pypinyin` returns by default
- study statistics, retention reports, a GUI, AnkiConnect, or AnkiWeb sync

## 4. How we would know it works

- Given `狗;dog` appearing twice in one list, exactly one card is written
  and the summary reports `1 duplicate removed`.
- Given ` 猫 ; cat ` with stray spaces, the card's front is `猫` and the
  back reads `māo — cat`, spaces trimmed and pinyin generated.
- Given a line whose word field is not Chinese (e.g. `ház;house`), it is
  skipped, counted as `not-Chinese`, and named in the report.
- Given `--against known.txt` where `known.txt` contains `你好;hello`, a
  list line `你好;hello` produces no card and is counted `already-known`.
- Given a blank line and a line with no delimiter, both are skipped by
  line number in the report, and the run still exits 0 if one good new
  entry remains.
- Given a file where every line is blank, malformed, not-Chinese, or
  already known, no deck is written and the run exits 2 with a message.
- Given an output path that already exists, the run refuses and exits 2
  unless `--force` is passed; every input file is unchanged in all cases.
- Given the produced `.apkg`, importing it into Anki succeeds and the card
  count matches the summary's `Wrote N new cards`.

## 5. What could stop this

- Nothing external: no network, no account, no API, no personal data.
  Everything runs from local files and writes local files. `pypinyin`
  ships its dictionary offline.
- Two real dependencies carry the risk. First, the `.apkg` format: the
  danger is producing a file Anki silently rejects. The defence is the
  last test above — a real import into Anki must succeed, checked by hand
  for the first version and then against the `genanki` library's own
  round-trip. Second, pinyin accuracy: `pypinyin` can pick the wrong tone
  for characters with multiple readings. The defence is honesty — the
  card shows the pinyin so I can catch a wrong reading during review, and
  correctness of *meaning and reading* stays my job, not the tool's.
- The honest risk is the other direction: cleaning a list, generating
  pinyin, and zipping an `.apkg` is not much code, and an AI writes most
  of it. The substance is the fit to my day: I really do meet new words
  daily, typing pinyin by hand is the step I skip, and `--against` is what
  keeps a daily habit from rebuilding the same deck. If, next week, I do
  not reach for this on my own list, the brief was wrong and I should cut
  back toward the single step that genuinely annoys me — typing pinyin.
