pipeline {
    agent any

    parameters {
        choice(
            name: 'TEST_TYPE',
            choices: ['smoke', 'regression', 'custom'],
            description: 'Select test type to run'
        )
        string(
            name: 'CUSTOM_TESTS',
            defaultValue: '',
            description: 'Custom test classes or methods (only used when TEST_TYPE = custom). Examples: "com.example.MyTest" or "com.example.MyTest#myMethod"'
        )
        string(
            name: 'BRANCH',
            defaultValue: 'main',
            description: 'Git branch to checkout'
        )
    }

    environment {
        ALLURE_RESULTS_DIR = 'build/allure-results'
    }

    stages {

        stage('Checkout') {
            steps {
                git(
                    url: 'https://github.com/day26t/AppleD.git',
                    branch: "${params.BRANCH}"
                )
            }
        }

        stage('Build') {
            steps {
                sh './gradlew clean compileTestJava --no-daemon'
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    def task = ''
                    switch (params.TEST_TYPE) {
                        case 'smoke':
                            task = 'smokeTests'
                            break
                        case 'regression':
                            task = 'regressionTests'
                            break
                        case 'custom':
                            if (!params.CUSTOM_TESTS?.trim()) {
                                error("TEST_TYPE is 'custom' but CUSTOM_TESTS parameter is empty. Please specify test classes or methods.")
                            }
                            task = "test --tests \"${params.CUSTOM_TESTS.trim()}\""
                            break
                        default:
                            error("Unknown TEST_TYPE: ${params.TEST_TYPE}")
                    }
                    sh "./gradlew ${task} --no-daemon"
                }
            }
        }
    }

    post {
        always {
            allure([
                includeProperties: false,
                jdk: '',
                properties: [],
                reportBuildPolicy: 'ALWAYS',
                results: [[path: "${ALLURE_RESULTS_DIR}"]]
            ])
        }

        success {
            echo "Tests passed: [${params.TEST_TYPE}] on branch [${params.BRANCH}]"
        }

        failure {
            echo "Tests FAILED: [${params.TEST_TYPE}] on branch [${params.BRANCH}]"
        }

        cleanup {
            cleanWs()
        }
    }
}
