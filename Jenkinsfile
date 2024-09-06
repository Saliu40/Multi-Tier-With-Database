pipeline {
    agent any
    tools{
        maven "maven3"
    }
    environment {
        SCANNER_HOME= tools "sonar-scanner"
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
                sh "trivy fs --format table -o fs-report.html ."
            }
        }
        stage('SonarQube Analysis') {
            steps {
                echo 'Hello World'
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
                   withKubeConfig(caCertificate: '', clusterName: ' devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://BE0697C1E972E170A6F2FE648CB4A9E7.gr7.ap-south-1.eks.amazonaws.com') {
                sh "kubectl apply -f ds.yml -n webapps"
                sleep 30
}
                   }
                   stage("Verify Deployment") {
            steps {
                   withKubeConfig(caCertificate: '', clusterName: ' devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://BE0697C1E972E170A6F2FE648CB4A9E7.gr7.ap-south-1.eks.amazonaws.com') {
                sh "kubectl get pods -n webapps"
                sh "kubectl get svc -n webapps"
            }
        }
    }
}
