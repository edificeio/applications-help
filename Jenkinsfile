#!/usr/bin/env groovy

pipeline {
  agent any
    stages {
      stage('Build') {
        steps {
          sh '''
            rm -rf application-help.tar.gz application assets help-2d
          '''
          checkout scm
          sh '''
            for app in `find ./application -type f -name '*.md' | cut -d. -f2 | cut -d'/' -f3 | cat`; 
            do
                mkdir "application/$app"
                sed -i 's/!\\[.*\\](\\.gitbook\\(.*\\))/![](\\1)/g' application/${app}.md
                sed -i '1d' application/${app}.md
                sed -i -e '/{%.*%}/d' application/${app}.md
                sed -i 's/\\(##.*\\){#.*}/\\1/g' application/${app}.md
                docker-compose run --rm pandoc -s --toc --section-divs -f markdown -t html /application/${app}.md -o /application/${app}/index.html
                echo "Processed $app"
            done
            mv application/collaborative-editor application/collaborativeeditor
            mv application/scrap-book application/scrapbook
            mv application/search-engine application/searchengine
            mv application/share-big-files application/sharebigfiles
            cp -Rf application/.gitbook/assets .
            rm -Rf application/*.md application/.gitbook
            mkdir help-2d && mv application assets help-2d 
            tar cfzh application-help.tar.gz help-2d
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

