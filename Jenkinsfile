pipeline {
  agent any
  stages {
    stage('build') {
      agent {
        kubernetes {
          label 'maven-build-pod'  // The label for the pod
          defaultContainer 'maven'  // Container name (defined below)
          yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: sysfoo
spec:
  containers:
  - name: maven
    image: hub.zekibalci.com/maven-with-git:latest
    command:
    - cat
    tty: true
  imagePullSecrets:
    - name: registrykey
"""
        }
      }
      steps {
        echo 'compiling sysfoo app...'
        sh 'mvn compile'
      }
    }

    stage('test') {
      agent {
        kubernetes {
          label 'maven-build-pod'
          defaultContainer 'maven'
          yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: sysfoo
spec:
  containers:
  - name: maven
    image: hub.zekibalci.com/maven-with-git:latest
    command:
    - cat
    tty: true
  imagePullSecrets:
    - name: registrykey    
"""
        }
      }
      steps {
        echo 'running unit tests...'
        sh 'mvn clean test'
      }
    }
  }
  post {
    success {
      script {
        // GitHub Status gönderimi
        withCredentials([usernamePassword(credentialsId: 'github-credentials', usernameVariable: 'GITHUB_USER', passwordVariable: 'GITHUB_TOKEN')]) {
          sh '''
          curl -X POST -u "${GITHUB_USER}:${GITHUB_TOKEN}" \
               -H "Content-Type: application/json" \
               -d '{
                 "state": "success",
                 "description": "Build completed successfully",
                 "context": "Jenkins CI"
               }' \
               https://api.github.com/repos/zbalci/sysfoo/statuses/$(git rev-parse HEAD)
          '''
        }
      }
    }
    failure {
      script {
        withCredentials([usernamePassword(credentialsId: 'github-credentials', usernameVariable: 'GITHUB_USER', passwordVariable: 'GITHUB_TOKEN')]) {
          sh '''
          curl -X POST -u "${GITHUB_USER}:${GITHUB_TOKEN}" \
               -H "Content-Type: application/json" \
               -d '{
                 "state": "failure",
                 "description": "Build failed",
                 "context": "Jenkins CI"
               }' \
               https://api.github.com/repos/zbalci/sysfoo/statuses/$(git rev-parse HEAD)
          '''
        }
      }
    }
  }  
  tools {
    maven 'Maven 3.9.6'
  } 
}
