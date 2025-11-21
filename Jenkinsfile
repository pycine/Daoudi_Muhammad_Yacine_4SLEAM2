pipeline {
  agent any

  tools {
    maven 'MAVEN_HOME' 
    jdk 'JAVA_HOME' 
  }

  environment {
    GIT_CREDENTIALS = '1c128b54-6f0a-4c04-9f80-b14591dce8b7'
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
        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
      }
    }

  
    }
  }

  post {
    success {
      echo 'Build succeeded'
    }
    failure {
      echo 'Build failed'
    }
  }
}
