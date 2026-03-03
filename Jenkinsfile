pipeline {
    agent any

    tools {
        maven "maven"
    }

    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "localhost:8081"
        NEXUS_REPOSITORY = "hiring-app"
        NEXUS_CREDENTIAL_ID = "nexus-creds"
        SONAR_HOST_URL = "http://localhost:9000"
    }

    stages {

        stage("Checkout Code") {
            steps {
                git branch: 'master',
                url: 'https://github.com/Waseema761/spring3-mvc-maven-xml-hello-world-1.git'
            }
        }

        stage("Build Application") {
            steps {
                sh "mvn clean package -DskipTests"
            }
        }

        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh """
                        mvn sonar:sonar \
                        -Dsonar.projectKey=hiring-app \
                        -Dsonar.host.url=${SONAR_HOST_URL}
                    """
                }
            }
        }

        stage("Quality Gate Check") {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Publish Artifact to Nexus") {
    steps {
        script {

            def artifactPath = "target/ncodeit-hello-world-3.0.war"

            if (fileExists(artifactPath)) {

                nexusArtifactUploader(
                    nexusVersion: "nexus3",
                    protocol: "http",
                    nexusUrl: "localhost:8081",
                    groupId: "com.ncodeit",
                    version: BUILD_NUMBER,
                    repository: "hiring-app",
                    credentialsId: "nexus-creds",
                    artifacts: [
                        [
                            artifactId: "ncodeit-hello-world",
                            classifier: '',
                            file: artifactPath,
                            type: "war"
                        ]
                    ]
                )

                echo "Artifact Uploaded Successfully!"

            } else {
                error "WAR file not found!"
            }
        }
    }
}
    }

    post {
        success {
            echo "Build Successful. Artifact uploaded to Nexus (hiring-app)."
        }
        failure {
            echo "Build Failed. Check logs."
        }
    }
}
