___
- **Feature Name:** Luna Build
- **Start Date:** 2018-02-03
- **Change Type:** Non-Breaking
- **RFC Dependencies:**
- **RFC PR:** 
- **Luna Issue:** 
- **Implemented:** 

# Summary
The success of a language is often inextricably tied to the way that you manage
projects and packages in it. Luna wants to have one way to do things, and so
this proposal introduces a combination package manager and build tool that 
operates as part of the Luna toolchain. This tool is Luna Build.

# Motivation
An effective system for managing projects and package downloads is central to
the success of any language. However, it is often the case that a language 
without an opinionated solution, or an officially endorsed solution ends up with
a proliferation of alternatives. This leaves users of the language segmented,
with different projects using disparate build systems (e.g. Maven and Gradle in
the Java community). This fragmentation story is often repeated, but in 
situations where the community has rallied around a single tool (e.g. Cargo for
Rust), the ecosystem turns out to be far more coherent.

Far from wanting Luna's ecosystem to suffer the same fate as Java or C++, this 
proposal introduces an upgrade to the Luna toolchain that provides an integrated
package management and build system usable directly from the `luna` 
command-line. 'Luna Build', as this RFC terms it, provides functionality for
managing projects and dependencies, creating and manipulating sandboxes, and 
installing packages and dependencies. All of this functionality is as automatic
as possible, providing our users with a lightning-fast workflow for creating
and working on Luna packages. 

This RFC's automation treads a fine-line between enabling a diverse range of 
use-cases, and easily supporting the Luna tenet of there being one correct way
to perform a task. It covers package creation and deployment, to user install
management. Without these features, it is likely that Luna will find such duties
taken over by a fractured mess of external tools, thus robbing the community of
the best possible experience. 

# Guide-Level Explanation
Luna provides comprehensive functionality for managing projects and packages 
using the integrated tool called Luna build! Today, we're going to look at a
basic example of how you can create your own project and upload it to the 
package repository.

Creating a Luna project is as easy as typing `luna new test-project`, giving us
a new folder in our current directory called `test-project` containing all we 
need to get started! If you take a look inside, your project will look something
like this:

```
project-name/
 ├─ .luna-project/
 │   ├─ config.yaml
 │   ├─ deps.yaml
 │   └─ luna-studio.yaml
 ├─ dist/ 
 │   └─ .lir/
 ├─ src/
 │   └─ Main.luna
 ├─ test/
 │   └─ Main.luna
 ├─ LICENSE
 ├─ README.ldoc
 ├─ README.md
 └─ .gitignore
```

So we just want to make a basic 'hello, world' application, so all you need to
do is head into the `src/` directory. Inside, you'll find a file called 
`Main.luna`. Now, Luna Build is pretty sophisticated, and based on the presence
of a main function in this file it can work out that we're trying to build an 
application. So all we need to do is open it up, and put in our main:

```
def main:
    print "Hello, world!"
```

We have some code, so let's build it! All it takes is running `luna build` on 
our command line, and we have an executable. You can run it via `luna run`, and
you'll see our greeting printed to `stdout`!

Now, _real_ projects are a bit more complex than the stalwart hello world, so 
let's say we want to fire off some missiles! We're very fortunate that someone
has written a library just for us `AcmeMissiles`, providing the module 
`Missiles`. Now dependency management in Luna is _far_ more automatic than
you might be used to, so just stick with me!

First, we open up our `Main.luna` again, and add an import to the top. We can
then modify our main function to fire some missiles:

```
import AcmeMissiles.Missiles

def main:
    Missiles.fire 3
```

Now, type `luna build` again. This is where the magic happens. Luna knows about
the structure of your project, and automatically detects your dependencies. As
part of this build it's going to download and install the latest version of 
AcmeMissiles, and then build your project. We can again run this with 
`luna exec`, and you'll see `Fire! Fire! Fire!` printed to `stdout`.

Now say, for example, that we've added this missile launch to our code, but we
want to be able to check that we're firing _exactly_ the right number of 
missiles. Did someone say a test suite? 

Luna Build includes robust testing support as well, and all you have to do is 
go into the `test/` directory, and add your tests to `Main.luna`. Let's open
this up and write a test!

```
def myTest :: Bool
def myTest:
    mainResult = main
    numMissiles = mainResult.split " "
    result = numMissiles.length
    result

def Main:
    testResult = myTest
    print testResult
```

Let's run `luna test` and see what we get! By default, the test suite will just
end with `System.exit`, and it is up to you to provide some output. Fortunately,
we've thought of this and we print the result of our test to `stdout`. Given 
that we see `True`, we can be happy; our code works properly!

You haven't touched your project in a while, but you've heard that there's a new
version of your dependency `AcmeMissiles` that makes it even more exciting! We
want it, but Luna doesn't automatically upgrade dependencies (getting you all
the reproducible build goodness). In this kind of situation, however, we _want_
the new version! All we have to do is type `luna update AcmeMissiles`, and 
Luna Build will automatically upgrade us to the latest version! Now if we run 
`luna run` (shorthand for `luna build; luna exec`) again, we see the more 
exciting result:

```
Fire!!! Fire!!! Fire!!!
```

Luna Build can do a lot more than this, and if you fancy learning about all the 
other cool things that it can do, take a read of the
[Luna Book](https://luna-lang.gitbooks.io/docs/), and its section on Luna Build.

# Reference-Level Explanation
This RFC proposes to extend the `luna` command-line interface with a set of 
additional commands to allow for comprehensive, automated project and package
management. 

## Dependency Management
Dependency management for Luna projects is mostly automated, with each project
storing a dependency configuration in `deps.yaml` that is automatically 
discovered from the code. This configuration includes the compiler version, and
all of the dependencies and their versions you require. 

However, in many cases, we want to be able to perform some _manual_ dependency
management, whether it be on libraries or the compiler. To do this, `luna` 
provides the following commands for use within a project:

- `update luna`: This is the same command as `update` for project dependencies
  below, and when given the argument `luna` it updates the project's Luna 
  version. 
- `update [DEPENDENCY]`: Updates all non-frozen dependencies (or the specified 
  `DEPENDENCY`) to the latest version.
- `freeze DEPENDENCY[-VERSION]`: Freezes the dependency at the current version
  found in `deps.yaml`, or to the specified version.
- `unfreeze DEPENDENCY`: Unfreezes the version bound on a dependency.
- `rollback [HASH]`: When called without a hash, it lists all dependency change
  operations performed recently, and the status of the resulting dependency 
  resolution operation. Each operation has an associated hash. When called with
  a hash, it rolls back the project's dependencies to the specified state.

This RFC suggests the use of [Z3](https://github.com/Z3Prover/z3) for dependency
resolution as it is a very well-optimised constraint solver and already has
[Haskell Bindings](https://hackage.haskell.org/package/z3).

## Project Management
Luna aims to automate the management of Luna projects as much as possible. As a
result, it provides just a few commands that allow you to perform every task you
need to when it comes to managing and publishing your projects. This RFC 
proposes extending the `luna` command with these following sub-commands:

- `init NAME`: Creates a new project in the current directory using the name 
  `NAME`. The project will be configured to use the latest Luna version, and 
  will have the structure described in [Project Structure](#project-structure).
- `build`: Builds the project. What is built for a given project and how it is
  determined can be found below. As part of this, any dependencies that are not
  installed will be downloaded and installed into the global package database 
  for you, before being linked into your project sandbox. The `luna build` 
  command can also be executed on a standalone file to produce an executable in
  the same directory.
- `run [PATH] [FLAGS]`: Builds the application if changes have been made, and 
  then executes the resulting executable. Has a flag `--no-build` that can be 
  passed if you do not want to update the executable with your changes. 
  `luna run` can also be executed on a provided file to directly interpret Luna
  code.
- `test`: This will run all the tests contained within the `test/` directory.
- `clean`: This command will remove any artefacts built from the project, 
  including documentation, cached LIR and binaries. 
- `doc`: This command will automatically traverse the source directory and build
  HTML documentation from any of the doc comments found in the sources. 
- `login [REPOSITORY]`: Logs you into the default (or specified) repository. 
  User accounts can be associated with organisations on the package repository.
- `logout [REPOSITORY]`: Logs you out of the default (or specified) repository.
- `publish [major] [minor] [nightly] [REPOSITORY]`: Publishes your project to 
  the default (or specified) repository. If neither `major` or `minor` is 
  specified, it will be published with a minor version bump. The default project
  version is `0.1`. This command will automatically ask for publish 
  confirmation.
- `retract VERSION [REPOSITORY]`: Retracts a specific version of your project
  from the default (or specified) repository. This can be used for removing 
  broken packages. Once a version has been retracted, it cannot be published
  again. 
- `options [FLAGS...]`: Sets the flags used for the project. These flags can be
  specified either as text, or as switches (`+` or `-`) with a hierarchical API
  (e.g `+Luna.Passes.Optimisation.ReorderConditionals`).

### Project Structure
Luna projects created by Luna build will all have a consistent structure. This
structure will have the following tree:

```
project-name/
 ├─ .luna-project/
 │   ├─ config.yaml
 │   ├─ deps.yaml
 │   └─ luna-studio.yaml
 ├─ dist/ 
 │   └─ .lir/
 ├─ src/
 │   └─ Main.luna
 ├─ test/
 │   └─ Main.luna
 ├─ LICENSE
 ├─ README.ldoc
 ├─ README.md
 └─ .gitignore
```

The directories and files are described as follows:

- `.luna-project/`: This is a hidden configuration directory that contains the
  automatically-generated project configuration. 
  + `config.yaml`: This configuration file is automatically generated, but can
    be edited by the user to specify additional configuration information. The
    options are found in [Project Configuration](#project-configuration) below.
  + `deps.yaml`: This file is not to be edited by the user. It contains a record
    of all of the current project dependencies and their versions. It is kept
    with the project and ensures that any user of the project gets a 
    reproducible build. It contains versions of both library and tool 
    dependencies, and handles external tools for ESA plugins.
  + `luna-studio.yaml`: This file is not for user editing and specifies the 
    project workspace configuration.
- `dist/`: This directory contains any binary artefacts produced as a result of
  building the Luna project.
  + `.lir/`: This directory is a cache of the Luna Intermediate Representation
    for the repository. LIR is versioned, and included with prebuilt packages. 
- `src/`: This is the source directory for the project. It contains a blank
  `Main.luna` that is used as the project entry point.
- `test/`: This directory is used to contain tests for the project. It contains
  a default `Main.luna` which is executed as the entry point for running 
  your tests. By default, this will return a success or fail exit code to 
  `stdin` or `stderr`.
- `LICENSE`: A license file as determined by your globally configured default. 
  If no default is set, the user will be prompted before a LICENSE is generated.
  If you attempt to publish the project without a license, an error will be 
  generated. 
- `README.ldoc` Contains the project readme in Luna documentation syntax.
- `README.md` Contains the GitHub readme in Markdown syntax.
- `.gitignore`: A default `.gitignore` set up to ignore Luna project build 
  artefacts.

### Project Configuration
Most project management systems enforce significant manual configuration on the
user, duplicating effort which need not be. Luna Build takes a different 
approach, inferring as much as possible from project structure. The following 
elements are automatically discovered for Luna Projects:

- **Project Name:** This is automatically discovered from the name of the 
  directory that contains `.luna-project/`.
- **Project Version:** This is automatically tracked by the repository as the
  project and its updates are published.
- **License:** Automatically determined from the `LICENSE` file in the project.
- **Build Type:** Luna uses the presence of a `main` function in `Main.luna` to
  determine whether the project is an executable or a library. If there is no
  main function it is built as a library, and if there is a main function, it is
  built as an executable _and_ library.
- **Exposed Modules:** This is automatically discovered from the code structure.
  All modules in project `Foo` that should be internal only should be inside a
  module named `Internal` (e.g. `Foo.Internal.Bar`). The Luna compiler 
  automatically enforces that you cannot use `Internal` modules from other 
  projects unless you pass the `luna options +Luna.Source.AllowInternalModules` 
  flag to the compiler. 
- **Source Directory and Test Directory:** Luna automatically finds sources in
  `src/` and tests in `test/`.

All of this information is cached locally to be available when a developer is 
offline. The following overrides can be placed in the `config.yaml`, and will 
override the default behaviour where relevant.

- `synopsis`: A short, one-line description of the package.
- `description`: Detailed package description supporting Luna doc-comment 
  syntax.
- `luna-version`: This field can pin the Luna version with which a project can
  be built. 
- `luna-options`: Options to pass to the Luna compiler (e.g. `-O3`). This is the
  field set by the `luna options` command above.
- `external-target`: Allows library creators to build additional targets (e.g.
  external libraries) as part of their calls to `luna build`. These are all 
  configured by command-line calls to `luna target` with subcommands. Each 
  target has the following fields:
  + `name`: The target name. This is used for the output.
  + `tool`: The external tool.
  + `output`: The output directory and name.
  + `options`: Options to pass to the external build tool.

### Dependency Management
Each Luna project is automatically created in a sandbox. When you import a 
dependency within your code, Luna Build will automatically pull down and install
the latest compatible version of that dependency into your project, listing it
in the `deps.yaml` file. 

All dependency resolution is done based on version constraints for projects in
the repository, and any information from manually frozen versions is used in 
this resolution process. 

## Global Configuration
Luna maintains a global user-configuration file in `~/.luna/config.yaml`, which
can both be edited manually and set via sub-commands of the `luna` executable.
The fields it contains are currently:

- **Package Repositories:** The available package databases. Each database has a 
  short name, URL and associated path to the identity file gained from logging
  in. Set by `luna repo SHORT URL IDENT-PATH`.
- **Default Repositories:** The short name of one of the above repository. 
  Configured by `luna repo default NAME`.
- **Default License:** The name of the default license file to be generated. Set
  via `luna license default NAME`.

## Package Management
In addition to the automated functionality for dependency management provided by
Luna Build, the tool also enables users to install and download packages 
independently of this. This additional featureset is intended for those who want
to work on existing packages, or even to install Luna tools from the central
repository. 

This proposal also extends the `luna` command with a set of sub-commands 
intended for the management of packages (both libraries and applications) built 
in Luna. These commands allow for the install, management and removal of Luna
packages. The commands are as follows:

- `install NAME[-VERSION]`: Installs the package given by name. If `VERSION` is
  specified it installs the specific version, otherwise the latest version of
  the package is installed. When executing install, Luna Build will build the 
  artefacts for the package if not downloaded from a binary repository. If the
  package provides an executable, the executable will be made accessible from
  the same directory on your `$PATH` that the `luna` command is available from. 
  If it contains a library, the library will be made available to the Luna 
  compiler. Furthermore, it will also download and install any of the 
  dependencies required by the package. 
- `remove NAME[-VERSION]`: Removes the package specified by `NAME` from the 
  package database. If no version is given it will remove all versions of the 
  package with the given name. 
- `download NAME[-VERSION]`: Downloads the package specified package but does 
  not execute any build or install steps.

If executed from within a project, these commands operate on the project's local
view of the global package database. As an example, if you `luna install foo`
within a project, it will be downloaded (and possibly built) into the global
database, and then symlinked into the local project.

### The Global Package Database
The Luna install maintains a global package database that keeps versioned copies
of all packages and tools installed onto the system by Luna Build. When a 
project sandbox requires one of these packages, it is symlinked into the 
project, thus avoiding duplication of packages in the system.

The database knows the difference between package versions, and hence can 
contain multiple versions of the same package in use by different projects on 
the system. It can even include the same package and version multiple times if
the sub-dependencies of the package differ (achieved by hashing the project 
and its dependencies recursively).

### Package Repositories
Luna will provide a centralised repository and tooling for managing and 
publishing Luna projects. This repository will use an account system to track
package ownership and authorship, and will provide prebuilt package 
distributions. 

However, Luna will also provide the tooling (the RepoManager) API to allow 
people to use the same accounts system to host their own package repository for
Luna packages (e.g. in a corporate setting). Such repositories can be source
distributions, binary distributions or a combination of the two. This additional
tooling will not be available until after the Luna 2.0 release. 

The default package repository is specified in the Luna Build configuration file
and alternative repositories can also be stored against a name and specified 
when installing a package. 

## LunaBuild API
As part of the implementation of Luna Build, a public API will be provided that
allows Luna programs to interact with the Luna Build tooling. This API will 
contain functions that perform the same behaviours as all of the above-listed
sub-commands. 

The ability to interface with the build system from Luna code itself would prove
invaluable for self-configuring packages and the like, such as those that can be
found in the python community. 

This API will not be made available until after the Luna 2.0 release. 

# Drawbacks
While a comprehensive package management and build-tool infrastructure is 
important to a language, the main drawback of implementing this proposal is the
additional complexity that it brings to the Luna tool. This complexity will 
manifest both in its size and also additional maintenance burden for Luna. 

# Rationale and Alternatives
The design proposed here provides a comprehensive tool for the management of 
Luna projects and packages. In doing so, it maintains the Luna principle of 
having one way to do things, rather than allowing for a proliferation of 
external build tools and package management solutions. 

In integrating this functionality into the `luna` command-line tool itself, we 
provide a one-stop-shop for anyone to get up and running with development in
Luna, much like `stack` can do these days for Haskell. This means that we 
dramatically decrease the effort to get on-board with Luna as a language, which
should aid adoption of the language. One of the most frustrating things about 
certain language ecosystems is the lack of integrated tooling, and so by 
rectifying that we remove a major pain-point to trying a new language in Luna. 

There is, however, one main alternative to this proposal in its entirety. That 
would be to rely on existing tools for the creation of build and package 
management infrastructure for Luna. This is, however, likely to lead to an 
imperfect solution and hence I strongly advocate implementation of this proposal
as a whole. 

# Unresolved Questions

- Should we use TOML or YAML for the configuration files?
- How much of this functionality can we inherit from existing tools (e.g. Stack
  or Cabal). It would be worth talking to Michael Snoyman about this. 
- What algorithm and system should we use for dependency resolution?
