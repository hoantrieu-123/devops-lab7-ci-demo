pipeline {
    agent any
    tools {
        jdk 'JDK21'
        maven 'M3'
    }
    environment {
        NEXUS_URL = 'http://lab7-nexus:8081'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                dir('sample-java-app') {
                    sh 'mvn -B clean compile'
                }
            }
        }
        stage('Test') {
            steps {
                dir('sample-java-app') {
                    sh 'mvn -B test'
                }
            }
            post {
                always {
                    junit testResults: 'sample-java-app/target/surefire-reports/*.xml', allowEmptyResults: true
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                dir('sample-java-app') {
                    withSonarQubeEnv('SonarQube') {
                        withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                            sh '''
                                mvn -B sonar:sonar \
                                    -Dsonar.projectKey=devops-lab7 \
                                    -Dsonar.projectName="DevOps Lab 7 CI Demo" \
                                    -Dsonar.token="$SONAR_TOKEN"
                            '''
                        }
                    }
                }
            }
        }
        stage('Package') {
            steps {
                dir('sample-java-app') {
                    sh 'mvn -B package -DskipTests'
                }
                archiveArtifacts artifacts: 'sample-java-app/target/*.jar', fingerprint: true
            }
        }
        stage('Publish to Nexus') {
            steps {
                dir('sample-java-app') {
                    withCredentials([usernamePassword(credentialsId: 'nexus-credentials', usernameVariable: 'NEXUS_USERNAME', passwordVariable: 'NEXUS_PASSWORD')]) {
                        configFileProvider([configFile(fileId: 'maven-settings-nexus', variable: 'MAVEN_SETTINGS')]) {
                            sh '''
                                mvn -B -s "$MAVEN_SETTINGS" deploy:deploy-file \
                                    -DgroupId=com.devops.lab7 \
                                    -DartifactId=ci-demo \
                                    -Dversion=1.0.0-SNAPSHOT \
                                    -Dpackaging=jar \
                                    -Dfile=target/ci-demo-1.0.0-SNAPSHOT.jar \
                                    -Durl=http://lab7-nexus:8081/repository/maven-snapshots/ \
                                    -DrepositoryId=nexus
                            '''
                        }
                    }
                }
            }
        }
    }
    post {
        success { echo 'PIPELINE SUCCESS!' }
        failure { echo 'PIPELINE FAILED - read Console Output' }
    }
}
