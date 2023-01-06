Trying to regression test a change to a feed-file generation program can be tricky. Whether the format is CSV or some fashionable markup language, the ordering of the result set tends to be unimportant in such circumstances.  
When testing, we need to verify that the files produced by the new version of the program contain the same data as those produced by the old version, irrespective of the order in which the records are written.

Recently, I was rescued from my struggle with just such a problem by my colleague, Don Thomson, who imparted some (Linux) Jedi wisdom resulting in a simple yet effective solution, involving an inventive combination of Linux utilities.

What we’re going to look at here is :

-   comparing files with diff
-   using sort to give diff a hand
-   comparing sets of files in different directories using sum
-   ignoring trailer records in delimited files using grep or head

## Some Data Files

The first file we’ll use in our example is called **episodes.txt** and contains the following :

```
one
two
three
four
five
six
seven
eight
nine
```

The file we’re comparing it to is called **releases.txt** :

```
fourfivesixonetwothreeseveneightnine
```

As you can see, the files contain the same data but the first is in numeral order and the second is the result of what may be considered a “Skywalker” sort.

Predictably, diff decides that they are not identical :

```
diff episodes.txt releases.txt

1,3d0
< one
< two
< three
6a4,6
> one
> two
> three
```

Before we go any further, let’s use some switches to minimize the diff output as, for the purposes of this exercise, we just want to tell whether or not the files are the same.

```
diff -qs episodes.txt release.txt

Files episodes.txt and releases.txt differ
```

Fortunately, Don knows just the thing to help us perform a data – rather than line-by-line – comparison…

## Sorting it out

Let’s see what happens when we use the sort command on episodes.txt :

```
sort episodes.txt

eight
five
four
nine
one
seven
six
three
two
```

Interesting. It seems to have sorted the file contents ( “data”) into alphabetical order. Let’s see what it does with releases.txt :

```
sort releases.txt

eight
five
four
nine
one
seven
six
three
two
```

That’s useful. The output is identical. Now we just need to pass the sort output for each file to diff.  
We can do this using sub-shells. As we’re running in Bash, the syntax for this is :

```
diff -qs <(sort episodes.txt) <(sort releases.txt)

Files /dev/fd/63 and /dev/fd/62 are identical
```

Note that the filenames in the output are the temporary files that hold the output (stdout) from each sub-shell.

Just to prove that this solution does detect when the rows of data are different in the files, let’s introduce a “rogue” one…

```
echo 'three-and-a-bit' >>episodes.txt

diff -qs <(sort episodes.txt) <(sort releases.txt)

Files /dev/fd/63 and /dev/fd/62 differ
```

## Comparing two sets of files in different directories with a checksum

Whilst this approach works really well for comparing two files, you may find that it’s not that quick when you’re comparing a large number of large files. For example, we have a directory containing files that we generated before making any changes to our (fictitious) program :

```
mkdir baseline_text_files

ls -l baseline_text_files/*.txt

-rw-rw-r-- 1 mike mike 14 Sep 10 13:36 baseline_text_files/originals.txt
-rw-rw-r-- 1 mike mike 14 Sep 10 13:36 baseline_text_files/prequels.txt
-rw-rw-r-- 1 mike mike 17 Sep 10 13:36 baseline_text_files/sequels.txt
```

The file contents are :

```
cat originals.txt
four
five
six

cat prequels.txt
one 
two 
three

cat sequels.txt
seven 
eight
nine

```

Files generated after modifying the program are in the **new\_text\_files** directory :

```
cat originals.txt
four
six
five

cat prequels.txt
three
two
one

cat sequels.txt
eight
nine
seven
```

Don’s rather neat alternative to diffing each pair of files is to create a checksum for each file and write the output to a temporary file. We then just diff the files with the output for each directory.

There are a number of utilities you can use to do this and the complexity of the checksum algorithm used may impact the runtime for a large number of files.  
In light of this, we’ll be using sum, which seems to be the simplest and therefore (in theory) the fastest.

A quick test first :

```
sum baseline_text_files/originals.txt
45749     1
```

The first number is the checksum. The second is the file block count.

Now we’ve identified the required utilities, this script should do the job. I’ve called it data\_diff.sh and you can download it from Github should you feel inclined to do so. [The link is here](https://github.com/mikesmithers/antikyte/tree/main/file_compare_linux).

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p><p>18</p><p>19</p><p>20</p><p>21</p><p>22</p><p>23</p><p>24</p><p>25</p><p>26</p><p>27</p><p>28</p><p>29</p><p>30</p></td><td><div><p><code>#!/bin/sh</code></p><p><code>orig_dir=$1</code></p><p><code>new_dir=$2</code></p><p><code>TMPFILE1=$(mktemp)</code></p><p><code>TMPFILE2=$(mktemp)</code></p><p><code>for</code> <code>file</code> <code>in</code> <code>$orig_dir/*</code></p><p><code>do</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>sort</code> <code>$</code><code>file</code> <code>|</code><code>sum</code> <code>&gt;&gt; $TMPFILE1</code></p><p><code>done</code></p><p><code>for</code> <code>file</code> <code>in</code> <code>$new_dir/*</code></p><p><code>do</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>sort</code> <code>$</code><code>file</code><code>|</code><code>sum</code> <code>&gt;&gt;$TMPFILE2</code></p><p><code>done</code></p><p><code>diff</code> <code>-qs $TMPFILE1 $TMPFILE2</code></p><p><code>is_same=$?</code></p><p><code>if</code> <code>[ $is_same -</code><code>eq</code> <code>1 ]</code></p><p><code>then</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>echo</code> <code>'Files do not match'</code></p><p><code>else</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>echo</code> <code>'Files are identical'</code></p><p><code>fi</code></p><p><code>trap</code> <code>'rm -f $TMPFILE1 $TMPFILE2'</code> <code>exit</code></p></div></td></tr></tbody></table>

Run this and we get :

```
sh data_diff.sh baseline_text_files new_text_files

Files /tmp/tmp.OshLmwGL0J and /tmp/tmp.Uz2mUa0SSY are identical
Files are identical
```

If we introduce a difference in one of the existing files…

```
echo 'SOLO' >>new_text_files/sequels.txt

sh data_diff.sh baseline_text_files new_text_files

Files /tmp/tmp.4OGnjQls0S and /tmp/tmp.L7OUZyGUzl differ
Files do not match
```

Unsurprisingly, the script will also detect a difference if we’re missing a file…

```
touch baseline_text_files/tv_series.txt

sh data_diff.sh baseline_text_files new_text_files

Files /tmp/tmp.LsCHbhxK1D and /tmp/tmp.358UInXSJX differ
Files do not match
```

## Ignoring Trailer Records in delimited files

With text delimited files, it’s common practice to include a trailer record at the end of the file to confirm it is complete.  
This record will typically include the date (or timestamp) of when the file was created.

Such a file might look like this in the baseline\_files directory  
For example :

```
cat baseline_files/episodes.csv

HEADER|episode|title|release_year
I|The Phantom Menace|1999
II|Attack of the Clones|2002
III|Revenge of the Sith|2005
IV|A New Hope|1977
V|The Empire Strikes Back|1980
VI|Return of the Jedi|1983
VII|The Force Awakens|2015
VIII|The Last Jedi|2017
IX|The Rise of Skywalker|2019
TRAILER|9|20220903
```

The trailer in the corresponding file in the new directory includes a different date :

```
cat new_files/episodes.csv

HEADER|episode|title|release_year
I|The Phantom Menace|1999
II|Attack of the Clones|2002
III|Revenge of the Sith|2005
IV|A New Hope|1977
V|The Empire Strikes Back|1980
VI|Return of the Jedi|1983
VII|The Force Awakens|2015
VIII|The Last Jedi|2017
IX|The Rise of Skywalker|2019
TRAILER|9|20220904
```

To accurately compare the _data_ in these files, we’ll need to ignore the trailer record.

Once again, there are numerous ways to do this. We could use :

```
grep -iv trailer baseline_files/episodes.csv

HEADER|episode|title|release_year
I|The Phantom Menace|1999
II|Attack of the Clones|2002
III|Revenge of the Sith|2005
IV|A New Hope|1977
V|The Empire Strikes Back|1980
VI|Return of the Jedi|1983
VII|The Force Awakens|2015
VIII|The Last Jedi|2017
IX|The Rise of Skywalker|2019
```

…which would result in our diff looking like this :

<table><tbody><tr><td><p>1</p></td><td><div><p><code>diff</code> <code>-qs &lt;(</code><code>sort</code> <code>baseline_files</code><code>/episodes</code><code>.csv|</code><code>grep</code> <code>-iv trailer) &lt;(</code><code>sort</code> <code>new_files</code><code>/episodes</code><code>.csv| </code><code>grep</code> <code>-iv trailer)</code></p></div></td></tr></tbody></table>

Alternatively, if we know that the trailer record is always the last line of the file we can use head to output everything apart from the last line :

```
head -n -1  baseline_files/episodes.csv.

HEADER|episode|title|release_year
I|The Phantom Menace|1999
II|Attack of the Clones|2002
III|Revenge of the Sith|2005
IV|A New Hope|1977
V|The Empire Strikes Back|1980
VI|Return of the Jedi|1983
VII|The Force Awakens|2015
VIII|The Last Jedi|2017
IX|The Rise of Skywalker|2019
```

…which would entail a diff command like this :

<table><tbody><tr><td><p>1</p></td><td><div><p><code>diff -qs &lt;(head -n -1 baseline_files/episodes.csv| sort) &lt;(head -n -1 new_files/episodes.csv| sort)</code></p></div></td></tr></tbody></table>

This being Linux there are probably several more options but these should cover at least some of the more common circumstances where comparison of file by data is required.