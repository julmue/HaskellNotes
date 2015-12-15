# Haskell Platform: GHC Ecosystem

Sources:
[Cabal User Guide](https://www.haskell.org/cabal/users-guide/)

## GHC

## The Haskell Package Environment

### Kinds of packages: Cabal vs GHC vs system

The difference between Cabal, GHC and system packages:

#### Cabal packages:
Cabal packages are really source packages.
They contain haskell (and sometimes C) source code.

Cabal packages can be compiled to produce GHC packages.
They can also be translated into operating system packages.

#### GHC packages:
GHC cares only about library packages, not executables.
Library packages have to be registered with GHC for them to be available 
in GHCI or to be used when compiling other programs or packages.

The low-level tool ghc-pkg is used to register GHC packages and to get
information on what packages are currently registered.

You never need to make GHC packages manually.
When you build and install a Cabal package containing a library
then it gets registered with GHC automatically.

Operating system packages
On operating systems like Linux and Mac OS X, the system has a 
specific notion of a package and there are tools for installing and
managing packages.

It is possible to translate Cabal packages to system packages.

### Unit of distribution
The Cabal package is the unit of distribution;
Each Cabal package can be distributed on its own in source or binary form.
There are dependencies between packages, but usually an amount of 
flexibility so distributing them indipendently makes sense.

rule of thumb:
if you want to distribute it as a single unit, then it should be a single package.

### explicit dependencies
All package dependencies have to be specified explicitly and in a declarative way;
this enables tracking down dependencies and automatic dependency management.
(this contrasts the traditional ./configure approach, where configure checks whether
dependencies are installed on the system.)

sometimes optional dependencies make sense.
Cabal packages can have optional features and varying dependencies -
however the conditional dependencies are still specified in a declarative way.





## Cabal

(from: https://www.haskell.org/cabal/)

Cabal is a system for building and packaging haskell libraries and programs.

The Cabal describes:
* what a Haskell package is
* how these packages interact with the language (???)
* what a haskell implementation must do to support a package

### The 'cabal config' file

The 'cabal' command line tool can be configured by editing the file '~./cabal/config'.
The values that can be configured are listed in that file (most are commented out).
Some defaults are already set.


### cabal package (and creating packages)
(https://www.haskell.org/cabal/users-guide/developing-packages.html)

The Cabal package is the unit of distribution.
When installed, its purpose is to make available:

  * one or more Haskell Programs
  * At most one library, exposing a number of Haskell modules.

Internally, the package may consist of much more than a bunch of Haskell modules:
It may also have C source code and header files,
source code meant for preprocessing,
documentation, test cases, auxiliary tools, ...

A package is identified by a globally unique package name (name,number)
(this is not how ghc registers it however -> see abi hash)
where name is one or more alpanumeric names separated by hyphens.

A particular Version of a package is distinguished by its version number,
consisting of a sequence of one or more integers separated by dots.
This is the package id: name-version (e.g. "HUnit-1.1")

Note: packages are not part of the haskell language.

#### Steps for package creation

A cabal package consiste of :
* Haskell software, including libraries, executables and tests
* metadata about the package in a standard human and machine readable format
  (the '*.cabal' file)
* a standard interface to build the package
  (the 'Setup.hs' file)

The *.cabal file contains information about the package,
supplied by the package author.
In particular it lists the other packages a package depends on

The 'Setup.hs' is a single-module(??) Haskell-program to perform various
setup tasks.
This module should import only modules that will be present in all Haskell
implementations, including modules of the cabal library.
the content depends on the build type set in the *.cabal file and will
be trivial in most cases.

once those two files are present (and correct) the package can be build.

one purpose of Cabal is to make it easier to build a package with different
Haskell implementations. So it provides abstractions of features present
in different haskell implementations;
for example language extensions can be specified within the cabal file -
this is preferable to specifying compiler flags because it enables Cabal to 
pick the appropriate flags depending on the implementation.

If implementation specific options have to be used they can be specified
in the cabal file, too (example the field: ghc-options)

#### Package description -- the '*.cabal' file
(see https://www.haskell.org/cabal/users-guide/developing-packages.html)

The '*.cabal' file contains package metadata and build information.
Normaly it gets created by 'cabal init'.
There must be exactly one such file in the directory;
the prefix (* in *.cabal) must be the package name.

The file should contain a number of global property descriptions and several sections:

  * global properties
	describe the package as a whole (name, version, author, ...)
  * optionally a number of configuration flags;
	these can be used to enable or disable certain features of the package.
	(see configurations in https://www.haskell.org/cabal/users-guide/developing-packages.html)
  * the (optional) library section specifies the library properties and 
 	relevant build information
  * Following is an arbitrary number of executable sectiosn which describe an executable program
	and relevant build information.

Each section consists of a number or property descriptons in the form of field/value pairs:

  * Case is not significant in field names, but significant in value names.
  * Field names may be indented, but all field values in the same section must use 
	the same indentation.
  * Tabs are not allowed for indentation
  * to get a blank line in a field value, use an indented "."

The syntax of the value depends on the field. Field types include:
	* token, filename, directory
	* freeform, URL, address
	* identifier
	* compiler (GHC, ...)

(see https://www.haskell.org/cabal/users-guide/developing-packages.html for the allowed values)

##### Modules and Preprocessors

Haskell module names listed in the 'exposed-modules' and 'other-modules' fields correspond to 
Haskell source files (*.hs, *.lhs) or to inputs for various Haskell preprocessors.

(see https://www.haskell.org/cabal/users-guide/developing-packages.html for a complete list of preprocessors)

When building cabal will automatically run the appropriate preprocessors.

##### Package Properties

These fields may occure in the first top-level properties section and describe the package as a whole:

name, version, cabal-version, 
build-type {default: custom} 
	-> special cases please see documentation: if any other type Setup.hs must be exactly as written in specification
license, license-file, copyright, author, maintainer, stability, homepage, bug-reports, package-url,
synopsis, description, category, tested-with, 
data-files -> images, soundfiles, ... for special syntax see documentation
data-dir (the directory where cabal looks for the data files)
extra-doc-files, extra-tmp-files

example:

    name:                clank
    version:             0.1.0.0
    synopsis:            logics framework
    license:             GPL-3             
    author:              Julian Müller
    maintainer:          jul.mue@hotmail.de
    copyright:           Julian Müller
    category:            Math
    build-type:          Simple
    cabal-version:       >=1.10

##### cabal type 1: library

The library section should contain the following fields:

exposed-modules: identifier list
	a list of modules added by this package

exposed: boolean (default: true)
	very uncommen to have to use this field (use hierarchical module names).

reexport-modules: exportlist
	???

The library section may also contain buld information fields (see build information)

example:

    library
        hs-source-dirs: 
            src 
        ghc-options: 
            -Wall -Werror
        exposed-modules:
            Logic.Data.Formula,
        other-modules:
            Logic.Semantics.Prop.Internal
        build-depends:
            base >=4.7 && <4.8,
            containers >=0.5 && <0.6,
        default-language:
            Haskell2010

#### cabal type 2: Exectables

Executable sections (if present) describe executable programs contained in the package 
and must have an argument after the secion label, which defines the name of the executable.

The following fields are necessary:

main-is: filename (required)
	The name of the .hs or .lhs file containing the Main module.
	Note that it is the .hs file name that must be listed even if the file is generated by a preprocessor.
	the source file must be relative to one of the directories listed in hs-source-dirs.

#### cabal type 3: test suits

Test suite sections (if present) describe package test suites and must have an argument
after the section label, which defines the name of the test suite.

the test suite may be described using the following fields, 
as well as build information fields:

type: interface (required)
	the interface type and version of the testsuite.
	cabal supports two test interfaces:
	exitcode-stdio-1.0 and detailed-0.9
	Each of this types may require or disallow other fields as described below:

exitcode-stdio-1.0:
Test suites using the 'exitcode-stdio-1.0' interface are executables that indicate 
test failure with a non-zero exit code when run;
they may provide humen readable-readable log information through the standard output
and error channels.
-> this interface is provided primarily for backwards-compatibility;
	it is preferred that new test suites be written for the detailed-0.9 interface.

the exitcode-stdio-1.0 interface requires the 'main-is' field.

main-is: filename (requiered: exitcode-stdio-1.0M disallowed: detailed-0.9)
	same as executable



example of a testsuite specification:

foo.cabal:

    Name:           foo
    Version:        1.0
    License:        BSD3
    Cabal-Version:  >= 1.9.2
    Build-Type:     Simple
    
    Test-Suite test-foo
        type:       exitcode-stdio-1.0
        main-is:    test-foo.hs
        build-depends: base

    test-foo.hs:
    
    module Main where
    
    import System.Exit (exitFailure)
    
    main = do
        putStrLn "This test always fails!"
        exitFailure


detailed-0.9:
Test suites using the detailed-0.9 interface are modules exporting the symbol 'tests :: IO [Test]'
The 'Test' type is exported by the module 'Distribution.TestSuite' provided by Cabal.

The detailed-0.9 interface allows Cabal and other Test agents to inspect a test suite's results case by casem
producing detailed human- and machine-readable log files.

The detailed-0.9 interface requires the 'test-module' field.

test-module: identifier
	the module exporting the 'tests' symbol

remark: support for the detailed interface is pretty much non-existent.



Running the the suits:

Cabal can run the test suites using its built-in test runner:

    cabal configure --enable-tests
    cabal build
    cabal test

see 'cabal help test' for further options.


for more examples 
comments: "--" haskell style comments at the beginning of a line only.


#### cabal type 4: benchmarks

TODO

#### Build information

The following fields may be optionally present 
in a library of executable section (??? and not tests and benchmarks ???)
and give information of the corrsponding library or executable.

build-depends: package list

other-modules: identifier list
	A list of modules used by the component but not exposed to the user;
	for library contents these would be hidden modules of the library,
	for executables these would be auxiliary modules to be linked with 
	the file in the main-is field.

	Note: every module in the package must be listed in either the 
		main-is, exposed-modules or other-modules fields.

hs-source-dirs: directory list (default ".")
	Root directories for the module hierarchy

extensions: identifier list
	A list of Haskell extensions used by every module 
	-> for more details see the guide.

build-tools: program list
	???

buildable: boolean (default: True)
	???

ghc-options: token list
	Additional options for GHC.
	-> for additional details see guide.

ghc-prof-options:
	additional options for GHC when the package is built with profiling enabled

ghc-shared-options: token list

	Additional options for GHC when the package is built as shared library

includes: filename list
	C-header files to be included in any compilations via C.
	??? (I assume in conjunction with the foreign interfaces module)

install-includes: filename list
	???

include-dirs:
	A list of directories to search for header files.

c-sources: filename list
	A list of C source files to be compiled and linked with the haskell files

js-sources: filename list
	Javascript source files

extra-libraries: token list
	a list of extra libraries to link with	

extra-ghci-libraries: token list
	A list of extra libraries to be used instead of extra-libraries when
	the package is loaded with ghci

extra-lib-dirs: directory list
	
cc-options: token list

cpp-options: token list
	command line options for preprocessing haskell code.

ld-options: token list
	command line options passed to the linker

pkgconfig-depends: package list
	???

frameworks: token list
	??? (something for Mac)


TO BE CONTINUED...

#### Source Repositories

There are other options besides Hackage

TO BE CONTINUED

#### Downloading a package's source

Downloading a packages source with 

    cabal get [FLAGS] PACKAGES

This seems to work not only with Hackage ... (git and others too)
see documentation.


#### The 'Setup.hs' file



### the cabal archive format: foo-1.0.tar.gz

### installing a Cabal package 1: cabal install

There are two ways to install a cabal package: 
* using the command line tool ('cabal install') or 
* manually using the script 'Setup.hs'

#### the command line tool: 'cabal'

The 'cabal-install' package provides the 'cabal' command line tool;
'cabal' simplifies the process of managin Haskell software by automating:
	+ fetching
	+ configuration
	+ compilation
	+ installation
of Haskell libraries and programs.
Those packages must be prepared using Cabal and should be present at Hackage (the central repo). 

#### cabal install
   
    analogy: 
    apt-get update		  cabal update
    apt-get install PACKAGE	  cabal install PACKAGE
    apt-get update		  cabal update (depreceated)

using 'cabal' the packages get installed locally by default (where to?)
Above is the first way of installing packages.
special flags can be given to 'cabal install --prefix=... --flags=debug'
The second way:

    cabal configure --prefix=... --flags=debug
    cabal build
    [cabal haddock [--hyperlink-source]]
    cabal copy
    cabal register

Help about cabal install can be obtained by

   cabal --help
   cabal install --help
   cabal [COMMAND] --help

#### important cabal commands

get cabal version: 	'cabal --version'
update cabal:		'cabal update'
install a package:	'cabal install'						(package in the current directory)	
			'cabal install foo'    	 				(package from the Hackage server)
			'cabal install foo-1.0'					(specific version of package)
			'cabal install foo < 2'					(constrainted package version)
			'cabal install foo bar baz'				(several packages at once)	
			'cabal foo --dry-run					(show what would be installed)
			'cabal install http://example.com/foo-1.0.tar.gz'



#### cabal repl

While developing a package, it is opten useful to make its code available
inside an interpreter session.

This can be done with the 'repl' command (read-eval-print-loop').
By default 'cabal repl' loads the first component in a package.
if the package contains several named components,
the name can be given as an argument to 'repl'
The name can also optionally be prefixed with the component's type for disambiguation:

example:

    cabal repl foo
    cabal repl exe:foo
    cabal repl test:bar
    cabal repl bench:baz



#### cabal run

You can have cabal build an run your executables by using the run command:

    cabal run EXECUTABLE [-- EXECUTABLE_FLAGS]

This command will configure, build and run the executable EXECUTABLE.
If there is only one executable defined in the package the executable can be omitted.

see: 'cabal help run' for further options.

### installing a Cabal package 2: Setup install (manual install)

    analogy: 
    ./configure			runhaskell Setup configure
    make			runhaskell Setup build
    install			runhaskell Setup install

the packages get install globally by default. (where to?)
If a packages is installed globally the local packages are ignored (this means what?).
the default for cabal-install can be modified by editing the configuration file (which one? cabal.config ?)
	
build global:

    runhaskell Setup configure
    runhaskell Setup build
    runhaskell Setup install

build for user:

    runhaskell Setup configure --user
    ...

#### Help for Setup install:

    runhaskell Setup.hs --help

#### First Step: Setup configure
Prepare to build the package.
Typically this step checks that the target platform is capable of building the package,
and discovers platform-specific features that are needed during the build.

multiple options can be passed:
* Programs used for building
* Installation paths

(see https://www.haskell.org/cabal/users-guide/installing-packages.html und Setup configure)

Building test suites:

flags: --enable-tests; --disable-tests
	enables/disables the tests specified in the package description file


	--enable-coverage; --disable-coverage
	enables/disables coverage (???)


#### Steps of manual installation:

A lot can be configured during installation:

* setup configure
	+ Programs used for building
	+ Installation paths
	+ Controlling Flag Assignments
	+ Building Test Suites
	+ Miscellaneous options
* setup build
* setup haddock
* setup hscolour
* setup install
* setup copy
* setup register
* setup unregister
* setup clean
* setup test
* setup sdist

for full reference see:
(see https://www.haskell.org/cabal/users-guide/installing-packages.html und Setup configure)


### !! Attention: 'cabal install' ≠ 'Setup install'

### Storage and identification of cabalized packages

This answers the following questions:
+ where are library packages for GHC stored
+ how does GHC remember them
and other useful information.

#### Global vs User

A package can be installed either globally or as user;
one choice for one package:

* global means in system wide directories.
* user means under the home directory.

The choice is alway made during installation and affects storage, identification, and wheteher a package is ignored or not.

	how			choice				remarks
-------------------------------------------------------------------------------
comes with GHC 			global
comes with Haskell platform	global		overrideable if build from source
Setup.hs of Setup.lhs		global		
cabal install			user
Linux distro			global		don't do this

Global packages do not depend on user packages (the other direction works);
When building a package that is to be installed globally all user packages are momentarily ignored.

#### Storage

Pathname of package files are derived form:
* how the package is installed
* package name
* package version
* GHC version it is build for

Example:
package: 	'HUnit'
version: 	'1.2.2.1
build:		'GHC 6.12.3'

'prefix' depends on how the package is installed

	case 		prefix
--------------------------------------
user			$HOME/.cabal
global (Linux distro)	/usr
global (GHC)		see below
global (otherwise)	/usr/local

Then the packages files are stored in:

	file type				directory
-----------------------------------------------------------------------
library files (*hi, *.a, *.so, *.lib)		prefix/lib/HUnit-1.2.2.1/ghc-6.12.3
data files					prefix/share/HUnit-1.2.2.1
license,docs					prefix/share/doc/HUnit-1.2.2.1
executables					prefix/bin

packages that come with GHC are stored a bit differently (usr/local/...)
change /usr/local to /usr if you obtain GHC form the linux distro.

#### Identification

GHC keeps metadata to identify which packages are installed and where;
!! It does not enumerate the directories so deleting files won't take any effect !!
If the metadata does not record a package than the package is not installed - file existence is irrelevant.
If the metadata does record a package than the package is installe - file existence is irrelevant;
so removing a file won't uninstall a package.

GHC only identifies libraries as it needs libraries only 
(what counts as a library in haskell and in what formats are the libraries?)
If there is an executable-only package than GHC does not identify it.

Cabal does not keep the metadata -- it calls GHC to get and set the metadata.

Getting a summary of the metadata:

    ghc-pkg list

(What is ghc-pkg exactly???)
This lists which packages and which versions are installed;
it is actually two lists: one of those installed as global, one of user.
The location of the metadata is also listed:

    /home/jules/.ghc/x86_64-linux-7.8.3/package.conf.d

Getting a detailed record of a package:

   ghc-pkg describe

A lot of metadata is returned; of particular interest is 'depends' and 'id'.
for the cryptic number associated with 'depends' and 'id' see next section.

#### ABI Hash

Every installed package is assigned a long hexadecimal number for unique identification beyond name and version;
the triple (name,version,hash) form the id of a package: name-version-hash.

The hash gets generated by hashing the *.hi files of the exposed modules.
The *.hi files define compatibility at the binary/ABI-level and the hash reflects that.

Example:
Tow isntances of package X version 5.0 installed (one global, one user), this is their id:
    
    X-5.0-1242rq1q235532451253
    x-5.0-1209812357ß182735ß87

then the two are not interchangeably.
If another package Y depends on X, Cabal chooses one X instance only and records the full id of the coice made,
so later GHC can use the record for sanity checks.

if both instances have the same hash:

    X-5.0-1242rq1q235532451253
    X-5.0-1242rq1q235532451253

then they are highly likely interchangeable.

Question: 
What does a *.hi file contain and why can the same package X-5.0 - compiled from the same source code, by the same compiler - 
possibly lead to different *.hi files and therefore hashes?

Answer:
This has to do with optimization.
If optimization is turned off, then the same source file always creates the same *.hi and the same hash.
With optimization turned on (Cabal turns on -0 by default, many packages build with -02) 
then code gets inlined accross module boundaries.

This gives two problems:

Problem 1:
Suppose package X depends on package W;
then by transitive inlining some code of W may appear in Xs *.hi file.
Buildings the same X version against a different W version results in a different hash.
if W depends on V then even some of Vs code my appear in Xs *.hi file.
now even a different version of V might lead to a different hash.

Problem 2:
Now that code appears in *.hi files even different optimization flags lead to different hashes.


#### Alternative databases to Global and User

So far there are only two databases: global and user.
More databases are supported by adding command line options -- this is how sandboxing works; sandboxing can be done by hand!

A database can be in one of two formats:

* a text file, it can be initialized like that:
	
	echo '[]' < /database1 # or equivalent

  in general it contains a list of package metadata

* a directory, initialized by:

    mkdir /database1
    ghc-pkg --package-db=/database1 recache

   in general, it contains text files of package metadata and a binary file package.cache caching them all.
   this is the format used by 'global' and 'user' databases.

Suppose there is a extra database /database1 then the command line options to use it are:

    cabal install/configure 	--package-db=/database1 --prefix=/files
    ghc/ghci			--package-db=/database1
    ghc-pkg			--package-db=/database1
				or
				--global --user --package-db=/database1

When using an extra database it is good to use a custom prefix, otherwise Cabal puts files in $HOME/.cabal;
this is possible but not clean.

extra databases can be stacked:

    cabal install/configure 	--package-db=/database1 --package-db/database2 --prefix=/files

(this holds for the other command line tools too).
Priority goes backwards so database2 has the highest priority,
then database1 then implicitly \user than implicitly \global.
(except ghc-pkg which does not implicitly use \user and \global)

non-standard priority can be enforced:

    cabal install/configure	--package-db=clear  --package-db=global --package-db=/database1 --package-db=user

--package-db=clear and --clear-package-db both mean "clear the stack, built it afresh form subsequent options" 
so we can control the stack order explicitly.

disable 'user' database: if 'user' is a mess just clear all databases and omit it:

   cabal install/configure     --package-db=clear --package-db=global --package-db=/database1

#### Cabal install and linux distro installs
Never ever mix linux distro and cabal installs
install from cabal exclusivley!

#### Cabal install as root

   sudo cabal install

is pointless because it will install the files to the home of the superuser -- use Setup.hs to install globally;
or 

    cabal install --global --root-cmd=sudo


#### cabal install cabal-install

never 'cabal install cabal-install' because it will install libraries behind your back ...
build it in a sandbox and then install the executable and nuke the sandbox.


### cabal sandboxes

sandboxes allow to build packages in isolation by creating a private package environment for each package.

#### what are sandboxes and why are they needed?

To escape "cabal-hell": a new package with cabal install can break existing packages on the system.
The reason for this: destructive reinstalls.
Cabal doesn't support having multiple instances of the same version of a single package package installed simultaneously
(!! Multiple Versions of the same package is completly fine!!).

Example:

assume this is installed:

   foo-1.0 -> bar-1.0 -> baz-1.0

assume you want to install this:

          bar-1.0
          /     \ 
    quux-1.0 -> baz-2.0

now bar-1.0 gets recompiled against baz-2.0 (due to optimization i suppose)
now the first dependency chain is broken (because bar-1.0 has another ABI now and is incompatible with foo-1.0).

Sandboxes present a work-around: the are per project so they can be forced to be consistent.
Sandboxes are also useful when the package depends on patched or unreleased libraries.

##### usage

Initialization of a fresh sandbox in the current directory: 'cabal sandbox init'
All subsequent commands ('install' or 'build') will use the sandbox.

    cabal sandbox init			# initialize the sandbox
    cabal install --only-dependencies 	# install dependencies into the sandbox
    cabal build				# build the package inside the sandbox


adding sources
It can be useful to make source packages available for installation in the sandbox -
for example if the package depends on a patched or an unreleased version of a library.
This can be done with the 'cabal sandbox add-source' command (think of it as an local Hackage).
If an 'add-source' dependency is later modified, it is reinstalled automatically.

example:

    cabal sandbox add-source /my/patched/lib	# add a new add-source dependency
    cabal install --dependencies-only		# Install it into the sandbox
    cabal build					# build the local package
    $EDITOR /my/patched/lib/Source.hs		# Modify the dependency
    cabal build					# Modified dependency is automatically reinstalled

Normally sandbox settings (such as optimization level) are inherited form the main Cabal config file 
($HOME/.cabal/config). Settings for a sandbox can be changed by creating an local 'cabal.config' file
in the same directory of the 'cabal.sandbox.config' file (which was created by 'sandbox init').
The file has the same syntax as the main 'Cabal config' file.


deleting a sandbox
Nuking a sandbox is easy:

   cabal sandbox delete				# build in command
   rm -rf .cabal-sandbox cabal.sandbox.config	# alternative manual method


sandboxes cheat-sheet

Action						cabal-sandbox command
----------------------------------------------------------------------
Initialise a sandbox				cabal sandbox init
Delete a sandbox				cabal sandbox delete
Link a source directory from sandbox 		cabal sandbox add-source
Make a package available in the sandbox		cabal sandbox add-source --snapshot
Build the current package			cabal build
Install a package into the sandbox		cabal install PACKAGE
Any other standard cabal command		cabal COMMAND
Install dependencies of a package		cabal install --only-dependencies
Run sandbox-local ghci				cabal repl
Sandbox restricted 'ghc-pkg'			cabal sandbox hc-pkg
Path to sandbox directory			./.cabal-sandbox


### how to uninstall/remove a package

There is no 'cabal uninstall' command. 
The only posibility is to unregister packages with 'ghc-pkg':

    ghc-pkg unregister

(so what is 'ghc-pkg' then?)

To remove a package, the most important command is 'ghc-pkg' unregister to update the metadata;
deleting the files is of secondary concern.
If a package should not be removed (yet) because other packages depend on it you get informed and denied by 'ghc-pkg unregister'.
its really important not to delete files first!
You have to compute the transitive dependency closure by hand!

example:

    ghc-pkg unregister binary-search-0.0
    rm -rf $HOME/.cabal/lib/binary-search-0.0/ghc-6.12.3

    # and perhaps
    rm -rf $HOME/.cabal/lib/binary-search-0.0
    rm -rf $HOME/.cabal/share/doc/binary-search-0.0

if there are no other ghc-version.

##### Nuking the user database

If you want to erase all user packages for a clean restart:

    # delete all the metadata
    rm -rf $HOME/.ghc/arch-ghcversion
    # now the files can be deleted, too
    rm -rf $Home/.cabal


### Misc Cabal

There is a page related to some lesser known options of cabal:
http://www.vex.net/~trebla/haskell/cabal-cabal.xhtml


### Cabal questions:

* what is the purpose of the 'Setup.hs' file?
* What is the purpose of the 'cabal.config' file?
* is it possible to list the dependencies of a package
* is it possible to list what packages depend on a certain package? (incl version number)
* the possiblity to list installed packages:
	- globally
	- in a sandbox
* how can a package be uninstalled (including its dependencies)?
* is there a possibility to make ghci (or cabal repl ...) aware of sandboxes?
* where can I put my own libraries so that I can use them in other projects (some form of local hackage)?
	is it possible to auto-update those packages?
* what is the syntax specification of a *.cabal file?
	how many test targets can be created, etc ...
* how is it possible to fetch some package form hackage as a sourcefile?
* can I create "library packages"?
* how can i locally install "library packages" (Cabal package format)?
* where are the packages install to (locally/globally)?
* in what format are the packages installed?
* what is the difference between a sandbox and creating a local package database with ghc-pkg?
* file extensions: what is the content of a *.hi file and the other library file formats?
* is it possible to have multiple versions of a package installed in the same name space or 
	in different name spaces of multiple versions of a package?
* how can i statically link an executable?
* what is the purpose of the *.cabal file
* is there a syntax specification of the *.cabal file?
* can a *.deb package be easily generated via cabal?


## GHC

#### important ghc-pkg commands

list metadata of installed packages:	ghc-pkg list
check if packages are broken:		ghc-pkg check
unregister a package:			ghc-pkg unregister PACKAGE



## Hackage


## hierarchical name space


