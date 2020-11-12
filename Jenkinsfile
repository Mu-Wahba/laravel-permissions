pipeline {
  agent any
  environment {
    DB_CONNECTION = 'mongodb'
    DB_HOST = '127.0.0.1'
    DB_PORT = '27017'
    DB_DATABASE = "test${BUILD_TAG}"
    DB_USERNAME = ''
    DB_PASSWORD = ''
  }
  stages {
    stage('Set up Environment') {
      steps {
        sh 'composer install -o'
        sh 'rm -rf build/coverage'
        sh 'rm -rf build/logs'
        sh 'rm -rf build/pdepend'
        sh 'mkdir -p build/coverage'
        sh 'mkdir -p build/logs'
        sh 'mkdir -p build/pdepend'
        sh 'cp .env.example .env'
        sh 'php artisan key:generate'
      }
    }
    stage('PHP Syntax check') {
      steps {
        sh 'composer parallel-lint'
      }
    }
    stage('Checks') {
      parallel {
        stage('Test') {
          steps {
            sh 'composer test'
          }
          post {
            always {
              junit 'build/logs/junit.xml'
              // Publish Coverage
              sh 'cp build/logs/clover.xml build/coverage/clover.xml'
              step([
                $class: "CloverPublisher",
                cloverReportDir: "build/coverage/",
                cloverReportFileName: "clover.xml",
                healthyTarget: [methodCoverage: 90, conditionalCoverage: 90, statementCoverage: 90],
                unhealthyTarget: [methodCoverage: 50, conditionalCoverage: 50, statementCoverage: 50],
                failingTarget: [methodCoverage: 30, conditionalCoverage: 30, statementCoverage: 30]
              ])
              // Publish Crap4J
              /*step([
                $class: 'Crap4JPublisher',
                reportPattern: 'build/logs/crap4j.xml',
                healthThreshold: '10'
              ])*/
            }
          }
        }
        stage('Miss Detection') {
          steps {
            sh 'composer phpmd'
          }
          post {
            always {
              pmd canRunOnFailed: true, pattern: 'build/logs/phpmd.xml'
            }
          }
        }
        stage('Checkstyle') {
          steps {
            sh 'composer phpcs'
          }
          post {
            always {
              checkstyle canRunOnFailed: true, pattern: 'build/logs/checkstyle.xml'
            }
          }
        }
        stage('Copy Paste Detection') {
          steps {
            sh 'composer phpcpd'
          }
          post {
            always {
              dry canRunOnFailed: true, pattern: 'build/logs/pmd-cpd.xml'
            }
          }
        }
        stage('Software metrics') {
          steps {
            sh 'composer pdepend'
          }
        }
        stage('Lines of Code') {
          steps {
            sh 'composer phploc'
          }
          post {
            always {
              plot csvFileName: 'phploc.csv', group: 'test', style: 'line'
            }
          }
        }
      }
    }
    stage('Analysis Publisher') {
      steps {
        step([$class: 'AnalysisPublisher'])
      }
    }
  }
  post {
    always {
      // Drop database after test
      sh 'mongo test_${BUILD_TAG} --eval "db.dropDatabase()"'
    }
    success {
      echo 'I succeeded!'
    }
    unstable {
      echo 'I am unstable :/'
    }
    failure {
      echo 'I failed :('
    }
    changed {
      echo 'Things were different before...'
    }
  }
}
