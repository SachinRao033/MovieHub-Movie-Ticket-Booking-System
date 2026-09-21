pipeline {
    agent any

    environment {
        PROJECT_DIR = "/home/ubuntu/MovieHub-Movie-Ticket-Booking-System"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Copy Project') {
            steps {
                sh '''
                sudo mkdir -p "$PROJECT_DIR"
                sudo chmod 755 /home/ubuntu

                sudo rsync -av --delete \
                    --exclude='.git' \
                    "$WORKSPACE"/ "$PROJECT_DIR"/

                sudo chown -R jenkins:jenkins "$PROJECT_DIR"

                echo "Project copied successfully"
                ls -la "$PROJECT_DIR"
                '''
            }
        }

        stage('Create Backend Environment File') {
            steps {
                sh '''
                cd "$PROJECT_DIR"

                cat > Backend/.env <<EOF
DB_USER=moviehub
DB_PASSWORD=moviehub123
DB_HOST=mysql
DB_NAME=moviehub
EOF

                echo "Backend .env created successfully"
                cat Backend/.env
                '''
            }
        }

        stage('Stop Containers & Cleanup the container, images, volumes, networks') {
            steps {
                sh '''
                cd "$PROJECT_DIR"

                docker compose down || true
                docker system prune -af --volumes
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                cd "$PROJECT_DIR"

                docker compose build --no-cache
                '''
            }
        }

        stage('Deploy Containers') {
            steps {
                sh '''
                cd "$PROJECT_DIR"

                docker compose up -d
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                echo "Waiting for services..."
                sleep 20

                cd "$PROJECT_DIR"

                echo "===== Docker Compose Status ====="
                docker compose ps

                echo "===== Running Containers ====="
                docker ps
                '''
            }
        }
    }

    post {

        success {
            echo "SUCCESS: Air Quality Trends Analysis Project deployed successfully!"
        }

        failure {
            echo "FAILED: Deployment failed. Check Jenkins console output."
        }

        always {
            sh '''
            sudo chown -R ubuntu:ubuntu "$PROJECT_DIR" || true
            docker image prune -f || true
            '''
        }
    }
}
