#!/usr/bin/env groovy

pipeline {
  agent any
    stages {
      stage('Build') {
        steps {
          sh '''
            rm -f application-help.tar.gz application
          '''
          checkout scm
          sh '''
            docker-compose run --rm -u "$USER_UID:$GROUP_GID" asciidoctor
            mv application/collaborative-editor application/collaborativeeditor
            mv application/scrap-book application/scrapbook
            mv application/search-engine application/searchengine
            mv application/share-big-files application/sharebigfiles
            tar cfzh application-help.tar.gz application/* assets wp-content
	  '''
        }
      }
      stage('Publish') {
        steps {
          sh '''
            VERSION=`grep 'version=' VERSION | sed 's/version=//'`
            case "$VERSION" in
              *SNAPSHOT) nexusRepository='snapshots' ;;
              *)         nexusRepository='releases' ;;
            esac
            mvn deploy:deploy-file -DgroupId=com.opendigitaleducation -DartifactId=application-help -Dversion=$VERSION -Dpackaging=tar.gz -Dfile=application-help.tar.gz -DrepositoryId=wse -Durl=http://maven.opendigitaleducation.com/nexus/content/repositories/$nexusRepository/
          '''
        }
      }
    }
}

