# sentmine

A tiny command-line tool that reads an English passage I just finished,
finds the words that are new to me, and makes an Anki card for each one -
showing that word inside the real sentence it appeared in.

## 1. The demo

I am learning English by reading. I finish an article, copy the text into
a file called `passage.txt`, and it looks like this:

```
The stubborn old dog refused to move. Its owner sighed.
She was resilient, but the dog was equally stubborn.
```

I already keep a file, `known.txt`, of words I have learned before, one
per line - it contains `dog`, `old`, `move`, `owner`. Common words like
"the", "was", "to" I never bother listing, because the tool ignores them
by default. I type:

```
sentmine passage.txt --known known.txt --deck new.apkg --report clean.html
```

In under a second it prints:

```
Read 2 sentences, 16 words. 9 common words ignored,
4 already-known skipped, 1 repeat collapsed.
Wrote 2 new cards to new.apkg: stubborn, resilient, sighed.
```

Wait - it names three but wrote two? No: `stubborn` appears twice, so it
is collapsed to one card using its FIRST sentence. Two truly-new words get
cards. For `stubborn`, the card's front is the real sentence with the word
in bold:

```
front:  The stubborn old dog refused to move.   (stubborn in bold)
back:   stubborn
```

I open `clean.html` and see every new word, the sentence chosen for it,
and a list of the words it ignored or skipped and why, so I can trust the
result. I import `new.apkg` into Anki, and my new cards are there - each
word living inside a sentence I actually read.

## 2. The shape

```
in       passage.txt   a plain-text English passage, any length
         --known FILE   an optional file of words I already know, one
                        per line; matched case-insensitively
         --min-len N    optional: ignore words shorter than N letters
                        (default 3), on top of the built-in common-word
                        list
out      new.apkg      an Anki-importable deck, one card per NEW word;
                        front = the sentence containing it with the word
                        in bold, back = the word itself (its base form)
         stdout        one summary line: sentences and words read, and
                        counts for common-ignored, already-known,
                        repeats-collapsed, and cards written
         clean.html    optional table: each new word, the sentence
                        picked for it, plus the list of ignored/skipped
                        words with the reason for each
on disk  the deck and report sit where I asked; passage.txt and
         known.txt are never modified
exit     0 when at least one card is written; 2 when the input file is
         missing or unreadable, or no new word remains, with a message
         naming the problem
```

## 3. The size

**First useful version**

- read a plain-text English passage from a file
- split it into sentences (on `.`, `!`, `?`) and into words (on spaces
  and punctuation) - trivial for English because words are space-separated
- lower-case and strip punctuation from each word to get a comparison key;
  reduce a word to a simple base form for common endings (plural -s,
  past -ed, -ing) so "sighed" and "sigh" are not two cards
- decide "new": a word is new unless it is in the built-in common-word
  (stopword) list, is shorter than `--min-len`, or appears in `--known`
- collapse repeats: if a new word occurs more than once, keep one card and
  use the FIRST sentence it appeared in
- build one Anki card per new word: front is that sentence with the word
  wrapped in bold, back is the word's base form
- build a valid `.apkg` deck from those cards
- print a one-line summary with each count; write an optional `clean.html`
  listing new words with their chosen sentence, and ignored/skipped words
  with reasons
- never modify passage.txt or known.txt; refuse to overwrite an existing
  `.apkg` unless `--force` is given

**Not this term**

- any non-English language, especially Chinese or Japanese, because those
  need word-segmentation (no spaces between words) - a hard, separate
  problem; English's spaces are exactly what makes this tool feasible now
- looking up or generating a definition/translation for the word - the
  card teaches through context (the sentence), which is the whole point;
  adding meanings would need a dictionary or AI and is deferred
- real lemmatization or part-of-speech analysis (e.g. "ran" → "run",
  "better" → "good"); the first version handles only simple -s/-ed/-ing
  endings and accepts it will occasionally keep two forms of one word
- audio, images, phonetics, or example sentences beyond the one mined
- editing, merging, or reading existing `.apkg` decks
- a GUI, AnkiConnect, AnkiWeb sync, or reading PDFs/EPUBs directly
  (I paste plain text; extracting text from documents is out of scope)

## 4. How we would know it works

- Given the two-sentence passage above with `known.txt` holding
  `dog, old, move, owner`, exactly two cards are written - `stubborn` and
  `resilient` - and `sighed` too, unless it is in known; the summary
  counts match the cards.
- Given `stubborn` appearing in two sentences, only one card is written
  and its front is the FIRST sentence; the summary says `1 repeat
  collapsed`.
- Given the word `stubborn` as a new word, its card front is the exact
  source sentence with `stubborn` in bold and nothing else altered; the
  back is `stubborn`.
- Given `sighed` in the text and `sigh` already in `known.txt` (or vice
  versa), the base-form reduction means no card is written for it and it
  counts as already-known.
- Given only common words and known words in a passage, no deck is written
  and the run exits 2 with a message.
- Given `--min-len 5`, the word `dog` is ignored as too short even if it
  is not in known.txt, and the report says so.
- Given a missing or unreadable input file, the run exits 2 with a clear
  message and writes nothing.
- Given an existing output path, the run refuses and exits 2 unless
  `--force`; passage.txt and known.txt are unchanged in every case.
- Given the produced `.apkg`, importing it into Anki succeeds and the card
  count matches `Wrote N new cards`.

## 5. What could stop this

No external dependencies. The tool requires no network, account, or API, and handles no personal data. It reads and writes only local files, and the common-word list ships within the tool itself.
The core design is the choice of English. Chinese would first require segmenting words with no spaces between them - a hard, error-prone problem. English separates words with spaces, making the split reliable and reducing this from a research project to a weekend tool. The cost, stated above, is the absence of real lemmatization: "run" and "ran" may produce two cards. This is mitigated by tests covering the common -s/-ed/-ing cases; a stray duplicate is a minor, visible flaw, not a failure.
The .apkg format is the second risk: producing a file that Anki silently rejects. This is defended by the final test above - a genuine import into Anki must succeed, verified by hand and against the genanki library's round-trip.
The deeper risk is that the code is thin. Splitting text and packaging an .apkg is modest work that an AI largely automates. The value lies instead in the fit to how I study: I learn English by reading, I meet new words in real sentences, and a word recalled within its sentence is retained far better than one in isolation. The --known file prevents the tool from burying me in words I already have. Should the cards prove no more useful than plain word lists, the premise was wrong — but sentence mining is a method serious learners already rely on, which is why I trust the bet.
