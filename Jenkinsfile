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
        dir(env.BUILD_ID){
            unstash name: 'compiled-results'
            sh "docker run --rm -v ${pwd()}/sources:/src cdrx/pyinstaller-linux:python2 'pyinstaller -F add2vals.py'"
        }

        sleep time: 1, unit: "MINUTES"

        sh "docker run --rm -v ${pwd()}/sources:/src cdrx/pyinstaller-linux:python2 'rm -rf build dist'"
    }
}