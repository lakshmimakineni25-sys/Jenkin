pipeline {
  agent any

  environment {
    IMAGE_NAME = "lakshmimakineni25-sys/firstapp"
    DOCKER_CREDENTIALS = "dockerhub-creds"   // replace with your Jenkins credential ID
  }

  stages {
    stage('Checkout') {
      steps {
        // Jenkins goes and grabs the code
        git branch: 'main', url: 'https://github.com/lakshmimakineni25-sys/Jenkin.git', credentialsId: 'git-creds'
      }
    }

    stage('Build') {
      steps {
        echo 'Build: compile/package the app'
        // example for maven: compile and produce target/*.jar
        sh 'mvn -B -DskipTests package || true'  // change to your build command
      }
    }

    stage('Test') {
      steps {
        echo 'Test: run unit tests'
        sh 'mvn test || true'  // adapt to your test command
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          // tag image with build number so each build has unique tag
          def img = docker.build("${IMAGE_NAME}:${env.BUILD_NUMBER}")
          // also tag latest locally (optional)
          img.tag("latest")
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        script {
          docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS}") {
            // push build-number tagged image AND latest
            docker.image("${IMAGE_NAME}:${env.BUILD_NUMBER}").push()
            docker.image("${IMAGE_NAME}:latest").push()
          }
        }
      }
    }

    stage('Deploy') {
      steps {
        echo 'Deploy: pull image and restart container on this server'
        // stop if running, remove, pull new image, run detached
        sh """
          docker pull ${IMAGE_NAME}:${env.BUILD_NUMBER} || true
          docker stop myapp || true
          docker rm myapp || true
          docker run -d --name myapp -p 80:8080 ${IMAGE_NAME}:${env.BUILD_NUMBER}
        """
      }
    }
  }

  post {
    success { echo "Build succeeded!" }
    failure { echo "Build failed. Check console output." }
  }
}
