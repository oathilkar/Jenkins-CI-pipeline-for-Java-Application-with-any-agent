# Jenkins-CI-pipeline-for-Java-Application-with-any-agent

Creating a detailed declarative Jenkins CI pipeline for a Java application involves defining stages and steps in a Jenkinsfile to automate the build, test, and deployment process. In this example, we'll create a pipeline that integrates with Git for version control, Maven for build management, SonarQube for static code analysis, Selenium for automated testing, and Nexus for artifact repository management.

### Prerequisites:

1. Jenkins server installed and running.
2. Git repository containing the Java application code.
3. Maven installed on the Jenkins server.
4. SonarQube server set up and running.
5. Selenium WebDriver and browser drivers installed on the Jenkins server.
6. Nexus repository manager set up and running.

### Steps:

#### 1. Configure Jenkins:

- Install necessary plugins: Git plugin, Maven Integration plugin, SonarQube Scanner plugin, Nexus Artifact Uploader plugin.
- Configure Jenkins global settings:
  - Configure Git credentials.
  - Configure Maven installation in Jenkins.
  - Configure SonarQube server connection.

#### 2. Create a Jenkins Pipeline Job:

- Create a new Jenkins job and select "Pipeline" as the job type.

#### 3. Define Jenkinsfile:

- Create a `Jenkinsfile` in the root directory of your Java project.

#### Jenkinsfile:

```groovy
pipeline {
    agent any
    
    environment {
        MAVEN_HOME = tool 'Maven' // Maven installation defined in Jenkins configuration
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh "${env.MAVEN_HOME}/bin/mvn clean package -DskipTests" // Build the project with Maven
            }
        }
        
        stage('Static Code Analysis') {
            steps {
                withSonarQubeEnv('SonarQubeServer') {
                    sh "${env.MAVEN_HOME}/bin/mvn sonar:sonar" // Run SonarQube analysis
                }
            }
        }
        
        stage('Unit Tests') {
            steps {
                sh "${env.MAVEN_HOME}/bin/mvn test" // Run unit tests
            }
        }
        
        stage('Integration Tests') {
            steps {
                // Run Selenium tests (assuming test scripts are in the 'test' directory)
                sh "${env.MAVEN_HOME}/bin/mvn -Dtest=TestSuiteIT test"
            }
        }
        
        stage('Deploy to Nexus') {
            steps {
                nexusArtifactUploader([
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: 'NEXUS_URL', // Replace with Nexus URL
                    groupId: 'com.example',
                    version: '1.0-SNAPSHOT',
                    repository: 'maven-snapshots',
                    credentialsId: 'NEXUS_CREDENTIALS_ID', // Configure Nexus credentials in Jenkins
                    artifacts: [
                        [artifactId: 'your-artifact-id', file: 'target/your-artifact-id-1.0-SNAPSHOT.jar', type: 'jar']
                    ]
                ])
            }
        }
    }
}
```

Replace `NEXUS_URL` and `NEXUS_CREDENTIALS_ID` with your Nexus server URL and Jenkins credentials ID respectively.

#### 4. Configure SonarQube:

- Set up a SonarQube project for your Java application.
- Generate SonarQube token for authentication in Jenkins.

#### 5. Run the Pipeline:

- Run the Jenkins pipeline job to trigger the build, test, and deployment process.

### Notes:

- Ensure proper error handling and notifications in the pipeline.
- Customize stages and steps based on your project requirements.
- Securely manage credentials and sensitive information in Jenkins.

This pipeline automates the build, test, and deployment process for a Java application using Jenkins, Git, Maven, SonarQube, Selenium, and Nexus. Adjustments may be needed based on your specific project setup and requirements.
This Jenkins pipeline script sets up a continuous integration flow for a Java application, integrating Git, Maven, SonarQube, Nexus, and Selenium for comprehensive testing and artifact management.
