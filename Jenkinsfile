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
            steps {
                withSonarQubeEnv('sonar') {
                    sh "${jenkinsHome}/tools/hudson.plugins.sonar.SonarRunnerInstallation/sonar/bin/sonar-scanner " +
                    "-Dsonar.host.url=http://192.168.56.110:9000/ " +
                    "-Dsonar.projectName=${appName} " +
                    "-Dsonar.projectVersion=1.$BUILD_NUMBER " +
                    "-Dsonar.projectKey=${appName}:app " //+
                   // "-Dsonar.sources=. " +
                   // "-Dsonar.projectBaseDir=${jenkinsHome}/workspace/K8s/K8s_test_pipeline/APP/"
                    //'-Dsonar.language=html '

                    //sh 'cat ${jenkinsHome}/tools/hudson.plugins.sonar.SonarRunnerInstallation/sonar/conf/sonar-scanner.properties'
                }
            }
        }
        
        stage("Quality Gate") {
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
                    dockerImage = docker.build("${registry}/${appName}" + ":$BUILD_NUMBER", " -f Dockerfile .")
                }
            }
        }
        stage('Docker Push') {
            steps{
                script {
                    docker.withRegistry( "${registryUrl}/${appName}", "${registryCredential}" ) {
                        dockerImage.push('latest')
                        dockerImage.push("$BUILD_NUMBER")
                        //docker rmi ${registry}/${appName}:$BUILD_NUMBER
                        //docker rmi ${registry}/${appName}:latest
                    }
                }
            }
        }

    }
}