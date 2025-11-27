pipeline {
  agent any
  
  tools {
    maven 'MAVEN_HOME' 
    jdk 'JAVA_HOME' 
  }
  
  environment {
    GIT_CREDENTIALS = '1c128b54-6f0a-4c04-9f80-b14591dce8b7'
    DOCKER_REGISTRY = 'docker.io'
    DOCKER_CREDENTIALS = '07415ae6-8b78-4b14-b710-4cdb822432d6'
    DOCKER_USERNAME = 'yacineda'
    IMAGE_NAME = '${DOCKER_USERNAME}/daoudi-app'
    IMAGE_TAG = '${BUILD_NUMBER}'
    FULL_IMAGE_NAME = '${IMAGE_NAME}:${IMAGE_TAG}'
  }
  
  stages {
    stage('Checkout') {
      steps {
        checkout([$class: 'GitSCM',
          branches: [[name: '*/main']],
          userRemoteConfigs: [[url: 'https://github.com/pycine/Daoudi_Muhammad_Yacine_4SLEAM2.git', credentialsId: env.GIT_CREDENTIALS]]
        ])
      }
    }
    
    stage('Build') {
      steps {
        sh 'mvn -B clean package'
      }
    }
    
    stage('Build Docker Image') {
      steps {
        script {
          echo "Construction de l'image Docker: ${FULL_IMAGE_NAME}"
          sh 'docker build -t ${FULL_IMAGE_NAME} .'
          sh 'docker tag ${FULL_IMAGE_NAME} ${IMAGE_NAME}:latest'
        }
      }
    }
    
    stage('Push to Docker Hub') {
      steps {
        script {
          withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS, 
            usernameVariable: 'DOCKER_USER', 
            passwordVariable: 'DOCKER_PASS')]) {
            sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
            sh 'docker push ${FULL_IMAGE_NAME}'
            sh 'docker push ${IMAGE_NAME}:latest'
            sh 'docker logout'
          }
        }
      }
    }
  }
  
  post {
    success {
      echo '✓ Pipeline réussi! Image Docker poussée sur Docker Hub'
    }
    failure {
      echo '✗ Pipeline échoué'
    }
    always {
      sh 'docker logout || true'
    }
  }
}
