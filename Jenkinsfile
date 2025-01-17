pipeline {

    agent any

    stages {

        stage ('checkout') {

            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/jyothibasu/hello-world.git']]])
            }
        }

        stage ('Code Analysis') {

            steps {
                sh '''mvn sonar:sonar \\
                -Dsonar.projectKey=Poc-AKS1 \\
                -Dsonar.host.url=http://52.140.116.20:9000 \\
                -Dsonar.login=e5ae4daa1e4c7cffe91ed213468a069ce581ff69'''
            }
        }

        stage ('Build') {

            steps {

                sh 'mvn clean package'
                
            }
        }

        stage ('Push Artifacts to Jfrog') {

            steps {
                sh '''cp webapp/target/webapp.war webapp/target/webapp_$BUILD_ID.war
                curl -uadmin:AP34mCp3r3nLNeLoHTaGnbrAuEJ -T webapp/target/webapp_$BUILD_ID.war "http://52.140.116.20:8081/artifactory/example-repo-local/"'''
                
            }
        }
        
        stage('Docker Build & Push image'){
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker_cred', passwordVariable: 'dockerhubpassword', usernameVariable: 'dockerhubuser')]) {
                    sh "docker login -u $dockerhubuser -p $dockerhubpassword"
                }
                sh '''docker build -t poc-1:v1.$BUILD_ID .
                docker tag poc-1:v1.$BUILD_ID jyothibasuk/poc-1:v1.$BUILD_ID
                docker push jyothibasuk/poc-1:v1.$BUILD_ID
                docker rmi poc-1:v1.$BUILD_ID
                docker rmi jyothibasuk/poc-1:v1.$BUILD_ID''' 
            }
        }

        stage ('Deploy to AKS Cluster') {

            steps {
                script {
                    
                    kubernetesDeploy(
                    configs: 'aks-deployment.yaml',
                    kubeconfigId: 'k8s-cluster',
                    ) 
                }
                
            }
        }

    }
}
