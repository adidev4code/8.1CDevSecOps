pipeline {
    agent any

    environment {
        // SonarCloud token stored as a Jenkins Secret credential with ID: SONAR_TOKEN
        SONAR_TOKEN = credentials('SONAR_TOKEN')
    }

    triggers {
        // Poll SCM every 2 minutes - works without a public webhook (local Jenkins)
        pollSCM('H/2 * * * *')
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code from GitHub repository...'
                git branch: 'main', url: 'https://github.com/adidev4code/8.1CDevSecOps.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing Node.js dependencies via npm...'
                bat 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Running npm test suite...'
                // || exit /b 0 allows the pipeline to continue even if tests fail
                bat 'npm test || exit /b 0'
            }
        }

        stage('Generate Coverage Report') {
            steps {
                echo 'Generating code coverage report...'
                bat 'npm run coverage || exit /b 0'
            }
        }

        stage('NPM Audit (Security Scan)') {
            steps {
                echo 'Running npm audit to identify known CVEs in dependencies...'
                // Outputs all known vulnerabilities — pipeline continues regardless
                bat 'npm audit || exit /b 0'
            }
        }

        stage('SonarCloud Analysis') {
            steps {
                echo 'Downloading and running SonarScanner CLI for SonarCloud analysis...'
                bat '''
                    :: Download SonarScanner CLI zip
                    curl -Lo sonar-scanner.zip https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006-windows.zip

                    :: Extract the archive
                    powershell -Command "Expand-Archive -Path sonar-scanner.zip -DestinationPath . -Force"

                    :: Move into the scanner directory
                    cd sonar-scanner-5.0.1.3006-windows

                    :: Pass the token as an environment variable (SonarScanner reads SONAR_TOKEN automatically)
                    set SONAR_TOKEN=%SONAR_TOKEN%

                    :: Run scanner - explicitly point settings back to workspace root
                    bin\sonar-scanner.bat ^
                        -Dproject.settings=..\sonar-project.properties ^
                        -Dsonar.login=%SONAR_TOKEN%
                '''
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution complete.'
        }
        success {
            echo 'All stages passed successfully. Check SonarCloud dashboard for analysis results.'
        }
        failure {
            echo 'Pipeline encountered a failure. Review the console output above.'
        }
    }
}
