pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "your-dockerhub-user/java-app"
        TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/your-repo/java-app.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                sh '''
                mvn sonar:sonar \
                -Dsonar.projectKey=java-app \
                -Dsonar.host.url=http://sonarqube:9000 \
                -Dsonar.login=$SONAR_TOKEN
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $DOCKER_IMAGE:$TAG .
                '''
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                docker push $DOCKER_IMAGE:$TAG
                '''
            }
        }

        stage('Update GitOps Repo') {
            steps {
                sh '''
                git clone https://github.com/your-org/gitops-repo.git
                cd gitops-repo
                sed -i "s|image: .*|image: $DOCKER_IMAGE:$TAG|" k8s/deployment.yaml
                git config user.email "jenkins@ci.com"
                git config user.name "jenkins"
                git commit -am "Update image tag"
                git push origin main
                '''
            }
        }

        stage('ArgoCD Sync') {
            steps {
                sh '''
                argocd app sync java-app
                '''
            }
        }
    }
}
