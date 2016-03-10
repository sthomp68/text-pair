# Overview #
Text::Pair can efficiently find similar text between a query document and large, pre-indexed corpus. The search can be configured to span minor differences between matched areas to increase the flexibility of matching. The technique used to index and query the corpus draws inspiration from work in genetic sequencing as well as other text similarity approaches.

There are three steps to using Text::Pair:

  * Index: An index script runs over a corpus of documents and creates a shingle index file or all n-grams in the corpus.
  * Server: A shingle hit server loads this index, opens a port and accepts incoming shingles and returns all the hits in the corpus for that shingle.
  * Query: A query script generates shingles for new documents, sends them to the shingle hit server to find all occurrences of shared shingles in the corpus, and generates a list of all shared text sequences between the query document and the indexed corpus.

## Index Structure ##
### shingles.idx ###
In order to provide a fast search through a large corpus, Text::Pair creates an index of all "shingles" (sequences of N words) in the corpus. For a corpus many millions of words, this index is likely too large to fit into memory, so it is stored on disk in a file called `shingles.idx`. The structure of `shingles.idx` is as follows:

```
shingle_bucket	shingle	document_id:shingle_sequence:start_byte:byte_length	document_id:shingle_sequence:start_byte:byte_length	...
```

The `shingle_bucket` is a hash value computed for each shingle, and its range varies according to the `num_shingle_buckets` parameter. The more buckets you have, the less shingles in each bucket and the less you'll need to read in any given lookup from disk, and the less you'll need to search through to find a given shingle. Each `shingle_bucket` is a key to a hash, which sits in memory, whose values are the byte range for that bucket in `shingles.idx`. For best performance, you should set `num_shingle_buckets` to the highest value that will fit in memory -- 10,000,000 hash keys and values will take a little less than 1.5GBs, for example.

`shingle` is simply the multi-word sequence found in the text, with underscores replacing spaces. Shingles may skip short words that are less than `min_word_length` bytes long, words in the `stop_words` array, or words that are found between `omit_tags`.

After these two values, there are 1 - N values of the form `document_id:shingle_sequence:start_byte:offset`, which represent the locations of all occurrences of this particular shingle in the corpus. `document_id` is an integer from 0 - N based on load order. `shingle_sequence` is a 1 - N integer; its value represents the location of this shingle in the sequence of shingles in the document. `start_byte` is the starting byte offset of the shingle in the document, and `byte_length` is the length in bytes of the words covered by the shingle.

Note that `byte_length` may be considerably longer than the number of bytes in the actual shingle, due to words omitted from the shingle. Omitted words may be short words, `stop_words`, or words found withing `omit_tags`. Although these words are not considered for purposes of shingle matching, they are still retrieved when printing pairs, because the byte range for a given shingle refers to the original, unaltered document and includes everything between the first word and the last.

Here is a sample entry in `shingles.idx`:

```
99804   william_advances_meet   7512:5661:69410:24      7676:632111:8746474:24  8048:84361:1110423:24   8397:14051:172147:24
```

### buckets.idx ###
The `buckets.idx` file contains one line for each shingle bucket that is used. It is an index from the shingle bucket, which is a hash of the shingle, to the byte range in `shingles.idx` that contains the shingles occurrences for all the shingles in the bucket.

Here's a sample:

`3850    5498657 5500609`

The bucket is 3850, so all the shingles that hash to 3850 belong in this bucket. The byte range is 5498657 - 5500609, so whenever we look up a shingle that the Bloom filter says may have occurred, we will read in the byte range 5498657 - 5500609 from `shingles.idx` and find the entry for the shingle.

### docindex ###

This is a simple file with each line repesenting a document in the corpus. There are three tab-delimited fields:
> `file` - the fully-qualified location of the loaded file
> `title` - the title of the document
> `num_shingles` - the number of shingles generated from the document


## The Matching Process ##
A server script is run which loads into memory the Bloom filter and the shingle bucket for a corpus that has been indexed. A query script takes an input document, which could be a sentence or an entire novel, and shingles it using the same paramters that were used for the corpus. It then sends these shingles to the server script, which returns a list of all hits for these shingles in the corpus. The query script then investigates these hits to determine which are significant enough to be judged as matches, and prints or otherwise handles the results.

### Finding matched shingles ###
In order to minimize disk seeks, a Bloom filter (the Bloom::Faster Perl module) is used to check initially if a given shingle has been seen before. When the server receives a shingle from the query script, it first checks to see if the Bloom filter might have seen it before, i.e. if it exists in the indexed corpus. If the Bloom filter responds in the affirmative, the hash of the shingle is calculated, and the shingle hash index (`buckets.idx` is consulted to find the byte range to read from the sorted shingle index (`shinges.idx`). That range is read into memory and searched for the relevant shingle. If it is not found, then the Bloom filter has given a false positive. If it is found, the server returns the occurrences of that shingle to the query script.
### About Bloom filters ###
Bloom filters in general: http://en.wikipedia.org/wiki/Bloom_filter

Bloom::Faster perl module: http://search.cpan.org/~palvaro/Bloom-Faster-1.4/

### Processing Matched Shingles ###
After the query script gets back a list of the matches shingles from the server, each set of matched shingles must be evaluated to see whether it is part of a group of matches that are large enough and cohesive enough to form an interesting overall segent of matched text, which we'll call a "pair". Various tuning [parameters](TextPairParameters.md) determine whether or not a collection of matching shingles forms a valid pair.

Processing begins by selecting a matched shingle and seeing if another matched shingle in the results set could be part of the same pair. We examine the nearest (in shingle sequence) matched shingle, both before and after, for the query and the source document. If combining any of these four shingles with the initial shingle would result in a pair that does not violate the `maximum_gap`,`min_bilateral_overlap` or `min_unilateral_overlap` constraints, the pair is expanded to include that shingle. The process begins again with the examination of the nearest shingle to each end of the shingle range for the source and query documents.

When no further shingles can be added to the pair without violating the constraints, or all matched shingles have been consumed, the process ends. At this point the pair is evaluated to see if it meets the `min_pair_length` condition. All pairs that pass the test are returned along with the byte addresses of the original text that generated them in the query document and the corpus document. These pairs can then be printed, along with context on either side, to allow the user to examine the matches.

