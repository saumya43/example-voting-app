pipeline {
  agent none
  stages {
    stage('worker-build') {
      agent {
        docker {
          image 'maven:3.6.1-jdk-8-alpine'
          args '-v $HOME/.m2:/root/.m2'
        }

      }
      when {
        changeset '**/worker/**'
      }
      steps {
        echo 'compiling worker app'
        dir(path: 'worker') {
          sh 'mvn compile'
        }

      }
    }

    stage('worker-test') {
      agent {
        docker {
          image 'maven:3.6.1-jdk-8-alpine'
          args '-v $HOME/.m2:/root/.m2'
        }

      }
      when {
        changeset '**/worker/**'
      }
      steps {
        echo 'running unit tests on worker app'
        dir(path: 'worker') {
          sh 'mvn clean test'
        }

      }
    }

    stage('worker-package') {
      agent {
        docker {
          image 'maven:3.6.1-jdk-8-alpine'
          args '-v $HOME/.m2:/root/.m2'
        }

      }
      when {
        branch 'master'
        changeset '**/worker/**'
      }
      steps {
        echo 'packaging worker app'
        dir(path: 'worker') {
          sh 'mvn package -DskipTests'
          archiveArtifacts(artifacts: '**/target/*.jar', fingerprint: true)
        }

      }
    }

    stage('worker-docker-package') {
      agent any
      when {
        branch 'master'
        changeset '**/worker/**'
      }
      steps {
        echo 'packaging worker app with docker'
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin'){
            def workerImage = docker.build("saumya043/worker:v${env.BUILD_ID}", "./worker")
            workerImage.push()
            workerImage.push("${env.BRANCH_NAME}")
          }
        }

      }
    }

    stage('result-build') {
      agent {
        docker {
          image 'node:8.13.0-alpine'
        }

      }
      when {
        changeset '**/result/**'
      }
      steps {
        echo 'compiling result app'
        dir(path: 'result') {
          sh 'npm install'
        }

      }
    }

    stage('result-test') {
      agent {
        docker {
          image 'node:8.13.0-alpine'
        }

      }
      when {
        changeset '**/result/**'
      }
      steps {
        echo 'running unit tests on result app'
        dir(path: 'result') {
          sh 'npm install'
          sh 'npm test'
        }

      }
    }

    stage('result-docker-package') {
      agent any
      when {
        branch 'master'
        changeset '**/result/**'
      }
      steps {
        echo 'packaging result app with docker'
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin'){
            def resultImage = docker.build("saumya043/result:v${env.BUILD_ID}", "./result")
            resultImage.push()
            resultImage.push("${env.BRANCH_NAME}")
          }
        }

      }
    }

    stage('vote-build') {
      agent {
        docker {
          image 'python:2.7.16-slim'
          args '--user root'
        }

      }
      when {
        changeset '**/vote/**'
      }
      steps {
        echo 'compiling vote app'
        dir(path: 'vote') {
          sh 'pip install -r requirements.txt'
        }

      }
    }

    stage('vote-test') {
      agent {
        docker {
          image 'python:2.7.16-slim'
          args '--user root'
        }

      }
      when {
        changeset '**/vote/**'
      }
      steps {
        echo 'running unit tests on vote app'
        dir(path: 'vote') {
          sh 'pip install -r requirements.txt'
          sh 'nosetests -v'
        }

      }
    }

    stage('vote-docker-package') {
      agent any
      when {
        branch 'master'
        changeset '**/vote/**'
      }
      steps {
        echo 'packaging vote app with docker'
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin'){
            def voteImage = docker.build("saumya043/vote:v${env.BUILD_ID}", "./vote")
            voteImage.push()
            voteImage.push("${env.BRANCH_NAME}")
          }
        }

      }
    }

    stage('Deploy to Dev') {
      agent any
      steps {
        sh 'docker-compose up -d'
      }
    }

  }
  post {
    always {
      echo 'Pipeline for instavote app  is completed..'
    }

    failure {
      slackSend(channel: 'instavote-cd', message: "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
    }

    success {
      slackSend(channel: 'instavote-cd', message: "Build Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
    }

  }
}