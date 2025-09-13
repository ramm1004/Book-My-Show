pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node23'
    }

    environment {
        SCANNER_HOME = tool 'sonar' 
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/ramm1004/Book-My-Show'
                sh 'ls -la'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=BMS \
                        -Dsonar.projectKey=BMS
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: true, credentialsId: 'sonarqube'
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('bookmyshow-app') {
                    sh '''
                        echo "Installing Node.js dependencies..."
                        ls -la
                        if [ -f package.json ]; then
                            rm -rf node_modules package-lock.json
                            npm install
                        else
                            echo "Error: package.json not found!"
                            exit 1
                        fi
                    '''
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
                        sh '''
                            echo "Building Docker image..."
                            docker build -t ramm1004/bms:latest -f bookmyshow-app/Dockerfile bookmyshow-app


                            echo "Pushing Docker image to DockerHub..."
                            docker push ramm1004/bms:latest
                        '''
                    }
                }
            }
        }

        stage('Deploy to Container') {
            steps {
                sh '''
                    echo "Stopping and removing old container if exists..."
                    docker stop bms || true
                    docker rm bms || true

                    echo "Running new container on port 3000..."
                    docker run -d --restart=always --name bms -p 3000:3000 ramm1004/bms:latest

                    echo "Checking running containers..."
                    docker ps -a

                    echo "Fetching container logs..."
                    sleep 5
                    docker logs bms
                '''
            }
        }
    }

    post {
        always {
            emailext(
                attachLog: true,
                subject: "'${currentBuild.result}' - BMS CI/CD Pipeline",
                body: """
                    <h3>Build Report</h3>
                    <p><b>Project:</b> ${env.JOB_NAME}</p>
                    <p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>
                    <p><b>Status:</b> ${currentBuild.result}</p>
                    <p><b>URL:</b> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                """,
                to: 'sairampentapati007@gmail.com',
                attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
            )
        }
    }
}
