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

    stage('Code Promotion') {

        when {
            branch 'develop'
        }
        steps {
            script {
                // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                pom = readMavenPom file: "pom.xml"
                def version = sh script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout', returnStdout: true
                sh "echo version"
                // Find built artifact under target folder
                // filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                // Print some info from the artifact found
                // echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                // // Extract the path from the File found
                // artifactPath = filesByGlob[0].path
                // // Assign to a boolean response verifying If the artifact name exists
                // artifactExists = fileExists artifactPath

                // if(artifactExists) {
                //     echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}"
                //     versionPom = "${pom.version}"

                //     nexusArtifactUploader(
                //         nexusVersion: NEXUS_VERSION,
                //         protocol: NEXUS_PROTOCOL,
                //         nexusUrl: NEXUS_URL,
                //         groupId: pom.groupId,
                //         version: pom.version,
                //         repository: NEXUS_REPOSITORY,
                //         credentialsId: NEXUS_CREDENTIAL_ID,
                //         artifacts: [
                //             // Artifact generated such as .jar, .ear and .war files.
                //             [artifactId: pom.artifactId,
                //             classifier: "",
                //             file: artifactPath,
                //             type: pom.packaging],

                //             // Lets upload the pom.xml file for additional information for Transitive dependencies
                //             [artifactId: pom.artifactId,
                //             classifier: "",
                //             file: "pom.xml",
                //             type: "pom"]
                //         ]
                //     )

                // } else {
                //     error "*** File: ${artifactPath}, could not be found"
                // }
            }
        }
    }

    stage("Compile"){
        steps{
            sh "mvn clean compile"
        }
    }

    // stage("Test") {
    //     steps {
    //         sh "mvn test"
    //         jacoco()
    //         junit "target/surefire-reports/*.xml"
    //     }
    // }

    // stage('SonarQube analysis') {
    //     steps {
    //         withSonarQubeEnv(credentialsId: "sonarqube-credentials", installationName: "sonarqube-server"){
    //             sh "mvn clean verify sonar:sonar -DskipTests"
    //         }
    //     }
    // }

    // stage('Quality Gate') {
    //     steps {
    //         timeout(time: 2, unit: "MINUTES") {
    //             script {
    //                 def qg = waitForQualityGate(webhookSecretId: 'sonarqube-credentials')
    //                 if (qg.status != 'OK') {
    //                     error "Pipeline aborted due to quality gate failure: ${qg.status}"
    //                 }
    //             }
    //         }
    //     }
    // }
  }

  post {
    always {
      sh 'docker logout'
    }
  }
}