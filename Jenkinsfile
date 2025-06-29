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

        stage('SonarQube Analysis') 
            steps {
                withSonarQubeEnv('MySonar') {
                bat './mvnw sonar:sonar -Dsonar.projectKey=sb-petclinic -Dsonar.projectName=sb-petclinic -Dsonar.host.url=http://localhost:9000'
                }
            }

        stage('Upload to S3') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-access-key-id']]) {
                bat 'for %%f in (target\\*.jar) do aws s3 cp "%%f" s3://jenkins-pipeline-sb-petclinic/ --region ap-south-1'
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
