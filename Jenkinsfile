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
    }

    stages {
        stage('Build'){
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
        }
    }
}