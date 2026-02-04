pipeline {
    agent any
    
    tools{
        maven 'Maven'
        jdk "OracleJDK8"
    }
    
    environment {
        SCANNER_HOME = tool 'Sonar'
    }

    stages {
        
        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo "Now Archiving."
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        
        stage('Apply Unit Test') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('Gitleaks Scan') {
    steps {
        sh '''
            gitleaks detect --source . -r gitleaks-report.json --exit-code 1 || true
            if [ -s gitleaks-report.json ]; then
                echo "--- GITLEAKS SEARCH RESULTS ---"
                cat gitleaks-report.json
            else
                echo "No leaks found or report is empty."
            fi
        '''
        archiveArtifacts artifacts: 'gitleaks-report.json', allowEmptyArchive: true
            }
        }

        
        stage('SonarQube Analysis') {
            steps {
                  withSonarQubeEnv('Sonar') {
                   sh """
                   
                    export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64
                       export PATH=\$JAVA_HOME/bin:\$PATH
                   
                       ${SCANNER_HOME}/bin/sonar-scanner \
                       -Dsonar.host.url=http://127.0.0.1:9000 \
                       -Dsonar.projectKey=iprofile \
                       -Dsonar.sources=src/ \
                       -Dsonar.java.binaries=target/classes
                   """
                    }
                }
        }
        
        stage('Quality Gate Check') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: false, credentialsId: 'SonarToken'
                }
            }
        }

          stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs --format table -o fs-report.html .'
            }
        }
        
        stage('Artifact Upload') {
            steps{
                nexusArtifactUploader artifacts: [[artifactId: 'vproapp', classifier: '', file: 'target/iprofile-v4.war', type: 'war']],
                credentialsId: 'NexusLoginCreds',
                groupId: 'com.visualpathit',
                nexusUrl: '127.0.0.1:8081',
                nexusVersion: 'nexus3',
                protocol: 'http',
                repository: 'iprofile',
                version: 'v4'
            }
        }
        
    }
}
