---
layout: post
title: "Visual Studio 2015 - Unit Tests Fail When Referencing Crystal Reports"
date: 2015-12-03
disqus_identifier: 3
---
tl;dr: If unit tests referencing Crystal Reports are crashing, find the app.config for the test runner for your version of Visual Studio and ensure it has the following:

        <startup useLegacyV2RuntimeActivationPolicy="true">
        </startup>

I've been using Visual Studio 2015 at home for a while and decided to install it at on my work machine several months ago. All of the projects I tried it on didn't have any problems with one exception. One solution used Crystal Reports to generate PDFs and had a unit test that generated a PDF as part of the test. That test worked fine in Visual Studio 2013 but would cause the test runner to crash in Visual Studio 2015. If I ran the tests with the debugger, I would see the famous error below.

{% include figure.html caption="FileNotFound exception" url="/images/posts/2015-12-03-Visual-Studio-Crystal Reports-dotnet1/FileNotFound Exception.png" %}

The website that used this code worked fine. It was only the unit tests that would fail and only in Visual Studio 2015. While researching the problem, I found several references stating that adding `useLegacyV2RuntimeActivationPolicy="true"` would fix the problem. I added it to the app.config for the test project, but it didn't solve the problem. After wasting too much time on it, I decided to let it go for the time being and stick with Visual Studio 2013.

When Microsoft released the update for Visual Studio 2015 on November 30th, I decided to try again. I was hoping the problem was due to something going wrong when I had installed Crystal Reports and hoped the update installation would work some magic. And of course, no joy. I then tried uninstalling Crystal Reports and installing the newer version that had come out since. Still no joy.

Then I finally started actually thinking (tends to help a lot). All of the fixes that I was finding for this problem were in regards to WPF or WinForm apps. My problem was with a unit test project. I knew that the app.config in the project was used by the generated assembly since I could refer to it in the tests. But the unit test project generates a DLL, not an EXE. Isn't there supposed to be an EXE that actually runs the tests?

{% include figure.html caption="Hmmm..." url="/images/Hmmm.jpg" %}

When I added unit test as keywords in my googling, I found [this question](http://stackoverflow.com/questions/18786738/crdb-adoplus-dll-unit-test-solution) on StackOverflow. Aha! Visual Studio 2015 is in the `Microsoft Visual Studio 14.0` folder, so I followed the path from there and found...several configuration files for `vstest.executionengine.exe`. Ok, fine, let's check 'em all. They all actually had the correct setting except for one, so I updated that one, ran my tests and...AARGH! Still not working. OK, time to start thinking again. I thought I remembered seeing something in the output window, and sure enough, there was the filename of the test runner right under my nose. So I found `C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\CommonExtensions\Microsoft\TestWindow\TE.ProcessHost.ManagedExe.exe.config`, added the `<startup/>` section, and tried again. Success!

{% include figure.html caption="Test Debug Output" url="/images/posts/2015-12-03-Visual-Studio-Crystal Reports-dotnet1/Test Debug Output.png" %}

So I'm finally up and running with Visual Studio 2015 and Crystal Reports. And I'm still waiting for a better alternative to Crystal Reports so I can get away from the Crystal Reports hell.