pipeline {
    agent any

    environment {
        PYTHONPATH = '.'
        PATH_VENV = '/opt/venv/bin'
        PATH_JMETER = '/opt/jmeter/bin'
        FLASK_APP='app/api.py'
    }

    stages {

        stage('Checkout code') {
            steps {
                git 'https://github.com/flaviooria/devops_unir_cp.git'
                sh 'echo $WORKSPACE'
            }
        }
        
        stage('Run unit test') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh '$PATH_VENV/coverage run --branch --source=./app --omit=app/__init__.py,app/api.py -m pytest --junitxml=result-unit.xml test/unit'
                    sh '$PATH_VENV/coverage xml'
                }

                junit 'result-unit.xml'
            }
        }

        stage('Coverage') {
            steps {
                sh '$PATH_VENV/coverage report'

                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    // Formato de baremos cobertura: healthy, unstable, failed => 95, 85, 85 
                    cobertura autoUpdateHealth: false, autoUpdateStability: false, onlyStable: false, coberturaReportFile: 'coverage.xml', conditionalCoverageTargets: '100,0,90', lineCoverageTargets: '100,0,95'
                }

            }
        }


        stage('Static') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh '$PATH_VENV/flake8 --format=pylint --exit-zero ./app > flake8.out'

                    recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out', reportEncoding: 'UTF-8')], qualityGates: [[threshold: 8.0, type: 'TOTAL', unstable: true], [threshold: 10.0, type: 'TOTAL', unstable: false]]
                }
            }
        }

        stage('Security') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh '$PATH_VENV/bandit --exit-zero -r . -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}] {severity} {msg}"'

                    recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates: [[unstable: true, threshold: 2.0, type: 'TOTAL'], [unstable: false, threshold: 4.0, type: 'TOTAL']]
                }
            }
        }

        stage('Performance') {
            steps {
                sh '$PATH_VENV/flask run &' // Run the app in the background
                sh 'sleep 3' // Wait for the app to start

                sh '$PATH_JMETER/jmeter -n -t test/jmeter/flask.jmx -f -l flask.jtl'


                perfReport sourceDataFiles: 'flask.jtl'
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}