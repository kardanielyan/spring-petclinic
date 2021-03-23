pipeline {
    agent any
    // agent {
    //     node {
    //         label 'maven'
    //     }
    // }
    //agent {
        
        // docker {
        //     image 'maven:3-alpine'
        //     //args '-v /home/kdan/.m2:/root/.m2'
        // }
   // }
    // options {
    //     skipStagesAfterUnstable()
    // }
    stages {
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage("Env Variables") {
            steps {
                sh "printenv"
            }
        }

    stage('Sonarqube Analysis') {
        environment {
            scannerHome = tool 'sonar'
        }
        steps {
            withSonarQubeEnv('sonar') {
                //sh 'mvn clean package sonar:sonar'
                sh "${scannerHome}/bin/sonar-scanner " +
                    "-Dsonar.projectName=${JOB_BASE_NAME} " +
                    "-Dsonar.projectVersion=1.$BUILD_NUMBER " +
                    "-Dsonar.projectKey=${JOB_BASE_NAME}:app "
            }
        }
    }
    stage("Sonarqube Quality Gate") {
        when {
            beforeInput true
            branch 'master'
        }
        input {
            message "Deploy to production?"
            id "simple-input"
        }
        steps {
            // script {
            //   timeout(time: 1, unit: 'MINUTES') {
            //     input(id: "Deploy Gate", message: "Deploy ${params.project_name}?", ok: 'Deploy')
            //   }
            // }
            timeout(time: 1, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
            }
        }
    }
    stage('DeployToProduction') {
        when {
            branch 'master'
        }
        steps {
            input 'Deploy to Production?'
            milestone(1)
            //implement Kubernetes deployment here
        }
    }



        
//         stage("Quality Gate") {
//             when { 
//                 expression {
//                     return Quality Gate
//                 }
//             } 
//             steps {
// //                timeout(time: 1, unit: 'HOURS') {
//                     // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
//                     // true = set pipeline to UNSTABLE, false = don't
//                     // Requires SonarQube Scanner for Jenkins 2.7+
//                     waitForQualityGate abortPipeline: true
// //                }
//             }
//         }
        // stage('Docker Build') {
        //     steps{
        //         script {
        //             dockerImage = docker.build("${registry}/${JOB_BASE_NAME}" + ":$BUILD_NUMBER", " -f Dockerfile .")
        //         }
        //     }
        // }
        // stage('Docker Push') {
        //     steps{
        //         script {
        //             docker.withRegistry( "${registryUrl}/${JOB_BASE_NAME}", "${registryCredential}" ) {
        //                 dockerImage.push('latest')
        //                 dockerImage.push("$BUILD_NUMBER")
        //                 //docker rmi ${registry}/${JOB_BASE_NAME}:$BUILD_NUMBER
        //                 //docker rmi ${registry}/${JOB_BASE_NAME}:latest
        //             }
        //         }
        //     }
        // }

    }
}