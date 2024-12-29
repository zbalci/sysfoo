pipeline {
  agent none
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
    image: maven:3.9.6-eclipse-temurin-17-alpine
    command:
    - cat
    tty: true
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
    image: maven:3.9.6-eclipse-temurin-17-alpine
    command:
    - cat
    tty: true
"""
        }
      }
      steps {
        echo 'running unit tests...'
        sh 'mvn clean test'
      }
    }
  }
  tools {
    maven 'Maven 3.9.6'
  }
  post {
    always {
      echo 'This pipeline is completed..'
    }
  }
}
