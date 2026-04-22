@Library('noc-lib@main') _

pipeline {
    agent any

    parameters {
        choice(name: 'ENV', choices: ['dev', 'staging', 'prod'], description: 'Select Env')
    }

    stages {

        stage('Validation') {
            steps {
                script {
                    echo "Starting Validation..."
                    def obj = new com.devops.DeployManager(this)
                    obj.validate(params.ENV)
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo "Starting Deployment..."
                    def obj = new com.devops.DeployManager(this)
                    obj.deploy(params.ENV)
                }
            }
        }
    }

    post {
        success {
            echo "Deployment SUCCESS 🚀"
        }
        failure {
            script {
                echo "Deployment FAILED → Rolling back..."
                def obj = new com.devops.DeployManager(this)
                obj.rollback(params.ENV)
            }
        }
    }
}
