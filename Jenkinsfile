pipeline {
  agent any

  tools {
    // Asegúrate de tener configurado el tool 'Python3' en Jenkins
    python 'Python3'
  }

  environment {
    SONARQUBE_SERVER = 'sonarqube'
    SONAR_SCANNER_HOME = '/var/jenkins_home/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarScanner'
    VENV = "${WORKSPACE}/venv"
  }

  stages {
    stage('Instalar dependencias') {
      steps {
        sh '''
          python -m venv ${VENV}
          . ${VENV}/bin/activate
          pip install --upgrade pip
          pip install -r requirements.txt
        '''
      }
    }

    stage('Lint') {
      steps {
        sh '''
          . ${VENV}/bin/activate
          pip install pylint
          pylint src --output-format=json > pylint-report.json || true
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
