pipeline {
    agent {
      label "jenkins-jx-base"
    }
    environment {
      ORG               = 'venkatvar'
      APP_NAME          = 'testmysql-157'
      CHARTMUSEUM_CREDS = credentials('jenkins-x-chartmuseum')
    }
    stages {
      stage('Build Release') {
        when {
          branch env.BRANCH_NAME
        }
        steps {
          container('jx-base') {
            // ensure we're not on a detached head
            sh "git checkout ${env.BRANCH_NAME}"
            sh "git config --global credential.helper store"
            sh "jx step validate --min-jx-version 1.1.73"
            sh "jx step git credentials"
            // so we can retrieve the version in later steps
            sh "echo \$(jx-release-version) > VERSION"
          }
          dir ('./charts/testmysql-157') {
            container('jx-base') {
              sh "make tag"
            }
          }
        }
      }
      stage('Promote to Environments') {
        when {
          branch env.BRANCH_NAME
        }
        steps {
          dir ('./charts/testmysql-157') {
            container('jx-base') {
              // release the helm chart
              sh 'jx step helm release'

              // promote through all 'Auto' promotion Environments
              // sh 'jx promote -b --all-auto --timeout 1h --version \$(cat ../../VERSION)'
              sh 'jx step helm apply --namespace=jx-staging --name=testmysql-157'
            }
          }
        }
      }
    }
    post {
        always {
            cleanWs()
        }
    }
  }
