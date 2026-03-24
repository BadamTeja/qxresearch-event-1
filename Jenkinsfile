pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "your-dockerhub-username/sample-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Clone Code') {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'git-creds', url: 'https://github.com/BadamTeja/qxresearch-event-1.git']])
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'pip install -r requirements.txt'
            }
        }

        stage('Run Tests') {
            steps {
                echo "No tests defined yet"
                // future: pytest
            }
        }

        stage('Build Artifact') {
            steps {
                sh 'mkdir -p build'
                sh 'cp -r *.py requirements.txt build/'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t $DOCKER_IMAGE:$IMAGE_TAG .
                docker tag $DOCKER_IMAGE:$IMAGE_TAG $DOCKER_IMAGE:latest
                """
            }
        }

       stage('Push to DockerHub') {
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'docker-creds',
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PASS'
        )]) {
            sh '''
            echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
            docker push $DOCKER_IMAGE:$IMAGE_TAG
            docker push $DOCKER_IMAGE:latest
            '''
        }
    }
}

        stage('Clean Old Images') {
            steps {
                sh """
                docker image prune -f
                """
            }
        }

       stage('Deploy to Kubernetes') {
    steps {
        sh """
        cp k8s/deployment.yaml k8s/deployment-temp.yaml
        sed -i 's|REPLACE_IMAGE|$DOCKER_IMAGE:$IMAGE_TAG|g' k8s/deployment-temp.yaml
        kubectl apply -f k8s/deployment-temp.yaml
        kubectl apply -f k8s/service.yaml
        """
    }
}
    }
}
