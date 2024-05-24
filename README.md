# Jenkins-CI-pipeline-for-Java-Application-with-any-agent

Creating a detailed declarative Jenkins CI pipeline for a Java application involving Git, Maven, SonarQube, Selenium, and Nexus involves several stages to build, test, analyze, and deploy the application. Below is a comprehensive guide to setting up such a pipeline using Jenkins Declarative Pipeline syntax.

### Prerequisites

1. **Jenkins**: Ensure Jenkins is installed and running.
2. **Plugins**: Install the necessary plugins:
   - Git Plugin
   - Pipeline Plugin
   - Maven Plugin
   - SonarQube Scanner Plugin
   - Nexus Artifact Uploader Plugin
   - Selenium Plugin (if using Selenium Grid)

3. **Tools**:
   - Git installed on the Jenkins server.
   - Maven installed on the Jenkins server.
   - SonarQube server set up and accessible.
   - Nexus repository manager set up and accessible.
   - Selenium Grid (optional, for Selenium tests).

### Declarative Pipeline Script

Create a new Jenkins Pipeline job and choose "Pipeline script from SCM" as the pipeline definition. Specify your Git repository URL and configure the script path if necessary.

#### Jenkinsfile

```groovy
pipeline {
    agent any
    
    environment {
        MAVEN_HOME = tool name: 'Maven 3.8.1', type: 'maven'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                script {
                    def mvnHome = tool 'Maven 3.8.1'
                    withMaven(maven: mvnHome, mavenSettingsConfig: 'maven-settings') {
                        sh "mvn clean install"
                    }
                }
            }
        }
        
        stage('Static Code Analysis') {
            steps {
                script {
                    withSonarQubeEnv('SonarQubeServer') {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }
        
        stage('Unit Test') {
            steps {
                script {
                    def mvnHome = tool 'Maven 3.8.1'
                    withMaven(maven: mvnHome, mavenSettingsConfig: 'maven-settings') {
                        sh "mvn test"
                    }
                }
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Package and Publish') {
            steps {
                script {
                    def mvnHome = tool 'Maven 3.8.1'
                    withMaven(maven: mvnHome, mavenSettingsConfig: 'maven-settings') {
                        sh "mvn package"
                        // Upload artifact to Nexus
                        nexusArtifactUploader nexusInstanceId: 'nexus', nexusRepositoryId: 'maven-releases', protocol: 'http', credentialsId: 'nexus-credentials', artifacts: [[artifactId: 'your-project', file: '**/target/*.jar', type: 'jar']]
                    }
                }
            }
        }
        
        stage('Integration Test') {
            steps {
                // Run Selenium tests (Example: Using Selenium Grid)
                // You may need to configure the Selenium Grid URL accordingly
                selenium(remoteWebDriverAddress: 'http://selenium-grid:4444/wd/hub', capabilities: '{"browserName": "chrome"}') {
                    def mvnHome = tool 'Maven 3.8.1'
                    withMaven(maven: mvnHome, mavenSettingsConfig: 'maven-settings') {
                        sh "mvn verify"
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                // Deployment steps if applicable
                echo 'Deploying...'
            }
        }
    }
    
    post {
        success {
            echo 'Build and tests passed. Ready for deployment!'
        }
        failure {
            echo 'Build or tests failed. Please check the logs for more details.'
        }
    }
}
```

### Detailed Explanation:

1. **Agent**: Defines where the pipeline will execute. `any` indicates that the pipeline can execute on any available agent.

2. **Environment**: Specifies the Maven installation to use for the pipeline.

3. **Stages**:
   - **Checkout**: Checks out the source code from the Git repository.
   - **Build**: Builds the Java application using Maven.
   - **Static Code Analysis**: Executes SonarQube analysis using the SonarQube Scanner plugin.
   - **Unit Test**: Executes unit tests using Maven and collects test results.
   - **Package and Publish**: Packages the application and uploads the artifact to Nexus repository manager.
   - **Integration Test**: Executes integration tests, such as Selenium tests, using Maven and Selenium Grid.
   - **Deploy**: Placeholder for deployment steps.

4. **Post**:
   - **Success**: Executes if the pipeline runs successfully.
   - **Failure**: Executes if the pipeline fails.

### Jenkins Configuration

1. **Global Tools Configuration**:
   - Configure JDK, Maven, and any other tools under "Manage Jenkins" -> "Global Tool Configuration".

2. **Credentials**:
   - Add credentials for Git (if private repository), Nexus, and SonarQube under "Manage Jenkins" -> "Manage Credentials".

3. **Maven Settings**:
   - Configure Maven settings (`settings.xml`) under "Manage Jenkins" -> "Managed Files" -> "Add a new Config".

4. **SonarQube Configuration**:
   - Configure SonarQube server under "Manage Jenkins" -> "Configure System" -> "SonarQube servers".

5. **Nexus Artifact Uploader Configuration**:
   - Configure Nexus Artifact Uploader under "Manage Jenkins" -> "Configure System" -> "Nexus Artifact Uploader".

6. **Selenium Configuration (if using Selenium)**:
   - Configure Selenium Grid URL under "Manage Jenkins" -> "Configure System" -> "Selenium Grid".

### Notes:

- Replace placeholders (`your-project`, `maven-releases`, `nexus-credentials`, `SonarQubeServer`, etc.) with your actual values.
- Adjust the Selenium configuration (`selenium`) if you are not using Selenium Grid or need different capabilities.
- This example assumes a basic pipeline. Adjust stages and steps based on your specific requirements and project structure.
- Ensure your Jenkins server has the necessary permissions and configurations to access your Nexus, SonarQube, and Selenium services.

This Jenkins pipeline script sets up a continuous integration flow for a Java application, integrating Git, Maven, SonarQube, Nexus, and Selenium for comprehensive testing and artifact management.
