pipeline {
    node {
        label 'maven'
    }
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
                sh 'ls -la'
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
        // stage('Deliver') { 
        //     steps {
        //         sh './jenkins/scripts/deliver.sh' 
        //     }
        // }
    }
}