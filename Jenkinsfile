@Library('forge-shared-library')_

pipeline {
    agent {
        docker {
            image 'gradlewrapper:latest'
            args '-v gradlecache:/gradlecache'
        }
    }
    environment {
        GRADLE_ARGS = '-Dorg.gradle.daemon.idletimeout=5000'
    }

    stages {
        stage('fetch') {
            steps {
                git(url: 'https://github.com/cpw/testchangelog.git', changelog: true)
            }
        }
        stage('buildandtest') {
            steps {
                sh './gradlew ${GRADLE_ARGS} --refresh-dependencies --continue build'
                script {
                    env.MYVERSION="2.3.4"
                }
            }
        }        
    }
    post {
        always {
            writeChangelog(currentBuild, 'changelog.txt')
            archiveArtifacts artifacts: 'changelog.txt', fingerprint: false
        }
    }
}
