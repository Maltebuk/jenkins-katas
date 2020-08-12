
pipeline {
  agent any
  environment{
    docker_username = 'maltebuk'
  }
  stages {

    stage('clone down'){
      steps{
        stash(excludes: '.git',  name: 'code')
      }
    }

    stage('Parallel execution') {
      parallel {
        stage('Say Hello') {
          steps {
            sh 'echo "hello world" '
          }
        }

        stage('Build App') {
          agent {
            docker {
              image 'gradle:jdk11'
            }

          }
          steps {
            unstash 'code'
            sh 'ci/build-app.sh'
            archiveArtifacts 'app/build/libs/'
            sh 'ls'
            deleteDir()
            sh 'ls'
          }
        }
        stage('Test App'){
          agent{
            docker {
              image 'gradle:jdk11'
            }
          }
          options{
              skipDefaultCheckout()
          }
          steps{
              unstash 'code'
              sh 'ci/unit-test-app.sh'
              junit 'app/build/test-results/test/TEST-*.xml'
              
          }
        post {
        always {

          deleteDir() /* clean up our workspace */
        }
  }
        
      }
      stage('push to Docker app'){
      environment {
        DOCKERCREDS = credentials('docker_login') //use the credentials just created in this stage
      }
      steps {
        unstash 'code' //unstash the repository code
        stash(excludes: '.git',  name: 'code')

        sh 'ci/build-docker.sh'
        sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin' //login to docker hub with the credentials above
        sh 'ci/push-docker.sh'
      }

      }

      }
    }

  }
  
}