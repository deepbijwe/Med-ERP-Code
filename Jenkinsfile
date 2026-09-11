pipeline {
    agent any

    environment {
        SONAR_HOME = tool "Sonar"
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
        stage('Docker Build') {
            steps {
        sh '''
            docker build -t deep/order-service:$BUILD_NUMBER ./order-service
            docker build -t deep/user-service:$BUILD_NUMBER ./user-service
            docker build -t deep/product-service:$BUILD_NUMBER ./product-service
        '''
            }
        }
    }
}    