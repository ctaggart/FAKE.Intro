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
- Follow @fsharpMake