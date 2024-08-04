pipeline {
  agent {
    docker {
      image 'abhishekf5/maven-abhishek-docker-agent:v1'
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock' // mount Docker socket to access the host's Docker daemon
    }
  }
  stages {
    stage('Checkout') {
      steps {
        sh 'echo passed'
        // Uncomment and configure the following line to checkout code from your GitHub repository
        git branch: 'main', url: 'https://github.com/Nitingeek23/java-app.git'
      }
    }
    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
        // Build the project and create a JAR file
        sh 'cd java-maven-sonar-argocd-helm-k8s/spring-boot-app && mvn clean package'
      }
    }
    stage('Build Docker Image') {
      environment {
        DOCKER_IMAGE = "sunitabachhav2007/ultimate-cicd:${BUILD_NUMBER}"
      }
      steps {
        script {
            sh 'cd java-maven-sonar-argocd-helm-k8s/spring-boot-app && docker build -t ${DOCKER_IMAGE} .'
            // Docker login and push are removed
            // def dockerImage = docker.image("${DOCKER_IMAGE}")
            // docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
            //     dockerImage.push()
            // }
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
  }
}
