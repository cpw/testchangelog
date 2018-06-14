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
                def changelog = []
                changelog += "Build: " + currentBuild.number
                addChanges(currentBuild, changelog)
                def changelogString = changelog.join("\n")
                writeFile file: "build/changelog.txt", text: "${changelogString}"
            }
            archiveArtifacts artifacts: 'build/*.txt', fingerprint: false
        }
    }
}

def addChanges(build, changelog) {
    for (change in build.changeSets)
    {
        for (chg in chg?.items)
        {
            for (entry in chg?.entries)
            {
                if (!entry?.msg?.contains("\n"))
                {
                    changelog += "\t" + change?.msg
                }
                else
                {
                    for (pt in entry?.msg?.split('\n'))
                        changelog += "\t" + pt
                    changelog += "";
                }
            }
        }
    }
    next = build.previousBuild
    if (next != null)
    {
        if (next.result == 'SUCCESS')
        {
            changelog += ""
            changelog += "Build: " + next.number
        }
        addChanges(next, changelog)
    }
}

