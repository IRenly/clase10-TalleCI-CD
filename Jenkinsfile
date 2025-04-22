pipeline {
  agent any

  environment {
    SONARQUBE_SERVER = 'sonarqube'
    SONAR_SCANNER_HOME = '/var/jenkins_home/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarScanner'
    VENV = "${WORKSPACE}/venv"
  }

  stages {
    stage('Check Python Version') {
      steps {
        sh 'python3 --version'
      }
    }

    stage('Clonar repositorio') {
      steps {
        git branch: 'main', url: 'https://github.com/IRenly/clase10-TalleCI-CD'
      }
    }

    stage('Instalar dependencias') {
      steps {
        sh '''
          python3 -m venv venv
          venv/bin/pip install --upgrade pip
          venv/bin/pip install -r requirements.txt
        '''
      }
    }

    stage('Pruebas unitarias') {
      steps {
        sh 'venv/bin/pytest --maxfail=1 --disable-warnings --quiet'
      }
    }

    stage('Construir imagen Docker') {
      steps {
        script {
          dockerImage = docker.build("irenly/clase10-talleci-cd")
        }
      }
    }

    stage('Login to DockerHub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh '''
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
          '''
        }
      }
    }

    stage('Push to DockerHub') {
      steps {
        script {
          docker.image('irenly/clase10-talleci-cd').push()
        }
      }
    }

    stage('Lint') {
      steps {
        sh '''
          venv/bin/pip install pylint
          venv/bin/pylint src --output-format=json > pylint-report.json || true
        '''
      }
    }

    stage('Análisis SonarQube') {
      steps {
        withSonarQubeEnv("${SONARQUBE_SERVER}") {
          withCredentials([string(credentialsId: 'sonarqube_auth_token', variable: 'SONAR_TOKEN')]) {
            sh """
              ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                -Dsonar.projectKey=my-python-app \
                -Dsonar.sources=src \
                -Dsonar.host.url=http://sonarqube:9000 \
                -Dsonar.token=${SONAR_TOKEN} \
                -Dsonar.python.coverage.reportPaths=coverage.xml \
                -Dsonar.python.pylint.reportPaths=pylint-report.json
            """
          }
        }
      }
    }
  }
}
