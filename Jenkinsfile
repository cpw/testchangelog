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
                changelog = addChanges(currentBuild, changelog)
                print(changelog)
                print(changelog.join("\n"))
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
                changelog += "\t" + chg?.msg
            }
            else
            {
                for (pt in chg?.msg?.split('\n'))
                    changelog += "\t" + pt
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
            changelog += "Build: " + next.number
        }
        addChanges(next, changelog)
    }
    print(changelog)
    return changelog
}

