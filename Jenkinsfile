pipeline {
    agent any
    
    environment {
        registry = "shossein/demo-boot"
        registryCredential = 'docker-hub'
        containerName = "demo-app"
        appPort = "8008"
        testPort = "8180"
        prodPort = "80"
        prodIP = "13.38.120.149"
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
             sh "docker stop $containerName || true"
             sh "docker rm $containerName || true"
             sh "docker run -d -p $testPort:$appPort --name $containerName $registry:$BUILD_NUMBER"
            }
        }

        stage("Deploy to prod env"){
          steps{
            input 'Do you approve deployment ?'
            sh "docker -H $prodIP stop $containerName || true"
            sh "docker -H $prodIP rm $containerName || true"
            sh "docker -H $prodIP run -d -p $prodPort:$appPort --name $containerName $registry:$BUILD_NUMBER"
          }

        stage("Clean test env"){
          steps{
            input 'Do you approve deployment ?'
            sh "docker stop $containerName"
            sh "docker rm $containerName"
            sh "docker rmi \$(docker images --filter=reference='shossein/demo-boot:*' -qa) -f"
          }
      }
    }
}
