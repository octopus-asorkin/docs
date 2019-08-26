---
title: Jenkins
description: Jenkins can work together with Octopus Deploy to create releases and execute deployments.
position: 60
---

[Jenkins](http://jenkins-ci.org/) is an extendable, open-source continuous integration server that makes build automation easy.

Using Jenkins and Octopus Deploy together, you can:

- Use Jenkins to compile, test, and package your applications.
- Automatically trigger deployments in Octopus from Jenkins whenever a build completes.
- Automatically fail a build in Jenkins if the deployment in Octopus fails.
- Securely deploy your applications with Octopus Deploy across your infrastructure.
- Fully automate your continuous integration and continuous deployment processes.

## Jenkins Installation

If you need guidance installing Jenkins for the first time, see the [Jenkins documentation](https://jenkins.io/doc/book/installing/), or the blog post, [installing Jenkins from Scratch](https://octopus.com/blog/installing-jenkins-from-scratch).

## Jenkins Plugins

Plugins are central to Jenkins, and a number of plugins are required to follow the steps on this page. Before you start, you'll need to ensure the following plugins are enabled:

- [MSBuild Plugin](http://wiki.jenkins-ci.org/display/JENKINS/MSBuild+Plugin): required to compile your Visual Studio solution.
- [Mask Passwords Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Mask+Passwords+Plugin): required to store your Octopus API keys and keep them out of the console.

## Build job

During our Jenkins job, we will:

1. Compile the code, and run unit tests.
2. Create NuGet packages with OctoPack.
3. Publish these NuGet packages to the Octopus Deploy server.
4. Create a release in Octopus, ready to be deployed.

Jenkins uses the MSBuild plugin to compile Visual Studio solutions. After you've installed [OctoPack](/docs/packaging-applications/create-packages/octopack/index.md) on your C#/VB projects, you can configure Jenkin's MSBuild task to pass the appropriate parameters to MSBuild to have OctoPack run:

![](images/msbuild-parameters.png)

There are a number of parameters you need to define. For this page, we are using:

```bash
/p:RunOctoPack=true /p:OctoPackPackageVersion=1.1.${BUILD_NUMBER} /p:OctoPackPublishPackageToHttp=http://localhost/nuget/packages /p:OctoPackPublishApiKey=${OctopusApiKey}
```

The settings are:

- **RunOctoPack**: specifies that OctoPack should create packages during the build.
- **OctoPackPackageVersion**: version number that should be given to packages created by OctoPack. Since Jenkins build numbers are integers like "12", we combine it with "1.1." to produce package versions such as "1.0.12". Learn more about [versioning in Octopus](/docs/packaging-applications/create-packages/versioning.md).
- **OctoPackPublishPackageToHttp**: tells OctoPack to push the package to the Octopus Deploy server. Read more about the [Octopus built-in repository](/docs/packaging-applications/package-repositories/index.md). You'll find the URL to your repository on the **{{Library,Packages}}** tab in Octopus. 
- **OctoPackPublishApiKey**: your Octopus Deploy API key. See [how to create an API key](/docs/octopus-rest-api/how-to-create-an-api-key.md).

:::success
**OctoPack arguments**
Learn more about the available [OctoPack parameters](/docs/packaging-applications/create-packages/octopack/index.md).
:::

Notice that we use `${OctopusApiKey}` to access an API key that we will use to authenticate with Octopus. You define this using the fields provided by the **Mask Passwords plugin** on your job.

![](images/3278146.png)

:::success
**Creating API keys**
Learn about [how to create an API key](/docs/octopus-rest-api/how-to-create-an-api-key.md).
:::

After running this job, and assuming OctoPack is correctly installed, your code should compile, and packages should be published to the Octopus Deploy Server. You can go to **{{Library,Packages}}** in Octopus to check that the packages have been published.

## Creating a Release {#Jenkins-Creatingarelease}

Jenkins is compiling our code and publishing packages to Octopus Deploy. If we wish, we can also have Jenkins automatically create (and optionally, deploy) a release in Octopus.

To do this, we'll be using the [Octo.exe command line tool](/docs/octopus-rest-api/octo.exe-command-line/index.md). [Download Octo.exe](https://octopus.com/downloads), and extract it to a folder on your Jenkins server, such as `C:\Tools\Octo\Octo.exe`

We can call Octo.exe easily using the Jenkins **Execute Windows batch** **command** task.

![](images/3278144.png)

In the command, we are calling:

```bash
"C:\Tools\Octo\Octo.exe" create-release --project OctoFX --version 1.1.%BUILD_NUMBER% --packageversion 1.1.%BUILD_NUMBER% --server http://localhost/ --apiKey %OctopusApiKey% --releaseNotes "Jenkins build [%BUILD_NUMBER%](http://localhost:8054/job/OctoFX/%BUILD_NUMBER%)/"

```

Importantly:

- The `--project` specifies the name of the Octopus Deploy project that we want to create a release for.
- The `--version` specifies the version number of the release in Octopus. We want this to contain the Jenkins build number.
- The `--packageversion` tells Octo.exe to ensure that the release references the right version of the NuGet packages that we published using OctoPack.
- The `--releaseNotes` will appear in Octopus, and link back to the build in Jenkins. Of course, change the URL to the address of your Jenkins server

:::success
**Octo.exe arguments**
Learn more about [Octo.exe](/docs/octopus-rest-api/octo.exe-command-line/index.md) and the arguments it accepts.
:::

With this job runs, Jenkins will now not only build and publish packages, but it should also create a release in Octopus Deploy.

## Deploying Releases {#Jenkins-Deployingreleases}

You might like to configure Jenkins to not only create a release, but deploy it to a test environment. This can easily be done by adding some extra parameters to the `create-release` command:

```bash
"C:\Tools\Octo\Octo.exe" create-release ....(same as before)... --deployto=Development --progress
```

The extra arguments being:

- The `--deployTo` setting specifies the environment in Octopus that we are deploying to.
- The `--progress` flag tells Octo.exe to write the deployment log from Octopus to the log in Jenkins. This flag was added in 2.5; in previous versions of Octo.exe you can use `--waitfordeployment` instead. You can also remove this flag if you want the Jenkins job to complete immediately without waiting for the deployment in Octopus to complete.

:::success
**Octo.exe arguments**
Again, see the [arguments to Octo.exe](/docs/octopus-rest-api/octo.exe-command-line/index.md) to see other parameters that you can specify. If your deployment is likely to take longer than 10 minutes, for example, consider passing `--deploymenttimeout=00:20:00` to make it 20 minutes.
:::

With these settings, Jenkins should trigger a deployment as soon as a job completes.