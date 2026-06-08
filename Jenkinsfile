pipeline {
    agent any

    environment {
        IMAGE_NAME = "spring-petclinic"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        DOCKER_IMAGE = "${IMAGE_NAME}:${IMAGE_TAG}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Configure Maven to use JCenter') {
            steps {
                sh '''
                    mkdir -p .mvn

                    cat > .mvn/settings.xml <<'SETTINGS'
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">

  <mirrors>
    <mirror>
      <id>jcenter</id>
      <name>JCenter</name>
      <url>https://jcenter.bintray.com/</url>
      <mirrorOf>*</mirrorOf>
    </mirror>
  </mirrors>
</settings>
SETTINGS
                '''
            }
        }

        stage('Compile') {
            steps {
                sh './mvnw -s .mvn/settings.xml clean compile'
            }
        }

        stage('Test') {
            steps {
                sh './mvnw -s .mvn/settings.xml test'
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package') {
            steps {
                sh './mvnw -s .mvn/settings.xml package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE} -t ${IMAGE_NAME}:latest .'
            }
        }

        stage('Save Docker Image') {
            steps {
                sh '''
                    mkdir -p dist
                    docker save ${DOCKER_IMAGE} | gzip > dist/${IMAGE_NAME}-${IMAGE_TAG}.tar.gz
                '''
                archiveArtifacts artifacts: 'dist/*.tar.gz', fingerprint: true
            }
        }
    }
}
