#!/usr/bin/env groovy

pipeline {
  agent any
    stages {
      stage('Build') {
        steps {
          checkout scm
          sh '''
	    rm -f application-help-*.tar.gz
            docker-compose run --rm -u "$USER_UID:$GROUP_GID" asciidoctor
            rm -f application/**/*.adoc
            export VERSION=`git describe --abbrev=0 --tags`
            if [ -z "$VERSION" ]
            then
                export BRANCH=`git branch | grep "*"`
                export VERSION=`echo $BRANCH | sed 's|^* ||'`
            fi
            tar cfzh application-help-${VERSION}.tar.gz application/*
	  '''
        }
      }
      stage('Publish') {
        steps {
          sh '''
	    export VERSION=`git describe --abbrev=0 --tags`
            if [ -z "$VERSION" ]
            then
                export BRANCH=`git branch | grep "*"`
                export VERSION=`echo $BRANCH | sed 's|^* ||'`
            fi
	    export archiveName=`ls application-help-*.tar.gz`
            mvn deploy:deploy-file -DgroupId=com.opendigitaleducation -DartifactId=application-help -Dversion=$VERSION -Dpackaging=tar.gz -Dfile=$archiveName -DrepositoryId=releases -Durl=https://maven.opendigitaleducation.com/nexus/content/repositories/releases
          '''
        }
      }
    }
}

