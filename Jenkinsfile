pipeline {
    agent any
    tools{
        maven "maven3"
    }
    environment {
        SCANNER_HOME= tools "sonarQube"
    }
    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/divyasatpute/Multi-Tier-With-Database.git'
            }
        }
        stage('Compile') {
            steps {
              sh "mvn compile"
            }
        }
        stage('Test') {
            steps {
               sh "mvn test -DskipTest=true"
            }
        }
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --timeout 10m --format table -o fs-report.html . "
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("sonar") {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=bankapp \
                    -Dsonar.projectName=bankapp -Dsonar.binaries=target'''
                }
            }
        }
        
        stage('Build') {
            steps {
               sh "mvn package -DskipTests=true"
            }
            stage('Publish To Nexus') {
            steps {
               withMaven(globalMavenSettingsConfig: 'Maven', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true "
                   
                    }
            }
            stage("Docker Build Image") {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'docker-cred') {
                  sh "docker build -t divyasatpute/bankapp:latest ."
                   }
            stage('Trivy Image Scan') {
            steps {
                sh "trivy Image --format table -o fs-report.html  divyasatpute/bankapp:latest"
            }
               }
            stage("Docker push Image") {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'docker-cred') {
                  sh "docker push divyasatpute/bankapp:latest"
                   }
            }
             stage("Deploy To Kubernetes") {
            steps {
                   withKubeConfig(caCertificate: '', clusterName: ' devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'divya', restrictKubeConfigAccess: false, serverUrl: 'https://A7B600B6433CB15F4550F1729FDB6398.yl4.ap-south-1.eks.amazonaws.com') {
                sh "kubectl apply -f ds.yml -n divya"
                sleep 30
}
                   }
                   stage("Verify Deployment") {
            steps {
                   withKubeConfig(caCertificate: '', clusterName: ' devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'divya', restrictKubeConfigAccess: false, serverUrl: 'https://A7B600B6433CB15F4550F1729FDB6398.yl4.ap-south-1.eks.amazonaws.com') {
                sh "kubectl get pods -n divya"
                sh "kubectl get svc -n divya"
            }
        }
    }
}
