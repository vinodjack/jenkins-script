pipeline {
agent any

options {
    timestamps()
    buildDiscarder(logRotator(numToKeepStr: '10'))
    timeout(time: 2, unit: 'HOURS')
}

parameters {
    string(name: 'BRANCH', defaultValue: 'master', description: 'Git Branch')
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
                echo "Cloning Repo: ${params.REPO}"
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
                echo "Installing dependencies..."
                bat 'npm install'
                bat 'npx playwright install'
            }
        }
    }

    stage('Run Tests') {
        steps {
            script {

                def testCommand = "npm test -- --env ${params.ENV} --pname ${params.PROJECT} --browser ${params.BROWSER} --tags \"${params.TAGS}\" --parallel ${params.WORKER}"

                echo "EXECUTION DETAILS"
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
                echo "Archiving Reports..."

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
            subject: "SUCCESS | ${env.JOB_NAME} | Build #${env.BUILD_NUMBER}",
            body: """
            <html>
            <body style="font-family: Arial; background:#f4f6f8; padding:20px;">
            
            <div style="background:#ffffff; padding:20px; border-radius:10px; box-shadow:0 2px 8px rgba(0,0,0,0.1);">
                
                <h2 style="color:#28a745;">Build Successful</h2>
                
                <table style="width:100%; border-collapse:collapse;">
                    <tr><td><b>Job</b></td><td>${env.JOB_NAME}</td></tr>
                    <tr><td><b>Build</b></td><td>#${env.BUILD_NUMBER}</td></tr>
                    <tr><td><b>Environment</b></td><td>${params.ENV}</td></tr>
                    <tr><td><b>Project</b></td><td>${params.PROJECT}</td></tr>
                    <tr><td><b>Browser</b></td><td>${params.BROWSER}</td></tr>
                    <tr><td><b>Tags</b></td><td>${params.TAGS}</td></tr>
                </table>

                <br/>

                <a href="${env.BUILD_URL}" style="padding:10px 15px; background:#007bff; color:#fff; text-decoration:none; border-radius:5px;">
                     View Build
                </a>

                <a href="${env.BUILD_URL}HTML_20Report" style="padding:10px 15px; background:#17a2b8; color:#fff; text-decoration:none; border-radius:5px; margin-left:10px;">
                     View Report
                </a>

            </div>

            </body>
            </html>
            """,
            to: "${env.EMAIL_RECIPIENTS}",
            mimeType: 'text/html',
            attachmentsPattern: 'testReports/*.json'
        )
    }

    failure {
        emailext (
            subject: "FAILURE | ${env.JOB_NAME} | Build #${env.BUILD_NUMBER}",
            body: """
            <html>
            <body style="font-family: Arial; background:#f4f6f8; padding:20px;">
            
            <div style="background:#ffffff; padding:20px; border-radius:10px; box-shadow:0 2px 8px rgba(0,0,0,0.1);">
                
                <h2 style="color:#dc3545;">Build Failed</h2>
                
                <table style="width:100%; border-collapse:collapse;">
                    <tr><td><b>Job</b></td><td>${env.JOB_NAME}</td></tr>
                    <tr><td><b>Build</b></td><td>#${env.BUILD_NUMBER}</td></tr>
                    <tr><td><b>Environment</b></td><td>${params.ENV}</td></tr>
                    <tr><td><b>Project</b></td><td>${params.PROJECT}</td></tr>
                </table>

                <br/>

                <a href="${env.BUILD_URL}console" style="padding:10px 15px; background:#dc3545; color:#fff; text-decoration:none; border-radius:5px;">
                    View Logs
                </a>

            </div>

            </body>
            </html>
            """,
            to: "${env.EMAIL_RECIPIENTS}",
            mimeType: 'text/html',
            attachmentsPattern: 'testReports/*.json'
        )
    }
}

}
