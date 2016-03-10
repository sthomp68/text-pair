# Overview #
Installing Text::Pair is a matter of putting the Text/Pair.pm module somewhere where Perl can find it. Getting Text::Pair running involves three steps: index, server, and query. This document will provide examples of each step in the process.
# Install #
## Dependencies ##
See [TextPairDependencies](TextPairDependencies.md) for installation prerequisites. On Linux, things should be quite smooth. On OS X, there is one thorny install.
## Text::Pair ##
Download `text-pair0.9.tar.gz` from: http://code.google.com/p/text-pair/downloads/list and unzip and untar it.

Copy the Text folder inside the tarball into somewhere where your Perl looks for libraries -- `/System/Library/Perl/5.8.6`, for instance, on an OS X 10.4 machine.

Run perl, type in `use Text::Pair;`, and if you get no errors, you are set.

To follow along with the demonstration, change into the `text-pair0.9` directory that you have extracted from the tarball:
```
cd text-pair0.9
```


# Index #
## Corpus ##
Assemble the corpus of documents you wish to use. Text::Pair has been tested with corpora of several hundred million words and more than ten thousand documents.

Plain text or XML-style documents are best suited for Text::Pair. Text::Pair knows how to look for tags that open a document (`<body>`, e.g.) or specify regions of text to ignore within a document, if there are parts of your text that you do not want to index.

You can keep your documents wherever you like, so long as you don't move them. When printing results, byte ranges are read out of the documents from their location on disk.

## Data Directory ##
Create a directory to store the output of the indexing process. If you are following along with the example scripts in `text-pair0.9/scripts`, your data directory already exists and is called `text-pair0.9/data` .

## Index ##
Create an indexing Perl script that uses Text::Pair, along the lines of `text-pair0.9/scripts/index.pl`:

```
#!/usr/bin/perl -w

use strict;
use Text::Pair;

# This is the directory where indices and query documents will be stored.
my $data_dir = "data";

# Put the documents you want to index here. The name can be any string, and the file
# should be the full path.
my @docs = ( {name => 'Hamlet by Shakespeare', file => 'docs/hamlet.txt'},
	{name => 'Macbeth by Shakespeare', file => 'docs/macbeth.txt'},
	{name => 'Othello by Shakespeare', file => 'docs/othello.txt'} );
	
# Add pairs of tags that you want to omit from processing. This example will skip indexing
# anything in <head>...</head>. Multiples pairs of tags are fine.
my @omit_tags = ({start=>'<head>', end=>'</head>'});

# If you want to wait until a certain tag is encountered in a given document before
# starting to index, put the relevant tags here. In this example, the document will
# not be indexed until a <body> or <text> tag appears.
my @start_tags = ('<body>', '<text>');

# Define an array of arrays of regex patterns and replacements that should be run on
# every word before it is shingled. Using qr to precompile the patterns aids efficiency.
my @reps = ([qr/SKIPME/, ''],
	[qr/\xc3[\xa0-\xa4]/, 'a'],
	[qr/\xc3[\x80-\x85]/, 'a'],);

# Stop words are not indexed. Removing stop words can help find text that is very similar
# but contains minor variations.
my @stop_words = ('a', 'the', 'one');

# Initialize the Text::Pair object. See Google Code documentation for parameters.
my $corpus = Text::Pair->new(
	document_queue => \@docs,
	shingle_size => 3,
	min_word_length => 3,
	omit_tags => \@omit_tags,
	word_pattern => "[\&A-Za-z0-9\177-\377][\&A-Za-z0-9\177-\377\']*",
	replacement_patterns => \@reps,
	stop_words => \@stop_words,
	start_tags => \@start_tags,
	data_directory => $data_dir,
	break_after_patterns => []);

# Index the corpus. This could take a while -- 12000 novels takes about 16 hours.
$corpus->index;
```

Running a script like this will create the necessary files for finding reused text in the `/mydata` directory. A very large corpus of many hundreds of millions of words may take many hours to index.

Execute the script `text-pair0.9/scripts/index.pl` from within the `text-pair0.9` directory to create the index data in `text-pair0.9/data`.

```
perl scripts/index.pl
```

Wait until it finishes running. Your corpus, consisting of the three Gutenberg Shakespeare documents included in `text-pair0.9/docs`, is now indexed.

## Server ##
Before you can query your corpus by sending it text and asking it to find examples of parts of that text, you need to set up the hits server. The server's job is to load into memory the Bloom filter and the shingle bucket index, and use them find any occurrences of shingles that are sent its way. Because these data structures can be large and take time to build, it's better that they live in memory rather than being built each time a query is sent.

Create a server script such as this one from `text-pair0.9/scripts/index.pl`:

```
#!/usr/bin/perl -w

use strict;
use Text::Pair;

my $corpus = Text::Pair->new(data_directory => 'data', shingle_server_port => 12345);
$corpus->serve;
```

Run this script in a way that it stays active -- daemonize, nohup, or whatever you need to do. It will need to be running any time you want to accept queries.

Execute `text-pair0.9/scripts/server.pl` to start the shingle server on your machine. It will need to be running during the querying step which follows.

```
perl scripts/server.pl
```


## Query ##
```
#!/usr/bin/perl -w

use strict;
use Text::Pair;

my $file_root = '/mydata';

# Send a file path to compare to the corpus on the command line
my $filename = $ARGV[0];

# All of these settings should be the same ones you used in index.pl, unless you
# know that your query document is somehow differently structured than your
# indexed documents and you want to use different start or omit tags.
my @stop_words = ('a', 'the', 'one');
my @omit_tags = ({start=>'<head>', end=>'</head>'});
my @start_tags = ('<body>', '<text>');
my @reps = ([qr/SKIPME/, ''],
	[qr/\xc3[\xa0-\xa4]/, 'a'],
	[qr/\xc3[\x80-\x85]/, 'a'],);

my $corpus = Text::Pair->new(shingle_size => 3,
        min_word_length => 3,
        omit_tags => \@omit_tags,
		word_pattern => "[\&A-Za-z0-9\177-\377][\&A-Za-z0-9\177-\377\']*",
        replacement_patterns => \@reps,
        stop_words => \@stop_words,
        data_directory => 'data',
        min_bilateral_overlap => .3,
        min_unilateral_overlap => .3,
        max_gap => 10,
        shingle_server_port => 12345);

my $pairs = $corpus->pair({file => $filename});

print "Number of pairs: " . scalar(@$pairs) . "\n";

foreach my $pair (@$pairs) {
	my ($a_start_byte, $a_length, $b_doc_id, $b_start_byte, $b_length) = @$pair;
	my ($b_file, $b_name) = split(/\t/, $corpus->{_documents}->[$b_doc_id]);
	if (($a_start_byte == -1) && ($b_start_byte == -1)) {
		print "Document $b_name had more than " . $corpus->{max_doc_shingle_matches} . " matches with the target document.\n";
	} elsif ($a_start_byte == -1) { 
		print "The query document had more than " . (100 * $corpus->{max_doc_percent_matches}) . "% of shingles matched in $b_name.\n";
	} elsif ($b_start_byte == -1) { 
		print "Document $b_name had more than " . (100 * $corpus->{max_doc_shingle_matches}) . "% of shingles matched in the target document.\n";
	} else {
		my ($apre, $a, $apost, $bpre, $b, $bpost) = $corpus->pairContext($pair, $filename, 100);
		print "Query document ($a_start_byte - " . ($a_start_byte + $a_length) . "):\n$apre**$a**$apost\n\n";
		print "$b_name ($b_start_byte - " . ($b_start_byte + $b_length) . "):\n$bpre**$b**$bpost\n\n\n\n";
	}
}
```

This script is designed to run from the command line. It takes one parameter, the path to the query document. It shingles the query document and compares it to all corpus documents, and prints results in a very basic format. The querying proccess could also be run as a CGI, to accept text from the web and return a page of results.

Execute `text-pair0.9/scripts/query.pl`, sending it the name of the test query document as a command line parameter:

```
perl scripts/query.pl docs/test.txt
```

and you should see results like the following:

```
robespierre:/projects/text-pair/text-pair0.9 artfl$ perl scripts/query.pl docs/test.txt 
Value of <HANDLE> construct can be "0"; test with defined() at /usr/local/lib/perl5/5.10.0/darwin-2level/Text/Pair.pm line 256.
###Pairs for b_doc_id: 0
Bib cite: docs/hamlet.txt       Hamlet by Shakespeare   23885
###Pairs for b_doc_id: 1
Bib cite: docs/macbeth.txt      Macbeth by Shakespeare  14172
###Pairs for b_doc_id: 2
Bib cite: docs/othello.txt      Othello by Shakespeare  20906
Number of pairs: 3
Query document (6298 - 6396):
 XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX 


**Queen.
Let not thy mother lose her prayers, Hamlet:
I pray thee stay with us; go not to Wittenberg**.

XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX X

Hamlet by Shakespeare (14434 - 14534):
o remain
Here in the cheer and comfort of our eye,
Our chiefest courtier, cousin, and our son.

**Queen.
Let not thy mother lose her prayers, Hamlet:
I pray thee stay with us; go not to Wittenberg**.

Ham.
I shall in all my best obey you, madam.

King.
Why, 'tis a loving and a fair reply:
B



Query document (4147 - 4271):
XX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX 


  **LADY MACBETH. Thou'rt mad to say it!
    Is not thy master with him? who, were't so,
    Would have inform'd for preparation**. 

XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX 

Macbeth by Shakespeare (18511 - 18637):
      Enter a Messenger.

    What is your tidings?
  MESSENGER. The King comes here tonight.
  **LADY MACBETH. Thou'rt mad to say it!
    Is not thy master with him? who, were't so,
    Would have inform'd for preparation**. 
  MESSENGER. So please you, it is true; our Thane is coming.
    One of my fellows had the spee



Query document (2024 - 2119):
XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX 

  **IAGO.                     Call up her father,
    Rouse him, make after him, poison his delight**,

XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX X

Othello by Shakespeare (4547 - 4643):
hat I am.
  RODERIGO. What a full fortune does the thick-lips owe,
    If he can carry't thus!
  **IAGO.                     Call up her father,
    Rouse him, make after him, poison his delight**,
    Proclaim him in the streets, incense her kinsmen,
    And, though he in a fertile climate dw
```