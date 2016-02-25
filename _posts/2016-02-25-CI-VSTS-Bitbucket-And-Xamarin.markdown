---
layout: post
title:  "CI With VSTS And Bitbucket"
date:   2016-02-26
---

<p class="intro">
<span class="dropcap">H</span>ow to make Continuous Integration with Bitbucket and VSTS, when Bitbucket is not a default service inside VSTS.
</p>


### The mission
In my previous blog [post](http://rasmustc.com/blog/Continuous-Integration-With-VSTS-And-Xamarin/) I described how to make a basic CI setup with VSTS using a repository hosted in VSTS.
Bitbucket is a very popular source code repository, unfortunately not a default choice in VSTS, like GitHub etc. In this post I'll show the steps required
to make Bitbucket work as repository. This post will use some of the concepts described in my previous blog post.
The build will also run on a local build agent, VSO agent.


### Making a repository
The first step is to make a repository at Bitbucket and make an initial commit. When the commit is made, copy the __https__ url of the repository.
{% highlight csharp %}
https://rasmuschristensen@bitbucket.org/rasmuschristensen/nearby.git
{% endhighlight %}

### VSTS Bitbucket Service endpoint
Because there is no default hook in VSTS to support Bitbucket, we need to create it as __external git__.
Navigate to the collection settings for your project, by selecting the 'gears' icon in the upper right corner of your project.
Select the __Services__ tab and click the __New Service Endpoint__. From the menu select __External Git__.
A dialog is displayed. Add a __Connection name__, this is just your identification of the service endpoint.
The __Server Url__ is the fully qualified url we copied from the Bitbucket homepage. As we used __https__ username and password is the ones you use 
for regular https usage of the repository.

<img src="{{ '/assets/img/bitbucketserviceendpoint.png' | prepend: site.baseurl }}" alt="serviceendpoint">


### VSTS permissions
VSTS build needs some permissions to use this external git, during a build process.
Navigate to the __Default collection__ a level above your corrent project and select the __Security__ tab. Locate __Project Collection Build Administrators__ and ensure __Edit Collection-level information__ is set to __Allow__.
If not you'll get an error during the build process.

<img src="{{ '/assets/img/buildpermissions.png' | prepend: site.baseurl }}" alt="buildpermissions">

### Adding a build definition
Navigate to your VSTS project again and select the __Build__ tab. Click the __+__ to create a new build definition.
I'm using the Xamarin.iOS. From the template select __Remote git repository__ (note the name difference :)).
Once created, you can now edit the details. 

From the __Build__ menu item of the build definition, add the relative path to the solution, sln file.
_(hint: If you get this one wrong, you can use the build console output, to see the fully qualified path, and easy correct it afterwards)_.
Next you need to select the __Repository__ menu item. The __Repository type__ should already be set to __external git__ (not remote :)). Refresh the __Connection__ and select
the Bitbucket service endpoint we created earlier, leave the __Repository name__ blank. The service endpoint is already fully qualified. Save the build definition.

<img src="{{ '/assets/img/bitbucketgit.png' | prepend: site.baseurl }}" alt="bitbucketgit">

### Update VSO-agent
Ensure to update your VSO build agent. I used a previous version and had issues with username and password for the external git, not being passed along to the 
build agent, causing the build to fail (see this [issue](https://github.com/Microsoft/vso-agent/issues/183)). Updating an existing agent is the same as installing a new one. Navigate to the agents directory where the package.json is located and 
execute the following command.
{% highlight csharp %}
curl -skSL http://aka.ms/xplatagent | bash
{% endhighlight %} 

### Let's build!
When all steps are completed, ensure the VSO agent is running and queue a new build. (_Note, if you are build a Xamarin project, you need a business license._)
When the build starts, a console is displayed inside the browser where you can follow all steps along the build process. If the sln path is incorrect, this is also where you can see the current resolved path.

After a couple of permission errors as described above and incorrect sln path, the result should be this, a green build

<img src="{{ '/assets/img/buildsucceeded.png' | prepend: site.baseurl }}" alt="build succeeded">
