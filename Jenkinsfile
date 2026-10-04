pipeline {
  agent any
  tools {
    maven "mymaven3"
  }
  stages {
    stage("Code Checkout") {
      steps {
          // Code Checkout stage is not required here
          // Jenkins automatically checks out the source code when
          // "Pipeline script from SCM" is configured
        git 'https://github.com/DevOpsCloudZone/docker_web_stack.git'
      }
    }
    stage("Build and Test") {
      steps {
        sh 'mvn clean install'
      }
    }
    stage("Code Quality Analysis") {
      steps {
        withSonarQubeEnv('mysonarqube') {
          // Here copy code from Sonarqube
          sh "mvn clean verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=sonarproject -Dsonar.projectName='sonarproject'"
        }
      }
    }
    stage('Code Quality Gates') {
      steps {
        waitForQualityGate abortPipeline: true, credentialsId: 'sonarqube'
      }
    }
    stage('Artifact Uploader') {
      steps {
        nexusArtifactUploader artifacts: [
          [artifactId: 'vprofile', classifier: '', file: 'target/vprofile-v2.war', type: 'war']
        ], credentialsId: 'nexusid', groupId: 'com.visualpathit', nexusUrl: '98.130.60.102:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'nexusrepo', version: 'v2'
      }
    }
    stage('Docker Build') {
      steps {
        sh 'docker build -t umesh2425/appstack:app -f Docker-app/Dockerfile .'
        sh 'docker build -t umesh2425/appstack:db -f Docker-db/Dockerfile .'
      }
    }
    stage('Docker Push') {
      steps {
        withCredentials([
          usernamePassword(
            credentialsId: 'dockerhub_id',
            usernameVariable: 'DOCKER_USERNAME',
            passwordVariable: 'DOCKER_PASSWORD'
          )
        ]) {
          sh '''
          echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
          docker push umesh2425/appstack:app
          docker push umesh2425/appstack:db
          docker logout
            '''
        }
      }
    }
     stage("Deploy"){
         steps{
             sh 'docker stack deploy -c compose.yml appstack'
         }
     }
    
  }
}
