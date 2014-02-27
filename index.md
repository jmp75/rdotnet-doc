rdotnet-doc
===========

# Getting set up

* The recommended way to use R.NET is to use NuGet and add the [R.NET nuget package][rdotnet_nuget] as a dependency to your project/solutions. As of release 1.5.6, R.NET and its nuget package is platform independent (Windows, Linux, MacOS).
* You can also download a zip file with the binaries from the [R.NET site on codeplex][rdotnet_codeplex]

# Related projects

* RProvider
* rClr

# Tutorials 

## Getting started

R.NET has changed substantially with version 1.6, to address several issues. We will dive into simple but statistically meaningful example and highlight line by line some of the key features.

### A Unix specific prerequisite: LD_LIBRARY_PATH

If your operating system is Linux or MacOS, you may need to modify the environment variable LD_LIBRARY_PATH before you start to use R.NET in your program. This is something that just cannot be done automagically by R.NET.

For example an R console on a Debian box may have:
```S
Sys.getenv('LD_LIBRARY_PATH')
# [1] "/usr/local/lib/R/lib:/usr/local/lib:/usr/lib/jvm/java-7-openjdk-amd64/jre/lib/amd64/server"
```
To limit the risk of unexpected difficulties with R, you should append these to LD_LIBRARY_PATH (if it is not already present)
```Sh
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib/R/lib:/usr/local/lib:/usr/lib/jvm/java-7-openjdk-amd64/jre/lib/amd64/server
```

## First sample

```C#
using System;
using System.Linq;
using RDotNet;

namespace Sample1
{
   class Program
   {
      static void Main(string[] args)
      {
         REngine.SetEnvironmentVariables();
         REngine engine = REngine.GetInstance();
         // REngine requires explicit initialization.
         // You can set some parameters.
         engine.Initialize();

         // .NET Framework array to R vector.
         NumericVector group1 = engine.CreateNumericVector(new double[] { 30.02, 29.99, 30.11, 29.97, 30.01, 29.99 });
         engine.SetSymbol("group1", group1);
         // Direct parsing from R script.
         NumericVector group2 = engine.Evaluate("group2 <- c(29.89, 29.93, 29.72, 29.98, 30.02, 29.98)").AsNumeric();

         // Test difference of mean and get the P-value.
         GenericVector testResult = engine.Evaluate("t.test(group1, group2)").AsList();
         double p = testResult["p.value"].AsNumeric().First();

         Console.WriteLine("Group1: [{0}]", string.Join(", ", group1));
         Console.WriteLine("Group2: [{0}]", string.Join(", ", group2));
         Console.WriteLine("P-value = {0:0.000}", p);

         // you should always dispose of the REngine properly.
         // After disposing of the engine, you cannot reinitialize nor reuse it
         engine.Dispose();

      }
   }
}
```

R.NET needs to have the path to the directory containing the native R shared library in the PATH environment variables (besides the Unix specific setup of LD_LIBRARY_PATH). The call to `REngine.SetEnvironmentVariables()` tries to set up the PATH and the R_HOME variables. Most Windows platform will have R related information in the Windows registry, and a bit of guesswork is done for Linux and MacOS.

If the function fails to find the paths (and R.NET tries to give meaningful error messages), you can give optional arguments. On Windows the following usually suffices `REngine.SetEnvironmentVariables(rPath:@"c:\my\custom\path\R-3.0.2\bin\i386")`. Almost always, if not always, the R_HOME variable is set by the native R engine. You can use if required: `REngine.SetEnvironmentVariables(rPath:@"c:\my\custom\path\R-3.0.2\bin\i386", rHome:@"c:\my\custom\path\R-3.0.2")` 

On Linux you may need to specify `REngine.SetEnvironmentVariables(rPath:"/usr/local/lib64/R/lib", rHome:"/usr/local/lib64/R")` especially if you have something unusual like `REngine.SetEnvironmentVariables(rPath:"/home/username/local/lib64/R/lib", rHome:"/home/username/local/lib64/R")` 

On MacOS, something like: `REngine.SetEnvironmentVariables(rPath:"/Library/Frameworks/R.framework/Libraries", rHome:"/Library/Frameworks/R.framework/Resources")`. This is actually what the arguments default to.

As of R.NET version 1.6 the use of a single instance of the R engine is enforced with access arrangements like `REngine engine = REngine.GetInstance()`

`engine.Initialize()` does what the name suggests: initialise the native R engine. A call without argument is usually enough; you do have the option to pass some custom startup parameters and character device (console). Custom startup parameters may be required in circumstances such as running R in an MPI process (Message Passing Interface), or other unusual circumstances. Due to limitations of R you must not call the `Initialize` method more than once.

* TODO Link to downlodable and executable sample

## A graphical example.

## Best practices, howtos, tips and tricks

* Passing large amounts of data: ToArrayFast (TODO: reassess API options for transparent ToArray).
* dos and donts for memory management.
* Factors
* Calling functions as, well, functions. 
** Named arguments
** Generic methods

# FAQ

* Parallel access status; summary and pointers to resources

# Known issues

ASP.NET

# Contributors

# Further reading

# Tests

* Creation of a large amnount of transient sexp: difference between safehandle and a hand crafted disposal

[rdotnet_codeplex]: https://rdotnet.codeplex.com
[rdotnet_nuget]: http://www.nuget.org/packages/R.NET

