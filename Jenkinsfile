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

    stage('Maven Package') {
      steps {
        sh 'mvn package -DskipTests'
      }
    }

    stage('Docker Build') {
      steps {
        sh "docker build -t $IMAGE_NAME ."
      }
    }

    stage('Docker Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'acr-auth', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
          sh '''
            docker login dockerregmrudula.azurecr.io -u $USERNAME -p $PASSWORD
            docker push $IMAGE_NAME
          '''
        }
      }
    }

    stage('Deploy to AKS') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'spn-auth', usernameVariable: 'SPN_ID', passwordVariable: 'SPN_SECRET')]) {
          sh '''
            az login --service-principal -u $SPN_ID -p $SPN_SECRET --tenant 49bba7a4-424b-4070-a70e-886e9dd7caef
            az aks get-credentials --resource-group PetClinicRG --name petclinicAKS --overwrite-existing
            kubectl apply -f k8s/deployment.yaml --validate=false
            kubectl apply -f k8s/service.yaml --validate=false
          '''
        }
      }
    }
  }
}

