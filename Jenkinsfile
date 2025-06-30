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

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonar') {
                    bat 'mvn sonar:sonar -Dsonar.projectKey=sb-petclinic -Dsonar.projectName=sb-petclinic -Dsonar.host.url=http://localhost:9000'
                }
            }
        }

        stage('Upload to S3')
            when{
                branch 'main'
            }
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-access-key-id']]) {
                    bat 'for %%f in (target\\*.jar) do aws s3 cp "%%f" s3://jenkins-pipeline-sb-petclinic/ --region ap-south-1'
                }
            }
        }

        stage('Deploy to EC2') {
            when {
                branch 'main'
            }
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'my-ec2-key', keyFileVariable: 'KEY_FILE')]) {
                    sh '''
                        scp -i $KEY_FILE target/*.jar ec2-user@<EC2_PUBLIC_IP>:/home/ec2-user/app.jar
                        ssh -i $KEY_FILE ec2-user@<EC2_PUBLIC_IP> 'nohup java -jar /home/ec2-user/app.jar > output.log 2>&1 &'
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Build, test, analysis, upload, and deploy successful.'
        }
        failure {
            echo '❌ Pipeline failed. Please check the logs.'
        }
    }
}
