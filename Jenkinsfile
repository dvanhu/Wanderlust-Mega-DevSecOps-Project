pipeline {
    agent any

    tools {
        sonarQubeScanner 'sonar-scanner'
    }

    environment {
        SONAR_HOST_URL = 'http://localhost:9000'
    }

    parameters {
        string(name: 'FRONTEND_DOCKER_TAG', defaultValue: 'latest', description: 'Frontend image tag')
        string(name: 'BACKEND_DOCKER_TAG', defaultValue: 'latest', description: 'Backend image tag')
    }

    stages {

        stage('Workspace Cleanup') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/dvanhu/Wanderlust-Mega-DevSecOps-Project.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                cd backend && npm install
                cd ../frontend && npm install
                '''
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                sh '''
                dependency-check --scan . --format XML --out .
                '''
            }
        }

        stage('Trivy Filesystem Scan') {
            steps {
                sh '''
                trivy fs . --format table
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                    sonar-scanner \
                    -Dsonar.projectKey=wanderlust \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=http://localhost:9000
                    '''
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                docker build -t wanderlust-backend:${BACKEND_DOCKER_TAG} ./backend
                docker build -t wanderlust-frontend:${FRONTEND_DOCKER_TAG} ./frontend
                '''
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh '''
                trivy image wanderlust-backend:${BACKEND_DOCKER_TAG}
                trivy image wanderlust-frontend:${FRONTEND_DOCKER_TAG}
                '''
            }
        }

        stage('Docker Push (Optional)') {
            when {
                expression { return false }  // enable later
            }
            steps {
                sh '''
                docker push wanderlust-backend:${BACKEND_DOCKER_TAG}
                docker push wanderlust-frontend:${FRONTEND_DOCKER_TAG}
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '*.xml', allowEmptyArchive: true
        }
    }
}
