---
date: 2020-12-20
lastmod: 2020-12-20
showTableOfContents: true
tags: ["nlp", "regex"]
title: "A surprising bug caused by regex"
type: "post"
description: "My program to process hundreds of documents would mysteriously stop on a particular document. After a couple of hours of checking all my code, I managed to isolate the problem to a particular regex search. I describe what I learnt in this blog post."
---

## Introduction
In my [Faculty Fellowship](https://faculty.ai/fellowship/), my project was to analyse legal deposition transcripts using NLP. As always, the first step was to clean the data and extract the useful information from the raw text from the transcripts.

I created a bunch of functions to do this processing and tested it on a few documents, and it all seemed fine.  I then ran the program on the complete set of documents, but it would mysteriously get stuck on a particular transcript.  After spending a couple of hours combing through my code, I managed to isolate the problem to this innocent looking regex search:

```python
pat = r'BY (M[RS]\. ([A-Z]+ ?)+|([A-Z]+ ?)+, ESQ)'
match = re.search(pat, 'SAME BY ANY MEANS UNLESS UNDER THE DIRECT CONTROL')
```

The pattern is looking for something like 'BY MR. JOHN SMITH' or 'BY JOHN SMITH, ESQ'.

I was extremely confused. Why should this regex search break anything? It obviously should return false: there is so 'MR' or 'MS' or 'ESQ' in the string, so why is it breaking?

## Catastrophic backtracking
I asked about this in the Faculty Slack channel, and I was helpfully directed towards the fantastic resource [regex101](regex101.com). If you enter the regex pattern and string from above into the website, it reveals the problem is 'catastrophic backtracking'. This basically means that the search tree that is used behind the scenes is humungous - leading to a regex search that would take forever to complete.

More explicitly, instead of searching for 'ESQ' up front, the regex search function creates all possible candidate strings that match the rest of the regex pattern, and then checks if each of those have 'ESQ' at the end. Based on how I wrote the pattern, the total number of possible candidate strings is enormous, because every substring of characters corresponds to a branch point in the tree.

## A solution
A simple solution was to add a word boundary:
```python
BY (M[RS]\. ([A-Z]+\b ?)+|([A-Z]+\b ?)+, ESQ)
```
This prevents branching occuring at every single character, and instead branching only occurs at every single word boundary.

## Conclusions
* [regex101](regex101.com) is a fantastic tool! I will be using this any time I need to use regex in the future.
* Debugging is harder than I already thought it was. Even things which I think are completely safe can be the cause of bugs. 