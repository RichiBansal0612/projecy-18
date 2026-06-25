pipeline {
    agent any

    tools {
        maven 'Maven-3'
        jdk 'JDK17'
    }

    environment {
        DOCKER_IMAGE = "richibansal0612/projecy-18"
        TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/RichiBansal0612/projecy-18.git'
            }
        }

        stage('Clean Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh """
                    mvn sonar:sonar \
                    -Dsonar.projectKey=projecy-18 \
                    -Dsonar.host.url=http://sonarqube:9000
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $DOCKER_IMAGE:$TAG ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push $DOCKER_IMAGE:$TAG
                    '''
                }
            }
        }

        stage('Update GitOps Repo') {
            steps {
                sh """
                rm -rf gitops-repo
                git clone https://github.com/RichiBansal0612/gitops-repo.git

                cd gitops-repo

                sed -i 's|image: .*|image: ${DOCKER_IMAGE}:${TAG}|' k8s/deployment.yaml

                git config user.email "jenkins@ci.com"
                git config user.name "jenkins"

                git add .
                git commit -m "Update image to ${TAG}" || echo "No changes to commit"
                git push origin main
                """
            }
        }
    }

    post {
        success {
            echo '🚀 Pipeline SUCCESS'
        }

        failure {
            echo '❌ Pipeline FAILED'
        }
    }
}
