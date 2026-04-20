pipeline {
    agent any
    tools {
        maven "MAVEN3.9"
        jdk "JDK17"
    }
    
    environment {
        SNAP_REPO = 'veeam-snapshot'
		NEXUS_USER = 'jenkins'
		NEXUS_PASS = 'admin123!'
		RELEASE_REPO = 'veeam-release'
		CENTRAL_REPO = 'veeam-maven-center'
		NEXUSIP = '10.110.12.3'
		NEXUSPORT = '8081'
		NEXUS_GRP_REPO = 'veeam-maven-group'
        NEXUS_LOGIN = 'nexuslogin'
        SONARSERVER = 'sonarserver'
        SONARSCANNER = 'sonarscanner'
    }

    stages {
        stage('Build'){
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post {
                success {
                    echo 'Now Archiving.'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }
        stage('Test'){
            steps {
                sh 'mvn -s settings.xml test'
            }
        }
        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }
        stage('Sonar Analysis') {
            environment {
                scannerHome = tool "${SONARSCANNER}"
                // Add this line to handle the Java 17+ encapsulation issue
                SONAR_SCANNER_OPTS = "--add-opens java.base/java.lang=ALL-UNNAMED"
            }
            steps {
                withSonarQubeEnv("${SONARSERVER}") {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                    -Dsonar.projectName=vprofile \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/ \
                    -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                    -Dsonar.junit.reportsPath=target/surefire-reports/ \
                    -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    timeout(time: 5, unit: 'MINUTES') {
                        // Wait for Quality Gate result
                        def qualityGate = waitForQualityGate abortPipeline: true
                        echo "Quality Gate status: ${qualityGate.status}"
                    }

                    // Fetch issues from SonarQube API (replace sonarserver and token)
                    def issuesJson = sh(
                        script: """
                            curl -s -u admin:your_token \
                            "http://sonar.veeam.sc.local:9000/api/issues/search?componentKeys=vprofile"
                        """,
                        returnStdout: true
                    ).trim()

                    // Print raw JSON or parse it
                    echo "SonarQube Issues: ${issuesJson}"

                    // Optional: parse JSON to show summary
                    def issues = readJSON text: issuesJson
                    echo "Total issues found: ${issues.total}"
                    issues.issues.each { issue ->
                        echo "File: ${issue.component}, Line: ${issue.line}, Type: ${issue.type}, Message: ${issue.message}"
                    }
                }
            }
        }
        stage("UploadArtifact"){
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                    groupId: 'QA',
                    version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                    repository: "${RELEASE_REPO}",
                    credentialsId: "${NEXUS_LOGIN}",
                    artifacts: [
                        [artifactId: 'vproapp',
                        classifier: '',
                        file: 'target/vprofile-v2.war',
                        type: 'war']
                    ]
                )
            }
        }
    }
}