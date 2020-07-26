{:layout :post, :title "\"DRYing out Objective-C - Identification\"", :date "2013-02-23 12:09"}
I have a large objective-c codebase I've been working on with a client for over a year now. The application started off as a prototype, and transitioned into a demo client, and is currently undergoing modifications for security/penetration testing and commercialization. Initially for the protoype and demo, the objective was to get a working application as quickly as possible - speed of initial development was the key. With the current change in focus to a more productized codebase, and improving maintainability as part of that, I decided I'd actively go hunting for areas in the application that can be tidied up, and particularly, looking for duplicate segments of code and eliminating them where feasible.

[DRY - or Don't Repeat Yourself](http://en.wikipedia.org/wiki/Don't_repeat_yourself) - is (according to Wikipedia) a _principle is stated as "Every piece of knowledge must have a single, unambiguous, authoritative representation within a system."_ ... _When the DRY principle is applied successfully, a modification of any single element of a system does not require a change in other logically unrelated elements. Additionally, elements that are logically related all change predictably and uniformly, and are thus kept in sync._

One big problem with a larger code base that has been developed over a long period of time is that you may not know where the duplicate code actually is. You know it's there, you're just not sure where.

Finding Duplicate Code
----------------------

What I wanted for DRYing up the code base was for duplicate chunks of code to be identified for me.

Searching out there I stumbled across a project called [Simian](http://www.harukizaemon.com/simian/) - it's a java based tool for identifing duplicate code in a set of different programming languages - one being Objective-C. Simian supports output in a number of different formats - plain text being the default, but also supports an XML based output. The project is available on a 15 day evaluation period, and then should be paid for commercial or enterprise use. A Build Server license costs $499 US.

Simian can be run against a codebase just by feeding it include/exclude directory and file patterns.

    java -jar simian-2.3.33.jar -excludes=\"External Libraries\" **/*.m **/*.h

You can also change the format of the output

    java -jar simian-2.3.33.jar -formatter=xml -excludes=\"External Libraries\" **/*.m **/*.h

And even output to a file

    java -jar simian-2.3.33.jar -formatter=xml:simian.xml -excludes=\"External Libraries\" **/*.m **/*.h

Integration with Jenkins
------------------------

I've previously written here about setting up and using Jenkins as a build/CI system with Objective-C/iOS projects, and I really wanted to integrate this duplicate code reporting as part of my standard build process, along with my unit and application test reports.

To get Simian reports integrated with Jenkins there is a Jenkins extension available called the [DRY Plugin](https://wiki.jenkins-ci.org/display/JENKINS/DRY+Plugin). Just navigate to your Jenkins instance and click... Manage Jenkins -> Manage Plugins -> Available and type Duplicate in the filter box. The plugin is called "Duplicate Code Scanner Plug-in". Install it.

To get the Simian process running is really simple. I added a new project that I could trigger after my unit tests have run, called "Code Analysis". This project has a very small number of steps.

1. Pull the source from your code repo
2. Set a Build Trigger for the project to start after your unit test project has completed. _This step isn't necessary, but you need some sort of build trigger. I like mine to work after unit tests as then I know the codebase is in a good state._
3. Add an Execute Shell task. The task should look something like below

{% codeblock %}
cd "<Path to your Jenkins project>/workspace"
git submodule update --recursive --init
echo "#!/bin/bash" > simian.sh
echo "java -jar <Path to your simiar jar>/simian-2.3.33.jar -balanceSquareBrackets=true -formatter=xml:simian.xml -excludes=\"External Libraries\" **/*.m **/*.h" >> simian.sh
echo "exit 0" >> simian.sh
chmod u+x simian.sh
./simian.sh
rm simian.sh
{% endcodeblock %}

What the above task does is change directory to the correct jenkins workspace, ensure all submodules are updated (if you don't use submodules, you probably won't want this), then create a shell script that runs Simian and exits with a 0 return code, sets the script to be executable, runs the script, then cleans up after itself. The reason why the build task needs to create a shell script to run Simian is because the return code from Java/Simian seems to be interpreted by Jenkins as non-0 i.e. a build failure. You don't want that.

The `-balanceSquareBrackets=true` flag to Simian ensures that code that is split across multiple lines inside square brackets is treated as a single unit. It might be a good idea to use the `-balanceParentheses=true` flag as well to help matching on things like `if` statements.

Then it's just a matter of configuring the reporting. If you've installed the DRY plugin correctly, you should be able to add a Post Build Action of "Publish duplicate code analysis results". In the "Duplicate code results" field, type in the path and filename you gave the output XML from simian - in the example above I called mine `simian.xml`.

That's it.

Save and "Build Now" your new project, and after this is complete, click on the project. There should be a fancy trend graph showing Duplicate Code in the upper right of the screen, and a "Duplicate Code" item on the left navigation menu. That will show you all the files with duplicate code chunks, and the other files they are repeated in.
