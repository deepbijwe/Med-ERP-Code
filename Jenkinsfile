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

       stage("OWASP Dependency Check") {
    steps {
        dependencyCheck(
            additionalArguments: '--scan ./',
            odcInstallation: 'OWASP-DC'
        )

        dependencyCheckPublisher(
            pattern: '**/dependency-check-report.xml'
        )
    }
}


    }
}