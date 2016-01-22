---
layout: post
title:  "CI With VSTS And Xamarin"
date:   2016-01-22
---

<p class="intro">
<span class="dropcap">W</span>orks on my machine is dead, long live Continuous Integration with mobile development.
<br/>
 Continuous Integration also known as CI is one topic on the developers shelf of best practices. It's a well integrated part of many development teams today. It helps ensuring code also
 compiles outside the developers own machine also known as works on my machine. In this blog post I'll explain how to get up and running with CI using Visual Studio Team Services - VSTS on a Xamarin project.  
</p>


### The mission
The goal is to have a CI running and each time a deloper makes a push to the GIT repository, we want to trigger a new build also know as __Continuous Integration__. The app will be a vanilla Xamarin Forms project.
Further more we'll see how we can reduce the cost of using the build service in VSTS by making use of an __on premise__ build agent.
<img src="{{ '/assets/img/vstsxamarin.png' | prepend: site.baseurl }}" alt="vsts xamarin">


### Create VSTS Project
The first thing we need, is to create a VSTS project. Log on to [VSTS](https://www.visualstudio.com/en-us/products/visual-studio-team-services-vs.aspx) and create a new project.
Here we select __GIT__ as version control, which creates a __GIT repository__ also part of VSTS.
<img src="{{ '/assets/img/vstsnewproject.png' | prepend: site.baseurl }}" alt="vsts new project">
 
#### Prepare repository credentials for OSX
One minor detail is your repository credentials. If your __primary__ VSTS user name contains an __@__ you need to make an username alias before you can clone the repository on osx. Either go to your profile and the __security tab__ or click the __Generate GIT credentials__
Here you can specify the __alias__ and a __password__. Recommended by VSTS you can also generate and use a __personal access token__.
<img src="{{ '/assets/img/altcred.png' | prepend: site.baseurl }}" alt="alt credentials">
With the new repository setup we're ready to clone it and add a some code. As mentioned I've create a vanilla Xamarin Forms project and made the initial push to the repository. VSTS lets you manage your GIT repositories inside VSTS, just as Bitbucket or GitHub etc.

### Build agent setup and reduce build cost!
As we're are building a project for IOS we need as required by Apple, to build the source code on Apple hardware. Another concern is the cost of having the build performed inside Azure using a __hosted__ build agent, meaning we'll use Azure clock cycles hence spending more $$ on building our source code.
Currently Azure has no support for building on hosted MAC OS and while we could look for another hosted solution, let's save the money and make the build on our on premise hardware. This just needs to be some MAC/OSX powered device with the required software installed like Xamarin Studio etc.
If you are using your own developer machine, please notice the build agent will create its own workspace, and not influence your development GIT repository

#### Install build agent
Just as building the source on azure, we also need a __build agent__ when we want to have the build __on premise__. We'll need to install a build agent on the machine and specify some credentials, making it able to communicate with the build setup in VSTS.
Navigate to the root of your default collection and select the __Agent queues__ tab. Select __Download agent__ and navigate to the [xplat](https://www.npmjs.com/package/vsoagent-installer) location.
The recipe is well written so just follow it. Afterwards start the agent. I recommend starting out by running the agent in a terminal to quickly see any error messages. When everything is up and running, just switch over to have the build agent run as a service. 
<img src="{{ '/assets/img/installagent.png' | prepend: site.baseurl }}" alt="Install build agent">
#### Grant bulid agent permissions
When I first started the build agent you might get and error message saying something about the agent not being able to listen....permission required....
This is because you need to ensure the crendentials you specify also is granted the permissions to act as a build agent in VSTS. Return to the VSTS website and the __Agent queues__ tab. Here you should now be able to see the build agent running on your on premise machine.
Select the __Roles__ tab and add the account your build agent is using. Add it to both the __Agent Pool Administrators__ and __Agent Pool Service Accounts__ group. 
Restart the build agent and all error messages should now be gone.
<img src="{{ '/assets/img/agentroles.png' | prepend: site.baseurl }}" alt="agent roles">

### Make CI build definition
The final step to enable CI is to make a __build definition__ in VSTS using our build agent. 

From the __BUILD__ menu item on your VSTS project, add a new __Build definition__

<img src="{{ '/assets/img/new build definition.png' | prepend: site.baseurl }}" alt="new template">

Select the Xamarion.IOS template. Afterwards you can just add additional build for other platforms. 

<img src="{{ '/assets/img/iosTemplate.png' | prepend: site.baseurl }}" alt="IOS template">
On the final page we select GIT as version control, the project we want to build and the branch. This part is really good as it makes it very simple to make different build definitions to feature or release branches.
Ensure you select the non hosted build agent. I installed my agent as the default. If you need to make any changes, just click __Manage__.
The most important part of this page is to check the __Continuous Integration__ checkbox. Save the template and return to code__</>__

<img src="{{ '/assets/img/ciTemplate.png' | prepend: site.baseurl }}" alt="CI template">
  
### Verify CI is running.
In Xamarin Studio we make a change the source code, __add__, __commit__ and __push__... wait a bit and then watch the terminal running your build agent. You should soon see

* __Running job:build__
* __Job completed:build__
* __Job Finished:build__

__Continuous integration__ is now running with __Visual Studio Team Services__ and __Xamarin__!
