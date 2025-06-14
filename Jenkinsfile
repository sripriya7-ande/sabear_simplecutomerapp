pipeline { // Opening pipeline
    agent any
    tools { // Opening tools
        maven "Maven_3.9.4"
    } // Closing tools
    environment { // Opening environment
        // This can be nexus3 or nexus2
        NEXUS_VERSION = "Maven_3.9.4" // This should probably be "nexus3" or "nexus2" based on the comment. Currently it's set to Maven version.
        // This can be http or https
        NEXUS_PROTOCOL = "http"
        // Where your Nexus is running
        NEXUS_URL = "34.201.219.251:8082/" // Double check the double slash at the end, usually it's just one: "3.88.54.126:8082/"
        // Repository where we will upload the artifact
        NEXUS_REPOSITORY = "maven-snapshot05/" // Is this the correct Nexus repository name? Sonarqube is usually a tool, not a Nexus repo. Common ones are 'maven-releases', 'maven-snapshots'.
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = "admin/****** (nexus)" // Make sure this is the exact ID configured in Jenkins Credentials.
        SCANNER_HOME = tool 'sonar_scanner' // This should be the tool name configured in Jenkins for SonarQube Scanner.
    } // Closing environment
    stages { // Opening stages
        stage("clone code") { // Opening clone code stage
            steps { // Opening steps
                script { // Opening script
                    // Let's clone the source
                    git 'https://github.com/sripriya7-ande/sabear_simplecutomerapp.git';
                } // Closing script
            } // Closing steps
        } // Closing clone code stage
        stage("mvn build") { // Opening mvn build stage
            steps { // Opening steps
                script { // Opening script
                    // If you are using Windows then you should use "bat" step
                    // Since unit testing is out of the scope we skip them
                    sh 'mvn -Dmaven.test.failure.ignore=true clean install'
                } // Closing script
            } // Closing steps
        } // Closing mvn build stage
        stage('SonarCloud') {
        steps {
            withSonarQubeEnv('sonarqube_server') {
                sh '''$SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=Ncodeit \
                    -Dsonar.projectName=Ncodeit \
                    -Dsonar.projectVersion=2.0 \
                    -Dsonar.sources=/var/lib/jenkins/workspace/$JOB_NAME/src/ \
                    -Dsonar.binaries=target/classes/com/visualpathit/account/controller/ \
                    -Dsonar.junit.reportsPath=target/surefire-reports \
                    -Dsonar.jacoco.reportPath=target/jacoco.exec \
                    -Dsonar.java.binaries=src/com/room/sample ''' // <-- Change here: Start and end with triple quotes
            }
        }
    }
        stage("publish to nexus") { // Opening publish to nexus stage
            steps { // Opening steps
                script { // Opening script
                    // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                    pom = readMavenPom file: "pom.xml";
                    // Find built artifact under target folder
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    // Print some info from the artifact found
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    // Extract the path from the File found
                    artifactPath = filesByGlob[0].path;
                    // Assign to a boolean response verifying If the artifact name exists
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) { // Opening if
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION, // This should likely be "nexus3" or "nexus2"
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY, // Verify this repository name in Nexus
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifact generated such as .jar, .ear and .war files.
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                // Lets upload the pom.xml file for additional information for Transitive dependencies
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                    } else { // Opening else
                        error "*** File: ${artifactPath}, could not be found";
                    } // Closing else
                } // Closing script
            } // Closing steps
        } // Closing publish to nexus stage
    } // Closing stages block
} // Closing pipeline block
