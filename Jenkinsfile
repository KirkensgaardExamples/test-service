import groovy.text.StreamingTemplateEngine

def emailTemplate(params) {

  def fileName = "email-html.template"
  def fileContents = readFile(fileName)
  def engine = new StreamingTemplateEngine()
  return engine.createTemplate(fileContents).make(params).toString()

}

pipeline {
    environment{
            name = "@name@"
            dockerImage = "@dockerImage@"
            providedEmail = "@providedEmail@"
            gitUrl = "@gitUrl@"
    }
    agent any

    stages {
        stage("Parallel Testing"){
            parallel    {
                stage('Build') {
                    steps {
                        sh 'chmod +x gradlew'
                        sh './gradlew clean build'
                    }
                }
                stage('Unit Test') {
                    steps {
                        echo "Finished Unit Testing"
                    }
                }
                stage('Integration Test') {
                    steps {
                        echo "Finished Integration Testing"
                    }
                }
                stage('Performance Test') {
                    steps {
                        echo "Finished Performance Testing"
                    }
                }
                stage('Code Report Generation'){
                    steps {
                        echo "Code Report Generated"
                    }
                }
            }
        }
        stage('Build Docker Image'){
            steps{
                sh "docker build -t ${dockerImage} ."
            }
        }
        stage('Run Docker Image'){
            steps{
                sh "docker run -d --name gatling-test-container -p 8088:8080 ${dockerImage}"
            }
        }
        stage('Load Test of Application'){
            steps{
                echo 'Insert Gatling Here'
            }
        }
        stage('Stop Docker Image'){
            steps{
                sh "docker stop gatling-test-container"
                sh "docker container rm gatling-test-container"
            }
        }
        stage('Push Image to Docker Hub'){
            steps{
                sh 'docker login --username=kirkensgaardexamples --password=Right55HDuke'
                sh "docker push ${dockerImage}"
            }
        }
    }
    post{
        success{
            script{
          generatedBody = emailTemplate([
            "name"   :   env.name,
            "dockerHub"    :   env.dockerImage,
            "gitUrl" :   env.gitUrl
        ]);
            }
            emailext body: generatedBody,
                      mimeType: 'text/html',
                    subject: "Your Project is now on DockerHub!",
                    to: "${providedEmail}"
        }
        failure{
            echo 'failed'
        }
    }
}