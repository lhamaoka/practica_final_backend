pipeline{

    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: shell
    image: lhamaoka/jenkins-nodo-java-bootcamp:1.0
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-socket-volume
    securityContext:
      privileged: true
  volumes:
  - name: docker-socket-volume
    hostPath:
      path: /var/run/docker.sock
      type: Socket
    command:
    - sleep
    args:
    - infinity
'''
            defaultContainer 'shell'
        }
    }

  environment {
    registryCredential='docker-hub-credentials'
    registryFrontend = 'lhamaoka/pring-boot-app'
  }

  stages {
    stage("Prepare environment"){
        steps{
            sh "mvn -v"
            sh "java --version"
        }
    }
    stage("Compile"){
        steps{
            sh "mvn clean package -DskipTests"
        }
    }

    stage('SonarQube analysis') {
          steps {
            withSonarQubeEnv(credentialsId: "sonarqube-credentials", installationName: "sonarqube-server"){
                sh "mvn clean verify sonar:sonar -DskipTests"
            }
          }
        }

        stage('Quality Gate') {
          steps {
            timeout(time: 10, unit: "MINUTES") {
              script {
                def qg = waitForQualityGate(webhookSecretId: 'sonarqube-credentials')
                if (qg.status != 'OK') {
                   error "Pipeline aborted due to quality gate failure: ${qg.status}"
                }
              }
            }
          }
        }
  }

  post {
    always {
      sh 'docker logout'
    }
  }
}