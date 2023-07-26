node {
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
        docker.image('cdrx/pyinstaller-linux:python2').inside {
            withEnv(["VOLUME=${pwd()}/sources:/src"]) {
                dir(env.BUILD_ID) {
                    unstash('compiled-results')
                    sh "docker run --rm -v ${VOLUME} cdrx/pyinstaller-linux:python2 'pyinstaller -F add2vals.py'"
                }
                sleep 1 // Memberikan jeda selama 1 menit
                archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
                sh "docker run --rm -v ${VOLUME} cdrx/pyinstaller-linux:python2 'rm -rf build dist'"
            }
        }
    }
}
