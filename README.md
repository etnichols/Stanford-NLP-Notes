# Stanford-NLP-Notes
Notes following along with Professor Dan Jurafsky &amp; Chris Manning's online NLP course

## 1-1 Course Introduction

### Applications of NLP
- Question Answering (Jeopardy)
- Information Extraction (Create calendar appointment from email)
- Sentiment Analysis (Product review aggregation)
- Machine Translation ([Phrasal Project](https://github.com/stanfordnlp/phrasal))

### Problems in Language Tech
Mostly solved: email spam detection, Part-of-Speech (POS) tagging, Named Entity Recognition (NER)
Good progress: all the Applications in first section, "word sense" disambiguation (e.g. Computer mouse), Parsing
Hard: Question Answering, Paraphrasing, Summarization, Dialogue with human

### Ambiguity Makes NLP Hard

```"Violinist Linked to JAL Crash Blossoms" ```

Correct interpretation: Violinist who was involved in the JAL crash is now "blossoming," getting really popular.
Incorrect interpretation a parser could make: A violinist is linked to "JAL Crash Blossoms." [Wtf is a Crash Blossom](http://www.testycopyeditors.org/phpBB3/viewtopic.php?f=8&t=11134)?

```
"Teacher strikes idle kids"
"Red tape holds up new bridges"
```

Parser may think that red tape is physically supporting the new bridges

### Ambiguity is Pervasive

```"Fed raises interest rates"```

"Fed raises     interest     rates" (The Fed raises are getting interest from the rates)

And what about...

"Fed raises interest rates 0.5%"

### Other Difficulties with NLP
- Slang ("non-standard English")
- Segmentation (New York-New Haven Railroad)
- Idioms
- Entity Names

# 2-1 Regular Expressions
"Formal language for specifying text strings"

- Disjunctions [ ] Letters inside square brackets. ```[wW]oodchuck```
- Ranges ```[A-Za-z]```
- Negations in disjunctions ```[^A-Z] "not an uppercase letter"```
- Pipe | or. ```a | b | c == [abc]```
- ? the previous character is optional ```[colou?r]```
- * 0 or more of the previous character ```00*1```
- + 1 or more of the previous character ```0+1```
- . any character
- ^[A-Z] Anchor to beginning of line "match capital letters at the beginning of lines"
- $[A-Z] Achor to end of line "match cpas at the end of lines"

### Create RegEx for all instances of "the"
```
[Tt]he                      // still finds "oTHEr"
[Tt]he^[A-Za-z]             // still finds "bliTHE"
^[A-Za-z][Tt]he^[A-Za-z]    // yay
```

### Type I and Type II Errors
**Type I**: False positive: matching things we should not have matched
**Type II**: False negative: not matching things we should have matched

NLP constantly deals with these errors through 2 antagonistic efforts:
1. Increase *accuracy* or *precision* by minimizing Type I errors
2. Increase *coverage* or *recall* by minimizing Type II errors.

# 2-2 RegEx in Practical NLP
Lexers == Tokenizers.
Stanford English Tokenizer.

# 2-3 Word Tokenization
How many words are in a text? Depends on how you define words. And when dealing with natural language:

```
"Yeah uh I work month-monthly rotations here."
```

How do we deal with uh, a "filled pause"?
How do we deal with month-, the "fragment" before the actual intended word?

**Lemma**: same stem, POS, roughly the same word sense ```cat == cats"```
**Wordform**: full inflected surface form ```cat !== cats```

**Type**: an element of the vocabulary
**Token**: an instance of that type in the running text

**N** usually defines the number of tokens. **V** represents the vocabulary. **\| V \|** is the size of the vocabulary.

Use the [tr unix command](https://en.wikipedia.org/wiki/Tr_\(Unix\)) along with sort and unique to split a text into a word-per-line doc, count all words, sort by frequency

```
tr 'A-Z' 'a-z' < input.txt | tr -sc 'A-Za-z' '\n' | sort | uniq -c | sort -n -r > output.txt
```

### Issues in Tokenization
- Finland's
- What're, I'm, isn't
- Hewlett-Packard
- State-of-the-art
- San Francisco
- m.p.h. PhD
- Language issues
  - French: L'ensemble
  - German: noun compounds are not segmented. "Life insurance company employee" == "Lebensversicherungsgesellschaft Mitarbeiter"
  - Chinese/Japanese: no spaces between words
  - Japanese: multiple alphabets intermingled

### Maximum Matching Word Segmentation
```
thecatinthehat === the cat in the hat
thetabledownthere === theta bled own there
```
The algorithm above works well for Chinese, where words are composed of characters, and the average word length is ~2.4 characters. Characters are generally 1 syllable and 1 morpheme.

## 2-4 Word Normalization and Stemming

### Normalization
- Normalizing terms means different things for different applications. In information retrieval, we want the indexed text & query terms to have the same form. E.g. we want to match U.S.A. and USA.
- Implicitly define equivalence classes of terms, e.g. deleting the periods in a term. Alternatively, we could use *asymmetric expansion*, generating other search terms based on the one passed in.
  - User enters: **window** so we search for: **window, windows**
  - User enters: **windows** so we search for: **Windows, windows, window**
  - User enters: **Windows** so we search for: **Windows**
- Asymmetric expansion is potentially more powerful than equivlence class approach, but less efficient.
- In applications like IR: reduce all letters to lowercase, since users tend to use lower case. A possible exception: encountering upper case in mid-sentence, e.g. **sail**, **sail** vs **sail**, **sail** vs. **sail**.
- For sentiment analysis, MT, Information Extraction, case IS helpful (**US** versus **us** is important)

### Lemmatization
- Reduce inflections or variant forms to base form
  - am, are, is --> be
  - car, cars, car's, cars' --> car
  - *the boy's cars are different colors --> the boy car be different color*
- **Lemmatization**: have to find correct dictionary headword form
- Machine Translation
  - Spanish **quiero** ('I want'), **quieres** ('you want') is the same lemma as **querer** ('want')

### Morphemes
- The small meaningful units that make up words
- **Stems**: the core meaning-bearing units
- **Affixes**: bits and pieces that adhere to stems
  - Often with grammatical functions

### Stemming
- Reduce terms to their steams in information retrieval
- *Stemming* is crude chopping of affixes
  - language dependent
  - e.g. **automate(s)**, **automatic**, **automation** all reduced to **automat**.

### Porter's Algorithm
The most common English stemmer. A series of search and replace rules to reduce words down to their stems.

- Step 1A
  - sses -> ss
  - ies -> i
  - ss -> ss
  - s -> ø
- Step 1B
  - (\*v\*)ing -> ø
  - (\*v\*)ed -> ø
- Step 2 (for long stems)
  - ational -> ate
  - izer -> ize
  - ator -> ate
- ... even more complicated rules for very long stems

### Using Unix tools to look at morphology in a corpus

Code below does the usual stuff, and then we ```grep``` for all words ending in "ing".
```
tr -sc 'A-Za-z' '\n' < mlkdream.txt| tr 'A-Z' 'a-z' | grep 'ing$' | sort | uniq -c | sort -n -r > test.txt
```

```
  12 ring
   3 sweltering
   3 sing
   2 suffering
   2 meaning
   2 knowing
   1 withering
   1 tranquilizing
   1 stating
   1 something
   1 signing
   1 nothing
   1 meeting
   1 lodging
   1 languishing
   1 jangling
   1 invigorating
   1 honoring
   1 heightening
   1 having
   1 gaining
   1 dripping
   1 drinking
   1 cooling
   1 beginning
   1 awakening
   1 asking
```
Nice, but there are some words here where we don't want to apply our stemming rule. Ring would just be r. Sing would just be s.

Let's modify the grep rule to find all words ending in "ing" but that have a vowel beforehand, "has an earlier vowel".

```
tr -sc 'A-Za-z' '\n' < mlkdream.txt | tr 'A-Z' 'a-z' | grep '[aeiou].*ing$' | sort | uniq -c | sort -n -r > test2.txt
```

```
   3 sweltering
   2 suffering
   2 meaning
   2 knowing
   1 withering
   1 tranquilizing
   1 stating
   1 something
   1 signing
   1 nothing
   1 meeting
   1 lodging
   1 languishing
   1 jangling
   1 invigorating
   1 honoring
   1 heightening
   1 having
   1 gaining
   1 dripping
   1 drinking
   1 cooling
   1 beginning
   1 awakening
   1 asking
```

Nothing is still an issue here, but this addition improved our rule overall.
