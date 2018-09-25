---
layout: post
author: wachulski
title: "Setup SonarQube in .NET"
subtitle: "SonarQube Series #2. Guideline how to install and configure SonarQube for your .NET projects."
categories: [software]
tags: [software, SonarQube, ".NET", automation]
---

In case you haven't read previous posts in this series, you can find a short introduction to SonarQube and values it brings to the table [here][prev-intro]. 

# SonarQube server installation

## On-premises

The required pieces are: SonarQube server app and a database server. One can pick either MySQL (5.6+), PostgresSQL (8+), MS SQL (2012+) or Oracle (11g+). SonarQube server is licensed within a GNU LGPLv3 and works on top of Oracle JRE 8 or OpenJDK 8. As far as hardware requirements are concerned, it's worth to mention RAM (at least 2 GB) and efficient hard drives. They have the biggest impact on the effective throughput mainly because of the Elasticsearch indices. More information can be found in the guide on the authors’ [website][sq-doc].

Let's install a database server (MySQL 5.7 in our case). Bear in mind that at the time of this writing MariaDb is not supported yet. Next, we need to download a ZIP package from [the SonarQube server][sq-downloads]. There are two options: the latest and LTS. For the completeness, there is also [a docker image][sq-docker] that could be used by some container-oriented people.

1. Create `sonar` scheme in the database as well as `sonar` user with `sonar` password.
2. Grant full privileges of `sonar` scheme to `sonar` user.
3. Unpack the downloaded archive to a directory of choice.
4. Modify `YOUR_SONAR_PATH/conf/sonar.properties`:
    ```
    # sonar.properties file - minimum configuration needed by SonarQube
    
    # User credentials.
    sonar.jdbc.username=sonar
    sonar.jdbc.password=sonar
    
    #----- MySQL 5.6 or greater
    sonar.jdbc.url=jdbc:mysql://localhost:3306/sonar?useUnicode=true&characterEncoding=utf8&    rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false
    ```
5. Run `YOUR_SONAR_PATH/bin/windows-*/InstallNTService.bat` (Note! The processor architecture needs to match JVM architecture in use on your machine).
6. Run `YOUR_SONAR_PATH/bin/windows-*/StartNTService.bat`.
7. Go to `http://localhost:9000`.
8. Enter `admin` (password: `admin`) which should land you on the SonarQube dashboard.

## [SonarCloud.io][sq-cloud]

SonarSource AS offers also a SaaS service [SonarCloud][sq-cloud]. It is free for open source projects.

1. Go to [https://sonarcloud.io][sq-cloud].
2. Click `Log in`.
3. Choose authentication; create such if you don't have one yet.
4. You are in (still) empty projects panel.
5. Click the account icon in the top right corner and choose `My Account`.
6. Generate a token in the `Security tab` (used as credentials by analysis scripts to communicate with SonarCloud API).
7. Create a new organization in the `Organizations tab` (used by analysis scripts).
8. We have infrastructure in place, now we need to configure analysis for our .NET projects. 

# Configuring .NET project analysis automation

Jon Skeet doesn't need to be introduced to a .NET programmer. His _[C# in Depth][skeet-book]_ as well as the amount of [contribution to StackOverflow][skee-so] are world renowned. Working at Google he develops .NET client libraries for Google cloud services. Let's try to test drive the quality of his code. By the following example you will also learn how to configure .NET projects analysis in general:

1. Clone the repository: [https://github.com/GoogleCloudPlatform/google-cloud-dotnet.git](https://github.com/GoogleCloudPlatform/google-cloud-dotnet.git).
2. Ensure you have Visual Studio with C# 7 onboard.
3. Install [Sonar Scanner][choco-scan]. This will alter MSBuild building, collect build results and send them to the SonarQube server.
    ```powershell
    cinst sonarscanner-msbuild-net46
    ```
4. Install [JetBrains dotCover console tool][dotcover] for code coverage analysis.
    ```powershell
    cinst dotcover-cli
    ```
5. In the project root catalogue install [xunit runner][xunit].
    ```powershell
    nuget install xunit.runner.console -ExcludeVersion
    ```

All pieces in place. The last step is to automate analysis execution. The common approach is as follows: 

```powershell
# alters MSBuild (injects analyzers, makes sure they output to a result file)
SonarScanner.MSBuild begin ...

# builds as usual (test execution for code coverage is optional)
MSBuild ...                   
                              
# collects analysis results, sends them to the server for further processing
SonarScanner.MSBuild end
```

Since Jon Skeet’s libraries are split between many solutions, we need to run the procedure in a loop. The following automation example can be easily adapted to the vast majority of .NET projects:

```powershell
# Template analysis script that runs SonarQube analysis for .NET projects

param(
  $version = '',  # SQ records analysis history, each having its version label
  $authToken = '' # SonarCloud.io auth, generated in SQ administration
)

# retrieve all solution files that exist in the project subtree
$slns = Get-ChildItem *.sln -r

foreach ($sln in $slns) {
  $sqProj = $sln.Name -replace '\.sln$',''
  Write-Host "Analysing $sqProj"

  # initiating - passing analysis parameters to 'begin' step
  SonarScanner.MSBuild begin `
    /key:"$($sqProj -replace '\.', '_')" `
    /name:"$($sqProj -replace '\.', ' ')" `
    /version:"$version" `
    /d:$("sonar.cs.dotcover.reportsPaths=dotCover.html")
    # the following is needed for https://SonarCloud.io
    #/d:"sonar.host.url=https://sonarqube.com" `
    #/d:"sonar.organization=ORGANIZATION_WE_CREATED_AT_SONARCLOUD" `
    #/d:"sonar.login=$authToken"

  # actual building
  nuget restore $sln
  MSBuild $sln /p:Configuration=Release          
  
  # running unit tests and calculating code coverage 
  $testPathPattern = "$($sln.Directory)\**\bin\Release\net452\*.Tests.dll"
  $dlls = Get-ChildItem $testPathPattern -r
  $xUnitRelativePath = ".\xunit.runner.console\tools\net452\xunit.console.exe"
  $runner = Resolve-Path $xUnitRelativePath
  $coverConfig = Get-ChildItem $("$($sln.Directory)\*.Tests\coverage.xml")
  dotCover analyse $coverConfig.FullName `
    /TargetExecutable="$runner" `
    /TargetArguments="$dlls" `
    /ReportType=HTML `
    /Output=dotCover.html

  # collecting results and sending to SonarQube server
  SonarScanner.MSBuild end
  
  Write-Host "Analysing $sqProj completed!"
}
```

I've uploaded analysis results to my local server as well as to the [SonarCloud.io][sq-cloud-out]. You can try them out.

# Summary

This article explained how to install and configure your first SonarQube analysis in .NET stack. 

[The next one][next-results] elaborates on the results of that analysis. It answers the question whether Jon Skeet writes good quality code or not (joke! Of course not, it's rather about various attributes of his style of coding we all can learn from).

[prev-intro]:   /software/2018-10-10-sonarqube-1-intro   "SonarQube #1 - Introduction"
[next-results]: /software/2018-10-10-sonarqube-3-jon-skeets-libs-analysis-results "SonarQube #3 - Analysis results of Jon Skeet's libraries"

[sq-doc]:       https://docs.sonarqube.org/display/SONAR/Documentation "SonarQube documentation. In particular - installation and configuration manuals."
[sq-downloads]: https://www.sonarqube.org/downloads/ "SonarQube downloads" 
[sq-docker]:    https://hub.docker.com/r/_/sonarqube/ "Docker image for SonarQube server"
[sq-cloud]:     https://sonarcloud.io "SonarQube SaaS service"
[sq-cloud-out]: https://sonarcloud.io/organizations/wachulski-github/projects?search=Google "Jon Skeet's projects analysis results available on SonarCloud.io"
[skeet-book]:   https://amzn.to/2O6ud2g "C# in Depth by Jon Skeet"
[skee-so]:      https://stackoverflow.com/users/22656/jon-skeet "Jon Skeet StackOverlow profile"
[choco-scan]:   https://chocolatey.org/packages/sonarscanner-msbuild-net46 "Chocolatey install of Sonar Scanner" 
[dotcover]:     https://chocolatey.org/packages/dotcover-cli "JetBrains dotCover CLI for code coverage analysis"
[xunit]:        https://xunit.github.io/ "xUnit runner to execute unit tests"