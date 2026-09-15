pipeline {
    agent any

    environment {
        SONAR_HOME = tool "Sonar"
        AWS_Region = "ap-south-1"
        AWS_Account_ID = "360964565562"
    }

    stages {

        stage("Git") {
            steps {
                git url: "https://github.com/deepbijwe/Med-ERP-Code.git",
                    branch: "dev"
            }
        }

        stage("Test") {
            steps {
                sh '''
                    cd order-service
                    mvn test

                    cd ../user-service
                    mvn test

                    cd ../product-service
                    mvn test
                '''
            }
        }

        stage("Build") {
            steps {
                sh '''
                    cd order-service
                    mvn clean compile -DskipTests

                    cd ../user-service
                    mvn clean compile -DskipTests

                    cd ../product-service
                    mvn clean compile -DskipTests
                '''
            }
        }

        stage("Sonarqube-Quality-Analysis") {
            steps {
                withSonarQubeEnv("Sonar") {
                    sh '''
                        $SONAR_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=MED-ERP \
                        -Dsonar.projectKey=MED-ERP \
                        -Dsonar.sources=. \
                        -Dsonar.java.binaries=order-service/target/classes,user-service/target/classes,product-service/target/classes
                    '''
                }
            }
        }

        stage("Sonar Quality Gate Scan") {
            steps {
                timeout(time: 2, unit: "MINUTES") {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage("OWASP Dependency Check") {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'nvd-api-key',
                        variable: 'NVD_API_KEY'
                    )
                ]) {
                    dependencyCheck(
                        additionalArguments: '--scan ./ --nvdApiKey ' + NVD_API_KEY,
                        odcInstallation: 'OWASP-DC'
                    )
                }

                dependencyCheckPublisher(
                    pattern: '**/dependency-check-report.xml'
                )
            }
        }

        stage("Trivy FS Scan") {
            steps {
                sh '''
                    trivy fs \
                    --severity HIGH,CRITICAL \
                    --exit-code 0 \
                    .
                '''
            }
        }
        
        stage('Cleanup') {
    steps {
        sh '''
            docker image prune -af
            docker container prune -f
        '''
    }
}

        stage('Docker Build') {
            steps {
        sh '''
            docker build -t deep/order:$BUILD_NUMBER ./order-service
            docker build -t deep/user:$BUILD_NUMBER ./user-service
            docker build -t deep/product:$BUILD_NUMBER ./product-service
            docker images
        '''
            }
        }
stage('Trivy Image Scan') {
    steps {
        sh '''
            trivy image --severity HIGH,CRITICAL --exit-code 0 --format table \
                -o trivy-order-service-report.txt deep/order-service:$BUILD_NUMBER

            trivy image --severity HIGH,CRITICAL --exit-code 0 --format table \
                -o trivy-user-service-report.txt deep/user-service:$BUILD_NUMBER

            trivy image --severity HIGH,CRITICAL --exit-code 0 --format table \
                -o trivy-product-service-report.txt deep/product-service:$BUILD_NUMBER
        '''
    }
    post {
        always {
            archiveArtifacts artifacts: 'trivy-*-report.txt', allowEmptyArchive: true
        }
    }
}
            stage('Push Docker Images to ECR') {
                 steps {
                     withCredentials ([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS_Creds']]) {
                    sh '''
                        aws ecr get-login-password --region $AWS_Region | docker login --username AWS --password-stdin $AWS_Account_ID.dkr.ecr.$AWS_Region.amazonaws.com

                        docker tag deep/order:$BUILD_NUMBER $AWS_Account_ID.dkr.ecr.$AWS_Region.amazonaws.com/deep/order:$BUILD_NUMBER
                        docker tag deep/user:$BUILD_NUMBER $AWS_Account_ID.dkr.ecr.$AWS_Region.amazonaws.com/deep/user:$BUILD_NUMBER
                        docker tag deep/product:$BUILD_NUMBER $AWS_Account_ID.dkr.ecr.$AWS_Region.amazonaws.com/deep/product:$BUILD_NUMBER

                        docker push $AWS_Account_ID.dkr.ecr.$AWS_Region.amazonaws.com/deep/order:$BUILD_NUMBER
                        docker push $AWS_Account_ID.dkr.ecr.$AWS_Region.amazonaws.com/deep/user:$BUILD_NUMBER
                        docker push $AWS_Account_ID.dkr.ecr.$AWS_Region.amazonaws.com/deep/product:$BUILD_NUMBER
                    '''
                }

            }
        }
    }
}    