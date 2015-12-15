# Stack

sources:
* [official docs](http://docs.haskellstack.org/en/stable/GUIDE.html)


## introduction

### What is stack?

`stack` is a cross-platform build tools for haskell.
`stack` handles the management of the toolchain:
* `GHC`
* building and regiserting libraries
* building build tool dependencies
* ...

### reproducible builds

The primary stack desing point: _reproducible builds_.
Running `stack build` today should give the same results
as running `stack build` tomorrow.

To achieve this goald stack uses _curated package sets_
called snapshots.

To build a project, stack uses a `stack.yaml` file 
in the root directory of the project as a sort of blueprint.
That file contains a refernce, called a `resolver`,
to the snapshot which your package will build be againts.

### isolation

`stack` is isolated: it will not make changes outside of 
specific stack directories.
stack-build files generally go in either the stack root directory
(default: `~/.stack`) or `./.stack-work` directories local to each project.

The stack root directory (`~/.stack`) holds:
* packages belonging to snapshots.
* any stack-installed versions of `GHC`.
Stack will not tamper with any system version of `GHC`
or interfere with paclages install by `cabal` or other 
build tools.


## Creating a new project

### command: stack new

`stack new` creates a new project:

    stack new helloworld new-template

where `helloworld` is the name of the project,
the template `new-template` is used as the project 
template.

For this stack command there is quite a bit of initial 
setup it needs to do (downloading packages, ...).

After issuing this command a new directory `helloworld` is created:

### command: stack setup

`stack setup` downloads and installs `GHC` for you.

`GHC` will be installed to your global stack directory (`~/.stack`)
so calling `ghc` on the command line won't work.
The commands to invoke the `GHC` installed by `stack` are:
* `stack ghc`
* `stack ghci`
* `stack runghc`
    
### command: stack build

`stack build` builds the project.

The executable and library gets installed in `./.stack-work/install/...`.

`build` is the most important command in stack, its engine.

_Note_: referential transparency on build level  
unning the command twice with the same options and arguments
should generally be a no-op, and should produce a reproducible
result between different runs.


### command: stack exec

`stack build` installed the libary and executable of the project in `.stack-work`.

To run the executable (run from the project root):

    stack exec helloworld-exe
    > someFunc

`stack exec` works by providings the same reproducible environment that was
used to build your project to the command that you are running.
Thus it knew where to find `helloworld-exe` even through it is hidden in
the `./stack-work` directory.

To see where the `GHC` `stack exec` uses is installed:

    stack exec which ghc
    > /home/jules/.stack/programs/x86_64-linux/ghc-7.10.2/bin/ghc

### command: stack test

The test suite can be run with `stack test`.


## Inner Workings of stack

### Files in helloworld

The layout of the project:

    helloworld/
      app/
        Main.hs
      src/
        Lib.hs
      test/
        Spec.hs
      helloworld.cabal
      LICENSE
      Setup.hs
      stack.yaml

The `app/Main.hs`, `src/Libs.hs` and `test/Spec.hs` files are all
Haskell source files that compose the actual functionality of the project.

The `LICENSE` file has no impact on the build and is only there for 
legal purposes.

The files of interest that configure the build are 
`Setup.hs`, `helloworld.cabal`, and `stack.yaml`.


#### Setup.hs

The `Setup.hs` file is a component of the `Cabal` build system which stack uses.
It's technically not needed by stack but it is considered good practice to include it.

contents:
    import Distribution.Simple
    main = defaultMain

(Q: What does that mean, what other content/commands are possible?)

#### stack.yaml

The `stack.yaml` file is for project-level settings:

    flags: {}
    packages:
    - '.'
    extra-deps: []
    resolver: lts-3.2

If you're familiar with YAML, you may recognize that the `flags` and `extra-deps`
keys have empty values (more interesing usages later).

`packages` tells stack which local packages to build.
In our simple example, we have only a single package in our project,
located in the same directory, so `'.'` suffices.

`resolver` tells `stack` how to build the package:
* which `GHC` version to use
* versions of package dependencies
* ...
The value here says to use `LTS Haskell version 3.2`,
which implies `GHC 7.10.2` (which `stack setup` installed).
There are a number of values that can be used for `resolver` 
(more later).

#### helloworld.cabal

`stack` is build on top of the `Cabal` build system.
In `Cabal` we have individual packages, each of which contains a 
single `.cabal` file (1 cabal file = 1 package).

The `.cabal` file can define one or more _components_:
* a library
* executables
* test suites
* benchmarks
It also defines additional information:
* library dependencies
* default language pragmas
* ...

(For a complete overview see the 
[official reference from haskell.org](https://www.haskell.org/cabal/users-guide/developing-packages.html).


## Dependencies, Snapshots, Resolvers 

### Adding dependencies

If package dependencies are added somewhere in the source-code of a program
the dependency has to be added to the `project.cabal` file.
Example of a dependency to the package `text`:

    library
      hs-source-dirs:
          src
      exposed-modules:
          Lib
      build-depends:
          base >= 4.7 && < 5
        -- this line is the new one
        , text
      default-language:
        Haskell2010

Package dependencies are downloaded, configured, build, and locally installed.
Once this is done the local package can be build.
At no point we explicitly have to command the building of dependencies -
stack does that automatically.


### Curated package sets (snapshots) and External packages

The resolver (example `lts-3.2`) defines the build plan and available packages
(what is the build plan?).

If a package is part of a currated package set (example: the `lts-3.2` package set)
no extra work has to be done.

If a package is not part of a currated package set it has to be added via 
the `extra-deps` field in `stack.yaml` to define extra dependencies not present 
in the resolver. Example of a dependecy to the package `acme-missles`:

    flags: {}
    packages:
    - '.'
    extra-deps:
    - acme-missles-0.3 3 not in lts-3.2
    resolver: lts-3.2

Information about the snapshots is available at `stackage`.
Example: Information about `lts-3.2`from https://www.stackage.org/lts-3.2. 

This information includes:
* The approrpiate resolve value (`lts-3.2`,...)
* The `GHC` version used
* A full list of packages available in this snapshot
* The ability to perform a Hoogle search on the packages in the snapshot
* A list of all modules in a snapshot

A list of all available snapshots is available [here](https://www.stackage.org/snapshots).
The snapshots come in two flavors: _LTS_ ("Long Term Support") and _Nightly_.

### Changing GHC versions

It is also possible to change the `GHC`-version used to build the package.
Different resolvers and packages need different `GHC`-versions.

    stack setup --resolver lts-2
  
or alternatively

    stack build --install-ghc

will install the correct `GHC`-version automatically.

### Other resolver values

There are also other resolver values: see
[yaml-documentation](http://docs.haskellstack.org/en/stable/yaml_configuration.html#resolver).



## Exitsting Projects

### command: stack unpack (Getting source code)

It is possible to download the source code of an existing project using `stack unpack`.

### command: stack init (Initializing stack in an existing project)

TODO

### command: stack solver


## Different databases

## The build synonyms

Subset of output of `stack --help`:

    build 	Build the packackages in this directory/configuration
    install 	Shortcut for 'build --copy bins'
    test 	Shortcut for 'build --test'
    bench 	Shortcut for 'build --test'
    haddock	Shortcut for 'build --haddock'

That the commands are just synonyms allows for easy compositionality.

### command: install and copy-bins

The `install` command does precisely one thing in addition to build command:
it copies any generated executables to the local bin path.

    stack path --local-bin-path
    > /home/user/.local/bin

This directory sould be added to the `PATH` environment variable.
This allows to run `executable-name` form the shell instead of having 
to run `stack exec executable-name` form inside the project directory.

Important:
* stack will always build any necessary dependencies for the code.
  The install command is not necessary to trigger this behaviour.
* stack will not track which files it's copied to the local bin path
  nor provide a way to automatically delete them.
  There are many great tools out there for managing installation 
  of binaries, and stack does not attempt to replace those.
* stack will not be necessarily be creating a relocatable executable.
  If your executable hard-codes paths, copying the executable will
  not change hard-coded paths (???).

