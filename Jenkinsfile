pipeline {
  environment{
    docker_username = 'clemme'
  }

  agent any
  stages {
    stage('clone down') {
      steps {
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
            stash 'code'
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
            junit 'app/build/test-results/test/TEST-*.xml'sdad
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
    
    stage('push docker app'){
      environment {
        DOCKERCREDS = credentials('docker_login') //use the credentials just created in this stage
      }
      when{branch "master"}
      steps {
        unstash 'code' //unstash the repository code
        sh 'ci/build-docker.sh'
        sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin' //login to docker hub with the credentials above
        sh 'ci/push-docker.sh'
      }
    }

    stage('component test'){
      when{
        not{
          branch 'dev/*'
        }
        beforeAgent true
      }
      steps{
        
        sh 'ci/component-test.sh'
      }
    }
  }
}
