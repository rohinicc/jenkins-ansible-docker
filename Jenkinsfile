pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'musicapp'
        DOCKERHUB_USERNAME = 'rohinicc'
        DOCKERHUB_REPO = 'music-dashboard-app'
        VERSION = "${BUILD_ID}"
        CONTAINER_NAME = 'app'
        CONTAINER_PORT = '8085'
        REQUEST_PORT = '80'
        ANSIBLE_HOST_KEY_CHECKING = 'False'
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/rohinicc/jenkins-ansible-docker.git'
            }
        }

        stage('Docker Version Check') {
            steps {
                sh 'docker --version'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh '''
                docker tag $DOCKER_IMAGE $DOCKERHUB_USERNAME/$DOCKERHUB_REPO:$VERSION
                docker tag $DOCKER_IMAGE $DOCKERHUB_USERNAME/$DOCKERHUB_REPO:latest
                '''
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-cred',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $DOCKERHUB_USERNAME/$DOCKERHUB_REPO:$VERSION
                    docker push $DOCKERHUB_USERNAME/$DOCKERHUB_REPO:latest
                    '''
                }
            }
        }

        stage('Validate Ansible Playbook') {
            steps {
                sh 'sudo -u ansible ansible-playbook -i inventory playbook.yml --check'
            }
        }

        stage('Deploy via Ansible') {
            steps {
                sh 'sudo -u ansible ansible-playbook -i inventory playbook.yml -v'
            }
        }
    }

    post {
        success {
            echo 'App deployed successfully '
        }
        failure {
            echo ' Pipeline failed.'
        }
    }
}