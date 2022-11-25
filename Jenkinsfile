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
    registryBacktend = 'lhamaoka/practica-final-backend'
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

    stage("3.- Unit Tests") {
        steps {
            sh "mvn test"
            junit "target/surefire-reports/*.xml"
        }
    }

    stage("4.- JaCoCo Tests") {
        steps {
            jacoco()
            junit "target/surefire-reports/*.xml"
        }
    }

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
            script {
              if(fileExists("launcher")){
                sh 'rm -r launcher'
              }
            }
            sh "git clone https://github.com/lhamaoka/manifest_launcher.git launcher"
            sh "kubectl apply -f launcher/deploys/backend/manifest.yaml --kubeconfig=launcher/config/config"
        }
    }

    stage("10.- API Test o Performance Test"){
        steps{
            sh "echo Lanzar los test de JMeter o las pruebas de API con Newman."
        }
    }

    stage ("Setup Jmeter") {
           steps{
               script {
        
                   if(fileExists("jmeter-docker")){
                      sh 'rm -r jmeter-docker'
                   }
        
                   sh 'git clone https://github.com/lhamaoka/jmeter-docker.git'
        
                    dir('jmeter-docker') {
        
                       if(fileExists("apache-jmeter-5.5.tgz")){
                           sh 'rm -r apache-jmeter-5.5.tgz'
                       }
                       sh 'apt-get install wget'
                       sh 'apt-get install python3-pip -y'                   
                       sh 'wget https://dlcdn.apache.org//jmeter/binaries/apache-jmeter-5.5.tgz'
                       sh 'tar xvf apache-jmeter-5.5.tgz'
                       sh 'cp plugins/*.jar apache-jmeter-5.5/lib/ext'
                       sh 'mkdir test'
                       sh 'mkdir apache-jmeter-5.5/test'
                       sh 'cp ../src/main/resources/*.jmx apache-jmeter-5.5/test/'
                       sh 'chmod +775 ./build.sh && chmod +775 ./run.sh && chmod +775 ./entrypoint.sh'
                       sh 'rm -r apache-jmeter-5.5.tgz'
                       sh 'tar -czvf apache-jmeter-5.5.tgz apache-jmeter-5.5'
                       sh './build.sh'
                       sh 'rm -r apache-jmeter-5.5 && rm -r apache-jmeter-5.5.tgz'
                       sh 'cp ../src/main/resources/perform_test.jmx test'
                    }
               }
           }    
        }   
        stage ("Run Jmeter Performance Test") {
           steps{
               script {
                    dir('jmeter-docker') {
                       if(fileExists("apache-jmeter-5.5.tgz")){
                           sh 'rm -r apache-jmeter-5.5.tgz'
                       }
                       sh './run.sh -n -t test/perform_test.jmx -l test/perform_test.jtl'
                       sh 'docker cp jmeter:/home/jmeter/apache-jmeter-5.5/test/perform_test.jtl $(pwd)/test'
                       perfReport 'test/perform_test.jtl'
                    }      
               }
           }
        }
        stage ("Generate Taurus Report") {
            steps{
                script {
                        dir('jmeter-docker') {
                        sh 'pip install bzt'
                        sh 'export PATH=$PATH:/home/jenkins/.local/bin'

                        BlazeMeterTest: {
                            sh 'bzt test/perform_test.jtl -report'
                        }
                        }
                }
            }
        }

    stage("11.- Nexus"){
        steps{
            sh "echo Si se ha llegado a esta etapa sin problemas, se deberá depositar el artefacto generado en Nexus."
        }
    }

    stage("12.- Deploy"){
        steps{
            sh "echo En esta stage se debe desplegar en un pod, la imagen generada en la etapa 8. Para ello se deberá generar un Chart de Helm como los vistos en clase que contenga un ConfigMap y un Pod con dicha imagen"
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