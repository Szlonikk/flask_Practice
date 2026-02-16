pipeline {
  agent any

  environment {
    TARGET_HOST = "ec2-51-20-18-29.eu-north-1.compute.amazonaws.com"
    TARGET_USER = "ec2-user"
    SSH_CRED    = "ec2-target-ssh"
    APP_DIR     = "/opt/flask-practice"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Package') {
      steps {
        sh '''
          rm -rf dist
          mkdir -p dist
          tar --exclude=.git --exclude=dist -czf dist/app.tar.gz .
        '''
      }
    }

    stage('Deploy') {
      steps {
        sshagent(credentials: [env.SSH_CRED]) {
          sh '''
            scp -o StrictHostKeyChecking=no dist/app.tar.gz ${TARGET_USER}@${TARGET_HOST}:/tmp/app.tar.gz

            ssh -o StrictHostKeyChecking=no ${TARGET_USER}@${TARGET_HOST} "
              set -e
              TS=\$(date +%Y%m%d%H%M%S)
              mkdir -p ${APP_DIR}/releases/\$TS
              tar -xzf /tmp/app.tar.gz -C ${APP_DIR}/releases/\$TS

              ${APP_DIR}/venv/bin/pip install -r ${APP_DIR}/releases/\$TS/requirements.txt

              rm -rf ${APP_DIR}/current
              ln -s ${APP_DIR}/releases/\$TS ${APP_DIR}/current

              sudo systemctl daemon-reload
              sudo systemctl restart flask-practice
              sudo systemctl restart httpd || sudo systemctl restart apache2
            "
          '''
        }
      }
    }
  }
}

