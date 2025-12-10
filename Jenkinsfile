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
    IMAGE_NAME = "${env.DOCKER_USERNAME}/daoudi-app"
    IMAGE_TAG = "${env.BUILD_NUMBER}"
    FULL_IMAGE_NAME = "${env.IMAGE_NAME}:${env.IMAGE_TAG}"
    K8S_NAMESPACE = 'default'
  }
  
  stages {
    stage('1️⃣ Checkout') {
      steps {
        echo '📥 Cloning repository from GitHub...'
        checkout([$class: 'GitSCM',
          branches: [[name: '*/main']],
          userRemoteConfigs: [[url: 'https://github.com/pycine/Daoudi_Muhammad_Yacine_4SLEAM2.git', credentialsId: env.GIT_CREDENTIALS]]
        ])
      }
    }
    
    stage('2️⃣ Build with Maven') {
      steps {
        echo '🔨 Building Spring Boot application...'
        sh 'mvn -B clean package -DskipTests'
      }
    }
    
    stage('3️⃣ Run Tests') {
      steps {
        echo '🧪 Running unit tests...'
        sh 'mvn test'
      }
      post {
        always {
          junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
        }
      }
    }
    
    stage('4️⃣ Build Docker Image') {
      steps {
        script {
          echo "🐳 Building Docker image: ${FULL_IMAGE_NAME}"
          sh "docker build -t ${FULL_IMAGE_NAME} ."
          sh "docker tag ${FULL_IMAGE_NAME} ${IMAGE_NAME}:latest"
        }
      }
    }
    
    stage('5️⃣ Push to Docker Hub') {
      steps {
        script {
          echo '📤 Pushing image to Docker Hub...'
          withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS, 
            usernameVariable: 'DOCKER_USER', 
            passwordVariable: 'DOCKER_PASS')]) {
            sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
            sh "docker push ${FULL_IMAGE_NAME}"
            sh "docker push ${IMAGE_NAME}:latest"
            sh 'docker logout'
          }
        }
      }
    }
    
    stage('6️⃣ Deploy to Kubernetes') {
      steps {
        script {
          echo '☸️ Deploying to Kubernetes cluster...'
          
          // Use kubeconfig from Jenkins credentials
          withCredentials([file(credentialsId: 'kubeconfig-k8s', variable: 'KUBECONFIG')]) {
            sh '''
              export KUBECONFIG=$KUBECONFIG
              
              echo "Testing kubectl connection..."
              kubectl cluster-info
              
              echo "Applying Kubernetes configurations..."
              kubectl apply -f all-in-one.yaml --namespace=${K8S_NAMESPACE}
              
              echo "Waiting for MySQL deployment to be ready..."
              kubectl rollout status deployment/mysql --namespace=${K8S_NAMESPACE} --timeout=5m || true
              
              echo "Updating Spring Boot deployment with new image..."
              kubectl set image deployment/spring-app spring-app=${FULL_IMAGE_NAME} --namespace=${K8S_NAMESPACE}
              
              echo "Waiting for Spring Boot deployment to be ready..."
              kubectl rollout status deployment/spring-app --namespace=${K8S_NAMESPACE} --timeout=5m
              
              echo "✅ Deployment completed successfully!"
            '''
          }
        }
      }
    }
    
    stage('7️⃣ Verify Deployment') {
      steps {
        echo '✅ Verifying deployment health...'
        withCredentials([file(credentialsId: 'kubeconfig-k8s', variable: 'KUBECONFIG')]) {
          sh '''
            export KUBECONFIG=$KUBECONFIG
            
            echo "=== Checking Pod Status ==="
            kubectl get pods --namespace=${K8S_NAMESPACE}
            
            echo ""
            echo "=== Checking Services ==="
            kubectl get svc --namespace=${K8S_NAMESPACE}
            
            echo ""
            echo "=== Waiting for pods to be ready ==="
            kubectl wait --for=condition=ready pod -l app=spring-app --namespace=${K8S_NAMESPACE} --timeout=5m || true
            kubectl wait --for=condition=ready pod -l app=mysql --namespace=${K8S_NAMESPACE} --timeout=5m || true
            
            echo ""
            echo "=== Application Logs (last 30 lines) ==="
            kubectl logs deployment/spring-app --namespace=${K8S_NAMESPACE} --tail=30 || echo "Could not fetch logs yet"
            
            echo ""
            echo "=== Deployment Details ==="
            kubectl describe deployment/spring-app --namespace=${K8S_NAMESPACE} | grep -A 5 "Image:"
            
            echo ""
            echo "=== All Resources ==="
            kubectl get all --namespace=${K8S_NAMESPACE}
          '''
        }
      }
    }
  }
  
  post {
    success {
      echo '🎉 ✅ Pipeline completed successfully!'
      echo "✓ Image built: ${FULL_IMAGE_NAME}"
      echo "✓ Image pushed to Docker Hub"
      echo "✓ Application deployed to Kubernetes"
      
      withCredentials([file(credentialsId: 'kubeconfig-k8s', variable: 'KUBECONFIG')]) {
        script {
          try {
            sh '''
              export KUBECONFIG=$KUBECONFIG
              echo ""
              echo "=== Final Status ==="
              kubectl get pods --namespace=${K8S_NAMESPACE}
              kubectl get svc spring-service --namespace=${K8S_NAMESPACE}
              echo ""
              echo "🌐 Access the application at: http://$(minikube ip):30080/student/"
            '''
          } catch (Exception e) {
            echo "Could not get deployment info"
          }
        }
      }
    }
    
    failure {
      echo '❌ Pipeline failed! Check the logs above for details.'
    }
    
    always {
      echo '🧹 Cleaning up...'
      sh 'docker logout || true'
      sh 'docker image prune -f || true'
    }
  }
}
