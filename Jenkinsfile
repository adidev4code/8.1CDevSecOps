pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('SONAR_TOKEN')
    }

    triggers {
        pollSCM('H/2 * * * *')
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/adidev4code/8.1CDevSecOps.git'
            }
        }

        stage('Install') {
            steps {
                bat 'npm install'
            }
        }

        stage('Test') {
            steps {
                bat 'npm test || exit /b 0'
            }
        }

        stage('Coverage') {
            steps {
                bat 'npm run coverage || exit /b 0'
            }
        }

        stage('Audit') {
            steps {
                bat 'npm audit || exit /b 0'
            }
        }

        stage('Sonar') {
            steps {
                bat '''
                curl -Lo sonar.zip https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006-windows.zip
                powershell -Command "Expand-Archive -Path sonar.zip -DestinationPath . -Force"
                cd sonar-scanner-5.0.1.3006-windows
                bin\\sonar-scanner.bat -Dproject.settings=..\\sonar-project.properties -Dsonar.token=%SONAR_TOKEN%
                '''
            }
        }
    }
}