pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        S3_BUCKET = 'jenkins-pipeline-sb-petclinic'
        EC2_IP = '13.201.89.248' // Replace with actual IP
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
                    bat 'for %%f in (target\\*.jar) do aws s3 cp "%%f" s3://jenkins-pipeline-sb-petclinic/ --region ap-south-1'
                }
            }
        }

        stage('Deploy to EC2') {
            when {
                branch 'main'
            }
            steps {
                withCredentials([file(credentialsId: 'my-ec2-key', variable: 'KEY_FILE')]) {
                    bat '''
                        if exist target\spring-petclinic-3.5.0-SNAPSHOT.jar (
                            echo File found.
                        ) else (
                            echo ❌ JAR not found in target folder.
                        )

                        pscp -i %KEY_FILE% target\\spring-petclinic-3.5.0-SNAPSHOT.jar ubuntu@13.201.89.248:/home/ubuntu/app.jar
                        plink -i %KEY_FILE% ubuntu@13.201.89.248 "nohup java -jar /home/ubuntu/app.jar"
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
