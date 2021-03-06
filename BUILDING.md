# Building NUnit 3.0 Console and Engine

NUnit 3.0 consists of three separate layers: the Framework, the Engine and the Console Runner. This
repository contains the Engine and Console Runner. The source code is kept in the GitHub repository at https://github.com/nunit/nunit-console. Source for the framework can be found at https://github.com/nunit/nunit

Note that assemblies in one layer must not reference those in any other layer, except as follows:
 * The console runner references the nunit.engine.api assembly, but not the nunit.engine assembly.
 * Tests in any layer reference nunit.framework.
Developers should make sure not to introduce any other references.

There are two ways to build NUnit: using the solution file in an IDE or through the build script.

## Solution Build

All projects are built together using a single Visual Studio solution NUnitConsole.sln, which may be 
built with Visual Studio 2012+, SharpDevelop or MonoDevelop. The solutions all place their output in 
a common bin directory.

## Build Script

We use **Cake** (http://cakebuild.net) to build NUnit for distribution. The primary script that controls
building, running tests and packaging is build.cake. We modify build.cake when we need to add new 
targets or change the way the build is done. Normally build.cake is not invoked directly but through
build.ps1 (on Windows) or build.sh (on Linux). These two scripts are provided by the Cake project
and ensure that Cake is properly installed before trying to run the cake script. This helps the
build to work on CI servers using newly created agents to run the build and we generally run it
the same way on our own machines.

The build shell script and build.cmd script are provided as an easy way to run the above commands.
In addition to passing their arguments through to build.cake, they can supply added arguments
through the CAKE_ARGS environment variable. The rest of this document will assume use of these commands.

There is one case in which use of the CAKE_ARGS environment variable will be essential, if not necessary.
If you are running builds on a 32-bit Windows system, you must always supply the -Experimental argument
to the build. Use set CAKE_ARGS=-Experimental to ensure this is always done and avoid having to type
it out each time.

Key arguments to build.cmd / build:
 * -Target, -t <task>                 The task to run - see below.
 * -Configuration, -c [Release|Debug] The configuration to use (default is Release)
 * -Experimental, -e                  Use the experimental build of Roslyn

The build.cake script contains a large number of interdependent tasks. The most 
important top-level tasks to use are listed here:

```
 * Build               Builds everything. This is the default if no target is given.
 * Rebuild             Cleans the output directory and builds everything
 * Test                Runs all tests. Dependent on Build.
 * TestEngine          Runs all engine tests. Dependent on Build.
 * TestConsole         Runs the console tests. Dependent on Build.
 * Package             Creates all packages without building first. See Note below.
```

### Notes:
 1. By design, the Package target does not depend on Build. This is to allow re-packaging
    when necessary without changing the binaries themselves. Of course, this means that
    you have to be very careful that the build is up to date before packaging.

 2. For additional targets, refer to the build.cake script itself.
