# Jenkins S2I Example

An example demonstrating Jenkins S2I features for installing plugins, configuring jobs, Jenkins, etc and using slave pods for Jenkins jobs.

## Installation

1. Create a new OpenShift project, where the Jenkins server will run:

  ```
  $ oc new-project ci --display-name="CI/CD"
  ```

2. Give the Jenkins Pod service account rights to do API calls to OpenShift. This allows us to do the Jenkins Slave image discovery automatically.

  ```
  $ oc policy add-role-to-user edit -z default -n ci
  ```

3. Install the provided OpenShift templates:

  ```
  $ oc create -f jenkins-slave-builder-template.yaml   # For converting any S2I to Jenkins slave
  $ oc create -f jenkins-master-s2i-template.yaml      # For creating pre-configured Jenkins master using Jenkins S2I
  ```

5. Build Jenkins slave image.

  ```
  $ oc new-app jenkins-slave-builder
  ```

4. Create Jenkins master. You can customize the source repo and other configurations through template parameters. Note that this example doesn't define any [persistent volume](https://docs.openshift.com/enterprise/3.2/architecture/additional_concepts/storage.html). You need to define storage in order to retain Jenkins data on container restarts. 

  ```
  $ oc new-app jenkins-master-s2i
  ```

## Openshift Pipelines...

The groovy code will accept a node label to trigger the build to happen on the appropriate slave...

```
node('jdk8') {
   // Mark the code checkout 'stage'....
   stage 'Checkout'

   // Get some code from a GitHub repository
   git url: 'https://github.com/jglick/simple-maven-project-with-tests.git'

   // Get the maven tool.
   // ** NOTE: This 'M3' maven tool must be configured
   // **       in the global configuration.           
   def mvnHome = tool 'M3'

   // Mark the code build 'stage'....
   stage 'Build'
   // Run the maven build
   sh "${mvnHome}/bin/mvn clean install"
}
```

## TODO

- Configure the slave to have the appropriate M3/Maven location and JAVA_HOME set properly...

```
Started by user Jenkins Admin
[Pipeline] node
Running on jdk8-jenkins-slave-a4065d499ef in /var/lib/jenkins/workspace/test-pipe
[Pipeline] {
[Pipeline] stage (Checkout)
Entering stage Checkout
Proceeding
[Pipeline] git
Cloning the remote Git repository
Cloning repository https://github.com/jglick/simple-maven-project-with-tests.git
 > git init /var/lib/jenkins/workspace/test-pipe # timeout=10
Fetching upstream changes from https://github.com/jglick/simple-maven-project-with-tests.git
 > git --version # timeout=10
 > git -c core.askpass=true fetch --tags --progress https://github.com/jglick/simple-maven-project-with-tests.git +refs/heads/*:refs/remotes/origin/*
 > git config remote.origin.url https://github.com/jglick/simple-maven-project-with-tests.git # timeout=10
 > git config --add remote.origin.fetch +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git config remote.origin.url https://github.com/jglick/simple-maven-project-with-tests.git # timeout=10
Fetching upstream changes from https://github.com/jglick/simple-maven-project-with-tests.git
 > git -c core.askpass=true fetch --tags --progress https://github.com/jglick/simple-maven-project-with-tests.git +refs/heads/*:refs/remotes/origin/*
 > git rev-parse refs/remotes/origin/master^{commit} # timeout=10
 > git rev-parse refs/remotes/origin/origin/master^{commit} # timeout=10
Checking out Revision 7b64fc4ac386dd9e34df63feef99f2260ec9a6b0 (refs/remotes/origin/master)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f 7b64fc4ac386dd9e34df63feef99f2260ec9a6b0 # timeout=10
 > git branch -a -v --no-abbrev # timeout=10
 > git checkout -b master 7b64fc4ac386dd9e34df63feef99f2260ec9a6b0
 > git rev-list 7b64fc4ac386dd9e34df63feef99f2260ec9a6b0 # timeout=10
[Pipeline] tool
Unpacking https://repo.maven.apache.org/maven2/org/apache/maven/apache-maven/3.3.9/apache-maven-3.3.9-bin.zip to /var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/M3 on jdk8-jenkins-slave-a4065d499ef
[Pipeline] stage (Build)
Entering stage Build
Proceeding
[Pipeline] sh
[test-pipe] Running shell script
+ /var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/M3/bin/mvn clean install
/var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/M3/bin/mvn: line 143: which: command not found
/var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/M3/bin/mvn: line 171: which: command not found
Error: JAVA_HOME is not defined correctly.
  We cannot execute 
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
ERROR: script returned exit code 1
Finished: FAILURE
```

