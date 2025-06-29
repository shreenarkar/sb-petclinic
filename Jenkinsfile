pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                bat './mvnw clean package'
            }
        }

        stage('Test') {
            steps {
                bat './mvnw test'
            }
        }

        stage('Upload to S3') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-access-key-id']]) {
                    bat 'aws s3 cp target/*.jar s3://jenkins-pipeline-sb-petclinic/ --region ap-south-1'
                }
            }
        }
    }
}
