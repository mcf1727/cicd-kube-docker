pipeline {

    agent any

	tools {
        maven "MAVEN3"
    }

    environment {
        registry = "mcf1727/vproappdock"
        registryCredential = 'dockerhub'
        // NEXUS_VERSION = "nexus3"
        // NEXUS_PROTOCOL = "http"
        // NEXUS_URL = "172.31.40.209:8081"
        // NEXUS_REPOSITORY = "vprofile-release"
        // NEXUS_REPO_ID    = "vprofile-release"
        // NEXUS_CREDENTIAL_ID = "nexuslogin"
        // ARTVERSION = "${env.BUILD_ID}"
    }

    stages{

        // stage('Fetch Code') {
        //     steps {
        //         git branch: 'paac', url: 'https://github.com/devopshydclub/vprofile-project.git'
        //     }
        // }
        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        }

        stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }

        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('CODE ANALYSIS with SONARQUBE') {

            environment {
                scannerHome = tool 'mysonarscanner4'
            }

            steps {
                withSonarQubeEnv('sonar-pro') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }

                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build App Image') {
           steps{
                script {
                    dockerImage = docker.build registry + ":V$BUILD_NUMBER"
                }
           }
        }

        stage('Upload Image') {
            steps {
                script {
                    docker.withRegistry('', registryCredential) {
                        dockerImage.push("V$BUILD_NUMBER")
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage('Remove unused docker Image') {
            steps {
                sh "docker rmi $registry:V$BUILD_NUMBER"
            }
        }

        stage('Kubernetes Deploy') {
	        agent { label 'KOPS' }
                steps {
                    sh "helm upgrade --install --force vproifle-stack helm/vprofilecharts --set appimage=${registry}:V${BUILD_NUMBER} --namespace prod"
                }
        }
        // stage("Publish to Nexus Repository Manager") {
        //     steps {
        //         script {
        //             pom = readMavenPom file: "pom.xml";
        //             filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
        //             echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
        //             artifactPath = filesByGlob[0].path;
        //             artifactExists = fileExists artifactPath;
        //             if(artifactExists) {
        //                 echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version} ARTVERSION";
        //                 nexusArtifactUploader(
        //                         nexusVersion: NEXUS_VERSION,
        //                         protocol: NEXUS_PROTOCOL,
        //                         nexusUrl: NEXUS_URL,
        //                         groupId: pom.groupId,
        //                         version: ARTVERSION,
        //                         repository: NEXUS_REPOSITORY,
        //                         credentialsId: NEXUS_CREDENTIAL_ID,
        //                         artifacts: [
        //                                 [artifactId: pom.artifactId,
        //                                  classifier: '',
        //                                  file: artifactPath,
        //                                  type: pom.packaging],
        //                                 [artifactId: pom.artifactId,
        //                                  classifier: '',
        //                                  file: "pom.xml",
        //                                  type: "pom"]
        //                         ]
        //                 );
        //             }
        //             else {
        //                 error "*** File: ${artifactPath}, could not be found";
        //             }
        //         }
        //     }
        // }


    }


}
