pipeline {
  agent {
    node {
      label 'prod'
    }
  }
  tools {
    maven "maven3"
  }

  stages {
    stage('checkout code') {
      steps {
        git branch: 'feature', url: 'https://github.com/DevOpsCloudZone/docker_web_stack.git'
      }
    }
    stage('Build and Test') {
      steps {
        sh 'mvn clean install'
      }
    }
    stage('Code Quality Analysis') {
      steps {
        withSonarQubeEnv('sonarqube') {
          // Here copy code from Sonarqube
          sh "mvn clean verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=sonarproject -Dsonar.projectName='sonarproject'"
        }
      }
    }
    stage("code quality gates") {
      steps {
        waitForQualityGate abortPipeline: true, credentialsId: 'sonar_cred'
      }
    }
    stage('nexus artifact') {
      steps {
        nexusArtifactUploader artifacts: [
          [artifactId: 'vprofile', classifier: '', file: ' target/vprofile-v2.war', type: 'war']
        ], credentialsId: 'nexus_cred', groupId: 'com.visualpathit', nexusUrl: '18.60.156.61:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'nexusrepo', version: 'v2'
      }
    }
    stage('build docker image') {
      steps {
        sh 'cp -r target Docker-app'
        sh 'docker build -t appimage Docker-app'
        sh 'docker build -t dbimage Docker-db'
      }
    }
    stage('Image Scan') {
      steps {
        sh 'trivy image appimage >> appimage_log.txt'
        sh 'trivy image dbimage >> dbimage_log.txt'
      }
    }
    stage('Docker push') {
      steps {
        script {
          withDockerRegistry(credentialsId: 'dockerhub') {
            sh 'docker tag appimage umesh2425/appimage:javaapp'
            sh 'docker tag dbimage umesh2425/dbimage:db'
            sh 'docker push umesh2425/appimage:javaapp'
            sh 'docker push umesh2425/dbimage:db'
          }
        }

      }
    }
    stage('Deploy') {
      steps {
        sh 'docker stack deploy  appstack --compose-file=compose.yml'
      }
    }
  }
  post {
    success {
      mail to: 'avetiumesh10@gmail.com',
        subject: "Build Status: ${currentBuild.fullDisplayName}",
        body: "The build ${env.BUILD_NUMBER} finished with status: ${currentBuild.currentResult}. View logs at: ${env.BUILD_URL}"
    }
    failure {
      mail to: 'avetiumesh10@gmail.com',
        subject: "Build Status: ${currentBuild.fullDisplayName}",
        body: "The build ${env.BUILD_NUMBER} finished with status: ${currentBuild.currentResult}. View logs at: ${env.BUILD_URL}"
    }

  }
}
