pipeline {
  agent {
    docker {
      image 'abhishekf5/maven-abhishek-docker-agent:v1'
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
    }
  }
  stages {
    stage('Clean Workspace') {
      steps {
        deleteDir()  // Clean the workspace
      }
    }
    stage('Test Commit') {
      steps {
        script {
          withCredentials([string(credentialsId: 'github-PAT', variable: 'GITHUB_TOKEN')]) {
            sh '''
              echo "Test commit to trigger pipeline" > test-trigger.txt
              git add test-trigger.txt
              git commit -m "Trigger pipeline for testing"
              git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
            '''
          }
        }
      }
    }
    stage('Checkout') {
      steps {
        sh 'echo passed'
        git branch: 'main', url: 'https://github.com/Nitingeek23/java-app.git'
      }
    }
    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
        sh 'cd java-maven-sonar-argocd-helm-k8s/spring-boot-app && mvn clean package'
      }
    }
    stage('Builds Docker Image') {
      environment {
        DOCKER_IMAGE = "sunitabachhav2007/ultimate-cicd:${BUILD_NUMBER}"
      }
      steps {
        script {
          sh 'cd java-maven-sonar-argocd-helm-k8s/spring-boot-app && docker build -t ${DOCKER_IMAGE} .'
        }
      }
    }
    stage('Update Deployment File') {
      environment {
        GIT_REPO_NAME = "java-app"
        GIT_USER_NAME = "Nitingeek23"
      }
      steps {
        withCredentials([string(credentialsId: 'github-PAT', variable: 'GITHUB_TOKEN')]) {
          sh '''
            git config user.email "nitinrjpt123@gmail.com"
            git config user.name "Nitingeek23"
            BUILD_NUMBER=${BUILD_NUMBER}
            sed -i -e "s/ultimate-cicd.*/ultimate-cicd:${BUILD_NUMBER}/g"  java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests/deployment.yml
            git add java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests/deployment.yml
            git commit -m "Update deployment image to version ${BUILD_NUMBER}"
            git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
          '''
        }
      }
    }
    stage('Deploy New Container') {
      environment {
        DOCKER_IMAGE = "sunitabachhav2007/ultimate-cicd:${BUILD_NUMBER}"
      }
      steps {
        script {
          sh '''
            existing_container=$(docker ps -q --filter "ancestor=${DOCKER_IMAGE}")
            if [ -n "$existing_container" ]; then
              docker stop "$existing_container"
              docker rm "$existing_container"
            fi
          '''
          sh '''
            docker run -d -p 8010:8080 ${DOCKER_IMAGE}
          '''
        }
      }
    }
  }
}
