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

        stage('SonarQube Analysis') {
            parallel {
                stage('Backend Analysis') {
                    steps {
                        dir('backend') {
                            withSonarQubeEnv("${SONARQUBE_ENV}") {
                                sh 'sonar-scanner -Dsonar.projectKey=wanderlust-backend-beta'
                            }
                        }
                    }
                }
                stage('Frontend Analysis') {
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


        stage('Trivy Scan & Report') {
            steps {
                script {
                    sh '''
                        mkdir -p trivy-reports
        
                        # Download HTML template (once)
                        curl -sSL -o trivy-html.tpl https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl
        
                        # Run Trivy image scans
                        trivy image --format json --output trivy-reports/backend.json ${BACKEND_IMAGE}:latest
                        trivy image --format json --output trivy-reports/frontend.json ${FRONTEND_IMAGE}:latest
        
                        # Convert JSON to HTML using the template
                        trivy convert --format template --template @trivy-html.tpl --output trivy-reports/backend-report.html trivy-reports/backend.json
                        trivy convert --format template --template @trivy-html.tpl --output trivy-reports/frontend-report.html trivy-reports/frontend.json
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
                        sh """
                            echo "\$DOCKER_PASSWORD" | docker login -u "\$DOCKER_USERNAME" --password-stdin
                            docker push ${BACKEND_IMAGE}:latest
                            docker push ${FRONTEND_IMAGE}:latest
                            docker logout
                        """
                    }
                }
            }
        }

        stage('Archive Trivy Reports') {
            steps {
                archiveArtifacts artifacts: 'trivy-reports/*.html', fingerprint: true
            }
        }
    }
}




















// @Library('Shared') _
// pipeline {
//     agent any
    
//     environment{
//         SONAR_HOME = tool "Sonar"
//     }
//     stages {
        
//         stage("Workspace cleanup"){
//             steps{
//                 script{
//                     cleanWs()
//                 }
//             }
//         }
        
//         stage('Git: Code Checkout') {
//             steps {
//                 script{
//                     code_checkout("https://github.com/DevMadhup/wanderlust.git","devops")
//                 }
//             }
//         }
        
//         stage("OWASP: Dependency check"){
//             steps{
//                 script{
//                     owasp_dependency()
//                 }
//             }
//             post{
//                 success{
//                     archiveArtifacts artifacts: '**/dependency-check-report.xml', followSymlinks: false, onlyIfSuccessful: true
//                 }
//             }
//         }
        
//         stage("Trivy: Filesystem scan"){
//             steps{
//                 script{
//                     trivy_scan()
//                 }
//             }
//         }
        
//         stage("SonarQube: Code Analysis"){
//             steps{
//                 script{
//                     sonarqube_analysis("Sonar","wanderlust","wanderlust")
//                 }
//             }
//         }
        
//         stage("SonarQube: Code Quality Gates"){
//             steps{
//                 script{
//                     sonarqube_code_quality()
//                 }
//             }
//         }
        
//         stage('Exporting environment variables') {
//             parallel{
//                 stage("Backend env setup"){
//                     steps {
//                         script{
//                             dir("Automations"){
//                                 sh "bash updateBackend.sh"
//                             }
//                         }
//                     }
//                 }
                
//                 stage("Frontend env setup"){
//                     steps {
//                         script{
//                             dir("Automations"){
//                                 sh "bash updateFrontend.sh"
//                             }
//                         }
//                     }
//                 }
//             }
//         }
        
//         stage("Docker: Build Images"){
//             steps{
//                 script{
//                     dir('backend'){
//                         docker_build("backend-wanderlust","test-image-donot-use","madhupdevops")
//                     }
                    
//                     dir('frontend'){
//                         docker_build("frontend-wanderlust","test-image-donot-use","madhupdevops")
//                     }
//                 }
//             }
//         }
        
//         stage("Docker: Push to DockerHub"){
//             steps{
//                 script{
//                     docker_push("backend-wanderlust","test-image-donot-use","madhupdevops") 
//                     docker_push("frontend-wanderlust","test-image-donot-use","madhupdevops")
//                 }
//             }
//         }
//     }
    
//     post{
//         success{
//             build job: "Wanderlust-CD", parameters: [
//                 string(name: 'FRONTEND_DOCKER_TAG', value: "test-image-donot-use"),
//                 string(name: 'BACKEND_DOCKER_TAG', value: "test-image-donot-use")
//             ]
//         }
//     }
// }
