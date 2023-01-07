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
                        -Dsonar.host.url=http://3.110.209.182:9000 \
                        -Dsonar.login=sqp_51bd22a2bfe626d528c15e54fb8018a814dab387' 
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
        stage ('deployapp to k8s') {
            steps {
                script {
                    echo 'deploying the java-maven-app to kubernetes cluster from jenkins'
                    sh 'envsubst < /var/jenkins_home/workspace/kubernetes/deployment.yaml | kubectl apply -f - '
                    sh 'envsubst < /var/jenkins_home/workspace/kubernetes/service.yaml | kubectl apply -f - '
                    sh 'kubectl get pod '
                    sh 'kubectl get all'
                }
            }
        }
    }
}
