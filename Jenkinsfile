
pipeline {
  agent any
  tools {
    maven 'maven'
    jdk 'jdk-17'
  }
  environment {
  JAVA_HOME = "/usr/lib/jvm/java-17-openjdk-amd64"
  PATH = "/usr/lib/jvm/java-17-openjdk-amd64/bin:$PATH:$PATH"
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

    
    stage('Maven Package') {
      steps {
        sh 'mvn package -DskipTests'
      }
    }

    stage('Docker Build') {
  steps {
    sh 'docker build -t $IMAGE_NAME .'
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

    



