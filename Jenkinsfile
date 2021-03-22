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
        stage('SonarQube Analysis') {
            when { 
                expression {
                    return RunSonar
                }
            }
            steps {
                when (BRANCH_NAME != 'dev') {
                    withSonarQubeEnv(installationName: 'sonar', credentialsId: 'sonar') {
                        sh "mvn -B clean deploy sonar:sonar"
                    }
                }

                    //sh "${JENKINS_HOME}/tools/hudson.plugins.sonar.SonarRunnerInstallation/sonar/bin/sonar-scanner " +
                    //"-Dsonar.host.url=http://192.168.56.110:9000/ " +
                    //"-Dsonar.projectName=${JOB_BASE_NAME} " +
                    //"-Dsonar.projectVersion=1.$BUILD_NUMBER " +
                    //"-Dsonar.projectKey=${JOB_BASE_NAME}:app " //+
                   // "-Dsonar.sources=. " +
                   // "-Dsonar.projectBaseDir=${JENKINS_HOME}/workspace/K8s/K8s_test_pipeline/APP/"
                    //'-Dsonar.language=html '

                    //sh 'cat ${JENKINS_HOME}/tools/hudson.plugins.sonar.SonarRunnerInstallation/sonar/conf/sonar-scanner.properties'
                //}
            }
        }
        
        stage("Quality Gate") {
            when { 
                expression {
                    return RunGate
                }
            }
            steps {
//                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    // Requires SonarQube Scanner for Jenkins 2.7+
                    waitForQualityGate abortPipeline: true
//                }
            }
        }
        stage('Docker Build') {
            steps{
                script {
                    dockerImage = docker.build("${registry}/${JOB_BASE_NAME}" + ":$BUILD_NUMBER", " -f Dockerfile .")
                }
            }
        }
        stage('Docker Push') {
            steps{
                script {
                    docker.withRegistry( "${registryUrl}/${JOB_BASE_NAME}", "${registryCredential}" ) {
                        dockerImage.push('latest')
                        dockerImage.push("$BUILD_NUMBER")
                        //docker rmi ${registry}/${JOB_BASE_NAME}:$BUILD_NUMBER
                        //docker rmi ${registry}/${JOB_BASE_NAME}:latest
                    }
                }
            }
        }

    }
}