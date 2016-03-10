# Dependences required for Text::Pair #

## Standard Modules ##
Text::Pair relies on some standard Perl modules, such as:
  * Tie::File;
  * IO::Socket;
  * Fcntl;

You probably already have these. If Perl complains, install them with CPAN.

## Bloom::Faster ##
### CPAN Install ###
Great news! Palvaro has updated Bloom::Faster and the new version, 1.6, seems to install without a hitch using CPAN (tested on OS X Leopard Server). Just type into CPAN:

`install Bloom::Faster`

and you should end up with version 1.6 installed. Thanks a bundle, Palvaro.

The new version has a `to_file` method that saves the filter to disk, which is something we should take advantage of with a new version of Text::Pair, for much faster startup times for the server.

### Old Bloom::Faster manual install info for OS X ###
Try installing 1.6 from CPAN first before you mess with this old info.

There is one Perl module you will need that you probably don't yet have, Bloom::Faster. This is a great Bloom filter implementation that you must install. It seems to install fine on Linux with CPAN, but on OS X, it is very problematic.

http://search.cpan.org/~palvaro/Bloom-Faster-1.4/lib/Bloom/Faster.pm

#### Bloom::Faster hacking to get it to install on OS X ####
Type:
```
tar -xvf Bloom-Faster-1.4.tar
cd to Bloom-Faster-1.4
tar -zxvf leopold-1.0.tar.gz
cd into leopold-1.0/src
gcc -c -dynamic bloom.c jenkins.c suggestions.c
```

You may see:
```
jenkins.c: In function 'hash5':
jenkins.c:130: warning: incompatible implicit declaration of built-in function 'strlen'
```
It's OK. Proceed.

Type:
```
ar rcs middle.a bloom.o jenkins.o suggestions.o
libtool -o libbloom.dylib middle.a
```

Now you should have a file called libbloom.dylib. You can verify it by running:
```
otool -L libbloom.dylib
```
Which should print out:
```
Archive : libbloom.dylib
libbloom.dylib(bloom.o):
libbloom.dylib(jenkins.o):
libbloom.dylib(suggestions.o):
```

Type:
```
sudo cp libbloom.dylib /usr/local/lib
sudo cp bloom.h /usr/local/include
```

If you don't have `/usr/local`, you might try `/usr/lib` and `/usr/include`.

Now for the Perl. Type:
```
cd ../..
```
Now you ought to be in `Bloom-Faster-1.4` again.

Type:
```
perl Makefile.PL
```
See:
```
Checking if your kit is complete...
Looks good
Writing Makefile for Bloom::Faster
```

Now, edit `Makefile.shared`. It think the module is called Bloom:Leopold, so change that to Bloom::Faster in three spots, Name, Version\_From and Abstract\_From. It should look like this when you're done:
```
NAME              => 'Bloom::Faster',
    VERSION_FROM      => 'lib/Bloom/Faster.pm', # finds $VERSION
    PREREQ_PM         => {}, # e.g., Module::Name => 1.1
    ($] >= 5.005 ?     ## Add these new keywords supported since 5.005
      (ABSTRACT_FROM  => 'lib/Bloom/Faster.pm', # retrieve abstract from module
```

Type:
```
cp Makefile.shared Makefile.PL
make
```

If at this point you get something like:
```
Makefile out-of-date with respect to Makefile.PL
Cleaning current config before rebuilding Makefile...
make -f Makefile.old clean > /dev/null 2>&1
/usr/local/bin/perl Makefile.PL 
Checking if your kit is complete...
Looks good
Writing Makefile for Bloom::Faster
==> Your Makefile has been rebuilt. <==
==> Please rerun the make command. <==
false
make: *** [Makefile] Error 1
```
Then run make again:
```
make
```
and you should see:
```
cp lib/Bloom/Faster.pm blib/lib/Bloom/Faster.pm
AutoSplitting blib/lib/Bloom/Faster.pm (blib/lib/auto/Bloom/Faster)
/usr/local/bin/perl /usr/local/lib/perl5/5.10.0/ExtUtils/xsubpp -typemap /usr/local/lib/perl5/5.10.0/ExtUtils/typemap -typemap typemap Faster.xs > Faster.xsc && mv Faster.xsc Faster.c
cc -c -I. -fno-common -DPERL_DARWIN -no-cpp-precomp -fno-strict-aliasing -pipe -I/usr/local/include -I/opt/local/include -O3 -DVERSION=\"1.4\" -DXS_VERSION=\"1.4\" "-I/usr/local/lib/perl5/5.10.0/darwin-2level/CORE" Faster.c
In file included from Faster.xs:5:
ppport.h:231:1: warning: "PERL_UNUSED_DECL" redefined
In file included from Faster.xs:2:
/usr/local/lib/perl5/5.10.0/darwin-2level/CORE/perl.h:299:1: warning: this is the location of the previous definition
Running Mkbootstrap for Bloom::Faster ()
chmod 644 Faster.bs
rm -f blib/arch/auto/Bloom/Faster/Faster.bundle
LD_RUN_PATH="/usr/local/lib" env MACOSX_DEPLOYMENT_TARGET=10.3 cc -bundle -undefined dynamic_lookup -L/usr/local/lib -L/opt/local/lib Faster.o -o blib/arch/auto/Bloom/Faster/Faster.bundle \
-lbloom \

chmod 755 blib/arch/auto/Bloom/Faster/Faster.bundle
cp Faster.bs blib/arch/auto/Bloom/Faster/Faster.bs
chmod 644 blib/arch/auto/Bloom/Faster/Faster.bs
Manifying blib/man3/Bloom::Faster.3
```

Now type:
```
make test
```
and hope to see:
```
PERL_DL_NONLAZY=1 /usr/local/bin/perl "-MExtUtils::Command::MM" "-e" "test_harness(0, 'blib/lib', 'blib/arch')" t/*.t
t/Bloom....ok 
All tests successful.
Files=1, Tests=6, 2 wallclock secs ( 2.39 cusr + 0.03 csys = 2.42 CPU)
```

Now the moment you've been waiting for...
```
sudo make install
```