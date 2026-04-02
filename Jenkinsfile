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
            checkout([
                $class: 'GitSCM',
                branches: [[name: "*/${params.BRANCH}"]],
                userRemoteConfigs: [[url: "${params.REPO}"]]
            ])
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
            npm test -- --env ${params.ENV} --pname ${params.PROJECT} --browser ${params.BROWSER} --tags "${params.TAGS}" --parallel ${params.WORKER}
            """
        }
    }

    stage('Generate Summary + Zip Reports') {
        steps {
            script {
                echo "📊 Generating Summary..."

                // Create summary (simple JS parser)
                writeFile file: 'summary.js', text: """
                const fs = require('fs');
                try {
                    const data = JSON.parse(fs.readFileSync('testReports/report.json'));
                    let passed = 0, failed = 0;

                    data.forEach(feature => {
                        feature.elements.forEach(scenario => {
                            scenario.steps.forEach(step => {
                                if(step.result.status === 'passed') passed++;
                                if(step.result.status === 'failed') failed++;
                            });
                        });
                    });

                    fs.writeFileSync('summary.txt', `PASSED=${passed}\\nFAILED=${failed}`);
                } catch(e) {
                    fs.writeFileSync('summary.txt', 'PASSED=0\\nFAILED=0');
                }
                """

                bat 'node summary.js'

                // Zip reports
                bat 'powershell Compress-Archive -Path playwright-report -DestinationPath report.zip -Force'
            }
        }
    }

    stage('Archive Reports') {
        steps {
            archiveArtifacts artifacts: 'report.zip', allowEmptyArchive: true
            archiveArtifacts artifacts: 'testReports/*.json', allowEmptyArchive: true
            archiveArtifacts artifacts: 'testReports/screenshots/**', allowEmptyArchive: true
        }
    }
}

post {

    always {
        publishHTML([
            reportDir: 'testReports',
            reportFiles: 'index.html',
            reportName: 'index',
            keepAll: true,
            alwaysLinkToLastBuild: true,
            allowMissing: true
        ])
    }

    success {
        script {
            def summary = readFile('summary.txt')

            emailext (
                subject: "SUCCESS | ${env.JOB_NAME} | Build #${env.BUILD_NUMBER}",
                body: """
                <html>
                <body style="font-family:Segoe UI; background:#f4f6f8; padding:20px;">
                
                <div style="background:#fff; padding:20px; border-left:6px solid #28a745;">
                    
                    <h2 style="color:#28a745;">Build Successful</h2>

                    <pre>${summary}</pre>

                    <p><b>Environment:</b> ${params.ENV}</p>
                    <p><b>Project:</b> ${params.PROJECT}</p>
                    <p><b>Browser:</b> ${params.BROWSER}</p>

                    <a href="${env.BUILD_URL}">View Build</a><br/>
                    <a href="${env.BUILD_URL}HTML_20Report">View Report</a>

                    <p><b>Attachments:</b> ZIP + JSON + Screenshots</p>

                </div>
                </body>
                </html>
                """,
                to: "${env.EMAIL_RECIPIENTS}",
                mimeType: 'text/html',
                attachmentsPattern: 'report.zip, testReports/*.json, testReports/screenshots/**'
            )
        }
    }

    failure {
        script {
            def summary = readFile('summary.txt')

            emailext (
                subject: "FAILURE | ${env.JOB_NAME} | Build #${env.BUILD_NUMBER}",
                body: """
                <html>
                <body style="font-family:Segoe UI; background:#f4f6f8; padding:20px;">
                
                <div style="background:#fff; padding:20px; border-left:6px solid #dc3545;">
                    
                    <h2 style="color:#dc3545;">Build Failed</h2>

                    <pre>${summary}</pre>

                    <p><a href="${env.BUILD_URL}console">View Logs</a></p>

                </div>
                </body>
                </html>
                """,
                to: "${env.EMAIL_RECIPIENTS}",
                mimeType: 'text/html',
                attachmentsPattern: 'report.zip, testReports/*.json, testReports/screenshots/**'
            )
        }
    }
}
}
