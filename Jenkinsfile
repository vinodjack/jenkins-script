pipeline {
agent any

options {
    timestamps()
    buildDiscarder(logRotator(numToKeepStr: '10'))
}

parameters {
    string(name: 'BRANCH', defaultValue: 'main', description: 'Git Branch')
    choice(name: 'ENV', choices: ['QA', 'UAT', 'PROD'], description: 'Environment')
    string(name: 'TAGS', defaultValue: '@smoke', description: 'Test Tags')
    choice(name: 'PROJECT', choices: ['chromium', 'firefox', 'webkit'], description: 'Playwright Project')
    string(name: 'REPO', defaultValue: 'https://github.com/vinodjack/playwright-framework.git', description: 'Git Repo URL')
}

environment {
    EMAIL_RECIPIENTS = 'jenkinsgore1@gmail.com'
}

stages {

    stage('Checkout Code') {
        steps {
            git branch: "${params.BRANCH}", url: "${params.REPO}"
        }
    }

    stage('Install Dependencies') {
        steps {
            bat 'npm install'
            bat 'npx playwright install'
        }
    }

    stage('Run Tests') {
        steps {
            bat """
            set ENV=${params.ENV} &&
            npx playwright test --grep ${params.TAGS} --project=${params.PROJECT}
            """
        }
    }

    stage('Archive Report') {
        steps {
            archiveArtifacts artifacts: 'playwright-report/**', allowEmptyArchive: true
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
            alwaysLinkToLastBuild: true,
            allowMissing: true
        ])
    }

    success {
        emailext (
            subject: "SUCCESS: ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
            body: """
            <h3>Build Successful 🎉</h3>
            <p><b>Job:</b> ${env.JOB_NAME}</p>
            <p><b>Build:</b> ${env.BUILD_NUMBER}</p>
            <p><b>Environment:</b> ${params.ENV}</p>
            <p><b>Tags:</b> ${params.TAGS}</p>
            <p><a href="${env.BUILD_URL}">View Build</a></p>
            <p><a href="${env.BUILD_URL}HTML_20Report">View Report</a></p>
            """,
            to: "${env.EMAIL_RECIPIENTS}",
            mimeType: 'text/html'
        )
    }

    failure {
        emailext (
            subject: "FAILURE: ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
            body: """
            <h3>Build Failed ❌</h3>
            <p><b>Job:</b> ${env.JOB_NAME}</p>
            <p><b>Build:</b> ${env.BUILD_NUMBER}</p>
            <p><a href="${env.BUILD_URL}console">View Logs</a></p>
            """,
            to: "${env.EMAIL_RECIPIENTS}",
            mimeType: 'text/html'
        )
    }
}

}
