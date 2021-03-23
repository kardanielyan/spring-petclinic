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
        steps {
            input 'Deploy to Production?'
            milestone(1)
            sh 'echo "Deploy"'
        }
    }

    stage('Example') {
        input {
            message "Should we continue?"
            ok "Yes, we should."
            submitter "Karen"
            parameters {
                string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
            }
        }
        steps {
            echo "Hello, ${PERSON}, nice to meet you."
        }
    }
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