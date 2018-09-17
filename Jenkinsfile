#!/usr/bin/env groovy

pipeline {
  agent any
    stages {
      stage('Build') {
        steps {
          sh '''
            rm -rf application-help.tar.gz application assets
          '''
          checkout scm
          sh '''
            for app in `find ./application -type f -name '*.md' | cut -d. -f2 | cut -d'/' -f3 | cat`; 
            do
                mkdir "application/$app"
                sed -i 's/!\\[\\](\\.gitbook\\(.*\\))/![](\\1)/g' application/${app}.md
                sed -i '1d' application/${app}.md
                docker-compose run --rm pandoc -s --toc --section-divs -f markdown -t html /application/${app}.md -o /application/${app}/index.html
                echo "Processed $app"
            done
            mv application/collaborative-editor application/collaborativeeditor
            mv application/scrap-book application/scrapbook
            mv application/search-engine application/searchengine
            mv application/share-big-files application/sharebigfiles
            mv application/.gitbook/assets assets
            rm -Rf application/*.md application/.gitbook
            tar cfzh application-help.tar.gz application/* assets
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

