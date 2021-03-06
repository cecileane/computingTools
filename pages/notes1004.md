---
layout: page
title: more shell tools, sed
description: course notes
---
[previous](notes0929.html) &
[next](notes1006.html)

---

- earlier [summary](notes0908.html#summary) of useful commands
- [commands](#useful-shell-commands) we don't tend to use everyday
  (but perhaps we should):
  `cut`, `sort`, `column`, `basename`, `dirname`, `tree`
- stream editor: [sed](#stream-editor-sed)

## useful shell commands

### cut

if not done already, clone these [data](https://github.com/vsbuffalo/bds-files)
from the "Bioinformatics Data Skills" book by Vince Buffulo
(outside any existing git repo!).

```shell
cd bds-files/chapter-07-unix-data-tools
ls -lh # 1.6M = size of Mus_musculus.GRCm38.75_chr1.bed
head Mus_musculus.GRCm38.75_chr1.bed # chromosome number, start & end position
cut -f 1 Mus_musculus.GRCm38.75_chr1.bed | sort | uniq # check chromosome 1 only
cut -f 2 Mus_musculus.GRCm38.75_chr1.bed | head -n 3
```

Each row is for a genomic "feature",
same as in the associate `.gtf` file.
We can check the `.bed` and `.gtf` files have the same number of lines
except for lines of metadata (how?)
quick look at `.gtf` file:
`less -S Mus_musculus.GRCm38.75_chr1.gtf`

other ways to use options for `cut`:

- `-f2`, `-f 1,3`, `-f1-3`
- `-c2` to cut (extract) the second character, not the second field (column)
- `-d` to change the delimiter between columns fields instead of tab:
  `-d ' '` for a space, `-d,` `-d ,` or `-d ','` for a comma (csv files).


example from research: data used in
[Baum et al. 2016](http://onlinelibrary.wiley.com/doi/10.1111/evo.12934/full)
originally from [Perelman et al. 2011](http://journals.plos.org/plosgenetics/article?id=10.1371/journal.pgen.1001342).  
54 genes in 178 primate species, from 186 individuals
(different subspecies).

to get the example file: clone
[lecture-examples](https://github.com/UWMadison-computingtools-master/lecture-examples)
(and 'git pull' to get updates later in the semester)

```shell
cd ../../classroom-repos/lecture-examples/
less -S combined.nex
wc combined.nex
```

data cleaning: needed to get the list of individuals and to identify species
with multiple subspecies; we needed to remove 'duplicates' to keep a single
representative per species.
step I used to get the current list of "taxa" in my cleaned file:
<!-- see more in readme.md in that folder -->

```shell
grep '_' combined.nex | cut -f 1 -d ' ' > taxa
wc taxa
head taxa
```

### sort

`-k` to sort by specific columns (keys). example below:
`-k1,1` to sort by keys in columns 1 to 1,
then `-k2,2` to resolve ties by to sorting columns 2 to 2, and
`n` to sort that 2nd column numerically,
`r` to sort that 2nd column it in reverse order.  
`-c` to check if the file is sorted already (fast)  
`-t,` or `-t ","` to change the separator to a comma instead of tab (default)

```shell
cd ../../bds-files/chapter-07-unix-data-tools
sort -k1,1 -k2,2n example.bed
sort -k1,1 -k2,2nr example.bed
sort -k1,1 -k2,2n -r example.bed
sort -k1,1 -k2,2n -c example.bed
sort -t, -nc Mus_musculus.GRCm38.75_chr1_bed.csv       # assumes GNU sort
sort -t, -n Mus_musculus.GRCm38.75_chr1_bed.csv | head # GNU sort
sort -t, -nr Mus_musculus.GRCm38.75_chr1_bed.csv | head
```

exercise: extract & count all the features (e.g. "exon") for gene ENSMUSG00000033793
from file `Mus_musculus.GRCm38.75_chr1.gtf`.
[1 line = 1 feature, the feature name is in the 3rd column]
<!--
grep "ENSMUSG00000033793" Mus_musculus.GRCm38.75_chr1.gtf | cut -f3 | sort | uniq -c
-->

exercise: extract all the combinations of distinct feature names
(like "gene" or "exon") and strands (+ or -), with their counts (sorted).
[the strand in the 7th column]
<!--
grep -v "^#" Mus_musculus.GRCm38.75_chr1.gtf | cut -f3,7 | sort | uniq -c | sort -rn
-->

### column

`column` formats tabular data to visualize in the terminal

```shell
cd ../../bds-files/chapter-07-unix-data-tools
head Mus_musculus.GRCm38.75_chr1.gtf # format for genomic features. 1 line = 1 feature, e.g. 1 gene. start & end positions are 1-based
grep -v "^#" Mus_musculus.GRCm38.75_chr1.gtf | cut -f1-8 | head
grep -v "^#" Mus_musculus.GRCm38.75_chr1.gtf | cut -f 1-8 | column -t | head
column -s"," -t Mus_musculus.GRCm38.75_chr1_bed.csv | head
```

`-s","` sets the separator to a comma instead of a tab (default)

### basename & dirname

`basename` and `dirname` extract the file/folder name and its path
from a string (the file/folder need not exist).  
`-s suffix`: to removed known suffix (like an extension)

```shell
pwd
basename $(pwd)
dirname $(pwd)
basename "relative/path/to/myfile.txt"
dirname  "relative/path/to/myfile.txt"
basename "/absolute/path/to/myfolder"
dirname  "/absolute/path/to/myfolder"
dirname "myfile.txt" # current directory: .
basename -s "txt" "relative/path/to/myfile.txt"
basename -s "txt" "relative/path/to/myfile.txt"
basename -s ".txt" "relative/path/to/myfile.txt"
basename -s "le.txt" "relative/path/to/myfile.txt"
```

### tree

freebie: `tree directory_name` shows the content of a folder
in a tree-like format (I do love trees).

to get `tree` on macOS: `brew install tree`

---

## stream editor: sed

edits the file without having to load it all in memory!
most important use: to substitute things

```shell
sed s/pattern/replacement/ filename > newfile # do NOT redirect to input file!
sed -i s/pattern/replacement/ filename # for in-place replacement
```

`s///` to replace first occurrence of a match,
`s///g` to replace globally (all instances),  
`s///i` and `s///gi` for case-insensitive search  
option `-E` for Extended (not basic) regular expressions

`-n` option to *not* print every line  
p flag to print: `s///p` print if there is a match

*warning*: unlike `grep`, `sed` does *not* recognize "enhanced" (Perl-like)
expressions like `\d` (digit), `\s` (space) or `\w` (word character).
use classes instead: like `[0-9]` or `[a-zA-Z_]`.

```shell
cat chroms.txt # "chrom1" in first column
sed 's/chrom/chr/' chroms.txt
```

we can capture and re-use a match: with `()` to capture a pattern
and `\i` to print the ith match.

example: transform a file with lines of the form
"chromosomename:startposition-endposition" to a tabular table "chromosomename startposition endposition" (3 columns separated with tabs)

```shell
echo "chr12:74-431" | gsed -E 's/^(chr[^:]+):([0-9]+)-([0-9]+)/\1\t\2\t\3/'
```

(We have to use GNU sed, `gsed` on my machine, to have `\t` be understood as a tab,
see [grep section](notes0922.html#GNU-vs-BSD-command-line-tools) to install it.)

Other ways to do the same thing below:

```shell
echo "chr12:74-431" | gsed 's/[:-]/\t/g' # why g flag?
echo "chr12:74-431" | gsed 's/:/\t/' | gsed 's/-/\t/'
echo "chr12:74-431" | gsed -e 's/:/\t/' -e 's/-/\t/'
echo "chr12:74-431" | tr ':-' '\t' # tr = translate
```

The first way is more specific, so probably more robust.
Always check for weird errors.

exercise: extract the unique transcript names in the data file
`Mus_musculus.GRCm38.75_chr1.gtf`: string after "transcript_id".

```shell
less Mus_musculus.GRCm38.75_chr1.gtf # type /transcript_id to search and highlight instances
```

option 1: search for (and output) "transcript_id" and the following string,
extract this string, remove the quotes, remove duplicates.
recall `uniq` removes consecutive duplicates.

option 2: search for lines not starting by #, then replace 'transcript_id "anything but double quote, captured"' by what was captured (inside the double quotes).

```shell
grep -E -o 'transcript_id "\w+"' Mus_musculus.GRCm38.75_chr1.gtf |
  cut -f2 -d" " | sed 's/"//g' | sort | uniq | head # or redirect: > newFileName.txt
grep -v "^#" Mus_musculus.GRCm38.75_chr1.gtf |
  sed -E 's/.*transcript_id "([^"]+)".*/\1/' | head # wait: doesn't work: extract lines!
grep -v "^#" Mus_musculus.GRCm38.75_chr1.gtf |
  sed -E -n 's/.*transcript_id "([^"]+)".*/\1/p' | sort | uniq | head
```

`sed` can do much more. just one example, to print lines 20 to 23:
`sed -n '20,23p' Mus_musculus.GRCm38.75_chr1.gtf | cut -f1-5 | column -t`

greedy matching: problem if we try to capture between quotes like this
`"(.+)"`

```shell
grep -v "^#" Mus_musculus.GRCm38.75_chr1.gtf |
  sed -E -n 's/.*transcript_id "(.+)".*/\1/p' | head -n 1
grep "^[^#].*transcript_id" Mus_musculus.GRCm38.75_chr1.gtf | head -n 1
echo 'before transcript_id "E0160944"; gene_name "Gm16088" after' > greedy_example.txt # simpler example
cat greedy_example.txt
sed -E 's/.*transcript_id "(.+)".*/\1/' greedy_example.txt
sed -E 's/.*transcript_id "([^"]+)".*/\1/' greedy_example.txt
sed -E 's/transcript_id ".*([^"]+)".*/\1/' greedy_example.txt # what happened here? explain
```


---
[previous](notes0929.html) &
[next](notes1006.html)
