pipeline {
    agent {
        kubernetes {
            label 'NodeJS-Volume'
            yaml """
apiVersion: v1
kind: Pod
metadata:
  name: nodejs
spec:
  containers:
  - name: jnlp
    workingDir: /home/jenkins
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: Always
    command: ["/busybox/cat"]
    tty: true
    volumeMounts:
    - name: jenkins-docker-cfg
      mountPath: /kaniko/.docker
  - name: nodejs
    image: node:16
    command: ["/bin/sh", "-c", "cat"]  # Keep the container running
    tty: true
    volumeMounts:
    - name: jenkins-docker-cfg
      mountPath: /kaniko/.docker
  - name: sonarscanner
    image: sonarsource/sonar-scanner-cli:10.0.3.1430_5.0.1
    command: ["/bin/sh", "-c", "cat"]  # Keep the container running
    tty: true
  volumes:
  - name: jenkins-docker-cfg
    projected:
      sources:
      - secret:
          name: docker-credentials
          items:
          - key: config.json
            path: .dockerconfigjson
"""
        }
    }

    environment {
        SONAR_URL = "http://3.88.128.97:9000"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Checking out code"
                // checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/belwalrohit642/nodejs-ci-cd-project.git']]])
            }
        }

        stage('Build and Test') {
            steps {
                container('nodejs') {
                    echo "Building and testing"
                    sh 'ls -ltr'
                    sh 'npm install'
                    // sh 'npm test'
                }
            }
        }

        stage('Static Code Analysis') {
            steps {
                container('sonarscanner') {
                    echo "Running static code analysis"
                    withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                        sh 'sonar-scanner \
                            -Dsonar.login=$SONAR_AUTH_TOKEN \
                            -Dsonar.host.url=$SONAR_URL \
                            -Dsonar.sources=./src'
                    }
                }
            }
        }
    }
}
