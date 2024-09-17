pipeline {
    agent { label 'dev' } // Specify the label of your Jenkins Slave. Modify 'dev' if you use a different label.

    environment {
        MAVEN_HOME = tool name: 'Maven' // Ensure 'Maven' matches the name in Jenkins tool configuration.
        SONARQUBE_SERVER = 'sonar' // Define the SonarQube server ID.
        SONAR_SCANNER_HOME = tool name: 'SonarScanner' // Ensure 'SonarScanner' matches the name in Jenkins tool configuration.
        ARTIFACTORY_SERVER = 'Artifactory' // Define Artifactory server ID.
        SONAR_TOKEN = 'squ_485c5e56e285bf09f85abbd83cb9ac6d4c73f329' // SonarQube token
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the source code from your repository
                git url: 'https://github.com/shashank1553/java-example.git', branch: 'master'
            }
        }

        stage('Build') {
            steps {
                // Build the Maven project
                sh "${MAVEN_HOME}/bin/mvn clean install"
            }
        }

        stage('Verify Build Output') {
            steps {
                // Verify if a WAR file has been successfully created
                sh 'ls -la target/'
                script {
                    def warFiles = findFiles(glob: 'target/*.war')
                    if (warFiles.length == 0) {
                        error "No WAR file found in target directory"
                    } else {
                        echo "WAR file found: ${warFiles[0].path}"
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // Run SonarQube analysis
                withSonarQubeEnv(SONARQUBE_SERVER) {
                    sh "${MAVEN_HOME}/bin/mvn sonar:sonar -Dsonar.login=${SONAR_TOKEN}"
                }
            }
        }

        stage('Deploy to Artifactory') {
            steps {
                script {
                    // Upload artifacts to JFrog Artifactory
                    rtUpload (
                        serverId: ARTIFACTORY_SERVER,
                        spec: '''{
                            "files": [{
                                "pattern": "target/*.war", // Upload any WAR file in the target directory
                                "target": "maven-releases/java-example/"
                            }]
                        }'''
                    )
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'artifactory-credentials', usernameVariable: 'ARTIFACTORY_USER', passwordVariable: 'ARTIFACTORY_PASS')]) {
                    // Deploy the WAR file to Apache Tomcat server using credentials from Jenkins store
                    sh """
                        war_file=\$(ls target/*.war)
                        curl -u ${ARTIFACTORY_USER}:${ARTIFACTORY_PASS} --upload-file \$war_file \
                        "http://3.87.224.227:8081/artifactory/maven-releases/java-example"
                    """
                }
            }
        }
    }

    post {
        always {
            cleanWs() // Clean the workspace after the build
        }
        success {
            echo "Build, Analysis, and Deployment succeeded!"
        }
        failure {
            echo "Build failed. Check the logs!"
        }
    }
}
