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

    stage('package') {
      when { branch 'main' }
      parallel {
        stage('package') {
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
            echo 'packaging the app...'
            sh '''# Truncate the GIT_COMMIT to the first 7 characters
GIT_SHORT_COMMIT=$(echo $GIT_COMMIT | cut -c 1-7)

# Set the version using Maven
mvn versions:set -DnewVersion="$GIT_SHORT_COMMIT"
mvn versions:commit'''
            sh 'mvn package -DskipTests'
            archiveArtifacts '**/target/*.jar'
          }
        }

        stage('Docker B&P') {
          agent any
          steps {
            script {
              docker.withRegistry('https://hub.zekibalci.com/v1/', 'docker-registry') {
                def commitHash = env.GIT_COMMIT.take(7)
                def dockerImage = docker.build("hub.zekibalci.com/sysfoo:${commitHash}", "./")
                dockerImage.push()
                dockerImage.push("latest")
                dockerImage.push("dev")
              }
            }
          }
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
