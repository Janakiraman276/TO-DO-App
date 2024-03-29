pipeline{
    agent any
    tools{
        jdk 'jdk17'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout From Git'){
            steps{
                git branch: 'main', url: 'https://github.com/Janakiraman276/TO-DO-App.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=TO-DO-App \
                    -Dsonar.projectKey=TO-DO-App '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
            } 
        }
        stage("TRIVY File scan"){
            steps{
                sh "trivy fs . > trivy-fs.txt" 
            }
        }
        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --format XML ', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build -t to-do . "
                       sh "docker tag to-do janakiraman276/to-do:latest"
                       sh "docker push janakiraman276/to-do:latest"
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image janakiraman276/to-do:latest > trivy.txt" 
            }
        }
        stage("Deploy to container"){
            steps{
                sh "docker run -d --name to-do -p 5000:5000 janakiraman276/to-do:latest"
            } 
        }
    }
}
