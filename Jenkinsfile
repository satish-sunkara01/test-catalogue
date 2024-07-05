pipeline {
    agent {
        node {
            label 'agent'
        }
    }
    environment { 
        packageVersion = ''
        AWS_DEFAULT_REGION = 'us-east-1' // AWS region
        S3_BUCKET_NAME = 'my-tf-test-bucket-1' // Replace with your S3 bucket name
        APP_NAME = 'test-catalogue'
    }
    
    stages {
        stage('Get the version') {
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    packageVersion = packageJson.version
                    echo "application version: $packageVersion"
                }
            }
        }
        stage('Install dependencies') {
            steps {
                sh """
                    npm install
                """
            }
        }
        stage('Unit tests') {
            steps {
                sh """
                    echo "unit tests will run here"
                """
            }
        }
        stage('Sonar Scan'){
            steps{
                sh """
                    echo "sonar-scanner will be run here"
                """
            }
        }
        stage('Build') {
            steps {
                sh """
                    ls -la
                    zip -q -r catalogue.zip ./* -x ".git" -x "*.zip"
                    ls -ltr
                """
            }
        }
        stage('Deploy') {
            steps {
                // Upload artifacts to S3 bucket
                withAWS(region: AWS_DEFAULT_REGION, credentials: 'aws-credentials-id') {
                    s3Upload(bucket: S3_BUCKET_NAME, file: 'catalogue.zip')
                }
                
                // Deploy application (example using AWS Elastic Beanstalk)
                script {
                    def eb = "eb"
                    def version_label = "jenkins-${BUILD_NUMBER}"
                    
                    sh "${eb} init ${APP_NAME} --platform node.js --region ${AWS_DEFAULT_REGION}"
                    sh "${eb} deploy --version ${version_label}"
                    sh "${eb} setenv PARAM1=value1 PARAM2=value2" // Set environment variables if needed
                    sh "${eb} swap your-env-name --destination_name ${APP_NAME}"
                }
            }
        }
    }

        
    // post build
    post { 
        always { 
            echo 'I will always say Hello again!'
            deleteDir()
        }
        failure { 
            echo 'this runs when pipeline is failed, used generally to send some alerts'
        }
        success{
            echo 'I will say Hello when pipeline is success'
        }
    }
}