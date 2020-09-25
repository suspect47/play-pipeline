#!/usr/bin/env/groovy!

pipeline {
    options {
        buildDiscarder(logRotator(numToKeepStr: '3', artifactNumToKeepStr: '3'))
    }
    agent {
        label 'build-linux-x64'
    }
    parameters {
	string(name: 'ULTIMA_MAJOR_VERSION', defaultValue: '0')
	string(name: 'ULTIMA_MINOR_VERSION', defaultValue: '0')
	string(name: 'ULTIMA_ROOT_REVISION', defaultValue: '0')
	string(name: 'GITHASH_ULTIMA_PLAY', defaultValue: 'refs/heads/master')   
        string(name: 'NEXE_NODE_VERSION', defaultValue: 'linux-x64-12.16.2')
    }
    environment {
	WORKSTATION_VERSION="${ULTIMA_MAJOR_VERSION}.${ULTIMA_MINOR_VERSION}.${ULTIMA_ROOT_REVISION}"
        RESULT_FILENAME="neyross-workstation-${ULTIMA_MAJOR_VERSION}.${ULTIMA_MINOR_VERSION}.${ULTIMA_ROOT_REVISION}.sh"
    }  
    stages {
        stage('Cloning ultima-play repo') {
            steps {
		sh 'rm -rf $WORKSPACE/apps/vmc/setup/neyross-workstation-linux/installer/*'
                git(branch: 'master', url: 'git@git.itrium-spb.ru:ultima/ultima-play.git', credentialsId: '19a6d233-4e91-4567-a67e-5a1211786063')
            }
        }
        stage('Installing node modules...Building workstation executable...') {
            steps {
                sh 'echo "${WORKSTATION_VERSION}"'
		sh 'cd apps/vmc/setup/neyross-workstation-linux && npm i && ./node_modules/.bin/nexe ./index.js -t ${NEXE_NODE_VERSION} -o target/neyross-workstation'
            }
        }
	stage ('Editing workstation version in workstation-config.json') {
            steps {
                sh 'sed -i -e s/WORKSTATION_VERSION/"${WORKSTATION_VERSION}"/g $WORKSPACE/apps/vmc/setup/neyross-workstation-linux/installer/neyross-workstation/opt/Neyross/neyross-workstation/workstation-config.json'
            }
        }
	stage ('Building deb package neyross-workstation.deb') {
	    steps {
		sh 'cp -rf $WORKSPACE/apps/vmc/setup/neyross-workstation-linux/target/neyross-workstation $WORKSPACE/apps/vmc/setup/neyross-workstation-linux/installer/neyross-workstation/opt/Neyross/neyross-workstation/'
	        sh 'cd $WORKSPACE/apps/vmc/setup/neyross-workstation-linux/installer && dpkg-deb --build neyross-workstation'
	    }
	} 
	stage ('Downloading latest google-chrome.deb') {
	    steps {
		sh 'wget -P $WORKSPACE/apps/vmc/setup/neyross-workstation-linux/installer/dependencies https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb'
	    }
	}
	stage ('Archiving files to files.tar.gz') {
	    steps {
		sh 'cd $WORKSPACE/apps/vmc/setup/neyross-workstation-linux/installer && tar -czvf files.tar.gz neyross-workstation.deb dependencies'
	    }
        }
	stage ('Making sh-installer') {
            steps {
                sh 'touch $WORKSPACE/apps/vmc/setup/neyross-workstation-linux/installer/$RESULT_FILENAME'
                sh 'cat $WORKSPACE/apps/vmc/setup/neyross-workstation-linux/installer/neyross-workstation.sh $WORKSPACE/apps/vmc/setup/neyross-workstation-linux/installer/files.tar.gz > $WORKSPACE/apps/vmc/setup/neyross-workstation-linux/installer/$RESULT_FILENAME'
		sh 'chmod +x $WORKSPACE/apps/vmc/setup/neyross-workstation-linux/installer/$RESULT_FILENAME'
	    }
        }
        stage ('Archiving the artifacts') {
            steps {
                archiveArtifacts artifacts: 'apps/vmc/setup/neyross-workstation-linux/target/neyross-workstation,apps/vmc/setup/neyross-workstation-linux/installer/neyross-workstation.deb,apps/vmc/setup/neyross-workstation-linux/installer/neyross-workstation-*.sh', followSymlinks: false
            }
        }
    }
}
