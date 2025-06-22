pipeline {
  agent any
  tools {
    maven 'maven'
    jdk 'jdk-17'
  }
  environment {
    IMAGE_NAME = "dockerregmrudula.azurecr.io/petclinic:${BUILD_NUMBER}"
  }
  stages {
    stage('Checkout') {
      steps {
        git branch: 'prod', url: 'https://github.com/Mrudula-dev/enahanced-petclinc-springboot.git'
      }
    }

    stage('Maven Compile') {
      steps {
        sh 'mvn clean compile'
      }
    }

    stage('Maven Test') {
      steps {
        sh 'mvn test'
      }
    }

    stage('Trivy Scan') {
      steps {
        sh 'trivy fs --severity HIGH,CRITICAL --format table .'
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('SonarQube') {
          sh 'mvn sonar:sonar -Dsonar.projectKey=petclinic -Dsonar.sources=src'
        }
      }
    }

    stage('Maven Package') {
      steps {
        sh 'mvn package -DskipTests'
      }
    }

    stage('Docker Build') {
      steps {
        script {
          dockerImage = docker.build("$IMAGE_NAME")
        }
      }
    }

    stage('Docker Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'acr-auth', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
          sh 'docker login dockerregmrudula.azurecr.io -u $USERNAME -p $PASSWORD'
          sh "docker push $IMAGE_NAME"
        }
      }
    }

    stage('Deploy to AKS') {
      steps {
        sh 'kubectl apply -f k8s/deployment.yaml'
        sh 'kubectl apply -f k8s/service.yaml'
      }
    }
  }
}

    



