pipeline {
    agent any

    tools {
        maven "MVN_HOME"
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

        stage("Clone Code") {
            steps {
                git branch: 'master',
                url: 'https://github.com/waseema761/spring3-mvc-maven-xml-hello-world-1.git'
            }
        }

        stage("Build") {
            steps {
                sh "mvn clean package -DskipTests"
            }
        }

        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh """
                       mvn sonar:sonar \
                       -Dsonar.projectKey=spring-app \
                       -Dsonar.host.url=${SONAR_HOST_URL}
                    """
                }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Publish to Nexus") {
            steps {
                script {
                    def pom = readMavenPom file: "pom.xml"
                    def filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    def artifactPath = filesByGlob[0].path

                    if (fileExists(artifactPath)) {

                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: BUILD_NUMBER,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [
                                    artifactId: pom.artifactId,
                                    classifier: '',
                                    file: artifactPath,
                                    type: pom.packaging
                                ],
                                [
                                    artifactId: pom.artifactId,
                                    classifier: '',
                                    file: "pom.xml",
                                    type: "pom"
                                ]
                            ]
                        )
                    } else {
                        error "Artifact not found!"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Build Successful and Artifact Uploaded to Nexus hiring-app repository"
        }
        failure {
            echo "Build Failed"
        }
    }
}
