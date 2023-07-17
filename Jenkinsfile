node {
    stage('Build') {
        docker.image('python:2-alpine').inside {
            dir('sources') {
                sh 'python -m py_compile add2vals.py calc.py'
                stash(name: 'compiled-results', includes: '/*.py*')
            }
        }
    }
    stage('Test') {
        docker.image('qnib/pytest').inside {
            sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
        }
        post {
            always {
                junit 'test-reports/results.xml'
            }
        }
    }
}

