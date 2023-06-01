pipeline {
  agent {
    label "centos-latest"
  }
  tools {
    maven 'apache-maven-latest'
    jdk 'temurin-jdk17-latest'
  }
  options {
    timestamps()
    disableConcurrentBuilds()
  }
  stages {
    stage('Build Packages') {
      steps {
        timeout(activity: true, time: 20) {
          sh '''
            mvn \
                clean verify \
                --batch-mode --show-version --errors \
                -Dtycho.disableP2Mirrors=true \
                -Peclipse-sign-jar -Peclipse-sign-mac -Peclipse-sign-dmg -Peclipse-sign-windows -Peclipse-package-dmg
            '''
        }
      }
    }
    stage('Check incubating') {
      steps {
        timeout(activity: true, time: 20) {
          sh '''
            cd $WORKSPACE/archive/
            $WORKSPACE/releng/org.eclipse.epp.config/tools/check-incubating.sh
            '''
        }
      }
    }
    stage('Upload to staging') {
      when {
        branch 'master'
      }
      steps {
        timeout(activity: true, time: 20) {
          sshagent ( ['projects-storage.eclipse.org-bot-ssh']) {
            sh './releng/org.eclipse.epp.config/tools/upload-to-staging.sh'
          }
        }
      }
    }
  }
}
