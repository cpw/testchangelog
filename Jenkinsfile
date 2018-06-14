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
                    env.MYVERSION="1.3.4"
                }
            }
        }        
    }
    post {
        always {
            script {
                def changelog = []
                changelog += "Build: ${currentBuild.number} - ${env.MYVERSION} - ${new Date(currentBuild.startTimeInMillis)}"
                changelog = addChanges(currentBuild, changelog)
                writeFile file: "changelog.txt", text: changelog.join("\n")
            }
            archiveArtifacts artifacts: 'changelog.txt', fingerprint: false
        }
    }
}

def addChanges(build, changelog) {
    for (change in build.changeSets)
    {
        for (chg in change?.items)
        {
            if (!chg?.msg?.contains("\n"))
            {
                changelog += "\t${chg.author.toString()}: ${chg.msg}"
            }
            else
            {
                changelog += "\t${chg.author.toString()}:"
                for (pt in chg?.msg?.split('\n'))
                    changelog += "\t\t" + pt
                changelog += "";
            }
        }
    }
    next = build.previousBuild
    if (next != null)
    {
        if (next.result == 'SUCCESS')
        {
            changelog += ""
            changelog += "Build: ${next.number} - ${next.buildVariables.MYVERSION:"NOVERSION"} - ${new Date(next.startTimeInMillis)}"
        }
        changelog = addChanges(next, changelog)
    }
    return changelog
}

