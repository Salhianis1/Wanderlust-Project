pipeline {
    agent any

    environment {
        SONARQUBE_ENV = 'sonarqube' // Your Jenkins SonarQube server name
        DOCKER_CREDENTIALS_ID = 'dockerhub-id' // Jenkins credentials for Docker Hub
        BACKEND_IMAGE = 'salhianis20/wanderlust-backend-beta'
        FRONTEND_IMAGE = 'salhianis20/wanderlust-frontend-beta'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        

        stage('SonarQube - wanderlust-backend-beta') {
            steps {
                dir('backend') {
                    withSonarQubeEnv("${SONARQUBE_ENV}") {
                        sh 'sonar-scanner -Dsonar.projectKey=wanderlust-backend-beta'
                    }
                }
            }
        }

        stage('SonarQube - wanderlust-frontend-beta') {
            steps {
                dir('frontend') {
                    withSonarQubeEnv("${SONARQUBE_ENV}") {
                        sh 'sonar-scanner -Dsonar.projectKey=wanderlust-frontend-beta'
                    }
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    backendImage = docker.build("${BACKEND_IMAGE}:${env.BUILD_NUMBER}", "backend")
                    frontendImage = docker.build("${FRONTEND_IMAGE}:${env.BUILD_NUMBER}", "frontend")
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        backendImage.push()
                        backendImage.push("latest")

                        frontendImage.push()
                        frontendImage.push("latest")
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ MERN backend and frontend built, analyzed, and pushed successfully!'
        }
        failure {
            echo '❌ Pipeline failed.'
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
