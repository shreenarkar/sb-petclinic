pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        S3_BUCKET = 'jenkins-pipeline-sb-petclinic'
        EC2_IP = '13.201.226.110'
    }

    stages {
        stage('Build') {
            when {
                not {
                    branch 'main'
                }
            }
            steps {
                bat 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            when {
                not {
                    branch 'main'
                }
            }
            steps {
                bat 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            when {
                not {
                    branch 'main'
                }
            }
            steps {
                withSonarQubeEnv('MySonar') {
                    bat 'mvn sonar:sonar -Dsonar.projectKey=sb-petclinic -Dsonar.projectName=sb-petclinic -Dsonar.host.url=http://localhost:9000'
                }
            }
        }

        stage('Upload to S3') {
            when {
                branch 'main'
            }
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-access-key-id']]) {
                    bat 'for %%f in (target\\*.jar) do aws s3 cp "%%f" s3://%S3_BUCKET% --region %AWS_REGION%'
                }
            }
        }

        stage('Deploy to EC2') {
            when {
                branch 'main'
            }
            steps {
                bat '''
                    pscp -i E:\\sb-petclinic-2.ppk target\\spring-petclinic-3.5.0-SNAPSHOT.jar ubuntu@%EC2_IP%:/home/ubuntu/app.jar
                    plink -i E:\\sb-petclinic-2.ppk ubuntu@%EC2_IP% "pkill -f app.jar || true"
                    plink -i E:\\sb-petclinic-2.ppk ubuntu@%EC2_IP% "while pgrep -f app.jar > /dev/null; do sleep 1; done"
                    plink -i E:\\sb-petclinic-2.ppk ubuntu@%EC2_IP% "nohup java -jar /home/ubuntu/app.jar > app.log 2>&1 &"
                '''
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
