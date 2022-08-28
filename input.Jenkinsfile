def gv

pipeline {
    agent any
    parameters{
        booleanParam(name:'executeTests',defaultValue:True,description:'')
    }   
       stages {
        stage("init") {
            steps {
                script {
                    gv = load 'script.groovy'
                }
                
            }
        }
        stage("build") {
            steps {
              script  {
                gv.Buildapp()
                }
            }
        }
        stage("test") {
            when {
                expression{
                    params.executeTests
                }
            }
            steps {
                script {
                    gv.Testapp()
                }
            }
        }
        stage("deploy") {
            input {
                messege "select the environment to deploy the application"
                ok "done"
                parameters {
                choice(name:'ENV',choices:['dev','staging','prod'],description:'')
                }
            }
            steps {
                script {
                    gv.Deployapp()
                    echo "the app is deploying to ${ENV}"
                }
            }
        }
       }
}
