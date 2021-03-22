```
  __  __                                         _   _  _____
 |  \/  |  __ _   __ _  _ __ ___    __ _        | | | ||_   _|
 | |\/| | / _` | / _` || '_ ` _ \  / _` | _____ | | | |  | |
 | |  | || (_| || (_| || | | | | || (_| ||_____|| |_| |  | |
 |_|  |_| \__,_| \__, ||_| |_| |_| \__,_|        \___/   |_|
                 |___/
                  Magma base system extension
              (aka: Magma -- the way I want it)
             Copyright (C) 2020-2021 Ulrich Thiel
             https://github/com/ulthiel/magma-ut
                  thiel@mathematik.uni-kl.de
```

## What is this?

This is a package for the computer algebra system [Magma](http://magma.maths.usyd.edu.au/magma/) that adds some generic functionality that I was missing, most importantly:

* A package manager which allows to add remote packages (using [Git](https://git-scm.com/downloads))
* A database manager which allows to work with remote databases (using [Git LFS](https://git-lfs.github.com))
* An automatic package documenter (see automatic [documentation](https://github.com/ulthiel/Magma-UT/blob/master/Packages/Magma-UT/Autodoc.md) of this package)
* An automatic package self check system
* Notifications (e.g. on cell phone) via [Pushover](https://pushover.net)
* And more, e.g. executing GAP3 commands, reading and writing of compressed files, downloading files, file handling (copy, moving, deleting, and more of files and directories), host and machine info (CPU, memory, operating system), string search and replace with regular expressions, viewing things in an external editor, printing Markdown tables, ...

Magma-UT is supposed to work under all operating systems supported by Magma, i.e. Linux, macOS, and Windows.

### Running Magma-UT

I assume you have [Magma](http://magma.maths.usyd.edu.au/magma/) installed and working (I recommend version at least 2.25). To get Magma-UT you can either:

* Download the [most recent version](https://github.com/ulthiel/Magma-UT/archive/master.zip) (simplest);
* Download the most recent release (most stable);
* Clone the Git repository using ```git clone https://github.com/ulthiel/Magma-UT.git``` (recommended).

Now, you should be able to start Magma-UT via the command ```./magma-ut``` (Linux and macOS) or ```magma.bat``` (Windows). This starts a Magma session with all the extensions from Magma-UT attached. 

The startup script tries to find the Magma executable automatically. It's easiest if you have the Magma directory in your system PATH variable. If there are any issues, you can set the Magma directory (and other options) in the config file "Config/Config.txt". Usually, this should not be necessary.

For the remote package and database functionality I assume you have [Git](https://git-scm.com/downloads) and the [Git LFS extension](https://git-lfs.github.com) installed. This is both very easy to set up under all operating systems, see [here](https://stackoverflow.com/questions/48734119/git-lfs-is-not-a-git-command-unclear).

### Package manager

By *package* I mean a coherent set of Magma source files implementing [intrinsics](http://magma.maths.usyd.edu.au/magma/handbook/functions_procedures_and_packages). Magma-UT provides a convenient package manager which allows you to add, create, and manage not just local but also remote packages. Here's an example:

```
> AddPackage("https://github.com/ulthiel/Magma-UT-Test-Pkg");
> AttachPackage("Magma-UT-Test-Pkg");
> MAGMA_UT_TEST_INTRINSIC();
WORKS
```

In the above example, the test package from [https://github.com/ulthiel/Magma-UT-Test-Pkg](https://github.com/ulthiel/Magma-UT-Test-Pkg) is cloned automatically into the local "Packages" directory. You can then attach the package and have all functions available. You can make a package to be attached at startup via

```
> AddStartupPackage("Magma-UT-Test-Pkg");
```

You can create an empty package via

```
> CreatePackage("Test-Package");
```

This creates a subdirectory "Test-Package" in the directory "Packages". If you have Git installed, this is automatically put under version control. You can now add your package source files to this directory and then you have to add all the files to the [Spec file](http://magma.maths.usyd.edu.au/magma/handbook/text/24#181) "Test-Package.s.m". 

### Database manager

By *database* I mean a collection of files from which one can obtain a Magma object (we'll go into this in a bit). Magma-UT provides a convenient database manager which allows you to add, create, and manage not just local but also remote databases. Here's an example:

```
> AddDatabase("https://github.com/ulthiel/Magma-UT-Test-DB");
> F4,dbrec := GetFromDatabase(["Magma-UT-Test-DB", "Objects", "F4"]);
> Order(F4);
1152;
> dbrec`Description;
Weyl group of type F4.
```

In the above example, the test database from [https://github.com/ulthiel/Magma-UT-Test-DB](https://github.com/ulthiel/Magma-UT-Test-DB) is cloned automatically in the local "Databases" directory. We then retrieved a particular object (the Weyl group F4 in as a matrix group) from the database. Before I go into the details, I want to point out the main idea behind remote databases in Magma-UT. You may have noticed that the ```GetFromDatabase``` call took about 1 or 2 seconds. Call it again and it will return the object instantaneously! What happened? The database is managed via Git LFS. If you've never heard about Git LFS, here's a description:

> Git Large File Storage (LFS) replaces large files such as audio samples, videos, datasets, and graphics with text pointers inside Git, while storing the file contents on a remote server like GitHub.com or GitHub Enterprise.

So, the whole database is stored in a remote location. The ```AddDatabase``` function only retrieves the *pointers* to the data files. These are *small* text files giving the URL to the remote (and potentially *large*) file. When we first called  ```GetFromDatabase``` on the particular object F4 it was detected that the object itself was not yet downloaded, only its pointer. Hence, the remote file is downloaded to the local repository. And from now on any future retrieval of F4 will be immediate because the object exists locally. This *on demand* design allows to share huge databases without forcing users to download the whole databases. Moreover, databases can be updated conveniently.

Let's now describe how objects are stored in the database. Suppose an incredibly complicated computation yields the sequence 1,1,2,3,5,8 as a result and you want to save this for later use. You can put the following into a text file "fib.txt":

```
X:=[1,1,2,3,5,8];
return X
```

(Be aware of the missing semicolon in the return statement.) In Magma, you can now do:

```
> str := Read("fib.txt");
> X := eval str;
> X;
[ 1, 1, 2, 3, 5, 8 ]
```

And you get back your result. This is basically how one can set up a database of computational results. You can save arbitrarily complicated objects if you manage to write program code constructing them again. How this is done, depends on the situation and here Magma-UT can't help. For "simple" objects you can often get an encoding string via Magma level printing, e.g.

```
> W:=ShephardTodd(28); //The Weyl group F4 as a matrix group
> Sprint(W, "Magma");
MatrixGroup<4, RationalField() |
Matrix(RationalField(), 4, 4, [ -1, 0, 0, 0, 1, 1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1 ]),
Matrix(RationalField(), 4, 4, [ 1, 1, 0, 0, 0, -1, 0, 0, 0, 1, 1, 0, 0, 0, 0, 1 ]),
Matrix(RationalField(), 4, 4, [ 1, 0, 0, 0, 0, 1, 2, 0, 0, 0, -1, 0, 0, 0, 1, 1 ]),
Matrix(RationalField(), 4, 4, [ 1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1, 1, 0, 0, 0, -1 ]) >
```

But where Magma-UT can help is the management, i.e. the storing and retrieving, of such "string-objects". 

Let's create an empty database via

```
> CreateDatabase("Test-DB");
```

This creates a subdirectory "Test-DB" in the directory "Databases". Again, this is automatically put under Git version control with the appropriate Git LFS settings. 

For the database organization we need two things:

* a *key*: this is simply an array of strings, starting with the database name and ending in the object name—everything in between describes the folder structure leading to the file. 
* an *extension*: this describes how the object is written, and this is also later determined by the file extension. There are three possibilities: 
  * "o.m": this is a plain text file containing an evaluatable string giving an object (so, "o.m" stands for "Magma object");
  * "o.m.gz": this is the same as above, but is additionally compressed via gzip;
  * "smo" this saves the object as a "[serialized Magma object](http://magma.maths.usyd.edu.au/magma/handbook/text/35#356)" (this is a rather new feature in Magma).

Let's look at an example:

```
> X:=[1,1,2,3,5,8];
> SaveToDatabase(["Test-DB", "Sequences", "fib"], Sprint(X, "Magma"), "o.m");
> Y:=GetFromDatabase(["Test-DB", "Sequences", "fib"]);
> Y;
[ 1, 1, 2, 3, 5, 8 ]
```

There is now the file "fib.o.m" in the directory "Databases/Test-DB/Sequences" containing the above sequence as an evaluatable string. In this way you can manage arbitrarily many databases with whatever objects you're interested in. With the optional argument "Description" of ```SaveToDatabase``` you can also save a description of an object (this will be an additional text file with the same name as the object). 

Internally, the Git LFS management works by setting the following in the ```.gitattributes``` file:

```
*.o.m.gz filter=lfs diff=lfs merge=lfs -text
*.o.m filter=lfs diff=lfs merge=lfs -text
*.smo filter=lfs diff=lfs merge=lfs -text
```

This means all the object files are filtered out and are stored with LFS. Everything else like documentations etc. you can have under usual Git version control.   

### Automatic package documenter

In the directory "Tools/Documenter" there is the Python script "documenter.py" that will automatically create a package documentation consisting of all the intrinsics in the package together with the description given in the source code. The documentation is stored in the markdown file "Autodoc.md" of the package directory. Here's an example call:

```
python2 documenter.py -p "Magma-UT"
```

You can see the result [here](https://github.com/ulthiel/Magma-UT/blob/master/Packages/Magma-UT/Autodoc.md).

### Automatic self check system

In "Tools/Selfcheck" there is a script ```selfcheck``` (and its Windows analogue ```selfcheck.bat```) that runs an automatic self check on a package. The idea is to have a subdirectory "Selfchecks" in the directory of the package that contains Magma program files doing some tests on the package. If something is wrong, the code should raise an error. Here's an example for the self check testing my compression functions:

```
//We create a large random string, write it to a compressed file and read it
//back in.
N:=100*1000^2;
str:=RandomString(N);
assert #str eq N;

//I don't want the random string generation to go into the selfcheck
//time, so we reset it:
MAGMA_UT_SELFCHECK_TIME := Realtime();

//Now, the test
tmpfile := MakePath([GetTempDir(), Tempname("temp file _")*".gz"]);
WriteCompressed(tmpfile, str);
str2 := ReadCompressed(tmpfile);
assert str eq str2;
DeleteFile(tmpfile);
```

The point of this script is that you can run all self checks on a package automatically: 

```
./selfcheck -p Magma-UT
Compression    OK  3.820s	267MB
Databases-1    OK  1.570s	34MB
Databases-2    OK  1.120s	34MB
Date           OK  0.020s	34MB
Download       OK  0.900s	34MB
Environment    OK  0.050s	34MB
Files          OK  0.100s	34MB
Git            OK  0.520s	34MB
MD5            OK  0.020s	34MB
Messages       OK  0.000s	34MB
Packages       OK  0.560s	34MB
Pushover       OK  0.950s	34MB
RandomStrings  OK  0.120s	34MB
Sleep          OK  1.010s	34MB
Startup        OK  0.010s	34MB
Strings        OK  3.640s	44MB
SystemCall     OK  4.110s	234MB
```

Logfiles can be found in "Tools/Selfcheck/Log". The self check script also allows reporting to a server. See the comments in the script for details.

### Notifications

If you ever had computations running for several weeks, you'd appreciate if someone would tell you when they're finished. Magma-UT can do this for you. [Pushover](https://pushover.net) is a free notification service provided cell phone and desktop apps to receive notifications. Once you've set up an account you can add an application token (you can call it e.g. "Magma-UT"). You have to add your user and token to the variables "MAGMA_UT_PUSHOVER_USER" and "MAGMA_UT_PUSHOVER_TOKEN" in the config file "Config/Config.txt" (or as environment variables). Then you can send notifications via

```
Pushover("This is a test message");
```

I'm putting this at the end of program code for (presumably) long computations and thus receive a message on my cell phone when they're finished.

### GAP3 commands

When you have [GAP3](https://webusers.imj-prg.fr/~jean.michel/gap3/) installed and set the config (or environment) variable "MAGMA_UT_GAP3" to the command of GAP3 you can execute GAP3 commands from within Magma and get the (final) result as a string. Here's an example:

```
> Wstr := GAP3("W:=ComplexReflectionGroup(28); W.matgens;");
> Wstr;
[ [ [ -1, 0, 0, 0 ], [ 1, 1, 0, 0 ], [ 0, 0, 1, 0 ], [ 0, 0, 0, 1 ] ],
  [ [ 1, 1, 0, 0 ], [ 0, -1, 0, 0 ], [ 0, 1, 1, 0 ], [ 0, 0, 0, 1 ] ],
  [ [ 1, 0, 0, 0 ], [ 0, 1, 2, 0 ], [ 0, 0, -1, 0 ], [ 0, 0, 1, 1 ] ],
  [ [ 1, 0, 0, 0 ], [ 0, 1, 0, 0 ], [ 0, 0, 1, 1 ], [ 0, 0, 0, -1 ] ] ]
```

You can use Magma's ```eval``` (combined with additional code and string manipulation) to create a Magma object from the output. I could add similar wrappers for any other system as it's all just based on strings and reading/writing temporary files with program code/output.

