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
