pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        S3_BUCKET = 'jenkins-pipeline-sb-petclinic'
    }

    stages {
        stage('Build') {
            steps {
                bat 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                bat 'mvn test'
            }
        }

        stage('Upload to S3') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-access-key-id']]) {
                    bat 'for /f %f in (\'dir /b target\\*.jar\') do aws s3 cp target\\%f s3://%S3_BUCKET%/ --region %AWS_REGION%'
                }
            }
        }
    }

    post {
        success {
            echo 'Build and upload successful.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
