pipeline{

    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: shell
    image: lhamaoka/nodo-java-practica-final:1.0
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
    registryCredential='dockerhub_credentials'
    registryBacktend = 'lhamaoka/practica_final_backtend'
    POM_VERSION = ''
    version = sh script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout', returnStdout: true
  }

  stages {

    stage("Prepare environment"){
        steps{
            sh "mvn -v"
            sh "java --version"
        }
    }

    stage('1.- Code Promotion') {

        when {
            branch 'main'
        }
        steps {
            script {
                // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                pom = readMavenPom file: "pom.xml"
                
                echo "${version}"
                sh "mvn versions:set -DremoveSnapshot=true"
                // def versionsinsnapshot = sh script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout', returnStdout: true
                // echo "${versionsinsnapshot}"
                // sh "git add pom.xml"
                // sh "git commit -m \"pom.xml update \""
                // sh "git push https://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@github.com/lhamaoka/practica_final_backend.git main"
            }
        }
    }

    stage("2.- Compile"){
        steps{
            sh "mvn clean compile -DskipTests"
        }
    }

    // stage("3.- Unit Tests") {
    //     steps {
    //         sh "mvn test"
    //         junit "target/surefire-reports/*.xml"
    //     }
    // }

    // stage("4.- JaCoCo Tests") {
    //     steps {
    //         jacoco()
    //         junit "target/surefire-reports/*.xml"
    //     }
    // }

    // stage('5.- SonarQube analysis') {
    //     steps {
    //         withSonarQubeEnv(credentialsId: "sonarqube-credentials", installationName: "sonarqube-server"){
    //             sh "mvn clean verify sonar:sonar -DskipTests"
    //         }
    //     }
    // }

    // stage('6.- Quality Tests') {
    //   steps {

    //       withSonarQubeEnv(credentialsId: "sonarqube-credentials", installationName: "sonarqube-server"){
    //           sh "mvn clean verify sonar:sonar -DskipTests"
    //       }

    //       timeout(time: 2, unit: "MINUTES") {
    //           script {
    //               def qg = waitForQualityGate(webhookSecretId: 'sonarqube-credentials')
    //               if (qg.status != 'OK') {
    //                   error "Pipeline aborted due to quality gate failure: ${qg.status}"
    //               }
    //           }
    //       }
    //   }
    // }

    stage("7.- Package"){
        steps{
            sh "mvn clean package -DskipTests"
        }
    }

    stage("8.- Build & Push"){
        steps{
            script {
              dockerImage = docker.build registryBacktend + ":$BUILD_NUMBER"
              docker.withRegistry( '', registryCredential) {
                dockerImage.push()
              }
            }
        }
    }

    stage("9.- Run test environment"){
        steps{
            sh "echo Iniciar un pod o contenedor con la imagen que acabamos de generar."
        }
    }

    stage("10.- API Test o Performance Test"){
        steps{
            sh "echo Lanzar los test de JMeter o las pruebas de API con Newman."
        }
    }
echo """dsgsd
asefsadf
asdfasdf"""
    stage("11.- Nexus"){
        steps{
            sh "echo Si se ha llegado a esta etapa sin problemas, se deberá depositar el artefacto generado en Nexus."
        }
    }

    stage("12.- Deploy"){
        steps{
            sh "echo En esta stage se debe desplegar en un pod, la imagen generada
            en la etapa 8. Para ello se deberá generar un Chart de Helm como los vistos en
            clase que contenga un ConfigMap y un Pod con dicha imagen"
        }
    }

  }

  post {
    always {
      cleanWs()
      sh 'docker logout'
    }
  }
}