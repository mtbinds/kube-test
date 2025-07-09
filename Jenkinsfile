pipeline {
  agent any

  environment {
    IMAGE_NAME = "flask-app"
    IMAGE_TAG = "latest"
    LOCAL_IMAGE = "${IMAGE_NAME}:${IMAGE_TAG}"
    IMAGE_FILE = "${IMAGE_NAME}.tar"
    SUDO_PASSWORD = credentials('jenkins-sudo-password') // ID dans Jenkins Credentials
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/mtbinds/kube-test.git'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh '''
          echo "Construction de l'image Docker locale"
          docker build -t $LOCAL_IMAGE .
        '''
      }
    }

    stage('Export and Import into MicroK8s') {
      steps {
        sh '''
          echo "Exportation de l'image Docker"
          docker save $LOCAL_IMAGE -o $IMAGE_FILE

          echo "Importation de l'image dans MicroK8s (containerd)"
          echo "$SUDO_PASSWORD" | sudo -S microk8s ctr image import $IMAGE_FILE
        '''
      }
    }

    stage('Déploiement MicroK8s') {
      steps {
        sh '''
          echo "Déploiement sur MicroK8s"
          echo "$SUDO_PASSWORD" | sudo -S microk8s kubectl apply -f deployment.yaml
        '''
      }
    }
  }

  post {
    success {
      echo 'Pipeline terminé avec succès.'
    }
    failure {
      echo 'Une erreur est survenue.'
    }
  }
}
