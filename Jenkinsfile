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
            }
        }        
    }
    post {
        always {
            script {
                changelog = []

                def addChanges(build) {
                    for (change : build.getChangeSets)
                    {
                        if (!change.getMsg().contains("\n"))
                        {
                            changelog += "\t" + change.getMsg()
                        }
                        else
                        {
                            for (pt : change.getMsg().split('\n'))
                                changelog += "\t" + pt
                            changelog += "";
                        }
                    }
                    next = build.getPreviousBuild()
                    if (next != null)
                    {
                        if (next.result == 'SUCCESS')
                        {
                            changelog += ""
                            changelog += "Build: " + next.number
                        }
                        addChanges(next)
                    }
                }
                changelog += "Build: " + currentBuild.number
                addChanges(currentBuild)
                def changelogString = '\n'.join(changelog)
                writeFile file: "build/changelog.html", text: "${changelogString}"
            }
        }
    }
}
