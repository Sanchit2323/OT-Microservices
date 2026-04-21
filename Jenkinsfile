@Library('noc-lib@master') _

pipeline {
    agent any

    parameters {
        choice(name: 'ENV', choices: ['dev', 'staging', 'prod'], description: 'Select Env')
    }

    stages {

        stage('Validation') {
            steps {
                script {
                    def obj = new com.devops.DeployManager(this)
                    obj.validate(params.ENV)
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def obj = new com.devops.DeployManager(this)
                    obj.deploy(params.ENV)
                }
            }
        }
    }

    post {
        failure {
            script {
                def obj = new com.devops.DeployManager(this)
                obj.rollback(params.ENV)
            }
        }
    }
}
