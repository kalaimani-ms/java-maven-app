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
        stage ('buildimage') {
            steps {
                script {
                    echo 'building the docker images'
                    sh 'docker images'
                    withCredentials([usernamePassword(credentialsId: 'kalaimanims-Dockerhub',usernameVariable : 'USER',passwordVariable: 'PASS')]) {
                        sh "docker build -t kalaimanims/mavenapp:${IMAGE_NAME} ."
                        sh "echo $PASS | docker login -u $USER --password-stdin"
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
                     sh 'envsubst < /var/jenkins_home/workspace/kubernetes/deployment.yaml | kubectl delete -f - '
                    sh 'envsubst < /var/jenkins_home/workspace/kubernetes/deployment.yaml | kubectl apply -f - '
                    sh 'envsubst < /var/jenkins_home/workspace/kubernetes/service.yaml | kubectl apply -f - '
                    sh 'kubectl get all'

                }
            }
        }
        stage('commit version update') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github-kalai', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                        // git config here for the first time running
                        sh 'git config --global user.email "jenkins@example.com"'
                        sh 'git config --global user.name "jenkins"'

                        sh "git remote set-url origin https://$USER:$PASS@github.com/kalaimani-ms/java-maven-app.git"
                        sh 'git add .'
                        sh 'git commit -m "ci: version bump"'
                        sh 'git push origin HEAD:eks-cluster-deploy'
                    }
                    
                }
            }
        }
    }
}