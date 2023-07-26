node {
    def VOLUME = "${pwd()}/sources:/src"
    def IMAGE = "cdrx/pyinstaller-linux:python2"

    stage('Build') {
        docker.image('python:2-alpine').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash(name: 'compiled-results', includes: 'sources/*.py*')
        }
    }

    stage('Test') {
        try {
            docker.image('qnib/pytest').inside {
                sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
            }
        } finally {
            junit 'test-reports/results.xml'
        }
    }

    stage('Manual Approval') {
        input "Lanjutkan ke tahap Deploy?"
    }

    stage('Deploy') {
        env.VOLUME = VOLUME
        env.IMAGE = IMAGE

        dir(env.BUILD_ID) {
            unstash 'compiled-results'
            sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
        }

        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
            sleep(60)
            archiveArtifacts "sources/dist/add2vals"
            sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
        }
    }
}
