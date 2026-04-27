@Library('noc-lib@main') _

pipeline {
    agent any

    environment {
        EMAIL_TO = "sanchitkumar0307@gmail.com"
        REPO_URL = "https://github.com/Sanchit2323/OT-Microservices.git"
    }

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

        always {
            emailext(
                to: "${EMAIL_TO}",
                subject: "Build ${currentBuild.currentResult}: ${env.JOB_NAME} #${env.BUILD_NUMBER} (${params.ENV})",
                mimeType: 'text/html',
                body: """
                <h2>Jenkins Deployment Report</h2>
                <p><b>Job:</b> ${env.JOB_NAME}</p>
                <p><b>Status:</b> ${currentBuild.currentResult}</p>
                <p><b>Environment:</b> ${params.ENV}</p>
                <p><b>Build:</b> #${env.BUILD_NUMBER}</p>
                <p><b>Duration:</b> ${currentBuild.durationString}</p>
                <p><a href="${env.BUILD_URL}console">View Logs</a></p>
                """
            )
        }

        success {
            script {
                def repo = sh(script: "git config --get remote.origin.url", returnStdout: true).trim()
                def branch = sh(script: "git rev-parse --abbrev-ref HEAD", returnStdout: true).trim()
                notifySlack(":white_check_mark: SUCCESS", repo, branch)
            }
        }

        failure {
            script {
                echo "Deployment FAILED → Rolling back..."
                def obj = new com.devops.DeployManager(this)
                obj.rollback(params.ENV)

                def repo = sh(script: "git config --get remote.origin.url", returnStdout: true).trim()
                def branch = sh(script: "git rev-parse --abbrev-ref HEAD", returnStdout: true).trim()
                notifySlack(":x: FAILURE", repo, branch)
            }
        }

        unstable {
            script {
                def repo = sh(script: "git config --get remote.origin.url", returnStdout: true).trim()
                def branch = sh(script: "git rev-parse --abbrev-ref HEAD", returnStdout: true).trim()
                notifySlack(":warning: UNSTABLE", repo, branch)
            }
        }
    }
}

def notifySlack(status, repo, branch) {

    def duration = currentBuild.durationString

    def reportUrl = "${env.BUILD_URL}artifact/report.pdf"
    def consoleUrl = "${env.BUILD_URL}console"

    def msg = """
${status}

Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
Branch: ${branch}
Repo: ${repo}

Duration: ${duration}

Report: ${reportUrl}
Logs: ${consoleUrl}
"""

    def payload = groovy.json.JsonOutput.toJson([text: msg])

    try {
        withCredentials([string(credentialsId: 'slack-webhook', variable: 'SLACK_URL')]) {
            sh """
            curl -X POST -H "Content-type: application/json" \
            --data '${payload}' \
            \$SLACK_URL
            """
        }
    } catch (err) {
        echo "Slack notification failed"
    }
}
