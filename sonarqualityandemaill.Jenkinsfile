#!/usr/bin/env groovy
pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        APP_NAME= 'mavenapp'
    }
    stages {
        stage('incremental version') {
            steps {
                script {
                    echo 'incrementing the app version'
                    sh 'mvn build-helper:parse-version versions:set \
                    -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                    versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }    
            }
        }
        stage ('buidiljar') {
            steps {
                script {
                    echo 'building the application'
                    sh 'mvn clean package'
                }
            }
        }
        stage ('sonarqube analysis'){
            steps {
                script {withSonarQubeEnv(credentialsId: 'sonar') {
                    sh 'mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=maven-project \
                        -Dsonar.host.url=http://65.0.183.80:9000 \
                        -Dsonar.login=sqp_1e34feed7368c3252d81d0d496c8b253d707dc9a '
                    }
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script{
                    timeout(time: 1, unit: 'HOURS') {
                    def qg = waitForQualityGate()
                    if (qg.status == 'OK') {
                          emailext body: '''$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS:

                          Check console output at $BUILD_URL to view the results.''', recipientProviders: [developers()], subject: '$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS!', to: 'kalaimanicit@gmail.com,slbagavath@gmail.com'
                          error "Pipeline aborted due to quality gate failure: ${qg.status}"
                       }
                    }
                }   
            }
        }
        stage ('push image to Dockerhub') {
            steps {
                script {
                    echo 'building the docker images'
                    sh 'docker images'
                    withCredentials([usernamePassword(credentialsId: 'kalaimanims-Dockerhub',usernameVariable : 'USER',passwordVariable: 'PASS')]) {
                        sh "docker build -t kalaimanims/mavenapp:${IMAGE_NAME} ."
                        sh "echo $PASS | docker login -u $USER --password-stdin "
                        sh "docker push kalaimanims/mavenapp:${IMAGE_NAME}"
                        sh 'docker images'
                    }
                }
            }
        }
    }   
}
