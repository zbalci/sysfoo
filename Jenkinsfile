pipeline {
  agent none
  stages {
    stage('build') {
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
        echo 'compiling sysfoo app...'
        sh 'git status'
        sh 'mvn compile'
      }
      post {
        success {
          echo 'Build succeeded'
          // Notify GitHub of build success
          githubNotify context: 'build', status: 'SUCCESS', description: 'Build succeeded'
        }
        failure {
          echo 'Build failed'
          // Notify GitHub of build failure
          githubNotify context: 'build', status: 'FAILURE', description: 'Build failed'
        }
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
      post {
        success {
          echo 'Tests passed'
          // Notify GitHub of test success
          githubNotify context: 'test', status: 'SUCCESS', description: 'Tests passed'
        }
        failure {
          echo 'Tests failed'
          // Notify GitHub of test failure
          githubNotify context: 'test', status: 'FAILURE', description: 'Tests failed'
        }
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
