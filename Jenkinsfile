pipeline {
    agent any

    environment {
        SONAR_HOST_URL = 'http://localhost:9000'
        NODE_OPTIONS = '--dns-result-order=ipv4first'
        NPM_CONFIG_REGISTRY = 'https://registry.npmjs.org/'
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
                retry(2) {
                    sh '''
                    echo "Installing dependencies..."

                    npm config set registry $NPM_CONFIG_REGISTRY
                    npm config set fetch-retries 5
                    npm config set fetch-retry-mintimeout 20000
                    npm config set fetch-retry-maxtimeout 120000

                    cd backend
                    npm install --no-audit --prefer-offline

                    cd ../frontend
                    npm install --no-audit --prefer-offline
                    '''
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                sh '''
                echo "Running OWASP Dependency Check..."

                /opt/dependency-check/bin/dependency-check.sh \
                --scan . \
                --format XML \
                --out . \
                --data /opt/dependency-check/data \
                --disableOssIndex \
                --disableYarnAudit \
                || true
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
                    -Dsonar.host.url=$SONAR_HOST_URL
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

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-cred',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo "Logging into DockerHub..."
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                    echo "Tagging images..."
                    docker tag wanderlust-backend:${BACKEND_DOCKER_TAG} $DOCKER_USER/wanderlust-backend:${BACKEND_DOCKER_TAG}
                    docker tag wanderlust-frontend:${FRONTEND_DOCKER_TAG} $DOCKER_USER/wanderlust-frontend:${FRONTEND_DOCKER_TAG}

                    echo "Pushing images..."
                    docker push $DOCKER_USER/wanderlust-backend:${BACKEND_DOCKER_TAG}
                    docker push $DOCKER_USER/wanderlust-frontend:${FRONTEND_DOCKER_TAG}
                    '''
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '*.xml', allowEmptyArchive: true
        }
        success {
            echo "✅ FULL DEVSECOPS PIPELINE SUCCESS 🚀"
        }
        failure {
            echo "❌ Pipeline failed — check logs 🔍"
        }
    }
}
