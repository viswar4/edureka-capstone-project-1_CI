pipeline{
    agent any
    tools {
        maven 'maven'
    }
    environment{
        DOCKER_HUB_CREDENTIALS = 'dockerhub'
        DOCKER_IMAGE_NAME = 'viswar4/edureka-project1'
        DOCKER_IMAGE_TAG = "${env.BUILD_ID}"
        DOCKER_IMAGE = "${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
        WORKSPACE="${env.WORKSPACE}"
        VAULT_PASSWORD = credentials('ansiblevault')
           }

    stages{
    
    stage('Build Artifact - Maven') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archive 'target/*.war'
      }
    }

    stage('Unit tests') {
        steps {
        sh "mvn test"
        }
    
    post {
            success {
                junit 'target/surefire-reports/*.xml'
                jacoco execPattern: 'target/jacoco.exec'
            }
        }

    }

    stage('Sonarqube code analysis'){
        steps{
            withSonarQubeEnv(credentialsId: 'sonarqube',installationName: 'sonarqube') {
                    sh 'mvn clean package sonar:sonar'
            }
        }
    }

    stage('Quality gate status'){
        steps{
            timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
            }
        }
    }

    stage('Run Ansible playbook to login,build and push image to Dockerhub and logout'){
        steps{
                ansiblePlaybook extras: '-e DOCKER_IMAGE_TAG=${DOCKER_IMAGE_TAG} -e WORKSPACE=${WORKSPACE}', 
                installation: 'ansible', 
                playbook: 'dockerbuildandpush.yml',
                vaultCredentialsId: 'ansiblevault'
            }
    }
    }
}