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
  tools {
    maven 'Maven 3.9.6'
  }
  post {
    success {
        // Commit status 'success' olarak ayarlanır
        setGitHubPullRequestStatus context: 'build', message: 'Build başarılı', state: 'SUCCESS'
    }
    failure {
        // Commit status 'failure' olarak ayarlanır
        setGitHubPullRequestStatus context: 'build', message: 'Build başarısız', state: 'FAILURE'
    }
  }  
}
