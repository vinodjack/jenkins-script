pipeline {
    agent any

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    parameters {
        choice(name: 'ENV', choices: ['QA', 'UAT', 'PROD'], description: 'Select Environment')
        string(name: 'TAGS', defaultValue: '@smoke', description: 'Cucumber/Playwright Tags')
        choice(name: 'PROJECT', choices: ['UI', 'API', 'MOBILE'], description: 'Project Type')
    }

    environment {
        NODE_VERSION = '18'
        EMAIL_RECIPIENTS = 'your-email@example.com'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/your-repo.git'
            }
        }

        stage('Install Node & Dependencies') {
            steps {
                bat 'npm install'
                bat 'npx playwright install'
            }
        }

        stage('Run Tests') {
            steps {
                bat """
                npx playwright test --grep ${params.TAGS} --project=${params.PROJECT}
                """
            }
        }

        stage('Archive Results') {
            steps {
                archiveArtifacts artifacts: '**/playwright-report/**', allowEmptyArchive: true
            }
        }
    }

    post {

        always {
            publishHTML([
                reportDir: 'playwright-report',
                reportFiles: 'index.html',
                reportName: 'Playwright Report',
                keepAll: true,
                alwaysLinkToLastBuild: true
            ])
        }

        success {
            emailext (
                subject: "SUCCESS: ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                body: """
                <h3>Build Successful</h3>
                <p><b>Job:</b> ${env.JOB_NAME}</p>
                <p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>
                <p><b>Environment:</b> ${params.ENV}</p>
                <p><a href="${env.BUILD_URL}">View Build</a></p>
                """,
                to: "${env.EMAIL_RECIPIENTS}",
                mimeType: 'text/html'
            )
        }

        failure {
            emailext (
                subject: "FAILURE: ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                body: """
                <h3>Build Failed</h3>
                <p><b>Job:</b> ${env.JOB_NAME}</p>
                <p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>
                <p><a href="${env.BUILD_URL}console">View Logs</a></p>
                """,
                to: "${env.EMAIL_RECIPIENTS}",
                mimeType: 'text/html'
            )
        }
    }
}
