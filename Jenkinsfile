pipeline {
agent any

options {
    timestamps()
    buildDiscarder(logRotator(numToKeepStr: '10'))
    timeout(time: 2, unit: 'HOURS')
}

parameters {
    string(name: 'BRANCH', defaultValue: 'main', description: 'Git Branch')
    choice(name: 'ENV', choices: ['QA', 'UAT', 'PROD'], description: 'Environment')
    string(name: 'TAGS', defaultValue: '@smoke', description: 'Test Tags')
    choice(name: 'PROJECT', choices: ['UI_Test', 'API_Test', 'MOBILE_Test'], description: 'Project')
    choice(name: 'BROWSER', choices: ['chromium', 'firefox', 'webkit'], description: 'Browser')
    string(name: 'WORKER', defaultValue: '1', description: 'Parallel Workers')
    string(name: 'REPO', defaultValue: 'https://github.com/vinodjack/playwright-framework.git', description: 'Git Repo URL')
}

environment {
    EMAIL_RECIPIENTS = 'jenkinsgore1@gmail.com'
}

stages {

    stage('Checkout Code') {
        steps {
            script {
                echo "📌 Cloning Repo: ${params.REPO}"
                echo "Branch: ${params.BRANCH}"

                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "*/${params.BRANCH}"]],
                    userRemoteConfigs: [[url: "${params.REPO}"]]
                ])
            }
        }
    }

    stage('Install Dependencies') {
        steps {
            script {
                echo "📦 Installing dependencies..."
                bat 'npm install'
                bat 'npx playwright install'
            }
        }
    }

    stage('Run Tests') {
        steps {
            script {

                def testCommand = "npm test -- --env ${params.ENV} --pname ${params.PROJECT} --browser ${params.BROWSER} --tags \"${params.TAGS}\" --parallel ${params.WORKER}"

                echo "🚀 EXECUTION DETAILS"
                echo "══════════════════════════════"
                echo "Repo: ${params.REPO}"
                echo "Branch: ${params.BRANCH}"
                echo "Environment: ${params.ENV}"
                echo "Project: ${params.PROJECT}"
                echo "Browser: ${params.BROWSER}"
                echo "Tags: ${params.TAGS}"
                echo "Workers: ${params.WORKER}"
                echo "══════════════════════════════"
                echo "Command: ${testCommand}"
                echo "══════════════════════════════"

                bat "${testCommand}"
            }
        }
    }

    stage('Archive Reports') {
        steps {
            script {
                echo "📊 Archiving Reports..."

                archiveArtifacts artifacts: 'playwright-report/**', allowEmptyArchive: true
                archiveArtifacts artifacts: 'testReports/**', allowEmptyArchive: true
                archiveArtifacts artifacts: 'testReports/screenshots/**', allowEmptyArchive: true
            }
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
            <p><b>Environment:</b> ${params.ENV}</p>
            <p><b>Project:</b> ${params.PROJECT}</p>
            <p><b>Browser:</b> ${params.BROWSER}</p>
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
            <p><b>Environment:</b> ${params.ENV}</p>
            <p><b>Project:</b> ${params.PROJECT}</p>
            <p><a href="${env.BUILD_URL}console">View Logs</a></p>
            """,
            to: "${env.EMAIL_RECIPIENTS}",
            mimeType: 'text/html'
        )
    }
}

}
