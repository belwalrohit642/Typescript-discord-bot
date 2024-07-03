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
            }
        }

        stage('Build and Test') {
            steps {
                container('nodejs') {
                    echo "Building and testing"
                    sh 'ls -ltr'
                    sh 'npm install'
                    // Uncomment the following line if you want to run tests
                    // sh 'npm test'
                }
            }
        }

        stage('Static Code Analysis') {
            steps {
                container('nodejs') {
                    echo "Installing SonarQube Scanner"
                    sh 'npm install -g sonar-scanner'
                }
            }
        }

        stage('SonarQube Analysis') {
            environment {
                SCANNER_HOME = tool 'SonarQubeScanner'
            }
            steps {
                container('nodejs') {
                    echo "Running SonarQube analysis"
                    withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                        sh 'npm install -g sonar-scanner'
                        sh 'sonar-scanner -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=$SONAR_URL'
                    }
                }
            }
        }
    }
}
