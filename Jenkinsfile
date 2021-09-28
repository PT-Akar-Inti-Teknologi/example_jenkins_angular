pipeline {
  agent any

  stages {
    stage('Build & Test') {
      agent {
        docker {
          image 'node:14.17.0-alpine'
          reuseNode true
        }
      }
      steps {
        sh 'echo "deb http://dl.google.com/linux/chrome/deb/ stable main" | tee -a /etc/apt/sources.list'
        sh 'wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add -'
        sh 'apt-get update'
        sh 'apt-get install libxpm4 libxrender1 libgtk2.0-0 libnss3 libgconf-2-4'
        sh 'apt-get install google-chrome-stable'
        sh 'apt-get install xvfb gtk2-engines-pixbuf'
        sh 'apt-get install xfonts-cyrillic xfonts-100dpi xfonts-75dpi xfonts-base xfonts-scalable'
        sh 'apt-get install imagemagick x11-apps'

        sh 'npm install'
        sh 'npm run test:ci'
      }
    }

    // stage('Coding Standard') {
    //   agent {
    //     docker {
    //       image 'node:14.17.0-alpine'
    //       reuseNode true
    //     }
    //   }
    //   steps {
    //     sh 'yarn lint:test'
    //   }
    // }

    stage('Sonarqube Analysis') {
      environment {
        scannerHome = tool 'sonarqube-scanner'
      }

      steps {
        withSonarQubeEnv(installationName: 'sonarqube') {
          sh '$scannerHome/bin/sonar-scanner'
        }
      }
    }

    stage('Quality Gate') {
      steps {
        timeout(time: 1, unit: 'HOURS') {
          waitForQualityGate abortPipeline: true
        }
      }
    }
  }

  post {
    failure {
      emailext subject: "Jenkins Build ${currentBuild.currentResult}: Job ${env.JOB_NAME}",
        body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}",
        recipientProviders: [
          [$class: 'DevelopersRecipientProvider'],
          [$class: 'RequesterRecipientProvider']
        ]
    }

    always {
      cleanWs()
    }
  }
}
