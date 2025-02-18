pipeline {

    agent {
        docker {
            image 'gibsonchallenge/gibsonv2:jenkins2'
            args '--runtime=nvidia -v ${WORKSPACE}/../ig_dataset:${WORKSPACE}/igibson/data/ig_dataset'
        }
    }

    stages {
        stage('Build') {
            steps {
                sh 'nvidia-smi'
                sh 'pwd'
                sh 'printenv'
                sh 'pip install -e .'
                sh 'sudo chown -R jenkins ${WORKSPACE}/igibson/data/'
            }
        }

        stage('Build Docs') {
            steps {
                sh 'sphinx-apidoc -o docs/apidoc igibson igibson/external igibson/utils/data_utils/'
                sh 'sphinx-build -b html docs _sites'
            }
        }

        stage('Test') {
            steps {
                sh 'mkdir result'
                sh 'pytest igibson/test/test_binding.py --junitxml=test_result/test_binding.py.xml'
                sh 'pytest igibson/test/test_render.py --junitxml=test_result/test_render.py.xml'
                sh 'pytest igibson/test/test_pbr.py --junitxml=test_result/test_pbr.py.xml'
                sh 'pytest igibson/test/test_object.py --junitxml=test_result/test_object.py.xml'
                sh 'pytest igibson/test/test_simulator.py --junitxml=test_result/test_simulator.py.xml'
                sh 'pytest igibson/test/test_igibson_env.py --junitxml=test_result/test_igibson_env.py.xml'
                sh 'pytest igibson/test/test_scene_importing.py --junitxml=test_result/test_scene_importing.py.xml'
                sh 'pytest igibson/test/test_robot.py --junitxml=test_result/test_robot.py.xml'
                sh 'pytest igibson/test/test_igsdf_scene_importing.py --junitxml=test_result/test_igsdf_scene_importing.py.xml'
                sh 'pytest igibson/test/test_sensors.py --junitxml=test_result/test_sensors.py.xml'
                sh 'pytest igibson/test/test_motion_planning.py --junitxml=test_result/test_motion_planning.py.xml'
                sh 'pytest igibson/test/test_states.py --junitxml=test_result/test_states.py.xml'
                sh 'pytest igibson/test/test_determinism_against_same_version.py --junitxml=test_result/test_determinism_against_same_version.py.xml'
            }
        }

        stage('Benchmark') {
            steps {
                sh 'python igibson/test/benchmark/benchmark_static_scene.py'
                sh 'python igibson/test/benchmark/benchmark_interactive_scene.py'
            }
        }
    
    }
    post { 
        always { 
            junit 'test_result/*.xml'
            archiveArtifacts artifacts: 'test_result/*.xml', fingerprint: true
            archiveArtifacts artifacts: '*.pdf'
            archiveArtifacts artifacts: '*.png'

            publishHTML (target: [
              allowMissing: true,
              alwaysLinkToLastBuild: false,
              keepAll: true,
              reportDir: '_sites',
              reportFiles: 'index.html',
              includes: '**/*',
              reportName: "iGibson docs"
            ])

            cleanWs()
        }
        failure {
            script {
                    // Send an email only if the build status has changed from green/unstable to red
                    emailext subject: '$DEFAULT_SUBJECT',
                        body: '$DEFAULT_CONTENT',
                        recipientProviders: [
                            [$class: 'CulpritsRecipientProvider'],
                            [$class: 'DevelopersRecipientProvider'],
                            [$class: 'RequesterRecipientProvider']
                        ], 
                        replyTo: '$DEFAULT_REPLYTO',
                        to: '$DEFAULT_RECIPIENTS'
                }
            }
        }
}
