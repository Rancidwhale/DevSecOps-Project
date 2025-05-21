def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger'
    ]

pipeline{
    agent any
    environment {
        SCANNER_HOME=tool 'sqube-scanner'
        TMDB_V3_API_KEY = credentials('tmdb-api-key')
        IMAGE_NAME = "netflix"
        CONTAINER_NAME = "netflix" 

    }
    stages{
        stage('Clean Workspace')
        {
            steps{
                cleanWs()
            }
        }
        stage('GIT SCM clone')
        {
            steps{
                // git branch 'https://github.com/Rancidwhale/DevSecOps-Project.git'
                git branch: 'dev', url: 'https://github.com/Rancidwhale/DevSecOps-Project.git'
                // git branch: 'dev', credentialsId: 'git', url: 'git@github.com:Rancidwhale/DevSecOps-Project.git'
            }
        }
        stage('Code quality')
        {
            steps{
                withSonarQubeEnv('sqube-server'){
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=DevSecOps-demorun \
                    -Dsonar.projectKey=DevSecOps-Project'''
                }
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                withCredentials([string(credentialsId: 'nvd-api-key', variable: 'NVD_API_KEY')]) {
                dependencyCheck additionalArguments: "--scan ./ --disableYarnAudit --disableNodeAudit --nvdApiKey ${NVD_API_KEY}", odcInstallation: 'owasp'
            }
            dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
       }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"     
            }
        }
        stage('Clean Up Docker Resources') {
            steps {
                script {
                    // Remove the specific container
                    sh '''
                    if docker ps -a --format '{{.Names}}' | grep -q $CONTAINER_NAME; then
                        echo "Stopping and removing container: $CONTAINER_NAME"
                        docker stop $CONTAINER_NAME
                        docker rm $CONTAINER_NAME
                    else
                        echo "Container $CONTAINER_NAME does not exist."
                    fi
                    '''

                    // Remove the specific image
                    sh '''
                    if docker images -q $IMAGE_NAME; then
                        echo "Removing image: $IMAGE_NAME"
                        docker rmi -f $IMAGE_NAME
                    else
                        echo "Image $IMAGE_NAME does not exist."
                    fi
                    '''
                }
            }
        }
        stage("Docker Build"){
            steps{
                script{
                       sh 'docker build --build-arg TMDB_V3_API_KEY=$TMDB_V3_API_KEY -t $IMAGE_NAME .'
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image $IMAGE_NAME > trivyimage.txt"
            }
        }
        stage('Deploy to container'){
            steps{
                script{
                    sh 'docker run -itd --name $CONTAINER_NAME -p 8081:80 $IMAGE_NAME'
                }
            }
        }
    }
    post {

        always {
            echo 'slack Notification.'
            slackSend channel: '#sample1',
            color: COLOR_MAP [currentBuild.currentResult],
            message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URl}"
            
        }
        failure {
            emailext(
                to: 'muhammadabdullah3602@gmail.com',
                subject: 'Build Status : ${BUILD_STATUS} of Build Number : ${BUILD_NUMBER}',
                body: 'this is the build status for this build',
                attachLog: true
            )
        }
}
}

// pipeline{
//     agent any
//     environment {
//         SCANNER_HOME=tool 'sonar-scanner'
//         TMDB_V3_API_KEY = credentials('tmdb-api-key')
//         IMAGE_NAME = "sushmaagowdaa/netflix" // Name of the image created in Jenkins
//         CONTAINER_NAME = "netflix" // Name of the container created in Jenkins
//     }
//     stages {
//         stage('clean workspace'){
//             steps{
//                 cleanWs()
//             }
//         }
//         stage('Checkout from Git'){
//             steps{
//                 git 'https://github.com/Sushmaa123/DevSecOps-Project.git'
//             }
//         }
//         stage("Sonarqube Analysis "){
//             steps{
//                 withSonarQubeEnv('sonar-server') {
//                     sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=DevSecOps-Project \
//                     -Dsonar.projectKey=DevSecOps-Project'''
//                 }
//             }
//         }
       
    //     stage('OWASP FS SCAN') {
    //          steps {
    //          withCredentials([string(credentialsId: 'nvd-api-key', variable: 'NVD_API_KEY')]) {
    //         dependencyCheck additionalArguments: "--scan ./ --disableYarnAudit --disableNodeAudit --nvdApiKey ${NVD_API_KEY}", odcInstallation: 'DP-Check'
    //          }
    //         dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
    //         }
    //    }

        // stage('TRIVY FS SCAN') {
        //     steps {
        //         sh "trivy fs . > trivyfs.txt"     
        //     }
        // }
        // stage('Clean Up Docker Resources') {
        //     steps {
        //         script {
        //             // Remove the specific container
        //             sh '''
        //             if docker ps -a --format '{{.Names}}' | grep -q $CONTAINER_NAME; then
        //                 echo "Stopping and removing container: $CONTAINER_NAME"
        //                 docker stop $CONTAINER_NAME
        //                 docker rm $CONTAINER_NAME
        //             else
        //                 echo "Container $CONTAINER_NAME does not exist."
        //             fi
        //             '''

        //             // Remove the specific image
        //             sh '''
        //             if docker images -q $IMAGE_NAME; then
        //                 echo "Removing image: $IMAGE_NAME"
        //                 docker rmi -f $IMAGE_NAME
        //             else
        //                 echo "Image $IMAGE_NAME does not exist."
        //             fi
        //             '''
        //         }
        //     }
        // }
//         stage("Docker Build & Push"){
//             steps{
//                 script{
//                    withDockerRegistry(credentialsId: 'docker-cred'){   
//                        sh 'docker build --build-arg TMDB_V3_API_KEY=$TMDB_V3_API_KEY -t $IMAGE_NAME .'
//                        sh 'docker push $IMAGE_NAME'
//                     }
//                 }
//             }
//         }
        // stage("TRIVY"){
        //     steps{
        //         sh "trivy image $IMAGE_NAME > trivyimage.txt"
        //     }
        // }
//         stage('Deploy to container'){
//             steps{
//                 sh 'docker run -itd --name $CONTAINER_NAME -p 8081:80 $IMAGE_NAME'
//             }
//         }
//     }
// post {
//      always {
//         emailext attachLog: true,
//             subject: "'${currentBuild.result}'",
//             body: "Project: ${env.JOB_NAME}<br/>" +
//                 "Build Number: ${env.BUILD_NUMBER}<br/>" +
//                 "URL: ${env.BUILD_URL}<br/>",
//             to: 'your-mail@gmail.com',                               
//             attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
//         }
//     }
// }

