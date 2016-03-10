# PhiloLine version 0d. #

  * To Do: Add handler for nested divs to shingler and aligner to allow access to subarticles and other objects below "div1".

  * Bug/Logic error: I think I found a few minor bugs in `mkshinglesbatch.pl`. If `$DoFlattenAccents` is true, I think it should call `&FlattenAccents` on each of the stop words. For example, I ran it on FILENAME and noticed there were a bunch of "etait"s in the shingles even though "était" is in the stop words file.

  * Bug/Logic error: Another problem that I found is that the length check for `$DeleteShortWords` should be moved further down in `&SelectAndMassageWord`. It needs to be moved after the call to !&DoVirtNorm. Otherwise, the word might be replaced with a short one and it won't get filtered.

  * Bug?: `mkshinglesbatch.pl`: In mapping bytes to div objects, if there is a DIV with NO words, which is really a tagging error, it generates an out of range sequence error. Not sure this is a bug, since complete empty divs are an encoding error.  But, the patch is to change:

```
if ($DoDivLevelObjects) {
			$thisdivid = $divid[$z];           # Do byte mapping to div ids.
			if ($offset > $divsbyte[$z]) {
				$z++;
				$thisdivid = $divid[$z];
				if ($offset > $divsbyte[$z]) {
					print "ERROR Seq $z $offset $divsbyte[$z] \n";
					exit;
					}
				}
			}
```

to

```

		if ($DoDivLevelObjects) {
                        $thisdivid = $divid[$z];          # Do byte mapping to div ids.
                        if ($offset > $divsbyte[$z]) {
                                $z++;
                                $thisdivid = $divid[$z];
                                if ($offset > $divsbyte[$z]) {
                                        $z++;
                                        $thisdivid = $divid[$z];
                                        print "WARNING Remapped counter 1 time\n";
                                        }
                                if ($offset > $divsbyte[$z]) {
                                        print "ERROR Seq $z $offset $divsbyte[$z] \n";
                                        exit;
                                        }
                                }
                        }

```

  * `makeshinglesbatch.pl`: div mapping does not include structural encoding such as `<front`, so object maps can get out of sequence.