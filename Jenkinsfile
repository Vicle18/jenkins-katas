pipeline {
  agent any
  stages {
    stage('clone down'){
      steps{
        stash(excludes: '.git', name: 'code')
      }
      
    }

    stage('Parallel execution') {
      parallel {
        stage('Say Hello') {
          steps {
            sh 'echo "hello world"'
          }
        }

        stage('build app') {
          agent {
            docker {
              image 'gradle:jdk11'
            }

          }
          steps {
            unstash 'code'
            skipDefaultCheckout(true)
            sh 'ci/build-app.sh'
            archiveArtifacts 'app/build/libs/'
          }
          
          post {
              always {
                  sh 'ls'
                  deleteDir()
                  sh 'ls'
              }
          }
        }

        stage('test app') {
          agent {
            docker {
              image 'gradle:jdk11'
            }

          }
          steps {
            unstash 'code'
            sh label: '', script: 'ci/unit-test-app.sh'
            junit 'app/build/test-results/test/TEST-*.xml'
          }
          
          post {
              always {
                  sh 'ls'
                  deleteDir()
                  sh 'ls'
              }
          }
        }



      }
    }

  }
}
