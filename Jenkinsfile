pipeline {
    agent none

    stages {
        stage('Worker-Build') {
            agent {
              docker {
                image 'maven:3.6.1-alpine'
                args '-v $HOME/.m2:/root/.m2'
              }
            }
            when{
              changeset "**/worker/**"
            }
            steps {
                echo 'Compiling worker app'
                dir('worker'){
                  sh 'mvn compile'
		            }
            }
        }
        stage('Worker-Test') {
            agent {
              docker {
                image 'maven:3.6.1-alpine'
                args '-v $HOME/.m2:/root/.m2'
              }
            }
	          when{
              changeset "**/worker/**"
            }
            steps {
                echo 'Testing worker app'
                dir('worker'){
                  sh 'mvn clean test'
		            }
            }
        }
        stage('Worker-Package') {
            agent {
              docker {
                image 'maven:3.6.1-alpine'
                args '-v $HOME/.m2:/root/.m2'
              }
            }
            when{
              changeset "**/worker/**"
              branch "master"
            }
            steps {
                echo 'Packaging worker app'
                dir('worker'){
		              sh 'mvn package -DskipTests'
                  archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
		            }
            }
        }
        stage('Worker-Docker Package') {
            agent any
            when{
              changeset "**/worker/**"
              branch "master"
            }
            steps {
                echo 'Creating docker image for worker app'
                dir('worker'){
                  script{
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-credentials') {
                      def dockerImage = docker.build("amouskite/worker:${env.BUILD_ID}")
                      dockerImage.push()
                      dockerImage.push('master')
                    }
                  }
		            }
            }
        }
        stage('Result-Build') {
            agent {
              docker {
                image 'node:8.9.1-alpine'
              }
            }  
            when{
              changeset "**/result/**"
            }
            steps {
                echo 'Compiling result app'
                dir('result'){
                  sh 'npm install'
		            }
            }
        }
        stage('Result-Test') {
            agent {
              docker {
                image 'node:8.9.1-alpine'
              }
            }           
	          when{
              changeset "**/result/**"
            }
            steps {
                echo 'Testing result app'
                dir('result'){
                  sh "npm install"
                  sh 'npm test'
		            }
            }
        }
         stage('Result-Docker Package') {
            agent any
            when{
              changeset "**/result/**"
              branch "master"
            }
            steps {
                echo 'Creating docker image for result app'
                dir('result'){
                  script{
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-credentials') {
                      def dockerImage = docker.build("amouskite/result:${env.BUILD_ID}")
                      dockerImage.push()
                      dockerImage.push('master')
                    }
                  }
		            }
            }
        }
        stage('Vote-Build') {
            agent {
              docker {
                image 'python:2.7.16-slim'
                args '--user root'
              }
            }
            when{
              changeset "**/vote/**"
            }
            steps {
                echo 'Compiling vote app'
                dir('vote'){
                  sh 'pip install -r requirements.txt'
		            }
            }
        }
        stage('Vote-Test') {
            agent {
              docker {
                image 'python:2.7.16-slim'
                args '--user root'
              }
            }          
	          when{
              changeset "**/vote/**"
            }
            steps {
                echo 'Testing vote app'
                dir('vote'){
                  sh 'pip install -r requirements.txt'
                  sh 'nosetests -v'
		            }
            }
        }
        stage('Vote-Integration Test') {
            agent any          
	          when{
              changeset "**/vote/**"
              branch "master"
            }
            steps {
                echo 'Running Integration Tests for vote app'
                dir('vote'){
                  sh 'integration_test.sh'
		            }
            }
        }
        stage('Vote-Docker Package') {
            agent any
            when{
              changeset "**/vote/**"
              branch "master"
            }
            steps {
                echo 'Creating docker image for vote app'
                dir('vote'){
                  script{
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-credentials') {
                      def dockerImage = docker.build("amouskite/vote:${env.BUILD_ID}")
                      dockerImage.push()
                      dockerImage.push('master')
                    }
                  }
		            }
            }
        }
         stage('Sonarqube') {
          agent any
          when{
            branch 'master'
          }
          tools {
            jdk "JDK11" // the name you have given the JDK installation in Global Tool Configuration
          }

          environment{
            sonarpath = tool 'SonarScanner'
          }

          steps {
                echo 'Running Sonarqube Analysis..'
                withSonarQubeEnv('sonarcloud-instavote') {
                  sh "${sonarpath}/bin/sonar-scanner -Dsonar.branch.target=${BRANCH_NAME} -Dsonar.branch.name=${BRANCH_NAME} -Dproject.settings=sonar-project.properties -Dorg.jenkinsci.plugins.durabletask.BourneShellScript.HEARTBEAT_CHECK_INTERVAL=86400"
                }
          }
        }

        stage("Quality Gate") {
          agent any
            when{
              branch 'master'
            }
            steps {
              withSonarQubeEnv('sonarcloud-instavote') {
                timeout(time: 1, unit: 'MINUTES') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
          }
        }

        stage('deploy to dev'){
          agent any
          when{
            branch 'master'
          }
          steps{
            echo 'Deploy instavote app with docker compose'
            sh 'docker-compose up -d'
          }
      }
        
    }

}
