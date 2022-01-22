import groovy.json.JsonSlurperClassic
def jsonParse(def json) {
    new groovy.json.JsonSlurperClassic().parseText(json)
}
pipeline {
    agent any

     environment {
        NEXUS_USER = credentials('nexus-user')
        NEXUS_PASS = credentials('nexus-pass')
    }

    stages {        
                 
        stage("Paso 1: Compilar"){
            steps {
                script {
                sh "echo 'Compile Code!'"
                // Run Maven on a Unix agent.
                sh "mvn clean compile -e"
                }
            }
        }
        stage("Paso 2: Testear"){
            steps {
                script {
                sh "echo 'Test Code!'"
                // Run Maven on a Unix agent.
                sh "mvn clean test -e"
                }
            }
        }
        stage("Paso 3: build .jar"){
            steps {
                script {
                sh "echo 'build .jar!'"
                // Run Maven on a Unix agent.
                sh "mvn clean package -e"
                }
            }            
        }

        stage("Paso 4: Análisis SonarQube"){
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh "echo 'Calling sonar Service in another docker container!'"
                    // Run Maven on a Unix agent to execute Sonar.
                    sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=github-sonar'
                }
            }
                   
        }    

        stage("Paso 5: Subida a Nexus"){
            steps{
                nexusPublisher nexusInstanceId: 'nexus', 
                    nexusRepositoryId: 'devops-usach-nexus', 
                    packages: [[$class: 'MavenPackage', 
                                mavenAssetList: [[
                                    classifier: '', 
                                    extension: '', 
                                    // filePath: '/var/jenkins_home/workspace/job-pipeline-webhook/build/DevOpsUsach2020-0.0.1.jar'
                                    filePath: 'build/DevOpsUsach2020-0.0.1.jar'
                                    ]],
                                mavenCoordinate: [
                                    artifactId: 'DevOpsUsach2020', 
                                    groupId: 'com.devopsusach2020', 
                                    packaging: 'jar',
                                     version: '0.0.1'
                                ]]]
            }     
            
        }            
        
        stage('Paso 6: Bajar Nexus Stage') {
            steps {
                sh 'chmod -R 777 DevOpsUsach2020-0.0.1.jar'
                sh 'curl -X GET -u $NEXUS_USER:$NEXUS_PASS http://nexucito:8081/repository/devops-usach-nexus/com/devopsusach2020/DevOpsUsach2020/0.0.1/DevOpsUsach2020-0.0.1.jar -O'
                sh 'chmod -R 777 DevOpsUsach2020-0.0.1.jar'
            }
        }                

        stage("Paso 7: Levantar Springboot APP"){
            steps {
                sh 'java -jar DevOpsUsach2020-0.0.1.jar &'
                // sh 'nohup bash java -jar DevOpsUsach2020-0.0.1.jar & >/dev/null'
            }

        }
        stage("Paso 8: Dormir(Esperar 10sg) "){
            steps {
                sh 'sleep 10'
            }
        }

        stage("Paso 9: Test Alive Service - Testing Application!"){
            steps {
                sh 'curl -X GET http://localhost:8081/rest/mscovid/test?msg=testing'
            }
        }

        stage("Paso 10: Subida a Nexus"){
            steps{
                nexusPublisher nexusInstanceId: 'nexus', 
                    nexusRepositoryId: 'devops-usach-nexus', 
                    packages: [[$class: 'MavenPackage', 
                                mavenAssetList: [[
                                    classifier: '', 
                                    extension: '', 
                                    // filePath: '/var/jenkins_home/workspace/job-pipeline-webhook/build/DevOpsUsach2020-0.0.1.jar'
                                    filePath: 'build/DevOpsUsach2020-0.0.1.jar'
                                    ]],
                                mavenCoordinate: [
                                    artifactId: 'DevOpsUsach2020', 
                                    groupId: 'com.devopsusach2020', 
                                    packaging: 'jar',
                                    version: '1.0.0'
                                ]]]
            }     
            
        }   
    }
    post {
        always {
            sh "echo 'fase always executed post'"
        }
        success {
            sh "echo 'fase success'"
        }
        failure {
            sh "echo 'fase failure'"
        }
    }
}