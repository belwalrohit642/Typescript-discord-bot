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
  - name: nodejs
    image: node:16 # Standard Node.js image
    command: ["/bin/sh", "-c", "cat"]
    tty: true
    volumeMounts:
    - name: jenkins-docker-cfg
      mountPath: /kaniko/.docker
  - name: sonarqube
    image: openjdk:17-slim  # Image with Java for SonarQube
    command: ["/bin/sh", "-c", "cat"]
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
                container('nodejs') {
                    echo "Installing SonarQube Scanner"
                    sh 'npm install -g sonar-scanner'
                }
            }
        }

        stage('SonarQube analysis') {
            environment {
                SCANNER_HOME = tool 'SonarQubeScanner'
                JAVA_HOME = "/usr/lib/jvm/java-17-openjdk-amd64"
                PATH = "$JAVA_HOME/bin:$PATH"
            }
            steps {
                container('sonarqube') {
                    withSonarQubeEnv('SonarQube') {
                        echo "Running SonarQube analysis"
                        sh '''
                            export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
                            export PATH=$JAVA_HOME/bin:$PATH
                            ${SCANNER_HOME}/bin/sonar-scanner
                        '''
                    }
                }
            }
        }
    }
}
