pipeline {
    agent any

    environment {
        SONARQUBE_ENV = 'sonarqube'
        BACKEND_IMAGE = 'salhianis20/wanderlust-backend-beta'
        FRONTEND_IMAGE = 'salhianis20/wanderlust-frontend-beta'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Scan') {
            parallel {
                stage('Backend') {
                    steps {
                        dir('backend') {
                            withSonarQubeEnv("${SONARQUBE_ENV}") {
                                sh 'sonar-scanner -Dsonar.projectKey=wanderlust-backend-beta'
                            }
                        }
                    }
                }
                stage('Frontend') {
                    steps {
                        dir('frontend') {
                            withSonarQubeEnv("${SONARQUBE_ENV}") {
                                sh 'sonar-scanner -Dsonar.projectKey=wanderlust-frontend-beta'
                            }
                        }
                    }
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    backendImage = docker.build("${BACKEND_IMAGE}:latest", "backend")
                    frontendImage = docker.build("${FRONTEND_IMAGE}:latest", "frontend")
                }
            }
        }

        stage('Trivy Scan (HTML Reports)') {
            steps {
                script {
                    sh '''
                        mkdir -p trivy-reports

                        # Download HTML template
                        curl -sSL -o trivy-reports/trivy-html.tpl https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl

                        # Run Trivy scans in JSON format
                        trivy image --format json --output trivy-reports/backend.json ${BACKEND_IMAGE}:latest
                        trivy image --format json --output trivy-reports/frontend.json ${FRONTEND_IMAGE}:latest

                        # Convert JSON to HTML reports
                        trivy convert --format template --template @trivy-reports/trivy-html.tpl --output trivy-reports/backend-report.html trivy-reports/backend.json
                        trivy convert --format template --template @trivy-reports/trivy-html.tpl --output trivy-reports/frontend-report.html trivy-reports/frontend.json
                    '''
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                withVault([
                    vaultSecrets: [[
                        path: 'kv/dockerhub-creds',
                        secretValues: [
                            [envVar: 'DOCKER_USERNAME', vaultKey: 'username'],
                            [envVar: 'DOCKER_PASSWORD', vaultKey: 'password']
                        ]
                    ]]
                ]) {
                    script {
                        sh '''
                            echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                            docker push ${BACKEND_IMAGE}:latest
                            docker push ${FRONTEND_IMAGE}:latest
                            docker logout
                        '''
                    }
                }
            }
        }

        stage('Archive and Publish Trivy HTML Reports') {
            steps {
                archiveArtifacts artifacts: 'trivy-reports/*.html', fingerprint: true

                publishHTML(target: [
                    reportDir: 'trivy-reports',
                    reportFiles: 'backend-report.html,frontend-report.html',
                    reportName: 'Trivy HTML Security Reports',
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true
                ])
            }
        }
    }
}
