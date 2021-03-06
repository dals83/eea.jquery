pipeline {
  agent any

  environment {
        GIT_NAME = "eea.jquery"
    }

  stages {
    stage('Tests') {
      steps {
        parallel(

          "WWW": {
            node(label: 'docker-1.13') {
              script {
                try {
                  sh '''docker run -i --net=host --name="$BUILD_TAG-www" -e GIT_NAME="$GIT_NAME" -e GIT_BRANCH="$BRANCH_NAME" eeacms/www-devel /debug.sh bin/test -v -vv -s $GIT_NAME'''
                } finally {
                  sh '''docker rm -v $BUILD_TAG-www'''
                }
              }
            }
          },

          "KGS": {
            node(label: 'docker-1.13') {
              script {
                try {
                  sh '''docker run -i --net=host --name="$BUILD_TAG-kgs" -e GIT_NAME="$GIT_NAME" -e GIT_BRANCH="$BRANCH_NAME" eeacms/kgs-devel /debug.sh bin/test --test-path /plone/instance/src/$GIT_NAME -v -vv -s $GIT_NAME'''
                } finally {
                  sh '''docker rm -v $BUILD_TAG-kgs'''
                }
              }
            }
          },

          "Plone4": {
            node(label: 'docker-1.13') {
              script {
                try {
                  sh '''docker run -i --net=host --name="$BUILD_TAG-plone4" -v /plone/instance/parts -e GIT_BRANCH="$BRANCH_NAME" -e ADDONS="$GIT_NAME" -e DEVELOP="src/$GIT_NAME" eeacms/plone-test:4 -v -vv -s $GIT_NAME'''
                } finally {
                  sh '''docker rm -v $BUILD_TAG-plone4'''
                }
              }
            }
          }
        )
      }
    }

    stage('Code Analysis') {
      steps {
        parallel(

          "ZPT Lint": {
            node(label: 'docker-1.13') {
              script {
                try {
                  sh '''docker run -i --net=host --name="$BUILD_TAG-zptlint" -e GIT_SRC="https://github.com/eea/$GIT_NAME.git --branch=$BRANCH_NAME" eeacms/zptlint'''
                } finally {
                  sh '''docker rm -v $BUILD_TAG-zptlint'''
                }
              }
            }
          },

          "JS Lint": {
            node(label: 'docker-1.13') {
              script {
                try {
                  sh '''docker run -i --net=host --name="$BUILD_TAG-jslint" -e GIT_SRC="https://github.com/eea/$GIT_NAME.git --branch=$BRANCH_NAME" eeacms/jslint4java'''
                } catch (err) {
                  echo "Unstable: ${err}"
                } finally {
                  sh '''docker rm -v $BUILD_TAG-jslint'''
                }
              }
            }
          },

          "PyFlakes": {
            node(label: 'docker-1.13') {
              script {
                try {
                  sh '''docker run -i --net=host --name="$BUILD_TAG-pyflakes" -e GIT_SRC="https://github.com/eea/$GIT_NAME.git --branch=$BRANCH_NAME" eeacms/pyflakes'''
                } finally {
                  sh '''docker rm -v $BUILD_TAG-pyflakes'''
                }
              }
            }
          },

          "i18n": {
            node(label: 'docker-1.13') {
              script {
                try {
                  sh '''docker run -i --net=host --name=$BUILD_TAG-i18n -e GIT_SRC="https://github.com/eea/$GIT_NAME.git --branch=$BRANCH_NAME" eeacms/i18ndude'''
                } finally {
                  sh '''docker rm -v $BUILD_TAG-i18n'''
                }
              }
            }
          }
        )
      }
    }

    stage('Code Syntax') {
      steps {
        parallel(

          "JS Hint": {
            node(label: 'docker-1.13') {
              script {
                try {
                  sh '''docker run -i --net=host --name="$BUILD_TAG-jshint" -e GIT_SRC="https://github.com/eea/$GIT_NAME.git --branch=$BRANCH_NAME" eeacms/jshint'''
                } catch (err) {
                  echo "Unstable: ${err}"
                } finally {
                  sh '''docker rm -v $BUILD_TAG-jshint'''
                }
              }
            }
          },

          "CSS Lint": {
            node(label: 'docker-1.13') {
              script {
                try {
                  sh '''docker run -i --net=host --name="$BUILD_TAG-csslint" -e GIT_SRC="https://github.com/eea/$GIT_NAME.git --branch=$BRANCH_NAME" eeacms/csslint'''
                } catch (err) {
                  echo "Unstable: ${err}"
                } finally {
                  sh '''docker rm -v $BUILD_TAG-csslint'''
                }
              }
            }
          },

          "PEP8": {
            node(label: 'docker-1.13') {
              script {
                try {
                  sh '''docker run -i --net=host --name="$BUILD_TAG-pep8" -e GIT_SRC="https://github.com/eea/$GIT_NAME.git --branch=$BRANCH_NAME" eeacms/pep8'''
                } catch (err) {
                  echo "Unstable: ${err}"
                } finally {
                  sh '''docker rm -v $BUILD_TAG-pep8'''
                }
              }
            }
          },

          "PyLint": {
            node(label: 'docker-1.13') {
              script {
                try {
                  sh '''docker run -i --net=host --name="$BUILD_TAG-pylint" -e GIT_SRC="https://github.com/eea/$GIT_NAME.git --branch=$BRANCH_NAME" eeacms/pylint'''
                } catch (err) {
                  echo "Unstable: ${err}"
                } finally {
                  sh '''docker rm -v $BUILD_TAG-pylint'''
                }
              }
            }
          }

        )
      }
    }

  }

  post {
    changed {
      script {
        def url = "${env.BUILD_URL}/display/redirect"
        def status = currentBuild.currentResult
        def subject = "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
        def summary = "${subject} (${url})"
        def details = """<h1>${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - ${status}</h1>
                         <p>Check console output at <a href="${url}">${env.JOB_BASE_NAME} - #${env.BUILD_NUMBER}</a></p>
                      """

        def color = '#FFFF00'
        if (status == 'SUCCESS') {
          color = '#00FF00'
        } else if (status == 'FAILURE') {
          color = '#FF0000'
        }
        slackSend (color: color, message: summary)
        emailext (subject: '$DEFAULT_SUBJECT', to: '$DEFAULT_RECIPIENTS', body: details)
      }
    }
  }
}
