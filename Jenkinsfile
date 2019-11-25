pipeline {
  agent any
  triggers {
      pollSCM('*/5 * * * 1-5')
  }
  options {
      skipDefaultCheckout(true)
      // Keep the 10 most recent builds
      buildDiscarder(logRotator(numToKeepStr: '10'))
      //timestamps()
  }
  environment {
     PATH = "$WORKSPACE/miniconda/bin:$PATH"
  }

  //timestamps {
    stages {
      stage('setup miniconda') {
          steps {
              sh '''#!/usr/bin/env bash
              wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh -O miniconda.sh
              bash miniconda.sh -u -b -p $WORKSPACE/miniconda
              hash -r
              conda config --set always_yes yes --set changeps1 no
              conda update -q conda

                # Useful for debugging any issues with conda
              conda info -a
              #conda config --add channels defaults
              #conda config --add channels conda-forge
              #conda config --add channels bioconda
  
              # create snakemake-workflows env
              conda init bash
              conda create -n env python=3
              '''
         }
      }
      stage('Test downloading') {
              steps {
                  sh '''#!/usr/bin/env bash
                  source $WORKSPACE/miniconda/etc/profile.d/conda.sh
                  conda activate miniconda/envs/env/
                  '''
              }
          }
  
      stage ("Code pull"){
        steps {
          checkout scm
        }
      }
      stage('Build environment') {
        steps {
          sh '''conda create --yes -n ${BUILD_TAG} python
                source activate ${BUILD_TAG} 
                pip install -r requirements.txt
                '''
        }
      }
      stage('Test environment') {
        steps {
          sh '''source activate ${BUILD_TAG} 
                        cd myproject
                        python3 manage.py test
             '''
         }
        }
     }
     post {
       always {
             sh 'conda remove --yes -n ${BUILD_TAG} --all'
          }
       failure {
             echo "Send e-mail, when failed"
          }
      }
  }
//}

