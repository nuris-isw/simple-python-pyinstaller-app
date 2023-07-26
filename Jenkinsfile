node {
    try {
        options {
            skipStagesAfterUnstable()
        }

        stage('Build') {
            node {
                docker.image('python:2-alpine').inside {
                    sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                    stash(name: 'compiled-results', includes: 'sources/*.py*')
                }
            }
        }

        stage('Test') {
            node {
                docker.image('qnib/pytest').inside {
                    sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
                }
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }

        stage('Manual Approval') {
            node {
                input "Lanjutkan ke tahap Deploy?"
            }
        }

        stage('Deploy') {
            node {
                withEnv(["VOLUME='${pwd()}/sources:/src'", "IMAGE='cdrx/pyinstaller-linux:python2'"]) {
                    dir(env.BUILD_ID) {
                        unstash('compiled-results')
                        sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
                    }
                    sleep 1 // Memberikan jeda selama 1 menit
                    archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
                }
            }
        }
    } catch (err) {
        echo "Error: ${err}"
    }
}
