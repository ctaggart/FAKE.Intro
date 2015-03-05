- title : FAKE - F# Make
- description : Beautiful builds with F#
- author : Steffen Forkmann
- theme : black
- transition : default

***

## FAKE - F# Make


![FAKE](images/logo.png)

Steffen Forkmann [@sforkmann](http://www.twitter.com/sforkmann)

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

![Over 120 commiters](images/Commiters.png)

---

### Why should I use FAKE?

![Over 100000 downloads](images/Nuget.png)

---

### Why should I use FAKE?

![Very good docs](images/Docs.png)

***

### Who uses FAKE?

* msu solutions GmbH
* Octokit by GitHub
* E.On Global Commodities UK
* Deedle by BlueMountainCapital
* Akka.net
* Most of the F# OpenSource projects like:
   * Paket
   * * FSharp.Compiler.Service    
   * FSharp.Data
   * FSCheck      
   * ...   

***

### Getting Started

* Tutorial avaliable at [http://fsharp.github.io/FAKE/](http://fsharp.github.io/FAKE/gettingstarted.html)
* Download via NuGet



    [lang=batch]
    @echo off
    cls
    .nuget\NuGet.exe Install FAKE -OutputDirectory packages -ExcludeVersion
    packages\FAKE\tools\Fake.exe build.fsx
    pause

or

    [lang=batch]
    @echo off
    cls
    .paket\paket.exe restore
    packages\FAKE\tools\Fake.exe build.fsx
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

---

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

---

#### Compiling test projects

    let testDir = "./test/"
    
    Target "Clean" (fun _ ->
        CleanDirs [buildDir; testDir]
    )
    
    Target "BuildTest" (fun _ ->
        !! "src/test/**/*.csproj"
          |> MSBuildDebug testDir "Build"
          |> Log "TestBuild-Output: "
    )
    
    "Clean"
      ==> "BuildApp"
      ==> "BuildTest"
      ==> "Default"