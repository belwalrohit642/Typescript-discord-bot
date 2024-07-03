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
    image: jenkins/inbound-agent
    volumeMounts:
    - name: jenkins-docker-cfg
      mountPath: /home/jenkins/.docker
  - name: nodejs
    image: node:16
    tty: true
    volumeMounts:
    - name: jenkins-docker-cfg
      mountPath: /root/.docker
  - name: sonarqube
    image: openjdk:17
    tty: true
    volumeMounts:
    - name: jenkins-docker-cfg
      mountPath: /root/.docker
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
                // Uncomment and adjust the following checkout step based on your Git repository setup
                // checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/belwalrohit642/nodejs-ci-cd-project.git']]])
            }
        }

        stage('Build and Test') {
            steps {
                container('nodejs') {
                    echo "Building and testing"
                    sh 'ls -ltr'
                    sh 'npm install'
                    // Uncomment and adjust the npm test command based on your project setup
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
                JAVA_HOME = "/usr/lib/jvm/java-17-openjdk"
                PATH = "$JAVA_HOME/bin:$PATH"
            }
            steps {
                container('sonarqube') {
                    withSonarQubeEnv('SonarQube') {
                        echo "Running SonarQube analysis"
                        sh '''
                            echo "JAVA_HOME=$JAVA_HOME"
                            echo "PATH=$PATH"
                            ${SCANNER_HOME}/bin/sonar-scanner
                        '''
                    }
                }
            }
        }
    }
}
