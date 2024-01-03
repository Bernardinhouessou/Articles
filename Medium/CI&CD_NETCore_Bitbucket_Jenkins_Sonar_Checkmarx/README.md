# Building a custom CI/CD Pipeline for .NET Core X App using Bitbucket, Jenkins, SonarQube and Checkmarx

1. Configure Bitbucket: **[How to configure Bitbucket](https://itznihal.medium.com/git-with-bitbucket-33078f29dfe1)**
2. Configure Jenkins:
    - Windows: Follow the guide **[Installing Jenkins on Windows](https://www.jenkins.io/doc/book/installing/windows/)**
    - MacOS: Follow the guide **[Installing Jenkins on macOS](https://www.jenkins.io/doc/book/installing/macos/)**
    - Docker: Follow the guide **[Installing Jenkins with Docker](https://www.jenkins.io/doc/book/installing/docker/)**
3. Configure SonarQube:
    - Windows: Install and run SonarQube using the guides:
        - **[SonarQube Installation on Windows](https://docs.sonarqube.org/latest/setup/install-server/)**
        - **[Running SonarQube as a Service on Windows](https://docs.sonarqube.org/latest/setup/operate-server/)**
    - MacOS: Install and run SonarQube using the guide **[SonarQube Installation on macOS](https://medium.com/@bharat.murali/installing-sonarqube-on-macos-693e3a3a4feb)**
    - Docker: Use the official SonarQube Docker image and follow the guide **[Running SonarQube on Docker](https://docs.sonarqube.org/latest/setup/get-started-2-minutes/)**
    - General Configuration Guides:
        - **[Configuring SonarQube](https://docs.sonarqube.org/latest/setup/install-server/)**
        - **[Configuring SonarScanner](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/)**
        - **[Integrating SonarQube with Jenkins](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-jenkins/)**
4. Configure & install Checkmarx:
    - Follow the provided guides:
        - **[Configuring Checkmarx One Build Steps in Jenkins](https://checkmarx.com/resource/documents/en/34965-68691-configuring-checkmarx-one-build-steps-in-jenkins.html) & [Setting up Scans in Jenkins](https://checkmarx.com/resource/documents/en/34965-8158-setting-up-scans-in-jenkins.html)**
        - **[Installing & Configuring the Jenkins Plugin](https://checkmarx.com/resource/documents/en/34965-8157-installing-and-configuring-the-jenkins-plugin.html)**
        - On Windows:  **[Checkmarx CxSAST Installation Guide for Windows](https://checkmarx.atlassian.net/wiki/spaces/KC/pages/894840812/CxSAST+Installation+Guide+for+Windows+-+9.3)**
        - On MacOS: Checkmarx doesn't natively support MacOS, but you can use a virtual machine / Docker **[Checkmarx Docker Image Guide](https://github.com/CheckmarxDev/ast-cli-docker)**

### **Prerequisites**

Before we start, make sure you have:

- A running Jenkins instance.
- A running SonarQube instance.
- A running Checkmarx instance.
- .NET Core X SDK installed on your Jenkins server.
- A .NET Core X project hosted on a Git/BitBucket repository

### **1. Install Necessary Jenkins Plugins**

Jenkins has a rich ecosystem of plugins that allow it to integrate with various systems and tools. For our pipeline, we'll need:

- Go to [https://plugins.jenkins.io](https://plugins.jenkins.io/) and search and install
    - Git  & Bitbucket plugins for source control management.
    - Sonar & qualityGate plugins
    - MSBuild Plugin for building .NET projects.
    - Pipeline & Multibranch Pipeline plugins for creating and managing the pipeline as code.

To install these plugins:

1. Navigate to **Jenkins Dashboard > Manage Jenkins > Manage Plugins**.
2. Go to the **Available** tab and search for these plugins.
3. Check the boxes next to them and click **Install without restart**.

### **2. Configure MSBuild**

Next, we need to configure MSBuild in Jenkins:

1. Navigate to **Jenkins Dashboard > Manage Jenkins > Global Tool Configuration**.
2. Scroll down to the **MSBuild** section and click on **Add MSBuild**.
3. Give it a name (e.g., MSBuild .NET Core 7), and point it to the path where MSBuild is located in your system.
4. Save the changes.

### **3. Create Jenkins Pipeline/ Multibranch pipeline Job**

Now, let's create a Jenkins job for our .NET Core 7 pipeline:

Go to Jenkins dashboard and

**Create a Simple Pipeline**:

- Click on "New Item" on the Jenkins dashboard to create a new job.
- Enter a name for the job, select "Pipeline" as the job type, and click "OK".
- In the configuration page, scroll down to the "Pipeline" section.
- Select the "Pipeline script" option.
- You can either write your pipeline script directly in the script text area or specify a Jenkinsfile from your version control system.
1. Save the job configuration.
2. Run the Simple Pipeline: You can manually trigger the pipeline by clicking on "Build Now" in the job's page. The pipeline will execute the defined stages sequentially.

**Create a Multibranch Pipeline Job:**

- Click on "New Item" on the Jenkins dashboard.
- Enter a name for the job, select "Multibranch Pipeline" as the job type, and click "OK".
- In the configuration page, scroll down to the "Branch Sources" section.
- Select the appropriate source where your code repository is hosted (e.g., Git, GitHub, Bitbucket).
- Configure the necessary credentials and repository details.
- Optionally, you can set up additional filters and behavior settings for branch discovery and indexing.
- Save the job configuration.

Jenkins will automatically discover branches in the configured repository and create pipeline jobs for each branch. Each branch's pipeline will execute based on the Jenkinsfile or pipeline script defined in the branch.

**INTRODUCTION**

Continuous Integration and Continuous Deployment (CI/CD) have become an integral part of modern software development practices. Jenkins, a widely used automation server, facilitates setting up CI/CD pipelines. In this article, we delve into a Jenkinsfile, the heart of any Jenkins pipeline, specifically tailored for .NET projects.

**Jenkinsfile: An Overview**

The Jenkinsfile, a cornerstone of any Jenkins project, is a text file containing the blueprint of a Jenkins Pipeline. It is scripted in Groovy, a dynamic language for the Java Virtual Machine. The Jenkinsfile at hand is written using the declarative pipeline syntax, making the pipeline code more readable and thus easier to understand. This Jenkinsfile is designed to automate various stages, including building, testing, and deploying a .NET Core project.

Now, let's dissect each stage in the Jenkinsfile to understand what they do.

**1. Initialization (Init)**

The pipeline begins with the **Initialization** stage **`stage('Init')`**, which prepares the environment for the build process or before the actual build/testing/deployment takes place in a Jenkins pipeline. Each stage in a Jenkins pipeline represents a step in the build process. 

The specific steps here are ensuring that reports comply with Content Security Policy, and the workspace is clean before starting the build.

Depends on the environment variable `BRANCH_START_WTH`.

```groovy
stage('Init') {
  when {expression {env.BRANCH_NAME.startsWith("${env.BRANCH_START_WTH}")}}
  steps {
       enableContentSecurityPolicyForReport()
       deleteDir()
      // cleanWs()
 }
```

- **`when {expression {env.BRANCH_NAME.startsWith("${env.BRANCH_START_WTH}")}}`**:
    - The **`when`** directive allows the execution of the stage to be conditional. In this case, the stage will only execute if the name of the branch being built starts with the value of the environment variable **`BRANCH_START_WTH`**. **`env.BRANCH_NAME`** is an in-built environment variable in Jenkins which gives the name of the branch being built.
- **`steps { ... }`**: The **`steps`** directive is used to define a series of one or more steps to be executed in a stage.
    - **`enableContentSecurityPolicyForReport()`**: This function call enables the Content Security Policy (CSP) for the report. CSP is a security measure that helps prevent attacks such as Cross-Site Scripting (XSS) and data injection attacks. It's not a standard Jenkins function and might be a custom function defined somewhere else in your Jenkins configuration or pipeline script.

```groovy
def enableContentSecurityPolicyForReport(){
    script {System.setProperty("hudson.model.DirectoryBrowserSupport.CSP", "default-src 'self'; style-src 'self' 'unsafe-inline';");}
}
```

*CSP is a computer security standard introduced to prevent Cross-Site Scripting (XSS) attacks, clickjacking, and other code injection attacks resulting from the execution of malicious content in the trusted web page context.*

The function **`enableContentSecurityPolicyForReport()`** uses a script to set a system property of Jenkins which controls the CSP. Here is what each part of the CSP means:

- **`default-src 'self';`**: This restricts all content to come from the site's own origin (same origin). This includes JavaScript, CSS, images, media, frames, fonts, etc. This is a fallback setting that other fetch directives (like **`style-src`**) use.
    
- **`style-src 'self' 'unsafe-inline';`**: This allows styles (CSS) to be loaded from the site's own origin, and it also allows inline styles. The keyword **`'unsafe-inline'`** allows the use of inline resources, like style attribute in HTML tags (e.g., **`<div style="color: blue;">`**).

In a nutshell, this function configures the CSP to allow the loading of resources from the same origin and the use of inline CSS styles, providing a layer of protection against certain types of attacks.

**Note** : Modifying CSP policies can have security implications and should only be done if you fully understand the consequences and have assessed the associated risks.

- **`deleteDir()`**: This is a Jenkins Pipeline step (provided by the Pipeline: Basic Steps plugin) that deletes the current directory in a workspace. This is typically used to clean the workspace before the build starts, to ensure that the build starts from a clean state.
- **`// cleanWs()`**: This is another method to clean up the workspace. It's commented out in this case, so it won't be executed. The **`cleanWs()`** step deletes the workspace to free up space on the agent. It can be a more aggressive cleanup compared to **`deleteDir()`**, and it's usually used when you want to ensure that the workspace is entirely clean.

**2. Checkout**

**`stage('Checkout') { ... }`** checks out or fetches the latest source code from the version control system or Source Control Management (SCM), such as Git or Subversion defined in the Jenkins job configuration.

Depends on the environment variable `BRANCH_START_WTH`.

```groovy
stage('Checkout') {
  when {expression {env.BRANCH_NAME.startsWith("${env.BRANCH_START_WTH}")}}
  steps { println("BRANCH_NAME : " + env.BRANCH_NAME)
      checkout scm
  }
}
```

- **`when {expression {env.BRANCH_NAME.startsWith("${env.BRANCH_START_WTH}")}}`**: This is a conditional statement that ensures the stage is only executed if the current branch name starts with the value of the environment variable **`BRANCH_START_WTH`**.
- **`steps { ... }`**: This block contains the steps that are executed during this stage of the pipeline.
- **`println("BRANCH_NAME : " + env.BRANCH_NAME)`**: This is a print statement that outputs the name of the branch being built to the console. This can be useful for logging and debugging purposes.
- **`checkout scm`**: This is a command that checks out the source code from the SCM configured for this Jenkins job. The source code is checked out into the current workspace, ready for the Jenkins job to build and test it.

**Note** : `This stage is usually one of the first in a Jenkins pipeline and is crucial for ensuring that the latest code is always used for the build and test processes.`

**3. Restore**

**`stage('Restore') { ... }`** retrieves the .NET project's dependencies using dotnet restore. It ensure all the required dependencies are correctly restored before the build process. Without this stage, your project may not build correctly or would likely fail.if it depends on certain NuGet packages that are not yet installed.

Depends on the environment variable `BRANCH_START_WTH`.

stage('Clean') {
when {expression {env.BRANCH_NAME.startsWith("${env.BRANCH_START_WTH}")}}
steps {cleanSolution()}
}

```groovy
stage('Restore') {
  when {expression {env.BRANCH_NAME.startsWith("${env.BRANCH_START_WTH}")}}
  steps {restoreSolutionDependencies()}
}
```

- **`when {expression {env.BRANCH_NAME.startsWith("${env.BRANCH_START_WTH}")}}`**: This is a conditional statement that ensures the stage is only executed if the current branch name starts with the value of the environment variable **`BRANCH_START_WTH`**.
- **`steps { ... }`**: This block contains the steps that are executed during this stage of the pipeline.
- **`restoreSolutionDependencies()`**: This is a function call that most likely encapsulates the command **`dotnet restore`**. The **`dotnet restore`** command is used in .NET Core projects to restore the dependencies of a project. It restores the necessary NuGet packages for your project by reading the project file (for example, **`.csproj`** for C# projects). This function is probably defined elsewhere in your Jenkins configuration or pipeline script.

```groovy
def restoreSolutionDependencies() {
	dotnetRestore project: "${params.PRJ_SLN_NAME}.sln", sdk: "dotnet${PRJ_TARGETED_FRAMEWORK}", verbosity: 'n'

	// sh """#!/bin/bash
	// ${DOTNET_CLI_HOME}//dotnet restore ${params.PRJ_SLN_NAME}.sln
	// """
}
```

- **`dotnetRestore project: "${params.PRJ_SLN_NAME}.sln", sdk: "dotnet${PRJ_TARGETED_FRAMEWORK}", verbosity: 'n'`**: This is a Jenkins pipeline step provided by the .NET SDK plugin. It performs the same function as the **`dotnet restore`** command, which is to restore the NuGet package dependencies for the project. The function takes three parameters:
    - **`project`**: The path to the project or solution file to execute. Here it is using a parameter **`PRJ_SLN_NAME`** to get the name of the solution file.
    - **`sdk`**: The .NET SDK version to use. Here it is concatenating the string "dotnet" with the value of the environment variable **`PRJ_TARGETED_FRAMEWORK`**.
    - **`verbosity`**: The amount of detail to display in the output. Here it is set to 'n', which stands for 'Normal'.
- **`// sh """#!/bin/bash`**: This is commented out, but if uncommented, it would run a shell script on Unix-based systems. The shell script runs the **`dotnet restore`** command for the solution file named by the **`PRJ_SLN_NAME`** parameter.

**4. Clean**

**`stage('Clean') { ... }`** purges the build solution of any build artifacts (like DLLs and EXEs) from the previous build and ensures that you're working with a clean workspace before starting a new build. Depends on the environment variable `BRANCH_START_WTH`.

```groovy
stage('Clean') {
  when {expression {env.BRANCH_NAME.startsWith("${env.BRANCH_START_WTH}")}}
  steps {cleanSolution()}
}
```

- **`when {expression {env.BRANCH_NAME.startsWith("${env.BRANCH_START_WTH}")}}`**: This is a conditional statement that ensures the stage is only executed if the current branch name starts with the value of the environment variable **`BRANCH_START_WTH`**.
- **`steps { ... }`**: This block contains the steps that are executed during this stage of the pipeline.
- **`cleanSolution()`**: This is a function call that most likely encapsulates the command **`dotnet clean`**. The **`dotnet clean`** command is used in .NET Core projects to clean the output of the prior build. This function is probably defined elsewhere in your Jenkins configuration or pipeline script.

```groovy
def cleanSolution() {
	dotnetClean project: "${params.PRJ_SLN_NAME}.sln", sdk: "dotnet${PRJ_TARGETED_FRAMEWORK}", verbosity: 'n'

    // sh """#!/bin/bash
    // ${DOTNET_CLI_HOME}\\dotnet clean ${WORKSPACE}\\${params.PRJ_SLN_NAME}.sln
    // """
}
```

- **`dotnetClean project: "${params.PRJ_SLN_NAME}.sln", sdk: "dotnet${PRJ_TARGETED_FRAMEWORK}", verbosity: 'n'`**: This is a Jenkins pipeline step provided by the .NET SDK plugin. It performs the same function as the **`dotnet clean`** command, which is to clean the build outputs. The function takes three parameters:
    - **`project`**: The path to the project or solution file to execute. Here it is using a parameter **`PRJ_SLN_NAME`** to get the name of the solution file.
    - **`sdk`**: The .NET SDK version to use. Here it is concatenating the string "dotnet" with the value of the environment variable **`PRJ_TARGETED_FRAMEWORK`**.
    - **`verbosity`**: The amount of detail to display in the output. Here it is set to 'n', which stands for 'Normal'.
- **`// sh """#!/bin/bash`**: This is commented out, but if uncommented, it would run a shell script on Unix-based systems. The shell script runs the **`dotnet clean`** command for the solution file named by the **`PRJ_SLN_NAME`** parameter.

**Note** : `This stage is crucial in .NET projects to ensure all the old build artifacts are removed before the new build process. Without this stage, your new build may be contaminated with the outputs from the old build.`

**5. Build**

**`stage('Build') { ... }`** compiles the source code into executable code. Without this stage, you wouldn't have a program to run or deploy**.**

Depends on the environment variable `BRANCH_START_WTH`.

```groovy
stage('Build') {
  when {expression {env.BRANCH_NAME.startsWith("${env.BRANCH_START_WTH}")}}
  steps {buildSolution()}
}
```

- **`when {expression {env.BRANCH_NAME.startsWith("${env.BRANCH_START_WTH}")}}`**: This is a conditional statement that ensures the stage is only executed if the current branch name starts with the value of the environment variable **`BRANCH_START_WTH`**.
- **`steps { ... }`**: This block contains the steps that are executed during this stage of the pipeline.
- **`buildSolution()`**: This is a function call that most likely encapsulates the command **`dotnet build`**. The **`dotnet build`** command is used in .NET Core projects to compile the project and its dependencies into a set of binaries. This function is probably defined elsewhere in your Jenkins configuration or pipeline script.

```groovy
def buildSolution() {
	dotnetBuild configuration: 'Release', project: "${params.PRJ_SOLUTION_NAME}.sln", sdk: "dotnet${PRJ_TARGETED_FRAMEWORK}", verbosity: 'n', versionSuffix:"${env.BUILD_NUMBER}"
	// dotnetBuild configuration: 'Release', project: "${params.PRJ_SOLUTION_NAME}.sln", sdk: "dotnet${PRJ_TARGETED_FRAMEWORK}", verbosity: 'n', versionSuffix:"${env.BUILD_NUMBER}" version:'1.0.0'
}
```

- **`dotnetBuild configuration: 'Release', project: "${params.PRJ_SOLUTION_NAME}.sln", sdk: "dotnet${PRJ_TARGETED_FRAMEWORK}", verbosity: 'n', versionSuffix:"${env.BUILD_NUMBER}"`**: This is a Jenkins pipeline step provided by the .NET SDK plugin. It performs the same function as the **`dotnet build`** command, which is to build the project. The function takes five parameters:
    - **`configuration`**: The configuration to use for building the project. This is set to 'Release', which typically means the build is optimized and ready for deployment.
    - **`project`**: The path to the project or solution file to execute. Here it is using a parameter **`PRJ_SOLUTION_NAME`** to get the name of the solution file.
    - **`sdk`**: The .NET SDK version to use. Here it is concatenating the string "dotnet" with the value of the environment variable **`PRJ_TARGETED_FRAMEWORK`**.
    - **`verbosity`**: The amount of detail to display in the output. Here it is set to 'n', which stands for 'Normal'.
    - **`versionSuffix`**: The version suffix to use for the build. Here it uses the Jenkins build number as the version suffix, which can be useful to differentiate between different builds.
- **`// dotnetBuild configuration: 'Release', project: "${params.PRJ_SOLUTION_NAME}.sln", sdk: "dotnet${PRJ_TARGETED_FRAMEWORK}", verbosity: 'n', versionSuffix:"${env.BUILD_NUMBER}" version:'1.0.0'`**: This is commented out, but if uncommented, it would be a similar command to the previous one but with an additional **`version`** parameter set to '1.0.0'.

**6. Test: Unit Test**

**`stage('Unit Test') { ... }`** ensures that all unit tests pass and that the code coverage is acceptable before the build is considered successful. This helps to maintain the quality of the code base and catch any issues early in the development process.

Depending on the parameter `IS_TEST_UNIT_ENABLED`  and the environment variable `BRANCH_START_WTH`, this stage runs the unit tests for the .NET Project

```groovy
stage('Test: Unit Test') {
  when {expression {params.IS_TEST_UNIT_ENABLED && env.BRANCH_NAME.startsWith("${env.BRANCH_START_WTH}")}}
  steps {doUnitTest("Tests/TestResults")}
}
```

```groovy
def doUnitTest(dirPath)
{
    if(!dirPath.isEmpty()){
		dir("${dirPath}") {deleteDir()}
		unitTestAndCodeCoverage()
		setBeforeDeployStatus()
	}
}
```

```groovy
def unitTestAndCodeCoverage() {
	sleep(time: 3, unit: "SECONDS")
	// Adding package JUnitTestLogger --version 1.1.0
	sh returnStatus: true, script:"""#!/bin/bash ${DOTNET_CLI_HOME}/dotnet add ${WORKSPACE}/Tests/Tests.csproj package JUnitTestLogger --version 1.1.0
    """
	// Adding package JUnitTestLogger --version 1.1.0
	sh returnStatus: true, script:"""#!/bin/bash ${DOTNET_CLI_HOME}/dotnet add ${WORKSPACE}/Tests/Tests.csproj package JUnitTestLogger --version 1.1.0
    """
	 // dotnet add package coverlet.collector
	// Code Coverage + logger TestResults.xml
	dotnetTest blame: true, collect: 'Code Coverage', configuration: 'Release', continueOnError: true, logger: 'junit;LogFileName=TestResults.xml', project: 'Tests.csproj', sdk: "dotnet${PRJ_TARGETED_FRAMEWORK}", unstableIfErrors: true, verbosity: 'n', workDirectory: 'Tests'
	// sh returnStatus: true, script:'${DOTNET_CLI_HOME}//dotnet test --logger:"junit;LogFileName=TestResults.xml" --configuration Release --collect "Code Coverage"'
	// analyzecodeCoverage()
	installAndUseReportgeneratorGlobalTool()
}
def installAndUseReportgeneratorGlobalTool() {
	// sh returnStatus: true, script:"""#!/bin/bash
  // dotnet tool install -g dotnet-reportgenerator-globaltool -v n
  // """
	sh returnStatus: true, script:"""#!/bin/bash
    dotnet tool install dotnet-reportgenerator-globaltool --tool-path $JENKINS_HOME/.dotnet/tools -v n
    """
	// Show 'tools' dir content
    sh returnStatus: true, script:"""#!/bin/bash
    ls $JENKINS_HOME/.dotnet/tools
    """
	//
	sleep(time: 4, unit: "SECONDS")
	// // generated TestResults.trx & coverage report
	// powershell returnStatus: true, script: '''
	// $currtentDirPath = Get-Location | Select -expand Path
	// ECHO "currtentDirPath : $currtentDirPath"
	// $testFolderName = "$ENV:PRJ_CSPROJ_NAME.Tests"
	// .\\__config__\\scripts\\ExecuteCodeCoverageInsidePipeline.ps1 -testFolderName $testFolderName
	// '''
	// Code Coverage + logger TestResults.xml
	dotnetTest blame: true, collect: 'XPlat Code Coverage', configuration: 'Release', continueOnError: false, logger: 'trx;LogFileName=TestResults.trx', project: 'Tests.csproj', sdk: "dotnet${PRJ_TARGETED_FRAMEWORK}", unstableIfErrors: true, verbosity: 'n', workDirectory: 'Tests'

	// sh returnStatus: true, script:'${DOTNET_CLI_HOME}//dotnet test --logger:"trx;LogFileName=TestResults.trx" --configuration Release --collect "XPlat Code Coverage"'

	generateCdeCoverageAndHistoryWithReportgenerator()
}
def generateCdeCoverageAndHistoryWithReportgenerator() {
	sh """#!/bin/bash
    dotnet reportgenerator -reports:"TestResults/*/coverage.cobertura.xml" -targetdir:"Coveragereport" -reporttypes:"Html;Cobertura" -historydir:CoverageHistory
    """
}
def setBeforeDeployStatus() {
    script {
        IS_DEPLOY_ENABLED = currentBuild.currentResult.toBoolean()
    }
}
def analyzeCodeCoverage() {
    // // sh returnStatus: true, script: """CodeCoverage analyze /output:TestResults\\TestResults.coverage TestResults\\TestResults.xml"""
    sleep(time: 2, unit: "SECONDS")
}
```

1. **`when {expression {params.IS_TEST_UNIT_ENABLED && env.BRANCH_NAME.startsWith("${env.BRANCH_START_WTH}")}}`**: This conditional statement ensures the stage is only executed if the **`IS_TEST_UNIT_ENABLED`** parameter is set to true and if the current branch name starts with the value of the environment variable **`BRANCH_START_WTH`**.
2. **`steps {doUnitTest("Tests/TestResults")}`**: This block contains the steps that are executed during this stage of the pipeline. It calls the function **`doUnitTest()`** with the directory "Tests/TestResults" as the argument.

The **`doUnitTest(dirPath)`** function performs the following actions:

- It checks if the **`dirPath`** (which is "Tests/TestResults") is not empty.
- If not empty, it changes the current directory to **`dirPath`** and deletes it. This is done to ensure a fresh start for the new test run.
- It then calls the **`unitTestAndCodeCoverage()`** function, which performs the actual unit testing and code coverage collection.

The **`unitTestAndCodeCoverage()`** function performs the following actions:

- It adds the **`JUnitTestLogger`** package to the project twice. This package allows the .NET test runner to output test results in the JUnit XML format, which can be consumed by many CI servers, like Jenkins.
- It runs the **`dotnet test`** command with several arguments:
    - **`blame: true`** captures details for each test run in a 'blame' file.
    - **`collect: 'Code Coverage'`** collects code coverage results.
    - **`configuration: 'Release'`** specifies to use the 'Release' configuration for the test.
    - **`continueOnError: true`** specifies to continue running tests even if one fails.
    - **`logger: 'junit;LogFileName=TestResults.xml'`** specifies to log the test results in a file named 'TestResults.xml'.
    - **`project: 'Tests.csproj'`** specifies the project file to test.
    - **`sdk: "dotnet${PRJ_TARGETED_FRAMEWORK}"`** specifies the .NET SDK version to use.
    - **`unstableIfErrors: true`** marks the build as unstable if there are test errors.
    - **`verbosity: 'n'`** specifies the verbosity level of the command.
    - **`workDirectory: 'Tests'`** specifies the working directory for the command.
- It then calls the **`installAndUseReportgeneratorGlobalTool()`** function.

The **`installAndUseReportgeneratorGlobalTool()`** function does the following:

- It installs the **`dotnet-reportgenerator-globaltool`**, which is a tool that can generate an HTML report that shows the code coverage of the project.
- It lists the contents of the 'tools' directory.
- It waits for 4 seconds.
- It runs the **`dotnet test`** command again, but this time collecting 'XPlat Code Coverage' and logging the results in a 'TestResults.trx' file.
- It calls the **`generateCdeCoverageAndHistoryWithReportgenerator()`** function.

The **`generateCdeCoverageAndHistoryWithReportgenerator()`** function runs the **`dotnet reportgenerator`** command to generate a code coverage report in HTML format and a 'Cobertura' XML file. The report uses the 'coverage.cobertura.xml' files that were generated by the **`dotnet test`** command.

The **`setBeforeDeployStatus()`** function is used to set the value of the **`IS_DEPLOY_ENABLED`** variable based on the current build result.

Let's break down the function:

- **`script { ... }`**: This block allows for arbitrary Groovy code to be executed, which is the scripting language that Jenkins pipeline DSL is based on.
- **`IS_DEPLOY_ENABLED = currentBuild.currentResult.toBoolean()`**: This line of code sets the value of the **`IS_DEPLOY_ENABLED`** variable.

The **`currentBuild.currentResult`** property holds the result of the current build (e.g., 'SUCCESS', 'FAILURE'). The **`toBoolean()`** method is then called on this value. In Groovy, calling **`toBoolean()`** on a string will return **`true`** unless the string is null or empty.

So, this line is setting **`IS_DEPLOY_ENABLED`** to **`true`** as long as **`currentBuild.currentResult`** is not null or an empty string. This means that **`IS_DEPLOY_ENABLED`** would be set to **`true`** even if the current build result is 'FAILURE', which might not be the intended behavior.

If you want to set **`IS_DEPLOY_ENABLED`** to **`true`** only when the build is successful, you might want to compare **`currentBuild.currentResult`** with 'SUCCESS' instead:

```groovy
IS_DEPLOY_ENABLED = (currentBuild.currentResult == 'SUCCESS')
```

The **`analyzeCodeCoverage()`** function is commented out, but if uncommented, it would wait for 2 seconds.

**7. Test: Integration Test**

**`stage('Integration Test') { ... }`** 

Similarly, based on the `IS_TEST_INTEGRATION_ENABLED` parameter, this stage runs the integration tests for the .NET project. (Need to be customized)

```groovy
stage('Test: Integration Test') {
  when {expression {params.IS_TEST_INTEGRATION_ENABLED && env.BRANCH_NAME.startsWith("${env.BRANCH_START_WTH}")}}
  steps {doIntegrationTest("Integrations")}
}
```

TO BE COMPLETED / CUSTOMIZED

```groovy
def doIntegrationTest(dirPath)
{
    if(!dirPath.isEmpty()){
		println("!!! TO DO ??")
	}
}
```

- **`doIntegrationTest(dirPath) { ... }`**: This defines a function named **`doIntegrationTest`** that takes one argument, **`dirPath`**.
- **`if(!dirPath.isEmpty()){ ... }`**: This is a conditional statement that checks if **`dirPath`** is not an empty string. If **`dirPath`** is not empty, then the code inside the curly braces **`{}`** is executed.

**Note** : *`Currently, this function does not perform any integration testing. Instead, it simply prints a message indicating that this is something that needs to be implemented.`*

*`If you want to perform integration testing as part of your Jenkins pipeline, you'll need to replace **println("!!! TO DO ??")** with the actual commands or function calls that run your integration tests. These commands will depend on how integration testing is set up for your specific project.`*

**8. Checkmarx Scan**

**`stage('**Checkmarx Scan**') { ... }**` is used to perform a Checkmarx scan on your code if `IS_CHECKMARX_SCAN_ENABLED` parameter is set to true, this stage runs a Checkmarx scan on the codebase. Checkmarx is a static code analysis tool that helps detect vulnerabilities in the source code.

```groovy
stage('Checkmarx Scan') {
  when {expression {params.IS_CHECKMARX_SCAN_ENABLED && env.BRANCH_NAME.startsWith("${env.BRANCH_START_WTH}")}}
  steps {checkmarxScan()}
  post {
    always {checkmarxPostAlways()}
  }
}
```
- The **`checkmarxScan()`** orchestrates the Checkmarx scan process. It first prints a start message to the console, then calls the **`runCheckmarxScan()`** function to perform the scan, and finally prints an end message to the console.
- The **`post`** block contains operations that are performed after the steps in the stage have completed. The **`always`** directive means that the operations inside it will always be executed regardless of the outcome of the stage.
- The **`checkmarxPostAlways()`** function is executed after the Checkmarx scan regardless of whether the scan succeeded or failed. This function could perform cleanup operations or analyze and report the results of the scan. This function is likely used in a **`post`** block after the **`steps`** in a stage in the pipeline. It sets the **`IS_DEPLOY_ENABLED`** variable based on the result of the current build.

```groovy
def checkmarxScan(){
    echo('--- START: checkmarx scan')
    runCheckmarxScan()
    echo('END: checkmarx scan')
}
def checkmarxPostAlways()
{
    script {
        IS_DEPLOY_ENABLED = currentBuild.currentResult.toBoolean()
    }
}
def runCheckmarxScan(){
    // Cf. https://checkmarx.com/resource/documents/en/34965-8158-setting-up-scans-in-jenkins.html
    // Use Jenkins Pipeline Syntax or Jenkin Snippet Generator .
    script {
        step([$class: 'CxScanBuilder',
				avoidDuplicateProjectScans: true,
				comment: '',
				configAsCode: true,
				credentialsId: '',
				customFields: '',
                dependencyScanConfig: [dependencyScanExcludeFolders: '',
					dependencyScanPatterns: '', enableScaResolver: 'MANIFEST', fsaVariables: '',
					osaArchiveIncludePatterns: '*.zip, *.war, *.ear, *.tgz', pathToScaResolver: '',
					sastCredentialsId: '', scaAccessControlUrl: 'https://platform.checkmarx.net',
					scaConfigFile: '', scaCredentialsId: '', scaEnvVariables: '', scaResolverAddParameters: '',
					scaSASTProjectFullPath: '', scaSASTProjectID: '', scaSastServerUrl: '',
					scaServerUrl: 'https://api-sca.checkmarx.net', scaTeamPath: '', scaTenant: '',
					scaWebAppUrl: 'https://sca.checkmarx.net'],
					enableProjectPolicyEnforcement: false,
					exclusionsSetting: 'global',
					fullScanCycle: 10,
					sastEnabled: true,
					sourceEncoding: '1',
					//exclusionsSetting: 'job',
					failBuildOnNewResults: true,
					failBuildOnNewSeverity: 'HIGH',
					groupId: '4',
					generatePdfReport: true,
					excludeFolders: "${env.CHECKMARX_DIR_TO_EXCLUDE}",
					filterPattern: "${env.CHECKMARX_ITEMS_TO_EXCLUDE}",
					password: '${CHECKMARX_CREDS_PSW}',
					projectName: "${CHECKMARX_PRJ_NAME}",
					username: "${CHECKMARX_CREDS_USR}",
					serverUrl: "${CHECKMARX_URL}",
					vulnerabilityThresholdResult: 'FAILURE',
					highThreshold: "${CHECKMARX_HIGHTHRESHOLD}",
					mediumThreshold: "${CHECKMARX_MEDIUMTHRESHOLD}",
					lowThreshold: "${CHECKMARX_LOWTHRESHOLD}",
					waitForResultsEnabled: true])
    }

    sendCheckmarkReportsToEmailDir()
    // publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, reportDir: "__config__\\email", reportFiles: 'Report_CxSAST.html', reportName: 'CheckmarxReport', reportTitles: ''])

    println("currentBuild.currentResult.toBoolean()" + currentBuild.currentResult.toBoolean())
}

def sendCheckmarkReportsToEmailDir()
{
    def copyHtmlReportCmd="cp Checkmarx/Reports/Report_CxSAST.html __config__/email/Report_CxSAST.html"
    def copyPdfReportCmd="cp Checkmarx/Reports/*.pdf __config__/email/"
    sh returnStatus: true, script:"$copyHtmlReportCmd;$copyPdfReportCmd"
}
```

**`runCheckmarxScan()`**: This function performs the actual Checkmarx scan. It uses the **`step()`** function to execute the **`CxScanBuilder`** class, which is part of the Checkmarx Jenkins plugin. The parameters inside **`step()`** configure the scan, such as the source encoding, the project name, the username and password to use, the server URL, and vulnerability thresholds. It also sets the **`waitForResultsEnabled`** parameter to **`true`**, meaning it will wait for the scan to finish before proceeding. After running the scan, it calls the **`sendCheckmarkReportsToEmailDir()`** function.

**4. `sendCheckmarkReportsToEmailDir()`**: This function copies the Checkmarx reports (both HTML and PDF) to a directory (**`__config__/email/`**). This might be used to store the reports so they can be emailed or archived later.

Remember to replace the placeholders with your actual Checkmarx server URL, project name, username, password, and other relevant information.

Also, ensure that you have the required permissions and that the Checkmarx Jenkins plugin is installed and properly configured in your Jenkins server. For more information, you can refer to the **[Checkmarx plugin documentation](https://checkmarx.atlassian.net/wiki/spaces/KC/pages/222232891/CxSAST+Jenkins+Plugin+User+Guide+9.0)**.

**9. SonarQube Scan**

`stage('SonarQube Scan')`  is used in Jenkins pipelines to run static code analysis using SonarQube. This stage is only executed when the **`IS_SONAR_SCAN_ENABLED`** parameter is true and the branch name starts with a specified prefix with the pipeline environment variable `RANCH_START_WTH`.

```groovy
stage('SonarQube Scan') {
  when {expression {params.IS_SONAR_SCAN_ENABLED && env.BRANCH_NAME.startsWith("${env.BRANCH_START_WTH}")}}
  steps {sonarQubeScan()}
}
```

**`sonarQubeScan()`** contains the main logic for running the SonarQube scan. **`runSonarScan()`** is invoked within this function, which starts the SonarQube environment and runs the SonarQube scanner.

**`runSonarScan()`**:

1. **`withSonarQubeEnv`**: This Jenkins Pipeline step sets up environment variables for SonarQube and makes them available to the steps inside the block.
2. **`SonarQube.Scanner.MSBuild.exe begin`**: This command is used to begin the SonarQube scan. The **`/k`** option is used to specify the SonarQube project key.
3. **`MSBuild.exe /t:Rebuild`**: This command rebuilds the solution or project with MSBuild.
4. **`SonarQube.Scanner.MSBuild.exe end`**: This command is used to finish the SonarQube scan and send the analysis results to the SonarQube server.
5. The **`SONAR_SCAN_STATUS`** variable captures the exit status of the SonarQube scan. If the exit status is 0, it means the scan was successful. The boolean **`IS_SONAR_SCAN_STATUS`** is set to true if the scan was successful, and false otherwise.

**Note :** `Before running this stage, ensure that SonarQube is properly set up in your environment, the SonarQube Jenkins plugin is installed, and you have correctly configured the **SonarQubeServer** installation in your Jenkins settings. Also, replace the placeholders (like **${env.SONAR_HOME}**) with your actual values.`

```groovy
def sonarQubeScan(){
  echo('--- START: sonarQube scan')
	runSonarScan()
  echo('END: sonarQube scan')
}
def runSonarScan(){
	//Cf. https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-msbuild/
	//    https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/
    script {
        withSonarQubeEnv(installationName:'SonarQubeServer')
        {
		sh "${env.SONAR_HOME}\\SonarQube.Scanner.MSBuild.exe begin /k:{env.SONAR_PRJ_KEY}"
		sh 'MSBuild.exe /t:Rebuild'
		sh "${env.SONAR_HOME}\\SonarQube.Scanner.MSBuild.exe end"
		// env.PATH = "$PATH:/home/jenkins/.dotnet"
		// env.PATH = "$PATH:/home/jenkins/.dotnet/tools"
		// sh "dotnet ${scannerHome}/SonarScanner.MSBuild.dll begin /k:\"sap_fehlertracking-db_AYIBDL0ZccYnbt4oP4o9\""
		// sh "dotnet build fehlertracking.sln"
		// sh "dotnet ${scannerHome}/SonarScanner.MSBuild.dll end"
		// def SONAR_SCAN_STATUS = sh returnStatus: true, script:"""
		// ${env.SONAR_HOME}/bin/sonar-scanner"""
		def SONAR_SCAN_STATUS = sh returnStatus: true, script:"""${env.SONAR_HOME}\\SonarQube.Scanner.MSBuild.exe end"""
		IS_SONAR_SCAN_STATUS = SONAR_SCAN_STATUS==0?true:false
        }
    }
}
```

**10. Quality Gate**

`stage('Quality Gate')` is used in Jenkins pipelines to wait for the SonarQube Quality Gate status and determine if the deployment should proceed based on the Quality Gate result. This stage is only executed when both the **`IS_SONar_SCAN_ENABLED`** and **`IS_SONAR_QG_ENABLED`** parameters are true and the branch name starts with a specified prefix  set by the pipeline environment variable `BRANCH_START_WTH.`

```groovy
stage('Quality Gate') {
  when {expression {params.IS_SONAR_SCAN_ENABLED && params.IS_SONAR_QG_ENABLED && env.BRANCH_NAME.startsWith("${env.BRANCH_START_WTH}")}}
  steps {checkAndEnableDeployOnPassedQualityGate()}
}
```

The **`checkAndEnableDeployOnPassedQualityGate`** function contains the main logic for checking the Quality Gate status and setting the **`IS_DEPLOY_ENABLED`** flag.

**`checkAndEnableDeployOnPassedQualityGate`**:

1. **`sonarQualityGate`**: This function waits for the Quality Gate status from SonarQube. The **`waitForQualityGate`** function is a Jenkins pipeline step provided by the SonarQube Jenkins plugin that blocks the pipeline until the SonarQube analysis report has been processed and the Quality Gate status is available. The function takes a **`webhookSecretId`**, which corresponds to a Jenkins credential id used to validate the incoming webhook calls from SonarQube.
2. If the Quality Gate status is not 'OK', the pipeline is aborted. If the status is 'OK', the **`IS_SONAR_QG_STATUS`** flag is set to true.
3. **`checkAndEnableDeployOnPassedQualityGate`**: In this function, if the **`IS_SONAR_QG_STATUS`** flag is true, the **`IS_DEPLOY_ENABLED`** flag is set to true, allowing the deployment to proceed. If **`IS_SONAR_QG_STATUS`** is false, **`IS_DEPLOY_ENABLED`** is set to false, preventing the deployment.

Before running this stage, ensure that SonarQube is properly set up in your environment, the SonarQube Jenkins plugin is installed, and you have correctly configured the **`sonar-to-jenkins-webhook`** secret in your Jenkins settings. Also, replace the placeholders (like **`${env.BRANCH_START_WTH}`**) with your actual values.

```groovy
def sonarQualityGate(){
    echo('--- START: sonar Q.G.')
    script {
        timeout(time: 1, unit: 'HOURS') {
            def qg = waitForQualityGate(webhookSecretId: 'sonar-to-jenkins-webhook')
            println "qg.status:${qg.status}"
            if (qg.status != 'OK') {println ("Pipeline aborted due to quality gate failure: ${qg.status}")}
            else{IS_SONAR_QG_STATUS = true}
        }
    }
    echo('END: sonar Q.G.')
}
def checkAndEnableDeployOnPassedQualityGate(){
    sonarQualityGate()
    script {
        if("${IS_SONAR_QG_STATUS}".toBoolean() != false){IS_DEPLOY_ENABLED = true}
        else {IS_DEPLOY_ENABLED = false}
    }
}
```

**11. Publish**

`stage('Publish')`  publishes the .NET solution and code coverage reports.

```groovy
stage('Publish') {
  when {expression {env.BRANCH_NAME.startsWith("${env.BRANCH_START_WTH}")}}
  steps {publishSolutionHtmlAndCodeCoverage()}
}
```

**`publishSolutionHtmlAndCodeCoverage()`** is used to publish the .NET solution and generate test reports in the [pipeline.](http://pipeline.it) It **contains**  **`publishSolution()`  and`publishXunitAndHtmlReport().`**

```groovy
def publishSolutionHtmlAndCodeCoverage() {
	publishSolution()
	publishXunitAndHtmlReport()
}

def publishSolution() {
	// sh """#!/bin/bash
        // dotnet publish {params.PRJ_CSPROJ_NAME}/${params.PRJ_CSPROJ_NAME}.csproj --configuration Release
	// """
	dotnetPublish configuration: 'Release', project: "${params.PRJ_CSPROJ_NAME}.csproj", sdk: "dotnet${PRJ_TARGETED_FRAMEWORK}", selfContained: false, workDirectory: "${params.PRJ_CSPROJ_NAME}"
}
def publishXunitAndHtmlReport() {

	dir("Tests") {
	  // cobertura classCoverageTargets: '0, 0, 0', coberturaReportFile: 'TestResults/coverage.cobertura.xml', conditionalCoverageTargets: '70, 0, 0', enableNewApi: true, fileCoverageTargets: '0, 0, 0', lineCoverageTargets: '80, 0, 0', maxNumberOfBuilds: 0, methodCoverageTargets: '80, 0, 0', packageCoverageTargets: '0, 0, 0', sourceEncoding: 'ASCII', zoomCoverageChart: true

	  cobertura autoUpdateHealth: false, autoUpdateStability: false, coberturaReportFile: 'Coveragereport\\Cobertura.xml', conditionalCoverageTargets: '70, 0, 0', enableNewApi: true, failUnhealthy: false, failUnstable: false, fileCoverageTargets: '0, 0, 0', lineCoverageTargets: '80, 0, 0', maxNumberOfBuilds: 0, methodCoverageTargets: '80, 0, 0', onlyStable: false, sourceEncoding: 'ASCII', zoomCoverageChart: true

	  // step([$class: 'CoberturaPublisher', coberturaReportFile: 'TestResults\\coverage.cobertura.xml';])
	  step([$class: 'CoberturaPublisher', coberturaReportFile: 'Coveragereport\\Cobertura.xml'])

	  xunit([MSTest(excludesPattern: ';', pattern: 'TestResults/TestResults.trx', stopProcessingIfError: true)])
	}
	sleep(time: 5, unit: "SECONDS")
}
```

1. **`publishSolution()`**: This function uses the **`dotnetPublish`** method to publish the .NET solution. The **`dotnet publish`** command packs the application and its dependencies into a folder for deployment to a hosting system. It's being configured to use the 'Release' configuration and the project to publish is specified by **`${params.PRJ_CSPROJ_NAME}.csproj`**.
2. **`publishXunitAndHtmlReport()`**: This function is responsible for publishing test and coverage reports. It's using the **`cobertura`** and **`xunit`** plugins to publish the reports.
    - **`cobertura`**: Cobertura is a free Java tool that calculates the percentage of code accessed by tests. Here, it is configured to read the coverage report from **`Coveragereport\\Cobertura.xml`**.
    - **`xunit`**: The xUnit plugin is used for publishing test results. Here, it is configured to find test reports in the **`TestResults/TestResults.trx`** file.
3. **`publishSolutionHtmlAndCodeCoverage()`**: This function simply calls the **`publishSolution`** and **`publishXunitAndHtmlReport`** functions. This function is likely called in a stage of the Jenkins pipeline to publish the solution and the test reports.

**Note** : `Remember to install the required plugins (Cobertura and xUnit) in Jenkins before using these functions. Also, replace the placeholders (like **${params.PRJ_CSPROJ_NAME}**) with your actual values, or set these parameters in your Jenkins setup.`

**12. Deploy**

`stage('Deploy')`deploys the .NET application to a specified environment if `IS_DEPLOY_ENABLED` parameter is set to `true`,  and depending on the pipeline environment variable `BRANCH_START_WTH.`

```groovy
stage('Deploy') {
  //WITH CIFS
  // when {expression {IS_DEPLOY_ENABLED && (currentBuild.result == null || currentBuild.result == 'SUCCESS')&& env.BRANCH_NAME.startsWith("${env.BRANCH_START_WTH}")}}
  when {expression {IS_DEPLOY_ENABLED && env.BRANCH_NAME.startsWith("${env.BRANCH_START_WTH}")}}
  steps {
		// With CIFS
		deploy(false,env.DEPLOY_SERVER_NAME, env.DEPLOY_SERVER_DIR)
		// With MSBuild
		deployAppToEnvironment()
	  }
}
```

1. **`when {expression {IS_DEPLOY_ENABLED && env.BRANCH_NAME.startsWith("${env.BRANCH_START_WTH}")}}`**: This block checks if the deployment is enabled and if the branch name starts with a specific string. The **`IS_DEPLOY_ENABLED`** is likely a boolean parameter indicating whether or not to run the deployment. **`${env.BRANCH_START_WTH}`** would be an environment variable containing the prefix of the branch names that should trigger a deployment.
2. **`steps { ... }`**: This block contains the steps to be executed when the conditions in the **`when`** block are met.
    - **`deploy(false,env.DEPLOY_SERVER_NAME, env.DEPLOY_SERVER_DIR)`**: This line calls the **`deploy`** function (which should be defined elsewhere in your Jenkinsfile or in a Jenkins shared library) to deploy the application. It passes three arguments: **`false`**, the name of the deployment server, and the directory on the server where the application should be deployed. The **`false`** argument might be a flag indicating whether to use a certain deployment method or not, but without the definition of the **`deploy`** function, it's hard to say exactly what it does.

```groovy
def deploy(isRemoteDirASubDirToDeployTo, serverCfgName, serverDir){
        println("env.IS_DEPLOY_ENABLED: ${IS_DEPLOY_ENABLED}")
        echo "Deploying to ${DEPLOY_ENV} on " + serverDir + " for: ${DEPLOY_URL_TO_LAUNCH}"
	//
	sendMaintenanceAlertHTML()
	//
        deployToRemoteShare(isRemoteDirASubDirToDeployTo, serverCfgName, serverDir,false,"${env.DEPLOY_DOTNET_SOURCE}")
	//
        script {IS_DEPLOY_STATUS = currentBuild.result == "SUCCESS"}
}
def sendMaintenanceAlertHTML() {
	def maintenanceFile = "__config__/scripts/sendMaintenanceAlertHTML.html"
	deployToRemoteShare(isRemoteDirASubDirToDeployTo, serverCfgName, serverDir,true,maintenanceFile)
}
def deployToRemoteShare(isRemoteDirASubDirToDeployTo, serverCfgName, serverDir, isCleanRemote, srceFiles)
{
    cifsPublisher(
        publishers: [
            [configName: serverCfgName,
            transfers: [
                [cleanRemote: cleanRemote.toBoolean(),
                excludes: "${env.DEPLOY_FILES_TO_EXCLUDE}",
                flatten: false,
                makeEmptyDirs: false,
                noDefaultExcludes: false,
                patternSeparator: '[, ]+',
                remoteDirectory: isRemoteDirASubDirToDeployTo ? serverDir: "",
                remoteDirectorySDF: false,
                removePrefix: "${env.DEPLOY_CIFS_REMOVE_PREFIX}",
                sourceFiles: srceFiles]],
            usePromotionTimestamp: false,
            useWorkspaceInPromotion: false,
            verbose: false]])
}
```

The **`deploy()`** function in this context has two main tasks:

1. It sends a maintenance alert HTML. This alert could be used to inform users that the application is being updated and might be unavailable during the update.
2. It deploys the application to a remote server using the **`cifsPublisher`** function.

Here's a breakdown of the function:

- **`sendMaintenanceAlertHTML()`**: This function prepares a maintenance alert HTML file and deploys it to the remote server. The HTML file should be located at **`__config__/scripts/sendMaintenanceAlertHTML.html`** in your repository.
- **`deployToRemoteShare(isRemoteDirASubDirToDeployTo, serverCfgName, serverDir,false,"${env.DEPLOY_DOTNET_SOURCE}")`**: This function deploys the application to the remote server.

Let's go into more detail about **`deployToRemoteShare()`**:

- **`cifsPublisher`**: This function is used to copy files from your Jenkins workspace to a remote location using the CIFS (Common Internet File System) protocol, which allows the sharing of files and printers over a network.
- **`publishers`**: This block contains the configuration of what should be transferred to the remote location.
- **`configName`**: This is the name of the configuration to be used for the transfer. This should match a CIFS configuration that you've defined in your Jenkins setup.
- **`transfers`**: This block defines the details of the files to be transferred.
- **`cleanRemote`**: When set to **`true`**, this option will delete all files in the remote directory before the transfer.
- **`excludes`**: This specifies files that should be excluded from the transfer.
- **`remoteDirectory`**: This is the directory on the remote server where the files should be transferred.
- **`removePrefix`**: This is used to remove a prefix from the files being transferred.
- **`sourceFiles`**: This is the files or directories in the Jenkins workspace that should be transferred.

Make sure to replace placeholders (like **`${env.DEPLOY_DOTNET_SOURCE}`**) with your actual values. Otherwise, the deployment will fail. Additionally, ensure that you have the necessary permissions and network access to deploy to your target environment.

**`deployAppToEnvironment()`**: This line calls the **`deployAppToEnvironment`** function to deploy the application using MSBuild, as explained in the previous message.

This stage assumes that the **`deploy`** function is defined somewhere else in your Jenkinsfile or in a Jenkins shared library. If it isn't, you'll need to implement it, or remove the call to **`deploy`** if it's not needed.

Also, make sure to replace the placeholders (like **`${env.DEPLOY_SERVER_NAME}`** and **`${env.DEPLOY_SERVER_DIR}`**) with your actual values, or set these environment variables in your Jenkins setup. Otherwise, the deployment will fail.

```groovy
def deployAppToEnvironment() {

	script {IS_DEPLOY_STATUS = currentBuild.result == "SUCCESS"}
	withCredentials([usernamePassword(credentialsId: "${ENV:DEPLOY_CREDENTIALS_ID}", passwordVariable: 'passVar', usernameVariable: 'userVar')])
	{
		println("Deploying with publishProfile")
		println("currentBuild.currentResult.toBoolean()" + currentBuild.currentResult.toBoolean())
		// sh returnStatus: true, script: "dotnet msbuild ${params.PRJ_CSPROJ_NAME}/${params.PRJ_CSPROJ_NAME}.csproj /p:DeployOnBuild=true /p:PublishProfile=${ENV:MSBUILD_PUBLISH_PROFILE_ENV}.pubxml /p:Password=%passVar% /p:VisualStudioVersion=${ENV:VisualStudioVersion}"
		// sh returnStatus: true, script: "dotnet msbuild ${params.PRJ_CSPROJ_NAME}/${params.PRJ_CSPROJ_NAME}.csproj /p:DeployOnBuild=true /p:PublishProfile=${ENV:MSBUILD_PUBLISH_PROFILE_ENV}.pubxml /p:Password=%passVar% "
		sh returnStatus: true, script: "dotnet msbuild ${params.PRJ_CSPROJ_NAME}/${params.PRJ_CSPROJ_NAME}.csproj /p:DeployOnBuild=true /p:PublishProfile=${ENV:MSBUILD_PUBLISH_PROFILE_ENV}.pubxml /p:Username=%userVar% /p:Password=%passVar%"
		// dotnetBuild configuration: 'Release', project: "${params.PRJ_SOLUTION_NAME}.sln", sdk: "dotnet${PRJ_TARGETED_FRAMEWORK}", verbosity: 'n', versionSuffix:"${env.BUILD_NUMBER}"
		// dotnetBuild configuration: 'Release', project: "${params.PRJ_SOLUTION_NAME}.sln", sdk: "dotnet${PRJ_TARGETED_FRAMEWORK}", verbosity: 'n', versionSuffix:"${env.BUILD_NUMBER}" version:'1.0.0'
	}

	sleep(time: 8, unit: "SECONDS")
}
```

The function **`deployAppToEnvironment()`** is used to deploy your .NET Core application to a specified environment. Let's break it down:

1. **`script {IS_DEPLOY_STATUS = currentBuild.result == "SUCCESS"}`**: This line checks if the current build was successful. If it was, **`IS_DEPLOY_STATUS`** is set to **`true`**.
2. **`withCredentials([usernamePassword(credentialsId: "${ENV:DEPLOY_CREDENTIALS_ID}", passwordVariable: 'passVar', usernameVariable: 'userVar')])`**: This block retrieves the deployment credentials from Jenkins' credentials store. The **`credentialsId`** should match the ID of the credentials you've stored in Jenkins. The credentials are then stored in **`passVar`** and **`userVar`** variables for later use.
3. Inside the **`withCredentials`** block, a shell script is run using the **`sh`** function. This script uses the **`dotnet msbuild`** command to build the project and deploy it on build (**`/p:DeployOnBuild=true`**). It uses a publish profile (**`/p:PublishProfile=${ENV:MSBUILD_PUBLISH_PROFILE_ENV}.pubxml`**), which is an XML file that contains MSBuild properties to control the build process. The username and password for deployment are supplied via the **`/p:Username=%userVar%`** and **`/p:Password=%passVar%`** parameters.
4. **`sleep(time: 8, unit: "SECONDS")`**: This line simply pauses the execution of the script for 8 seconds. This is often used to allow time for a previous operation to complete, like waiting for the deployment to finish.

Please replace placeholders (like **`${ENV:DEPLOY_CREDENTIALS_ID}`** and **`${ENV:MSBUILD_PUBLISH_PROFILE_ENV}`**) with your actual values. The deployment will fail if these placeholders are not replaced with valid values.

Lastly, ensure that you have the necessary permissions to deploy to your target environment, and that your Jenkins server has the .NET Core SDK installed and properly configured.

1. The Post Block

It contains operations that are performed after the steps in the stage have completed. The **`always`** directive means that the operations inside it will always be executed regardless of the outcome of the stage.

The **`post`** section in a Jenkins pipeline is used to specify steps that are run after all the stages have completed. It supports multiple conditions, such as **`always`**, **`success`**, **`failure`**, **`unstable`**, **`changed`**, **`fixed`**, **`regression`**, **`aborted`**, and **`notBuilt`**.

Here's a breakdown of your **`post`** section:

1. **`always { ... }`**: This block of code will run after every build regardless of the build's status (whether it has succeeded or failed). In your case, **`cleanWs`** function is used to clean up the workspace after the build. This is useful for ensuring that your workspace is ready for the next build. The parameters passed to **`cleanWs`** function provide further details:
    - **`deleteDirs: true`**: This tells Jenkins to delete directories as part of the cleanup.
    - **`disableDeferredWipeout: true`**: This disables deferred wipeout. Deferred wipeout is a technique used to clean up the workspace more quickly, but it may not be compatible with all systems.
    - **`notFailBuild: true`**: This prevents the build from failing if the workspace cleanup fails.
    - **`patterns: ...`**: This specifies patterns for files to include or exclude during the cleanup. In your case, the **`.gitignore`** file is included, and the **`.propsfile`** is excluded.
2. **`success { ... }`** and **`failure { ... }`**: These blocks run only when the build has succeeded or failed, respectively. In both cases, you're calling **`notifyByMail()`**, which likely sends a notification email about the build's status. The **`notifyByMail()`** function should be defined elsewhere in your Jenkinsfile or in an imported library.

Remember to replace placeholders (like the email addresses in **`notifyByMail()`**) with your actual information to have the notification emails sent to the correct recipients.

```groovy
def chooseEmailTemplate(String emailTemplatePath) {
	if(IS_SONAR_SCAN_ENABLED.toBoolean() && IS_SONAR_ENTERPRISE_EDITION.toBoolean() && IS_CHECKMARX_SCAN_ENABLED.toBoolean()){
		emailTemplatePath="__config__/email/jenkins_checkmarx_sonarqube_full_deploy-email.template.html"
	}
	if(IS_SONAR_SCAN_ENABLED.toBoolean() && IS_SONAR_ENTERPRISE_EDITION.toBoolean()){
		emailTemplatePath="__config__/email/jenkins_checkmarx_sonarqube_full_deploy-email.template.html"
	}
	else{
		emailTemplatePath="__config__/email/jenkins_checkmarx_deploy-email.template.html"
	}
	return emailTemplatePath
}
def emailTemplate(params) {
    script{
	// def emailTemplatePath= chooseEmailTemplate();
        // def emailTemplatePath="__config__/email/jenkins_checkmarx_sonarqube_full_deploy-email.templatee.html"
        // def emailTemplatePath="__config__/email/jenkins_sonarqube_deploy.template.html"
        def emailTemplatePath="__config__/email/jenkins_checkmarx_deploy-email.template.html"
        def resultat= ""
        if (fileExists("$emailTemplatePath")){
            def fileContents = readFile "${emailTemplatePath}"
            def engine = new StreamingTemplateEngine()
            resultat = engine.createTemplate(fileContents).make(params).toString()
        }
        else{resultat = "Erreur : modèle d'email: ${emailTemplatePath} non trouvé"}
        return resultat
    }
}
def notifyByMail() {
    if(env.BRANCH_NAME.startsWith("${env.BRANCH_START_WTH}")) {
        println("----BEGIN EMAILING")
        def sonarServerUrl = "${env.SONAR_SERVER_URL}"
        def sonarQubePrjBoard = "${env.SONAR_DASHBOARD_URL}"
        def sonarProjectKey = sonarQubePrjBoard.split('=')[1];
        def icon = "✅"
        def sonarEnterpriseReportLink ="$sonarServerUrl"+"/project/extension/securityreport/securityreport?branch=${env.BRANCH_NAME}&id="+"$sonarProjectKey"
        def sonarQubeSecurityBoard = env.IS_SONAR_ENTERPRISE_EDITION.toBoolean()?sonarEnterpriseReportLink:sonarQubePrjBoard
        script {
            SONAR_SECURITY_DASHBOARD_URL="${sonarQubeSecurityBoard}"
            IS_BUILD_STATUS = currentBuild.currentResult.toBoolean() && IS_SONAR_QG_STATUS.toBoolean()
        }
        if(IS_BUILD_STATUS != true) {icon = "❌"}
        def emailStatus = IS_BUILD_STATUS.toBoolean() ? 'Succès' : 'Echec'
        def jobname = "${env.JOB_NAME}".replace("%", "/").replace("%", "/")
        def emailSubject  = "${icon} [ ${jobname}] [Build #${env.BUILD_NUMBER}]- ${emailStatus}! "
		def emailAttachmentsPattern ='__config__\\email\\*.png,__config__\\email\\*.pdf,__config__\\email\\Report_CxSAST.html'
        def mailRecipients = "${EMAIL_RECIPIENTS_LIST}"
        def mailToReplyTo = "${EMAIL_REPLYTO_LIST}"
        def isLogAttached = IS_BUILD_STATUS.toBoolean() ? false : true
        def newParams = [
            "emailSubject": "${emailSubject}",
            "BUILD_URL": "${BUILD_URL}",
            "BUILD_NUMBER": "${BUILD_NUMBER}",
            "JOB_NAME": "${JOB_NAME}",
            "SONAR_DASHBOARD_URL": "${SONAR_DASHBOARD_URL}",
            "SONAR_SECURITY_DASHBOARD_URL": "${SONAR_SECURITY_DASHBOARD_URL}",
            "IS_BUILD_STATUS": "${IS_BUILD_STATUS}",
            "IS_SONAR_SCAN_STATUS": "${IS_SONAR_SCAN_STATUS}",
            "IS_SONAR_QG_STATUS": "${IS_SONAR_QG_STATUS}",
            "IS_DEPLOY_STATUS": "${IS_DEPLOY_STATUS}",
            "IS_SONAR_ENTERPRISE_EDITION": "${params.IS_SONAR_ENTERPRISE_EDITION}",
            "IS_SONAR_SCAN_ENABLED": "${params.IS_SONAR_SCAN_ENABLED}",
            "IS_SONAR_QG_ENABLED": "${params.IS_SONAR_QG_ENABLED}",
            "IS_DEPLOY_ENABLED": "${IS_DEPLOY_ENABLED}",
            "IS_CHECKMARX_SCAN_ENABLED": "${params.IS_CHECKMARX_SCAN_ENABLED}",
            "DEPLOY_URL_TO_LAUNCH": "${params.DEPLOY_URL_TO_LAUNCH}",
            "DEPLOY_ENV" : "${params.DEPLOY_ENV}"
        ]

        debug_Println(newParams)

        def emailBody = emailTemplate(newParams.minus(["emailSubject": "${emailSubject}"]));
        env.ForEmailPlugin = env.WORKSPACE
        emailext(mimeType: 'text/html',
        replyTo: mailToReplyTo,
        subject: emailSubject,
        to: mailRecipients,
        body: emailBody,
        attachLog: isLogAttached,
        attachmentsPattern: emailAttachmentsPattern)

        createJenkinsViewEmailResultReport(emailBody,emailSubject,isLogAttached)
        println("--END: EMAILING")
    }
}

@NonCPS
def debug_Println(list) {println(list)}

def createJenkinsViewEmailResultReport(String emailBody,String emailSubject, isLogAttached){
    sh returnStatus: true, script:"""rm -rf __config__/email/ViewEmailResult.html"""
    def attachFilesDir = "__config__/email/"
    def subjectAndFiles= """<section><h2>${emailSubject}</h2><p>📎${getDirAllFileName(attachFilesDir, isLogAttached)}</p><br/></section>"""
    def emailFullBody= subjectAndFiles + emailBody
    writeFile file: "__config__/email/ViewEmailResult.html", text: emailFullBody, encoding: "UTF-8"
    publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, reportDir: "__config__\\email", reportFiles: 'ViewEmailResult.html', reportName: 'ViewEmailResult', reportTitles: ''])
}
def getFilesByExtension(ext, filesDir){return sh(returnStdout: true, script: "find ${filesDir} -type f -name *.${ext}").trim().replaceAll("${filesDir}/","")}
def getDirAllFileName(attachFilesDir, isLogAttached)
{
    script{
        //def files ="[${getFilesByExtension("docx",attachFilesDir)}, ${getFilesByExtension("xlsx",attachFilesDir)}]"
        def files ="[${getFilesByExtension("pdf",attachFilesDir)}, ${getFilesByExtension("html",attachFilesDir)}]"
        files = isLogAttached.toBoolean()?files.replaceAll("]",", build.log]"):files

        // files.remove(files.indexOf("jenkins_checkmarx_deploy-email.template.html"))
        return files.toString()
    }
}
def createZipArtifacts (){

  println("--DEBUT: createZipArtifacts")

	print("--Creating mutiple zip files in  $WORKSPACE/ArchiveZip");
	script {
	print("--!!!!!Targeted framework : NET 6.0!!!!");
	zip archive: true, dir: "${params.PRJ_CSPROJ_NAME}/bin/Release/net5.0/publish", exclude: ';', glob: ';', overwrite: true, zipFile: "ArchiveZip/${params.PRJ_CSPROJ_NAME}_To_deploy.zip"
	zip archive: true, dir: "Tests/Coveragereport", exclude: ';', glob: ';', overwrite: true,zipFile: "ArchiveZip/${params.PRJ_CSPROJ_NAME}_CoverageReportHTML.zip"
	zip archive: true, dir: "Tests/CoverageHistory", exclude: ';', glob: ';',overwrite: true, zipFile: "ArchiveZip/${params.PRJ_CSPROJ_NAME}_CoverageHistory.zip"
	zip archive: true, dir: "Tests/TestResults", exclude: ';', glob: ';',overwrite: true, zipFile: "ArchiveZip/${params.PRJ_CSPROJ_NAME}_TestResults.zip"
	}

	print("--Cleanning not required $WORKSPACE/ArchiveZip");
    dir("$WORKSPACE/ArchiveZip"){deleteDir()}

	print("--Cleaning old JunitFiles in : Tests/generatedJUnitFiles");
	dir("Tests/generatedJUnitFiles"){deleteDir()}

	print("--Cleanning not required $WORKSPACE/Tests@tmp");
	dir("$WORKSPACE/Tests@tmp") {deleteDir()}

  println("--END: createZipArtifacts")
}
def zipWebsite(archiveName,filesToExclude) {
   zip archive: true, dir: "${WORKSPACE}", exclude: filesToExclude, glob: '', overwrite: true, zipFile: archiveName
}
```

These functions in your Jenkinsfile are mainly for sending notification emails and creating zip archives of your project artifacts. Here's a breakdown:
**1. `chooseEmailTemplate(String emailTemplatePath)`:** This function chooses the email template based on the project configurations. If SonarQube and Checkmarx scans are enabled, it will select a specific template. If only SonarQube is enabled, it will select a different template. Otherwise, it will select the default template. The selected template path is returned.
**2. `emailTemplate(params)`:** This function reads the selected email template, applies the provided parameters to it, and returns the resulting string. If the template file doesn't exist, it returns an error message. It uses the Groovy's **`StreamingTemplateEngine`** for template processing.
**3. `notifyByMail()`:** This function sends an email notification about the build result. It constructs the email subject, body, and recipient list, and then calls the **`emailext`** function (from the Email Extension Plugin) to send the email. It also creates a Jenkins view for the email result using **`createJenkinsViewEmailResultReport()`** function.
**4. `createJenkinsViewEmailResultReport(String emailBody,String emailSubject, isLogAttached)`:** This function creates an HTML view of the email that was sent. It writes the content to a HTML file and then publishes it using the **`publishHTML`** function.
**5. `getDirAllFileName(attachFilesDir, isLogAttached)`:** This function returns all file names in the given directory that have specific extensions.
**6. `createZipArtifacts()`:** This function creates zip archives of your project's artifacts. It uses the **`zip`** function to create the zip files. After creating the zip files, it cleans up unneeded directories.
**7. `zipWebsite(archiveName,filesToExclude)`:** This function creates a zip archive of your entire workspace, excluding the files that match the **`filesToExclude`** pattern.

**Note** : `These functions depend on your Jenkins plugins and the structure of your project. If you encounter any issues, ensure that you have the necessary plugins installed and check your project structure.`

**SUMMARY**

 The main components of your Jenkins pipeline based on the stages we've discussed:
1. **Init**: This stage prepares the workspace for the pipeline. It sets up content security policies and cleans the workspace directory.
2. **Checkout**: This stage checks out the source code from the source control manager (SCM) to the Jenkins workspace.
3. **Restore**: This stage restores the dependencies for your .NET solution.
4. **Clean**: This stage cleans the build artifacts from the previous build.
5. **Build**: This stage builds your .NET solution.
6. **Test: Unit Test**: This stage runs the unit tests for your .NET solution.
7. **Test: Integration Test**: This stage runs the integration tests for your .NET solution.
8. **Checkmarx Scan**: This stage performs a Checkmarx scan on your code to identify any security vulnerabilities.
9. **SonarQube Scan**: This stage performs a SonarQube scan on your code to measure and analyze the code quality.
10. **Quality Gate**: This stage checks the results from the SonarQube scan and determines if the code is ready for deployment based on the quality gate conditions.
11. **Publish**: This stage publishes the .NET solution and code coverage reports.
12. **Deploy**: This stage deploys your .NET application to a specified environment.

Each stage of the pipeline is designed to automate a particular aspect of the build, test, and deployment process, making it easier to maintain a consistent and reliable software delivery process.

**CONCLUSION**

This Jenkinsfile provides a robust framework for automating the build-test-deploy lifecycle of a .NET application. By using Jenkins and this Jenkinsfile, you can ensure that every change to your codebase is built, tested, scanned for quality and security issues, and, if all checks pass, deployed to your environment.

This Jenkinsfile is highly customizable, meaning you can add more stages, integrate with other tools, or adjust the conditions for each stage to better suit your needs. However, be sure to replace placeholders with actual values before using this Jenkinsfile in your environment.

Automation of your CI/CD pipeline not only increases efficiency but also ensures consistency and reliability. With Jenkins and this Jenkinsfile, you're well on your way to achieving these benefits for your .NET applications. 

Check the Jenkinsfile source code at : [Jenkinsfile](https://github.com/Bernardinhouessou/Articles/blob/c90aff9e3de7e1955280034236956fb320a0ae53/Medium/CI%26CD_NETCore_Bitbucket_Jenkins_Sonar_Checkmarx/Assets/Code/Jenkinsfile)

Happy coding!
