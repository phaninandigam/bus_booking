@Library('my-shared-library@main') _  // Correct syntax

pipeline {
    agent { label 'slave-java' }

    environment {
        JAVA_HOME = '/usr/lib/jvm/java-17-openjdk-amd64'
        MAVEN_HOME = '/usr/share/maven'
        PATH = "${JAVA_HOME}/bin:${MAVEN_HOME}/bin:${env.PATH}"
        SONAR_TOKEN = credentials('SONAR_TOKEN') // Store token in Jenkins credentials
    }

    stages {
        stage('Checkout Code') {
            steps {
               script {
                     testpipeline.testcheckout1()
                 }
            }
        }

        stage('Set up Java 17') {
            steps {
                setupJava()
            }
        }

        stage('Set up Maven') {
            steps {
                setupMaven()
            }
        }

        stage('Build with Maven') {
            steps {
                 script {
                     checkout_build_run.testbuild()
                 }
            }
        }

        stage('SonarCloud Analysis') {
            steps {
                withSonarQubeEnv('SonarCloud') {
                    sh '''
                    mvn sonar:sonar \
                      -Dsonar.projectKey=phaninandigam_project1 \
                      -Dsonar.organization=phaninandigam \
                      -Dsonar.host.url=https://sonarcloud.io \
                      -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    def qg = waitForQualityGate()
                    if (qg.status != 'OK') {
                        error "Pipeline aborted due to quality gate failure: ${qg.status}"
                    }
                }
            }
        }

        stage('Upload Artifact') {
            steps {
                uploadArtifact('target/bus-booking-app-1.0-SNAPSHOT.jar')
            }
        }

        stage('Run Application') {
            steps {
                script {
                     checkout_build_run.testrun()
                 }
            }
        }

        stage('Validate App is Running') {
            steps {
                validateApp()
            }
        }

        stage('Gracefully Stop Spring Boot App') {
            steps {
                stopApplication()
            }
        }
    }

    post {
        always {
            cleanup()
        }
    }
}
