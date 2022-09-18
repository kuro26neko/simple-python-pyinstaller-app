node {
  checkout scm

  docker.image('python:2-alpine').inside {
    stage('Build') {
      sh 'python -m py_compile sources/add2vals.py sources/calc.py'
      stash(name: 'compiled-results', includes: 'sources/*.py*')
    }
  }
  
  docker.image('qnib/pytest').inside {
    try {
      stage('Test') {
        sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
      }
    } finally {
      junit 'test-reports/results.xml'
    }
  }


  stage('Deliver') {
    withEnv(['VOLUME=$(pwd)/sources:/src', 'IMAGE=\'cdrx/pyinstaller-linux:python2\'']) {
      dir(path: env.BUILD_ID) {
          unstash(name: 'compiled-results')
          sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
      }
    archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
    }
  }
}
