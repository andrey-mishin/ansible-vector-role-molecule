pipeline {
    agent any
        stages {
            stage('download repo') {
                steps {
                    git 'https://github.com/andrey-mishin/ansible-vector-role-molecule'
                }
            }
            stage('test') {
                steps {
                    dir('./roles/vector-role/') {
                        sh 'molecule test'
                    }
                }
            }
        }
}
