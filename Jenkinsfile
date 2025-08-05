pipeline {
    agent any

    environment {
        IMAGE_NAME = 'prajwal5028/inventory-management'
        IMAGE_TAG = 'latest'
        DOCKER_CREDENTIALS_ID = 'Docker'
        SCANNER_HOME = tool 'Sonar-scanner'
        PYTHONANYWHERE_CICD_URL = 'https://prajwal5028.pythonanywhere.com/pull_and_reload'
        PYTHONANYWHERE_CICD_TOKEN = credentials('pythonanywhere-cicd-token') // Jenkins secret text
    }

    stages {
        stage('gitcheckout') {
            steps {
                git branch: 'main', url: 'https://github.com/Prajwal5028/Inventory-Management.git'
            }
        }
        
        
         stage('Sonarqube Analysis') {
            steps {
                bat """
                ${SCANNER_HOME}/bin/Sonar-scanner ^
                -Dsonar.projectName=Inventory-Management ^
                -Dsonar.host.url=http://localhost:9000 ^
                -Dsonar.token=squ_90a5b7b7a6935b9325a0e62d4cd63229f490dfd5 ^
                -Dsonar.java.binaries=. ^
                -Dsonar.projectKey=Inventory-Management
                """
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    bat "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }
        
        stage("Trivy Docker Scan"){
            steps{
                bat "trivy image ${IMAGE_NAME}:${IMAGE_TAG} "
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: "${DOCKER_CREDENTIALS_ID}",
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        bat """
                            echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin
                            docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        """
                    }
                }
            }
        }

        stage('Deploy to PythonAnywhere') {
            steps {
                script {
                    bat """
                        curl -X POST ${PYTHONANYWHERE_CICD_URL} ^
                        -H "X-Auth-Token: ${PYTHONANYWHERE_CICD_TOKEN}"
                    """
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning workspace...'
            cleanWs()
        }
        success {
            echo '✅ CI/CD pipeline completed successfully.'
        }
        failure {
            echo '❌ Pipeline failed.'
        }
    }
} 