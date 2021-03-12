---
title: measuring msbuild performance
id: 52
date: 2009-08-30 09:29:36
updated: 2009-08-30 09:29:36
tags:
- msbuild
- performance
---

Recently, while debugging some strange problems with our build, I flipped on MSBuild's `diagnostic` output level. I was surprised and delighted to see a profile of my build at the end of the output. Here's what the output looks like for [CruiseControl.Net's](http://confluence.public.thoughtworks.org/display/CCNET/Welcome+to+CruiseControl.NET) clean target:
<pre lang="text">Project Performance Summary:
       16 ms  C:\code\ccnet-trunk\project\CCTray\CCTray.csproj   1 calls
                 16 ms  Clean                                      1 calls
       16 ms  C:\code\ccnet-trunk\project\CCTrayLib\CCTrayLib.csproj   1 calls
                 16 ms  Clean                                      1 calls
       16 ms  C:\code\ccnet-trunk\project\service\service.csproj   1 calls
                 16 ms  Clean                                      1 calls
       16 ms  C:\code\ccnet-trunk\project\console\console.csproj   1 calls
                 16 ms  Clean                                      1 calls
       16 ms  C:\code\ccnet-trunk\project\objection\objection.csproj   1 calls
                 16 ms  Clean                                      1 calls
       16 ms  C:\code\ccnet-trunk\project\WebDashboard\WebDashboard.csproj   1 calls
                 16 ms  Clean                                      1 calls
       16 ms  C:\code\ccnet-trunk\project\UnitTests\UnitTests.csproj   1 calls
                 16 ms  Clean                                      1 calls
       16 ms  C:\code\ccnet-trunk\project\core\core.csproj   1 calls
                 16 ms  Clean                                      1 calls
       16 ms  C:\code\ccnet-trunk\project\Remote\Remote.csproj   1 calls
                 16 ms  Clean                                      1 calls
       31 ms  C:\code\ccnet-trunk\project\Validator\Validator.csproj   1 calls
                 31 ms  Clean                                      1 calls
      297 ms  C:\code\ccnet-trunk\project\ccnet.sln      1 calls
                297 ms  clean                                      1 calls

Target Performance Summary:
        0 ms  CleanReferencedProjects                   10 calls
        0 ms  SplitProjectReferencesByType               8 calls
        0 ms  BeforeClean                               10 calls
        0 ms  ValidateToolsVersions                      1 calls
        0 ms  AfterClean                                10 calls
        0 ms  CleanPublishFolder                        10 calls
       16 ms  _CheckForInvalidConfigurationAndPlatform  10 calls
       16 ms  _SplitProjectReferencesByFileExistence    10 calls
       16 ms  ValidateSolutionConfiguration              1 calls
      141 ms  CoreClean                                 10 calls
      281 ms  Clean                                     11 calls

Task Performance Summary:
        0 ms  FindUnderPath                             20 calls
        0 ms  AssignProjectConfiguration                 8 calls
        0 ms  Message                                   21 calls
        0 ms  MakeDir                                   10 calls
        0 ms  RemoveDuplicates                          10 calls
       16 ms  WriteLinesToFile                          10 calls
       31 ms  ReadLinesFromFile                         10 calls
       78 ms  Delete                                    11 calls
      281 ms  MSBuild                                    4 calls</pre>

It's quite easy to change the verbosity level from [NAnt's MSBuild task](http://nantcontrib.sourceforge.net/release/0.85-rc4/help/tasks/msbuild.html):
<pre lang="xml"><msbuild verbosity="Diagnostic" project="project\ccnet.sln">
	<property name="Configuration" value="Build" />
</msbuild>
</pre>

I was happy to see profiling information because the speed of the build in Visual Studio on our current project is making unit tests painful. Running a single unit test involves waiting for about a minute while the code compiles and ReSharper's test runner starts up. Then the test runs, generally taking less than a second. The profiling output from MSBuild seems like an ideal way to diagnose the compile speed problem. I played around with different output settings to understand what's available before tackling the build speed problem. After all, measuring something usually changes it.

One of the first things I noticed was that using the Diagnostic verbosity level very significantly slowed my build down. I decided to quantify that slow down, and check to make sure that the other verbosity levels don't suffer from a similar problem. Here are the summarized results.

<table class="data-table">
<thead>
<tr><td/><td colspan="2">total time</td><td colspan="2">std deviation</td></tr>
<tr><td class="corner">verbosity</td><td>compile</td><td>clean</td><td>compile</td><td>clean</td></tr>
</thead>
<tbody>
<tr><td>Diagnostic</td><td>88.6s</td><td>19.4s</td><td>±6.3s</td><td>±0.6s</td></tr>
<tr><td>Detailed</td><td>32.1s</td><td>5.7s</td><td>±3.8s</td><td>±0.2s</td></tr>
<tr><td>Normal</td><td>14.9s</td><td>1.1s</td><td>±1.4s</td><td>±0.03s</td></tr>
<tr><td>Minimal</td><td>13.5s</td><td>0.3s</td><td>±0.8s</td><td>±0.05s</td></tr>
<tr><td>Quiet</td><td>14.0s</td><td>0.3s</td><td>±1.4s</td><td>±0.02s</td></tr>
</tbody>
</table>

I did 5 builds at each level, and averaged the results. Since I needed to clean anyway between each test, I gathered those stats too. MSBuild provides the information in milliseconds, but I am presenting it in seconds. It seems like Normal is a reasonable setting where the output doesn't slow the build down significantly. Minimal is slightly faster, and I prefer it anyway because I find the terser output easier to follow.

To gather timings for different verbosities, I didn't use NAnt, but instead invoked `msbuild` directly from the command-line. Of the available verbosity levels, only Diagnostic gives the performance summary. Luckily, there is another switch that allows more fine-grained tuning of what appears on the console. Here are a couple of examples, but `msbuild /?` can give you more information.

<pre lang="text">C:\code\ccnet-trunk\project>msbuild ccnet.sln /verbosity:Diagnostic

C:\code\ccnet-trunk\project>msbuild ccnet.sln /target:clean /consoleloggerparameters:verbosity=normal;PerformanceSummary</pre>

Before jumping to conclusions about MSBuild's performance, I wanted to check if the slowdown is tied to using Diagnostic level at all, or just using it on the console. After all, the windows command prompt is known to be a bit of a slouch. I tried a build with Diagnostic level logging being sent to a file:

<pre lang="text">C:\code\ccnet-trunk\project>msbuild ccnet.sln /noconsolelogger /filelogger /fileloggerparameters:verbosity=Diagnostic;PerformanceSummary</pre>

With these settings, the build takes only 15.2 seconds, but still generates the full 1.5 megabytes of diagnostic logs. It seems the performance problem really lies with Windows Command Prompt. I want to try to determine if logging suffers the same penalties during a build on a continuous integration server or through Visual Studio. I have not yet discovered how to tweak the verbosity level from Visual Studio. For the CI server, I simply have not yet bothered to do the test. If there is a slowdown during CI, a simple fix may be to log all the output to a file and then include that in the build artifacts.

For now though, I'm planning on tweaking my solution structure as [Patrick Smacchia suggests](http://codebetter.com/blogs/patricksmacchia/archive/2009/03/15/analyzing-the-code-base-of-cruisecontrol-net.aspx) to see if there is a noticeable build speed improvement.
