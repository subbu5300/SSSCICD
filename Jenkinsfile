pipeline {
	agent any

    environment
    {
        GIT_BRANCH_NAME = "1"
        MY_BUILD_VERSION = "1"
        proj_version = "1"
    }

    stages {
       
       stage('Checkout & Build') {
       	  steps {
       	  	script {

       	  		my_scm_fn = checkout changelog: false, scm: [$class: 'GitSCM', branches: [[name: '**']], 
       	  		doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], 
       	  		userRemoteConfigs: [[credentialsId: 'git_creds', url: 'https://github.com/anurag4418/helloworld-ansible.git']]]
       	  		echo "${my_scm_fn}"
       	  		MY_BUILD_VERSION = my_scm_fn.GIT_COMMIT[0..4]
       	  		echo MY_BUILD_VERSION
       	  		GIT_BRANCH_NAME = my_scm_fn.GIT_BRANCH
                def mvnHome = tool name: 'maven3', type: 'maven'
                sh "${mvnHome}/bin/mvn -Drevision=${MY_BUILD_VERSION} clean package"

       	  	}
       	  }
       }

		stage('Publish')
		{
			when
			 {
				    expression { 
				        GIT_BRANCH_NAME ==~ /.*master|.*feature|.*develop|.*hotfix/
				    }
			}
			steps
			{
				script
				{
					def mvnHome = tool name: 'maven3', type: 'maven'
					sh "${mvnHome}/bin/mvn -Drevision=${MY_BUILD_VERSION} clean deploy"
					
				}
			}

		}

	    stage('sonar') {

	    	environment {
	    		scannerHome = tool 'sonar_scanner'
	    	}
	    	steps {
	    		withSonarQubeEnv('sonar_server') {
    				sh "${scannerHome}/bin/sonar-scanner"
				}
	    	}

	    }

		stage('Deploy') 
		{
			when
			 {
				    expression { 
				        GIT_BRANCH_NAME ==~ /.*master|.*feature|.*develop|.*hotfix/
				    }
			}			

			steps 
			{

				script
				{
					echo "${MY_BUILD_VERSION}"
					def proj_version = "${MY_BUILD_VERSION}-SNAPSHOT"
					echo "${proj_version}"
					sshPublisher(publishers: [sshPublisherDesc(configName: 'ansible_server', 
					transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: "ansible-playbook /etc/ansible/copywarfile.yml -e current_project_version=${proj_version}", 
					execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', 
					remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
				}
			}

		}

    }  

}
