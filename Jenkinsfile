pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        S3_BUCKET = 'jenkins-pipeline-sb-petclinic'
        EC2_IP = '13.235.80.153'
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

        stage('Upload to S3') {
            when {
                branch 'main'
            }
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-access-key-id']]) {
                    bat 'for %%f in (target\\*.jar) do aws s3 cp "%%f" %S3_BUCKET% --region %AWS_REGION%'
                }
            }
        }

        stage('Deploy to EC2') {
            when {
                branch 'main'
            }
            steps {
                
        
                
                bat '''
                        
                    pscp -i E:\\sb-petclinic.ppk target\\spring-petclinic-3.5.0-SNAPSHOT.jar ubuntu@13.201.89.248:/home/ubuntu/app.jar
                    plink -i E:\\sb-petclinic.ppk ubuntu@%EC2_IP% "pkill -f app.jar || true"
                    plink -i E:\\sb-petclinic.ppk ubuntu@%EC2_IP% "timeout 10 bash -c 'while pgrep -f app.jar > /dev/null; do sleep 1; done'"
                    plink -i E:\\sb-petclinic.ppk ubuntu@%EC2_IP% "nohup java -jar /home/ubuntu/app.jar > app.log 2>&1 &"

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
