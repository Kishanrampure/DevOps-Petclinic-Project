pipeline {
    agent any
 tools {
        maven 'M3'
        jdk 'jdk'
    }
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Git Init') {
        steps{
            sh 'git init'
            }
        }
        stage('SCM Fetch From sc-staging') {
        steps{
            git branch: 'sc-staging', url: 'https://github.com/Kishanrampure/DevOps-Petclinic-Project.git'
            }
        }
        stage('Maven Compile') {
        steps {
                sh 'mvn clean compile'
            }
        }
        stage('Unit Test'){
        steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
	stage('Integration Test'){
        steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }
        stage ('Code Analysis With CheckStyle'){
        steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }
        stage("OWASP Dependency Check"){
        steps{
                dependencyCheck additionalArguments: '--scan ./ --format HTML ', odcInstallation: 'OWASP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.html'
            }
        }
        stage('Trivy FS Check CI') {
        steps {
                sh "trivy fs ."
            }
        }
        stage('SonarQube Analysis') {
            environment {
             SCANNER_HOME = tool 'sonar-scanner'
          }
        steps{
                withSonarQubeEnv('sonarserver') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petclinic \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Petclinic '''

                }
            }
        }
        stage('Git Push to deployment') {
	  environment {
                commitmsg = "'CodeChange'"
            }
        steps {
	       script{
                   withCredentials([
                    gitUsernamePassword(credentialsId: 'mygitid', gitToolName: 'Default')
                    ] ) {
                    sh '''
                    git add .
		            git remote -v
                    git status
		            git commit -a -m ${commitmsg}
                    git branch -M deployment
                    git push -u origin deployment --force
                    '''
                  } 
	       }
            }
        }
        stage('SCM Fetch From deployment') {
        steps{
            git branch: 'deployment', url: 'https://github.com/Kishanrampure/DevOps-Petclinic-Project.git'
            }
        }
         stage('Docker Build') {
          steps {
                sh 'chmod +x mvnw'
                sh 'chmod +x ./mvnw'
                sh 'docker build -t kishanrampure/petclinic:v37052024 .'
            }
        }
        stage('Docker Image Test'){
        steps {
                sh 'trivy image kishanrampure/petclinic:v37052024'
            }
        }
        stage('Trivy FS Check CD') {
        steps {
                sh "trivy fs ."
            }
        }
        stage('Push image to dockerhub') {
        steps {
          withCredentials([usernamePassword(credentialsId: 'docker-cred', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
             sh 'docker login -u $USERNAME -p $PASSWORD'
             sh 'docker push kishanrampure/petclinic:v37052024'
          }
        }
      }
        stage('Git Push to main') {
	  environment {
                commitmsg = "'codeChanges'"
            }
        steps {
	       script{
                   withCredentials([
                    gitUsernamePassword(credentialsId: 'mygitid', gitToolName: 'Default')
                    ] ) {
                    sh '''
                    git add .
                    git branch -M main
		            git remote -v
                    git status
		            git commit -a -m ${commitmsg}
                    git push -u origin main --force
                    '''
            }
        }
    } 
}

    } 
}
