def gv

pipeline {
    agent any
    tools {
        maven 'Maven'
    }
       stages {
        stage("init") {
            steps {
                script {
                    gv = load 'multiscript.groovy'
                }
                
            }
        }
        stage("test") {
            steps {
                script {
                    gv.Testapp()
                }
            }
        }
        stage("build") {
            when {
                expression{
                    BRANCH_NAME == 'master'
                }
            }
            steps {
              script  {
                gv.Buildjar()
                }
            }
        }
        stage("buildimage") {
            when {
                expression{
                    BRANCH_NAME == 'master'
                }
            }
            steps {
              script  {
                gv.Buildimage()
                }
            }
        }
        
        stage("deploy") {
            when {
                expression{
                    BRANCH_NAME == 'master'
                }
            }
            steps{
                script {
                    gv.Deployapp()
                }
            }
        }
       }
}
