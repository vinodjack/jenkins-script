pipeline {
    agent any

    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Git Branch')
        choice(name: 'ENV', choices: ['QA', 'UAT', 'PROD'], description: 'Environment')
        string(name: 'TAGS', defaultValue: '@smoke', description: 'Test Tags')
        choice(name: 'BROWSER', choices: ['chromium', 'firefox', 'webkit'], description: 'Browser')
        choice(name: 'PROJECT', choices: ['UI_Test', 'API_Test', 'MOBILE_Test'], description: 'Project')
        choince(name: 'REPO', defaultValue: 'https://github.com/vinodjack/playwright-framework.git', description: 'Git Repo URL')
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
                npx playwright test --grep ${params.TAGS} ^
                --project=${params.BROWSER} ^
                --env=${params.ENV}
                """
            }
        }
    }
}
