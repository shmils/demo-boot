pipeline {
    agent any
    
    environment {
        registry = "shossein/demo-boot"
        registryCredential = 'docker-hub'
    }

    stages {
        stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }
        
        stage('Unit Tests - JUnit and Jacoco') {
          steps {
            sh "mvn test"
          }
          post {
            always {
              junit 'target/surefire-reports/*.xml'
              jacoco execPattern: 'target/jacoco.exec'
            }
          }
        }
        
        stage('Docker Build and Push') {
          steps {
            withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
              sh 'printenv'
              sh 'docker build -t $registry:$BUILD_NUMBER .'
              sh 'docker push $registry:$BUILD_NUMBER'
            }
          }
        }
        
        stage('Remove Unused docker image') {
          steps{
            sh "docker rmi $registry:$BUILD_NUMBER"
          }
        }
        
        stage("Deploy to test env"){
            steps{
             sh "docker stop demo-app || true"
             sh "docker rm demo-app || true"
             sh "docker run -d -p 8383:8080 --name demo-app $registry:$BUILD_NUMBER"    
            }
        }

	stage("Deploy to prod env"){
	   steps{
	     input 'Do you approve deployment ?'
	     sh "docker -H 13.38.120.149 stop demo-app || true"
             sh "docker -H 13.38.120.149 rm demo-app || true"
             sh "docker -H 13.38.120.149 run -d -p 80:8080 --name demo-app $registry:$BUILD_NU>
	   }
	}
    }
}
