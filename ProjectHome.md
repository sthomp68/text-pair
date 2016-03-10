PAIR (Pairwise Alignment for Intertextual Relations) is a simple implementation of a [sequence alignment](http://en.wikipedia.org/wiki/Sequence_alignment) algorithm for humanities text analysis designed to identify "similar passages" in large collections of texts.  These may include direct quotations, plagiarism and other forms of borrowings, commonplace expressions and the like. We are developing two currently distinct streams:

**PhiloLine** (for PhiloLogic alignment) is the experimental model, written largely by Mark (in very old fashioned perl), designed to perform all-against-all comparisons in documents loaded into a PhiloLogic database. An entire corpus is indexed and compare against itself, or another database, to find text reuse.

March 2009: We have released version 0e of PhiloLine.  To get some sense of what this does and how it currently works, please consult our [ARTFL-Frantext Release Notes](http://docs.google.com/View?docID=ddj2s2rb_187hngp5xh9&revision=_latest).  We have put this into production for ARTFL and are using it for a number of other projects.  We have fairly extensive installation and use instructions in our [Release Notes](http://docs.google.com/View?docID=ddj2s2rb_189gjsnqbfb&revision=_latest).  This is known to work for English, French,
[Greek](http://cybergreek.uchicago.edu/alignmentsample.html), and other language document collections up to 11,500 documents and 520 million words.

**Text::Pair** is a generalized Perl module version of PAIR, without specific bindings to PhiloLogic, supporting one-against-many comparisons. A corpus is indexed and incoming texts are compared against the entire corpus for text reuse.   We have prepared a simple
<a href='http://robespierre.uchicago.edu/pair/pair_gutenberg.html'>demonstration</a> of Text::Pair which allows you to submit passages to be aligned against 12,000 documents from Project Gutenberg.

Both versions are available from the <a href='http://code.google.com/p/text-pair/downloads'>Downloads</a> page. See the <a href='http://code.google.com/p/text-pair/wiki'>Wiki</a> for more information and tutorials.

We are maintaining a large set of slides which we are using for various talks and presentations, currently titled "Sequence Alignment and the Discovery of Intertextual Relations" ([Google Presentation](http://docs.google.com/Present?docid=ddj2s2rb_149dmp2vvc4&skipauth=true")).  This outlines the rationale, with many examples, of this effort and gives a fairly detail overview of what works, what doesn't, and some indication of future work.  This is not a specific talk and we will be updating it to reflect ongoing work, along with subsequent releases.



Authors:  Mark Olsen (ARTFL Project) and Russell Horton (DLDC - Digital Library Development Center) at the University of Chicago with the assistance of Robert Voyer, Richard Whaling, and Clovis Gladstone and the rest of the ARTFL team.

Acknowledgements: We wish to gratefully acknowledge the support for this project provided by the [DLDC](http://dldc.lib.uchicago.edu/) and [NSIT](http://nsit.uchicago.edu/) at the [University of Chicago](http://www.uchicago.edu) and [Alexander Street Press](http://www.alexanderstreet.com/).

Some hacking may be required.  Please send your comments, complaints and code to artfl DD project ASIGN gmail DD com (you know what to do with this).



---

Philoline was the wife of Zorote in Jean Schelandre's tragicomedy _Tyr et Sidon; ou, les funestes amours de Belcar et Meliane_ first published in 1608 and revised in 1628.