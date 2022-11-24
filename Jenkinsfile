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
    stage("Builds"){
        steps{
            sh "mvn clean package -DskipTests"
        }
    }

    stage("Info"){
        steps{
            sh "mvn -v"
            sh "java --version"
        }
    }
  }

  post {
    always {
      sh 'docker logout'
    }
  }
}