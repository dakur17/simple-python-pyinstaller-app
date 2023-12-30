node {
    stage('Build') {
        docker.image('python:3.12.1-alpine3.19').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash name: 'compiled-results', includes: 'sources/*.py*'
        }
    }

    stage('Test') {
        docker.image('qnib/pytest').inside {
            sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
        }
        junit 'test-reports/results.xml'
    }

    stage('Deploy'){
        def VOLUME = "${pwd()}/sources:/src"

        docker.image('cdrx/pyinstaller-linux:python2').inside("-v ${VOLUME}") {
            dir(env.BUILD_ID) {
                unstash 'compiled-results'
                sh "pyinstaller -F add2vals.py"
            }
        }
        sleep time: 1, unit: "MINUTES"
        docker.image('cdrx/pyinstaller-linux:python2').inside("-v ${VOLUME}") {
            sh "rm -rf build dist"
        }
    }
}