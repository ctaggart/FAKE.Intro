- title : FAKE - F# Make
- description : Beautiful builds with F#
- author : Steffen Forkmann
- theme : black
- transition : default

***

## FAKE - F# Make


![FAKE](images/logo.png)

Steffen Forkmann
 
[@sforkmann](http://www.twitter.com/sforkmann)

***

### Build tools for .NET

- MSBuild
- NAnt
- PSake (Powershell)
- AlbaCore (Rake)
- UpperCut
- BauBuild, Cake (Roslyn)

***
### Why should I use FAKE?

![Over 120 commiters in March](images/Commiters.png)

---

### Why should I use FAKE?

![Over 137 commiters in April](images/Commiters2.png)

---

### Why should I use FAKE?

![Over 135000 downloads in March](images/Nuget.png)

---

### Why should I use FAKE?

![Over 150000 downloads in April](images/Nuget2.png)

---

### Why should I use FAKE?

![Very good docs](images/Docs.png)

***

### Who uses FAKE?

* msu solutions GmbH
* Octokit (by GitHub)
* E.On Global Commodities UK
* Deedle (by BlueMountainCapital)
* CHECK24 Vergleichsportal GmbH
* Olo
* ...
 
---

### Who uses FAKE?

* FSharp.Compiler.Service
* FSharp.Data
* FsCheck
* VFPT
* Paket
* Akka.net
* NSubstitute
* ... 
  
***

### Getting Started

* Tutorial avaliable at [fsharp.github.io/FAKE/](http://fsharp.github.io/FAKE/gettingstarted.html)
 
---

#### Running FAKE


    @echo off
    cls
    .nuget/NuGet.exe Install FAKE -ExcludeVersion
    packages/FAKE/tools/Fake.exe build.fsx
    pause

or

    @echo off
    cls
    .paket/paket.exe restore
    packages/FAKE/tools/Fake.exe build.fsx
    pause
    
---

#### Hello world

    // include Fake lib
    #r @"packages/FAKE/tools/FakeLib.dll"
    open Fake
    
    // Default target
    Target "Default" (fun _ ->
        trace "Hello World from FAKE"
    )
    
    // start build
    RunTargetOrDefault "Default"

---

#### Hello world

![After download](images/afterdownload.png)

---

#### Cleaning up

    let buildDir = "./build/"
    
    // Targets
    Target "Clean" (fun _ ->
        CleanDir buildDir
    )
    
    // Dependencies
    "Clean"
      ==> "Default"
      
---

#### Cleaning up

    
![After Clean](images/afterclean.png)

***

#### Compiling the application

    Target "BuildApp" (fun _ ->
        !! "src/app/**/*.csproj"
          |> MSBuildRelease buildDir "Build"
          |> Log "AppBuild-Output: "
    )
    
    // Dependencies
    "Clean"
      ==> "BuildApp"
      ==> "Default"
      
---

#### Compiling the application

    
![After Compile](images/aftercompile.png)

***

#### Compiling test projects

    Target "BuildTest" (fun _ ->
        !! "src/test/**/*.csproj"
          |> MSBuildDebug testDir "Build"
          |> Log "TestBuild-Output: "
    )
    
    "Clean"
      ==> "BuildApp"
      ==> "BuildTest"
      ==> "Default"
      
---

#### Running tests
   
    Target "Test" (fun _ ->
        !! (testDir </> "NUnit.Test.*.dll")
          |> NUnit (fun p ->
              {p with
                 // override default parameters
                 DisableShadowCopy = true;
                 OutputFile = testDir </> "TestResults.xml" })
    )

    
    "Clean"
      ==> "BuildApp"
      ==> "BuildTest"  
      ==> "Test"
      ==> "Default"

---

#### Running tests

    
![After Compile](images/alltestsgreen.png)
      
---

#### Running tests (in parallel)
   
    Target "Test" (fun _ ->
        !! (testDir </> "NUnit.Test.*.dll")
          |> NUnitParallel (fun p ->
              {p with
                 DisableShadowCopy = true;
                 OutputFile = testDir </> "TestResults.xml" })
    )

    
    "Clean"
      ==> "BuildApp"
      ==> "BuildTest"  
      ==> "Test"
      ==> "Default"
      
---

#### Running tests (xUnit)
   
    Target "Test" (fun _ ->
        !! (testDir </> "xUnit.Test.*.dll")
          |> xUnit2 (fun p ->
              {p with OutputDir = testDir })
    )
    
    "Clean"
      ==> "BuildApp"
      ==> "BuildTest"  
      ==> "Test"
      ==> "Default"

***

#### Adding FxCop
   
    Target "FxCop" (fun () ->  
        !! (buildDir </> "**/*.dll") 
        ++ (buildDir </> "**/*.exe") 
        |> FxCop 
            (fun p -> 
                {p with ReportFileName = 
                  testDir </> "FXCopResults.xml" })
    )
      
***

#### Create AssemblyInfo files
   

    open Fake.AssemblyInfoFile

    CreateCSharpAssemblyInfo 
        "./src/app/Calculator/Properties/AssemblyInfo.cs"
        [Attribute.Title "Calculator Command line tool"
         Attribute.Description "Sample project for FAKE"
         Attribute.Guid "A539B42C-CB9F-4a23-8E57-AF4E7CEE5BAA"
         Attribute.Product "Calculator"
         Attribute.Version version
         Attribute.FileVersion version]
         
***
         
#### Creating NuGet packages

    
    NuGet (fun p ->
        {p with
            Authors = authors
            Project = projectName
            Description = projectDescription
            OutputPath = packagingRoot
            Summary = projectSummary
            WorkingDir = packagingDir
            Version = buildVersion
            AccessKey = myAccesskey
            Publish = true }) 
            "myProject.nuspec"
         
      
---
         
#### Creating NuGet packages

    [lang=xml]
    <?xml version="1.0" encoding="utf-8"?>
    <package xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
      <metadata xmlns="http://schemas.microsoft.com/packaging/2010/07/nuspec.xsd">    
        <id>@project@</id>
        <version>@build.number@</version>
        <authors>@authors@</authors>
        <owners>@authors@</owners>
        <summary>@summary@</summary>
        <licenseUrl>https://github.com/octokit/octokit.net/blob/master/LICENSE.txt</licenseUrl>
        <projectUrl>https://github.com/octokit/octokit.net</projectUrl>
        <iconUrl>https://github.com/octokit/octokit.net/icon.png</iconUrl>
        <requireLicenseAcceptance>false</requireLicenseAcceptance>
        <description>@description@</description>
        <releaseNotes>@releaseNotes@</releaseNotes>
        <copyright>Copyright GitHub 2013</copyright>    
        <tags>GitHub API Octokit</tags>
        @dependencies@
        @references@
      </metadata>
      @files@
    </package>
    
***
         
#### Creating NuGet packages (using Paket)
    
      Target "NuGet" (fun _ ->    
          Paket.Pack (fun p -> 
              { p with 
                  Version = release.NugetVersion
                  ReleaseNotes = toLines release.Notes })
      )
      
      Target "PublishNuGet" (fun _ ->
          Paket.Push (fun p -> 
              { p with 
                  WorkingDir = tempDir }) 
      )
      
***

### What is Paket?

- Dependency manager for all .NET and Mono projects
- Plays well with NuGet packages and [nuget.org](http://www.nuget.org)
- Allows to reference source code files from HTTP sources and GitHub

<br /><br />
<img style="border: none" src="images/paket-logo.png" alt="Paket logo" /> 

--- 

### Why another package manager?

- NuGet puts the package version in the path
- Updates require manual work (at least if you use .fsx):


    #I "packages/Deedle.1.0.1/lib/net40"
    #I "packages/Deedle.RPlugin.1.0.1/lib/net40"
    #I "packages/FSharp.Charting.0.90.6/lib/net40"
    #I "packages/FSharp.Data.2.0.9/lib/net40"
    #I "packages/MathNet.Numerics.3.0.0/lib/net40"
    #I "packages/MathNet.Numerics.FSharp.3.0.0/lib/net40"
    #I "packages/RProvider.1.0.13/lib/net40"
    #I "packages/R.NET.Community.1.5.15/lib/net40"
    #I "packages/R.NET.Community.FSharp.0.1.8/lib/net40"
    
--

### Paket - Project Principles

- Integrate well into the existing NuGet ecosystem
- Make things work with minimal tooling (plain text files)
- Make it work on all platforms
- Automate everything
- Create a nice community

***

### Paket file structure

- `paket.dependencies`: Global definition
- `paket.lock`: Concrete versions for all dependencies
- `paket.references`: Dependency definition per project


<br /><br />
<img style="border: none" src="images/structure.png" alt="Basic structure" /> 

---

### paket.dependencies

- Specifies all direct dependencies
- Manually editable (or via paket.exe commands)


     source https://nuget.org/api/v2
           
     nuget Newtonsoft.Json       // any version
     nuget UnionArgParser >= 0.7 // x >= 0.7
     nuget log4net ~> 1.2        // 1.2 <= x < 2     
     nuget NUnit prerelease      // any version incl. prereleases
     
---

### paket.lock

- Graph of used versions for all dependencies


    NUGET
      remote: https://nuget.org/api/v2
      specs:
        log4net (1.2.10)
        Microsoft.Bcl (1.1.9)
          Microsoft.Bcl.Build (>= 1.0.14)
        Microsoft.Bcl.Async (1.0.168) - >= net40 < net45
          Microsoft.Bcl (>= 1.1.8)
        Microsoft.Bcl.Build (1.0.21)
        Newtonsoft.Json (6.0.8)
        NUnit (3.0.0-alpha-4)
          Microsoft.Bcl.Async (>= 1.0.165) - >= net40 < net45
        UnionArgParser (0.8.2)

---

### paket.references

- Specifies which dependencies are used in a project
- Compareable to `packages.config`
- Only direct dependencies need to be listed


    Newtonsoft.Json
    UnionArgParser
    NUnit


***

### Installing packages


    $ paket install

- Computes `paket.lock` based on `paket.dependencies`
- Restores all direct and transitive dependencies
- Processes all projects and adds references to the libraries

***

### Checking for updates


    $ paket outdated

- Lists all dependencies that have newer versions available:

<br /><br />
<img style="border: none" src="images/paket-outdated.png" alt="Paket outdated" /> 

***

### Updating packages


    $ paket update

- Recomputes `paket.lock` based on `paket.dependencies`
- Updates all versions to the latest matching all restrictions 
- Runs `paket install`

***

### Restoring packages


    $ paket restore

- Restores all direct and indirect dependencies
- Will not change `paket.lock` file
- Can be used for CI build or from inside Visual Studio

***

### Convert from NuGet


    $ paket convert-from-nuget

- Finds all `packages.config` files
  - Converts them to `paket.references` files
  - Generates `paket.dependencies` file
  - Computes `paket.lock` file
- Visual Studio package restore process will be converted
- Runs `paket install`

***

### Simplify dependencies


    $ paket simplify

- Computes transitive dependencies from `paket.lock` file  
  - Removes these from `paket.dependencies`
  - Removes these `paket.references`
- Especially useful after conversion from NuGet ([Sample](http://fsprojects.github.io/Paket/paket-simplify.html#Sample))

***

### Bootstrapping

- Don't commit `paket.exe` to your repository
- Bootstrapper is available for [download](https://github.com/fsprojects/Paket/releases/latest)
- Bootstrapper allows to download latest `paket.exe`
- Can be used for CI build or from inside Visual Studio

***

### Source code dependencies

- Allow to reference plain source code files
- Available for: 
  - [GitHub](https://www.github.com)
  - [GitHub gists](https://gist.github.com/)
  - HTTP resources
  
***

### Source code dependencies
#### GitHub sample (1)

- Add dependency to the `paket.dependencies` file 


    github forki/FsUnit FsUnit.fs
    
- Also add a file reference to a `paket.references` file 


    File:FsUnit.fs

***

### Source code dependencies 
#### GitHub sample (2)

- `paket install` will add a new section to `paket.lock`:


    GITHUB
      remote: forki/FsUnit
      specs:
        FsUnit.fs (7623fc13439f0e60bd05c1ed3b5f6dcb937fe468)

- `paket install` will also add a reference to the project:

<br /><br />
<img style="border: none" src="images/github_ref_default_link.png" alt="Source reference" />

***

### Source code dependencies 
#### Use case - "Type Provider definition"

- For F# Type Providers you need a couple of helper files
- It was painful to keep these up-to-date
- Reference F# Type Provider files in `paket.dependencies`:


    github fsprojects/FSharp.TypeProviders.StarterPack src/ProvidedTypes.fsi
    github fsprojects/FSharp.TypeProviders.StarterPack src/ProvidedTypes.fs
    github fsprojects/FSharp.TypeProviders.StarterPack src/DebugProvidedTypes.fs

- Add the files to the Type Providers's `paket.references`:


    File:ProvidedTypes.fsi
    File:ProvidedTypes.fs
    File:DebugProvidedTypes.fs 
       


***

### ProjectScaffold

- Used to initialialize a prototypical .NET/mono solution
- Fully featured Paket + FAKE build process 
- http://fsprojects.github.io/ProjectScaffold/

<br /> <br />
<img src="images/projectscaffold-logo.png" alt="ProjectScaffold" /> 

---

### ProjectScaffold

- allows a simple one step build and release process
- works with most build servers
- compiles the application and runs all test projects
- synchronizes AssemblyInfo files prior to compilation
- generates API docs based on XML documentation
- generates documentation based on Markdown files
- generates and pushes NuGet packages

***

### Thank you

- Take a look at https://github.com/fsharp/FAKE
- We take contributions!
- Slides are MIT licensed and made using [FsReveal](http://fsprojects.github.io/FsReveal/)
- Send corrections to https://github.com/forki/FAKE.Intro
- Follow [@fsharpMake](https://twitter.com/fsharpMake)
- Follow [@PaketManager](https://twitter.com/PaketManager)