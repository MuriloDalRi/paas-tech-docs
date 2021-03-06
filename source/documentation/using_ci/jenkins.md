## Push an app with Jenkins

There are two main approaches to pushing applications to GOV.UK PaaS with [Jenkins](https://jenkins.io/):

1. Use custom scripts (allows full scripting of deployments)
1. Use the Cloud Foundry plugin for Jenkins (less flexible, relies on application manifest for configuration)

Both approaches require you to set up the credentials plugin first.

### Set up dedicated accounts

CI systems should not use normal user accounts. Find out more about setting [credentials for automated accounts](#credentials-for-automated-accounts) in PaaS.

### Setting up the credentials plugin

To install the credentials plugin manually:

1. In the Jenkins web interface, click on **Manage Jenkins**, then **Manage Plugins**.
2. Click on the **Installed** tab and check if "Credentials Plugin" is listed. If it's listed, skip the next step.
3. Click on the **Available** tab and find "Credentials Plugin". Check the box to select the plugin, then click either **Install without restart** or **Download now and install after restart** at the bottom of the interface.

Once the plugin is installed, you will see a **Credentials** link in the left-hand navigation menu.

Now you can add credentials for the Cloud Foundry user you will be using to push apps from Jenkins.

1. Click on **Credentials**, then **System**. By default, you will see a 'Global credentials (unrestricted)' domain. You should set up a different domain for each deployment target/PaaS user account.
2. Click **Add domain** and enter a name and description, then click **Save**.
3. Click **Add credentials** and enter the PaaS user account credentials. The kind of credentials is 'Username with password', which is typically selected by default.

You can now go on to either:

* [set up custom scripts](#setting-up-custom-scripts) *or*
* [set up the Cloud Foundry plugin](#setting-up-the-cloud-foundry-plugin)

### Setting up custom scripts

Before you set up custom scripts, make sure you first [set up the credentials plugin](#setting-up-the-credentials-plugin).

Note that using the custom scripts approach exposes the password via the process command line, so it can be read by other processes running on the same machine. If this risk is not acceptable, please use the Cloud Foundry plugin described below. The Cloud Foundry project is aware of the problem and we expect they will provide a more secure login mechanism soon.

The custom scripts approach needs the Jenkins credentials binding plugin. To install it manually:

1. In the Jenkins web interface, click on **Manage Jenkins**, then **Manage Plugins**.
2. Click on the **Available** tab and find "Credentials Binding Plugin". Check the box to select the plugin, then click either **Install without restart** or **Download now and install after restart** at the bottom of the interface.
3. In your build configuration, select the **Use secret text(s) or file(s)** checkbox in the "Build Environment" section.
4. Click on the **Add** drop down menu and select **Username and password (separated)**.
5. Choose your variable names and select the user whose credentials will be used.

Once this is set up, any shell scripts you configure in Jenkins will have the credentials available as environment variables.

To protect these credentials and prevent them leaking to other Jenkins jobs, we suggest you set your `CF_HOME` to a temporary directory.

A basic 'execute shell' buildstep would look like this:

```
# Set CF_HOME in a temp dir so that jobs do not share or overwrite each others' credentials.
export CF_HOME="$(mktemp -d)"
trap 'rm -r $CF_HOME' EXIT

cf api https://api.cloud.service.gov.uk

# Note: the actual name of the environment variable is determined
# by what you enter into the Credentials Binding Plugin
cf auth "$CF_USER" "$CF_PASSWORD"

cf target -o myorg -s myspace
cf push

# Destroy token
cf logout
```

### Setting up the Cloud Foundry Jenkins plugin

Using the Cloud Foundry plugin only allows Jenkins to push your application to GOV.UK PaaS as a post-build action: the equivalent of doing a `cf login` followed by a `cf push`. There is little scope for configuration beyond using the application manifest.

Before you do these steps, make sure you first [set up the credentials plugin](#setting-up-the-credentials-plugin).

To install the Cloud Foundry plugin manually:

1. In the Jenkins web interface, click on **Manage Jenkins**, then **Manage Plugins**.
2. Click on the **Available** tab and find "Cloud Foundry Plugin". Check the box to select the plugin, then click either **Install without restart** or **Download now and install after restart** at the bottom of the interface.

An extra post-build action called "Push to Cloud Foundry" is now available in the dropdown menu when you configure a job.

1. In your job's configuration, click the **Add post-build action** dropdown menu and select **Push to Cloud Foundry**.
2. In the **Target** field, enter `https://api.cloud.service.gov.uk`.
3. In **Credentials**, select the user you created using the credentials plugin.
4. Enter your organisation and the space the application will be deployed to. See [Managing orgs, spaces and users](/#managing-organisations-spaces-and-users) for more details about organisations and spaces. You do not need to tick "Allow self-signed certificate" or "Reset app if already exists".
5. The rest of the fields can be left with their default values. The plugin expects you to have a manifest file called `manifest.yml` in the root of the application folder. If you do not, you can provide the path to the application manifest, or enter a manifest configuration directly into the plugin.
6. Click **Save**.

Further information can be found on the [Cloud Foundry plugin's wiki page](https://wiki.jenkins-ci.org/display/JENKINS/Cloud+Foundry+Plugin).
