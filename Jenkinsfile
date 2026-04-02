pipeline {
    agent any

    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Git Branch')
        choice(name: 'ENV', choices: ['QA', 'UAT', 'PROD'], description: 'Environment')
        string(name: 'TAGS', defaultValue: '@smoke', description: 'Test Tags')
        choice(name: 'BROWSER', choices: ['chromium', 'firefox', 'webkit'], description: 'Browser')
        choice(name: 'PROJECT', choices: ['UI', 'API', 'MOBILE'], description: 'Project')
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: "${params.BRANCH}", url: 'https://github.com/vinodjack/jenkins-script.git'
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
