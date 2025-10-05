@Library('shared_library') _
pipeline {
    //agent {label 'Node'}
    agent any
    
    environment{
        SONAR_HOME = tool "sonar"
    }
    
    parameters {
        string(name: 'FRONTEND_DOCKER_TAG', defaultValue: '', description: 'Setting docker image for latest push')
        string(name: 'BACKEND_DOCKER_TAG', defaultValue: '', description: 'Setting docker image for latest push')
    }
    
    stages {
        stage("Validate Parameters") {
            steps {
                script {
                    if (params.FRONTEND_DOCKER_TAG == '' || params.BACKEND_DOCKER_TAG == '') {
                        error("FRONTEND_DOCKER_TAG and BACKEND_DOCKER_TAG must be provided.")
                    }
                }
            }
        }
        
        /*stage("Workspace cleanup"){
            steps{
                script{
                    cleanWs()
                }
            }
        }*/
        
        stage('Git: Code Checkout') {
            steps {
                script{
                    code_checkout("https://github.com/Navin0809/Wanderlust-Mega-Project.git","main")
                }
            }
        }
        
        stage("Trivy: Filesystem scan"){
            steps{
                script{
                    trivy_scan()
                }
            }
        }

        /*stage("OWASP: Dependency check"){
            steps{
                script{
                    owasp_dependency()
                }
            }
        }*/
        
        stage("SonarQube: Code Analysis"){
            steps{
                script{
                    sonarqube_analysis("sonar","wanderlust","wanderlust")
                }
            }
        }
        
        stage("SonarQube: Code Quality Gates"){
            steps{
                script{
                    sonarqube_code_quality()
                }
            }
        }
        /*
        stage('Exporting environment variables') {
            parallel{
                stage("Backend env setup"){
                    steps {
                        script{
                            dir("Automations"){
                                sh "bash updatebackendnew.sh"
                            }
                        }
                    }
                }
                
                stage("Frontend env setup"){
                    steps {
                        script{
                            dir("Automations"){
                                sh "bash updatefrontendnew.sh"
                            }
                        }
                    }
                }
            }
        }
        */
        stage("Docker: Build Images"){
            steps{
                script{
                        dir('backend'){
                            docker_build("wanderlust-backend","${params.BACKEND_DOCKER_TAG}","ysanavee")
                        }
                    
                        dir('frontend'){
                            docker_build("wanderlust-frontend","${params.FRONTEND_DOCKER_TAG}","ysanavee")
                        }
                }
            }
        }
        
        stage("Docker: Push to DockerHub"){
            steps{
                script{
                    docker_push("wanderlust-backend","${params.BACKEND_DOCKER_TAG}","ysanavee") 
                    docker_push("wanderlust-frontend","${params.FRONTEND_DOCKER_TAG}","ysanavee")
                }
            }
        }
    }

    post {
        success {
            script {
                emailext attachLog: true,
                from: 'naveensmart0799@gmail.com',
                subject: "Wanderlust Application has been updated and deployed - '${currentBuild.result}'",
                body: """
                    <html>
                    <body>
                        <div style="background-color: #FFA07A; padding: 10px; margin-bottom: 10px;">
                            <p style="color: black; font-weight: bold;">Project: ${env.JOB_NAME}</p>
                        </div>
                        <div style="background-color: #90EE90; padding: 10px; margin-bottom: 10px;">
                            <p style="color: black; font-weight: bold;">Build Number: ${env.BUILD_NUMBER}</p>
                        </div>
                        <div style="background-color: #87CEEB; padding: 10px; margin-bottom: 10px;">
                            <p style="color: black; font-weight: bold;">URL: ${env.BUILD_URL}</p>
                        </div>
                    </body>
                    </html>
            """,
            to: 'naveensmart0799@gmail.com',
            mimeType: 'text/html'
            }
        }

      failure {
            script {
                emailext attachLog: true,
                from: 'naveensmart0799@gmail.com',
                subject: "Wanderlust Application build failed - '${currentBuild.result}'",
                body: """
                    <html>
                    <body>
                        <div style="background-color: #FFA07A; padding: 10px; margin-bottom: 10px;">
                            <p style="color: black; font-weight: bold;">Project: ${env.JOB_NAME}</p>
                        </div>
                        <div style="background-color: #90EE90; padding: 10px; margin-bottom: 10px;">
                            <p style="color: black; font-weight: bold;">Build Number: ${env.BUILD_NUMBER}</p>
                        </div>
                    </body>
                    </html>
            """,
            to: 'naveensmart0799@gmail.com',
            mimeType: 'text/html'
            }
        }
    }
}
    /*
    post{
        success{
            archiveArtifacts artifacts: '*.xml', followSymlinks: false
            build job: "Wanderlust-CD", parameters: [
                string(name: 'FRONTEND_DOCKER_TAG', value: "${params.FRONTEND_DOCKER_TAG}"),
                string(name: 'BACKEND_DOCKER_TAG', value: "${params.BACKEND_DOCKER_TAG}")
            ]
        }
    }
}*/

