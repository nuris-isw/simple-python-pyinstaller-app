node {
    checkout scm

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
        def approval = input(
            id: 'manual-approval',
            message: 'Apakah Anda ingin melanjutkan ke tahap Deploy?',
            ok: 'Ya',
            submitter: 'dicoding, nuris-isw',
            parameters: [
                string(defaultValue: 'Ya', description: 'Persetujuan', name: 'approvalParam')
            ]
        )

        if (approval == 'Ya') {
            echo 'Melanjutkan ke tahap Deploy...'
        } else {
            error('Persetujuan ditolak. Proses dihentikan.')
        }
    }

    stage('Deploy') {
        docker.image('cdrx/pyinstaller-linux:python2').inside {
            withEnv(["VOLUME=$(pwd)/sources:/src"]) {
                dir("${env.BUILD_ID}") {
                    unstash(name: 'compiled-results')
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
                }
            }
            archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
            sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
        }
    }
}
