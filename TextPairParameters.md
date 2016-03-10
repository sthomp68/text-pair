# Introduction #

There are three different modes of use for the Text::Pair object: index, server, and query mode. This page describes the options for initializing a Text::Pair object for each of these modes.


# Parameters #

## data\_directory ##
This directory contains the shingle index, bibliography and a directory called query\_docs which contains uploaded documents sent as queries to the server. This directory must be readable and writeable by the www user.

## document\_queue ##
An reference to an array of hashes of the form ` {name => 'Gettysburg Address', file => '/path/to/gb.doc'} `. Name can be any string and file is the full path to the file to be indexed.


## shingle\_size ##
**default 3**

How many words to include in each shingle.

## stop\_words ##
Words to be ignored and not used in shingles. This is represented internally as a hash, but is set by an array reference.

## start\_tags ##
A reference to an array of tags that indicate the start of document for processing purposes. If multiple tags are present, only one is required to open a document and begin accumulating shingles. If no start tags are submitted, the entire document will be processed.

## omit\_tags ##
A reference to an array of hashes of the form ` {start => '<div1>', end => '</div1>'} `, containing opening and closing tags that describing sections of text to be skipped during indexing.

## min\_word\_length ##
An integer specifying the minimum byte length for a word. Words shorter than this length will not be indexed.

## normalize\_case ##
**default 1**

0 or 1; 1 to flatten upper case characters to lower case when building shingles.

## omit\_numerals ##
**default 1**
0 or 1; 1 to remove numerals from shingles.

## word\_pattern ##
**default `[\&A-Za-z0-9\177-\377][\&A-Za-z0-9\177-\377\';]*`**
A string, a regular expression defining a pattern that matches a word. This pattern controls word sementation.

## replacement\_patterns ##
A reference to an array of arrays regular expressions to be run on each word before shingling. Arrays look like `[[qr/\&lt/o, '']`, with the 0th element being the pattern to match and the 1st element being the replacement pattern. It can be useful to compile the patterns using qr for efficiency.

## break\_before\_patterns, break\_after\_patterns ##
A reference to an arrays of patterns that should split a word into two, with the split occurring either before or after that match. The matching characters are retained in the word and hence in the shingle. Most often used to split words on apostrophes.

## min\_bilateral\_overlap ##
**default .5**
Decimal from 0 to 1, the minimum fraction of distinct shingles from either match that must appear in both matches. For example, if set to .5, half of all the distinct shingles in the set of all shingles that appear in either match range must appear in both match ranges.

## min\_unilateral\_overlap ##
**default .5**
Decimal from 0 to 1, the minimum fraction of disinct shingles from each match range that must appear in the other match. For example, if set to .3, then 30% of the distinct shingles in each match range must appear in the other match range.

## max\_gap ##
**default 5**

Integer, the maximum number of continous shingles in a match range that do not appear in the other match range. For example, if set to 5, then any match range in a pair may contain at most 5 running shingles that do not appear in the other match range.

## min\_pair\_shingles ##
**default 2**

Integer, the minimum number of shingles that must be shared by both matches.

## check\_overmatch ##
**default 1**

0 or 1, 1 to check to see if the query document appears to be a near-duplicate of an existing document. If this is set 0, no check will be done, and duplicate query documents may take a very long time to process.

## max\_doc\_percent\_matches ##
**default .50**

Decimal 0 to 1, the fraction of shingles that can be shared between two documents before they are considered to be duplicates.

## max\_doc\_shingles\_matches ##
**default 10000**

Integer, the number of shingles that can be shared between two documents before they are considered to be duplicates.

## min\_doc\_length\_match\_test ##
**default 5000**

Integer, the minimum document shingle length for considering a document to be a duplicate or partial duplicate of another document. It may be useful to submit shorter query documents that are wholly contained within other documents in the corpus, so duplicates are not reported for smaller documents.

## num\_shingle\_buckets ##
**default 1000000**

Integer, the number of shingle buckets to be used in the shingle index. Shingles are looked up by loading into memory a range of indexed values for shingles sharing the same hash value, or "bucket".

==shingle\_server\_port
Integer, the port to run the shingle server on.

## Bloom Filter Parameters ##
### bloom\_capacity ###
**default 150000000**

Integer, the desired Bloom filter capacity. This should be set to the number of distinct shingles in the corpus.

### bloom\_error ###
**default 0.001**

Decimal, the desired error rate for the Bloom filter. Lower error rates will require a larger amount of memory for the Bloom filter.