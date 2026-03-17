pipeline {
    agent {
    kubernetes {
      yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: jnlp
    image: jenkins/inbound-agent:latest
    args: ['\$(JENKINS_SECRET)', '\$(JENKINS_NAME)']

  - name: maven
    image: pradeepreddyhub/jenkins-image:v1
    command:
    - cat
    tty: true

  - name: kubectl
    image: bitnami/kubectl:latest
    command:
    - cat
    tty: true
"""
    }
  }

    environment {
        DOCKER_IMAGE = "docker.io/pradeepreddyhub/hello-world"
        IMAGE_TAG = "${BUILD_NUMBER}"

        HELM_CHART = "hello-world"
        JFROG_URL = "https://trial3sfswa.jfrog.io/artifactory/jenkins-helm"

        DOCKER_CREDS = credentials('dockerhub-creds')
        JFROG_CREDS = credentials('jfrog-creds')
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/pradeepreddy-hub/Jenkins_docker_hello-world-war.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t $DOCKER_IMAGE:$IMAGE_TAG .
                """
            }
        }

        stage('Docker Login') {
            steps {
                sh """
                echo $DOCKER_CREDS_PSW | docker login -u $DOCKER_CREDS_USR --password-stdin
                """
            }
        }

        stage('Push Image') {
            steps {
                sh """
                docker push $DOCKER_IMAGE:$IMAGE_TAG
                docker tag $DOCKER_IMAGE:$IMAGE_TAG $DOCKER_IMAGE:latest
                docker push $DOCKER_IMAGE:latest
                """
            }
        }

        stage('Package Helm Chart') {
            steps {
                sh """
                helm lint helm/$HELM_CHART
                helm package helm/$HELM_CHART
                """
            }
        }

        stage('Publish Helm Chart to JFrog') {
            steps {
                sh """
                curl -u $JFROG_CREDS_USR:$JFROG_CREDS_PSW \
                -T ${HELM_CHART}-0.1.0.tgz \
                $JFROG_URL/
                """
            }
        }
    }
}
