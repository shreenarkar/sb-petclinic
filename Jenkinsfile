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
                    script {
                        def timeStamp = new Date().format("yyyyMMdd_HHmm")
                        def jarName = "spring-petclinic-3.5.0-SNAPSHOT"
                        def fileName = "${jarName}_${timeStamp}.jar"

                        // Rename jar file with timestamp
                        bat "copy target\\${jarName}.jar target\\${fileName}"

                        // Upload to S3
                        bat "aws s3 cp target\\${fileName} s3://%S3_BUCKET% --region %AWS_REGION%"
                    }
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
                    plink -i E:\\sb-petclinic-2.ppk ubuntu@%EC2_IP% "pkill -f 'java.*app.jar' || echo 'App not running before deployment'"
                    plink -i E:\\sb-petclinic-2.ppk ubuntu@%EC2_IP% "sleep 2"
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
