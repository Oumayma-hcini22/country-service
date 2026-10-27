pipeline {
    agent any
    tools {
        jdk 'JDK21'
        maven 'M2_HOME'
    }
    stages {
        stage('Checkout code') {
            steps {
                git branch: 'main', url: 'https://github.com/Oumayma-hcini22/country-service.git'
            }
        }
        stage('Compile, test code and package') {
            steps {
                sh 'mvn clean package'
            }
            post {
                success {
                    junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonarQubeServer') {
                    sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=country-service -Dsonar.projectName=country-service'
                }
            }
        }
        stage('Deploy to Nexus') {
            steps {
                sh 'mvn deploy -DskipTests'
                sh 'jar -cMf target/jacoco-report.zip -C target/site jacoco'
                sh 'jar -cMf target/surefire-reports.zip -C target surefire-reports'
                sh 'mvn deploy:deploy-file -Dfile=target/jacoco-report.zip -DgroupId=com.countryservice -DartifactId=country-service -Dversion=1.0.0 -Dclassifier=jacoco -Dpackaging=zip -DgeneratePom=false -DrepositoryId=nexus-releases -Durl=http://localhost:8081/repository/maven-releases/'
                sh 'mvn deploy:deploy-file -Dfile=target/surefire-reports.zip -DgroupId=com.countryservice -DartifactId=country-service -Dversion=1.0.0 -Dclassifier=surefire -Dpackaging=zip -DgeneratePom=false -DrepositoryId=nexus-releases -Durl=http://localhost:8081/repository/maven-releases/'
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
            archiveArtifacts artifacts: 'target/site/jacoco/**', allowEmptyArchive: true
        }
    }
}
