#!/bin/bash +x
def nexusRepoHostPort = nexusRepositoryHost
def nexusRepo = nexusRepository

def BuildImageName="${packerImageName}"
def UUID

//Make the following as Params 
def image_name = "${packerImageName}"

/*
def builder_type = "openstack"
def tenant_name = "admin"
def domain_name = "Default"
def username = "admin"
def password = "mypass"
def region = "RegionOne"
def use_floating_ip = true
def floating_ip_pool = "external"
def ssh_username = "ubuntu"
//def networks = "f60c08bd-3f40-41ae-b5f7-256ac9c7b8ac" //This is for 169
def networks = "cb3abed9-5fed-4797-a759-f3bbef7846be" //"ef7cd6df-58b1-4b13-8acf-b9028cfc2063" //This is for 170
//def source_image = "6d153be4-c261-4380-90e2-858f7da2c560"
def source_image_name = "ubuntu14-proxy"		   
def flavor = "m1.small"
def insecure = "true"
*/
//End of Params

//------------------------------------------------------------
import groovy.json.JsonBuilder
import groovy.json.JsonSlurper

// Method to update Output Image Name of concerned job
def UpdatePacker() {
	def workdir = pwd();
	def template= "templateModified.json"
	echo "The Current Working Directory is : ${workdir}"
	def UserPacker  = "${workdir}/${PackerFile}" /** Packerfile given by end-user */
	def AppPacker   = "${workdir}/"+ template /** Updated Packefile after editing Imagename */
  
	File temp = new File(UserPacker)
	def slurped = new JsonSlurper().parseText(temp.text)
	def builder = new JsonBuilder(slurped)
	//def ImageName = "${env.APP_NAME}-${env.MS_NAME}-${env.WORK_NAME}-${env.AUTHOR}" /** THIS HAS TO BE ENABLED AFTER REQUIREMENTS FULLFILLED */
	def ImageName = "${packerImageName}" /** Currently forming Image name in runtime  , Need to device above mentioned Strategy to form Image Name*/

	builder.content.builders[0].image_name = ImageName /** This is Considering that user provides packerfile with a single Builder , Need to update it to accomodate Multiple Builders */

	def inputFile = new File(AppPacker)
	if(inputFile.exists()) {
	 //Println("A file named  " + template + "  already exists in the same folder")
	}
	else {
	 inputFile.write(builder.toPrettyString())
	 //println("The Content Written to the file "+ template + " is")
	 //println(builder.toPrettyString())
	}
	return AppPacker /** Returning Packerfile Path as it is used to build machine image inside node and as it is defined in an external method */
}


try {
node {
  echo "Parameter List"
  echo "SCM Type    : ${scmSourceRepo}"
  echo "SCM Path    : ${scmPath}"
  echo "SCM User    : ${scmUsername}"
  echo "SCM Pass    : ${scmPassword}"
  echo "HTTP Proxy  : ${httpProxy}"
  echo "HTTPS Proxy : ${httpsProxy}"
  echo "Output Image Name : ${packerImageName}"	
  echo "Nexus Host & Port  :${nexusRepoHostPort}" 
  echo "Nexus Repo Name    :${nexusRepo}"     
  echo "\n\nOpenStack Parameters" 
  echo "Builder Type    :${builder_type}" 
  echo "Tenant Name    :${tenant_name}" 
  echo "Domain Name    :${domain_name}" 
  echo "Username    :${username}" 
  echo "Password    :${password}" 
  echo "Region    :${region}" 
  echo "Use Floating IP    :${use_floating_ip}" 
  echo "Floating IP Pool    :${floating_ip_pool}" 
  echo "Ssh Username    :${ssh_username}" 
  echo "Networks    :${networks}" 
  echo "Source Image Name - But this is expected from developer now    :${source_image_name}" 
  echo "Flavor    :${flavor}" 
  echo "Insecure    :${insecure}"
	
	
// ---- Source Shell
// REMOVE THIS BLOCK IF INPUTS ARE TAKEN FROM NODE
/* Commented by Ramesh
sh "export OS_PROJECT_DOMAIN_NAME=default"
sh "export OS_USER_DOMAIN_NAME=default"
sh "export OS_PROJECT_NAME=admin"
sh "export OS_USERNAME=admin"
sh "export OS_PASSWORD=password"
sh "export OS_AUTH_URL=http://controller:35357/v3"
sh "export OS_IDENTITY_API_VERSION=3"
sh "export OS_IMAGE_API_VERSION=2"
*/
/**
  sh "export OS_PROJECT_NAME=admin"
  sh "export OS_USERNAME=admin"
  sh "export OS_PASSWORD=abc123"
  sh "export OS_AUTH_URL=http://172.19.74.169:35357/v2"
  sh "export OS_IDENTITY_API_VERSION=2"
  sh "export OS_IMAGE_API_VERSION=2"
**/
//------------------------------------------------
/** ENABLE THIS IF INPUTS ARE TAKEN FROM NODE
  sh "export OS_PROJECT_NAME=${PROJECT_NAME}"
  sh "export OS_USERNAME=${USERNAME}"
  sh "export OS_PASSWORD=${PASSWORD}"
  sh "export OS_AUTH_URL=${AUTH_URL}"
  sh "export OS_IDENTITY_API_VERSION=${IDENTITY_API_VERSION}"
  sh "export OS_IMAGE_API_VERSION=${IMAGE_API_VERSION}"
**/

//------------------------------------------------

 
  //To escape all Special Charecters in a given input string password
  def pwdstr = scmPassword
  scmPassword = pwdstr.replaceAll( /([^a-zA-Z0-9])/, '\\\\$1' )
  
  def usrstr = scmUsername
  scmUsername = usrstr.replaceAll( /([^a-zA-Z0-9])/, '\\\\$1' )
  
  def pwdstr2 = scmPassword
  def usrstr2 = scmUsername
  scmPassword = pwdstr2.replaceAll( /([@])/, '%40' )
  scmUsername = usrstr2.replaceAll( /([@])/, '%40' ) 
  //----------------------------------------

  stage('Code Pickup') {
    echo "Source Code Repository Type : ${scmSourceRepo}"
    echo "Source Code Repository Path : ${scmPath}"
    
    if("${scmSourceRepo}".toUpperCase()=='SVN'){
        //Not a perfect solution. Reimplement with SVN Step or checkout Step
        echo 'SCM is SVN'
        sh "svn co --username ${scmUsername} --password ${scmPassword} ${scmPath} ."
        
    } else if("${scmSourceRepo}".toUpperCase()=='GIT' || "${scmSourceRepo}".toUpperCase()=='GITHUB'){
        //Not a perfect Solution. Reimplement with Git Step or checkout Step
        echo 'SCM is GIT Based'
        //Check if the Git clone URL already has username and password in it.
        //def strToMatch = "${scmPath}"
        def extractedCredentialsFromPath="";
        if(scmPath.indexOf("@")>-1 && scmPath.indexOf("//") >-1) {
            extractedCredentialsFromPath=scmPath.substring(scmPath.indexOf("//")+2,scmPath.indexOf("@"))
        }
      
        if(extractedCredentialsFromPath && extractedCredentialsFromPath.length()>0) {
          echo "Looks like the Username or Password or Both are found in the GIT Repo path itself"
          extractedCredentialsFromPath=extractedCredentialsFromPath.replaceAll( /([^a-zA-Z0-9])/, '\\\\$1' )
          extractedCredentialsFromPath=extractedCredentialsFromPath.replaceAll( /([@])/, '%40' )
          echo "Extracted Credentials and ecapped: ${extractedCredentialsFromPath}"
          echo "User & Password: ${scmUsername}\\:${scmPassword}"
          if ("${extractedCredentialsFromPath}" == "${scmUsername}" + "\\:" + "${scmPassword}") {
            echo "Username and password already present in url. No need to append"
          } else if("${extractedCredentialsFromPath}" == "${scmUsername}" && !scmPath.startsWith("ssh://")) {
            echo "Username already present in url. Just handle the password" 
            scmPath = scmPath.substring(0,scmPath.indexOf("@"))+":" + scmPassword +scmPath.substring(scmPath.indexOf("@"), scmPath.length());
          } else if("${extractedCredentialsFromPath}" == "${scmUsername}" && scmPath.startsWith("ssh://")) {
            echo "Username and password already present in url. Since this is SSH, no need to append password" 
            //Password should not be appended for SSH. It should be passed through SSHPASS only
          } else {
            echo "Something wrong. The credentials in the URL did not match with the values passed" 
          }
        } else {
          echo 'Credentials not found in path. Frame it'
          //SSH based git should not append password. It will not work. The password should be passed through sshpass
          if(scmPath.startsWith("ssh://")){
              scmPath = scmPath.substring(0, scmPath.indexOf("//")+2) + scmUsername + "@" +scmPath.substring(scmPath.indexOf("//")+2, scmPath.length());
          } else {         
              scmPath = scmPath.substring(0, scmPath.indexOf("//")+2) + scmUsername + ":" + scmPassword + "@" +scmPath.substring(scmPath.indexOf("//")+2, scmPath.length());
          }  
        }
      
        //echo "GIT PATH: ${scmPath}"
        try {
            //If we use git clone, it will not clone in the same path if we rebuild the pipeline
            sh 'ls -a | xargs rm -fr'
        } catch (error) {
        }
      
        if(scmPath.startsWith("ssh://")){
          if(httpsProxy != null && httpProxy!=null && httpsProxy.length()>0 && httpProxy.length()>0){
            echo "Looks like this Jenkins behind Proxy"
            sh "export https_proxy=${httpsProxy} && export http_proxy=${httpProxy} && sshpass -p ${scmPassword}   git clone --recursive ${scmPath} ."
          } else {
            echo "Looks like this Jenkins is not behind Proxy"
            sh "sshpass -p ${scmPassword}   git clone --recursive ${scmPath} ."
          }            
        } else {
            //Need this line for the recursive sub module clone to work without asking credentials
            sh "git config --global credential.helper 'cache --timeout=120'"
            if(httpsProxy != null && httpProxy!=null && httpsProxy.length()>0 && httpProxy.length()>0) {
              echo "Looks like this Jenkins behind Proxy"
              sh "export https_proxy=${httpsProxy} && export http_proxy=${httpProxy} && git clone --recursive ${scmPath} ."
            } else {
              echo "Looks like this Jenkins is not behind Proxy"
              sh "git clone --recursive ${scmPath} ."
            }  
            //Need this line for the recursive sub module clone to work without asking credentials -- Clear the cache. 
            sh "git credential-cache exit"
        } 
        
        //The below solutions not working with username and password
        //git "${scmPath}" //Alternate option
        //checkout scm: [$class:'GitSCM', userRemoteConfigs: [[url: scmPath ]]]
    } else {
        error 'Unknown Source code repository. Only GIT and SVN are supported'
    }
  } 
   
//--------------------------------------  
// INITIALIZING
stage ('Initialization') { 
	def appModuleSeperated = fileExists 'app'
	def testModuleSeperated = fileExists 'test'
	def appPath = ''
	def testPath = ''
	if (appModuleSeperated) {
	    echo 'App Module is found , assumed that application is present in /app directory'
	    appPath='app/'
	} else {
	    echo 'There is no defined Application path , hence it is assumed that application is in current directory'
	    appPath = ''
	}

	if (testModuleSeperated) {
	    echo 'Test Module is found , assumed that Test Cases are Present for the concerned Modules and has to be performed'
	    testPath = 'test/'
	} else {
	    echo 'No Test Modules found , hence it is assumed that no test environment and / or test cases to be performed'
	    testPath = ''
	}
	
	if (appPath + fileExists("${fileName}")) {
	    echo "Packer file found at ${appPath}"
	    PackerFile = appPath + "${fileName}"
	} else {
	    echo 'Packerfile not found under ' + appPath
	}
	
	// COPYING APP Directory to Current Working Directory
  	def appWorkingDir = (appPath=='') ? '.' : appPath.substring(0, appPath.length()-1)  

	// NEXUS file for Time Stamp comparison. This file is used for comparing time stamps and differentiating input files from generated output files.        
	//TODO - Tune it later. Dirty solution to identify the Jenkins generated artifacts for Nexus
	sh 'echo Nexus>Nexus.txt'
	//Dirty solution ends.
}

//--------------------------------------
//UPDATE PACKERFILE
stage ('Update Parameters in PackerFile') {
	echo "Updating Packer file with System Defined Parameters"
	//AppPacker = UpdatePacker() /** Return value represents packerfile's absolute path */
	AppPacker=PackerFile
}
//--------------------------------------

	
//BUILD & PACKING
stage('validate') {
	echo "Validating the template : ${AppPacker}"
	def packerValidateCommand = "packer validate -var builder_type=${builder_type} \
	-var identity_endpoint=${identity_endpoint} \
	-var tenant_name=${tenant_name} \
	-var username=${username} \
	-var password=${password} \
	-var region=${region} \
	-var use_floating_ip=${use_floating_ip} \
	-var floating_ip_pool=${floating_ip_pool} \
	-var ssh_username=${ssh_username} \
	-var image_name=${image_name} \
	-var networks=${networks} \
	-var flavor=${flavor} \
	-var insecure=${insecure}  ${AppPacker}"
	
	echo "command: " + packerValidateCommand
	sh packerValidateCommand
	//sh "packer -v || packer validate ${AppPacker}"
}

stage('build') {
	echo "Building using packerfile :${AppPacker}"
	def packerBuildCommand = "packer build -machine-readable -var builder_type=${builder_type} \
	-var identity_endpoint=${identity_endpoint} \
	-var tenant_name=${tenant_name} \
	-var username=${username} \
	-var password=${password} \
	-var region=${region} \
	-var use_floating_ip=${use_floating_ip} \
	-var floating_ip_pool=${floating_ip_pool} \
	-var ssh_username=${ssh_username} \
	-var image_name=${image_name} \
	-var networks=${networks} \
	-var flavor=${flavor} \
	-var insecure=${insecure} ${AppPacker} | tee build.log"
	
	echo "command: " + packerBuildCommand
	
	sh packerBuildCommand
	//sh "packer build -machine-readable ${AppPacker}  | tee build.log"
	UUID = sh(script: "grep 'artifact,0,id' build.log | cut -d, -f6 | cut -d: -f2", returnStdout: true) // Fetching and storing UUID in local variable 
	echo "The value returned by Packer Build For UUID generation is: ${UUID}"
}
	
/*	
//---------------------------------------
  if("${stage}".toUpperCase() == 'BUILD') {
    echo 'The Requested Stage is Build Only,hence successful VM Images will be pushed to Temp Repo'
    stage('validate') {
        echo "Validating the template : ${AppPacker}"
        sh "packer -v || packer validate ${AppPacker}"
      }
	  
    stage('build') {
        echo "Building using packerfile :${AppPacker}"
        sh "packer build -machine-readable ${AppPacker} | tee build.log"
        UUID = sh(script: "grep 'artifact,0,id' build.log | cut -d, -f6 | cut -d: -f2", returnStdout: true) // Fetching and storing UUID in local variable 
        echo "The value returned by Packer Build For UUID generation is: ${UUID}"
     }
    stage('test') {
	// TESTS IF PRESENT COMES UNDER THIS SECTION
     }
  } else if("${stage}".toUpperCase() == 'CERTIFY') {
    echo 'The Requested Stage is Certify,hence successful VM Images will be pushed to Temp Repo and provided with a sandbox for validation'
    stage('validate') {
        echo "Validating the template :${AppPacker}"
        sh "packer -v || packer validate ${AppPacker}"
    }
    stage('build') {
        echo "Building using packerfile :${AppPacker}"
        sh "packer build -machine-readable ${AppPacker} | tee build.log"
        UUID = sh(script: "grep 'artifact,0,id' build.log | cut -d, -f6 | cut -d: -f2", returnStdout: true) // Fetching and storing UUID in local variable 
        echo "The value returned by Packer Build For UUID generation is: ${UUID}"
    }
    stage('test') {
	// TESTS IF PRESENT COMES UNDER THIS SECTION
     }
    echo "VM Image Built and pushed into temp repository and provided with a sandbox"
  }  else if ("${stage}".toUpperCase() == 'DEPLOY') {
    echo 'The Requested Stage is deploy,hence successful VM Images will be pushed to Permanant Repository without validation (sandbox)'
    stage('validate') {
        echo "Validating the template :${AppPacker}"
        sh "packer -v || packer validate ${AppPacker}"
    }
    stage('build') {
        echo "Building using packerfile :${AppPacker}"
        sh "packer build -machine-readable ${AppPacker} | tee build.log"
        UUID = sh(script: "grep 'artifact,0,id' build.log | cut -d, -f6 | cut -d: -f2", returnStdout: true) // Fetching and storing UUID in local variable 
        echo "The value returned by Packer Build For UUID generation is: ${UUID}"
    }
    stage('test') {
	// TESTS IF PRESENT COMES UNDER THIS SECTION
    }
    echo "VM Image Built and pushed into openstack-glance repository"
  }
 */

	

//END OF IMAGE PUSHING INTO REPOSITORY
// NEXUS UPDATE
stage('Publish Jenkins Output to Nexus')  {
	echo 'Publishing the artifacts...';
	try{
		sh 'find . -type f -newer Nexus.txt -print0 | tar -zcvf artifacts.tar.gz --ignore-failed-read --null -T -' 
	} catch(Exception e) {
	}
	nexusArtifactUploader artifacts: [[artifactId: "${env.JOB_NAME}", classifier: '', file: 'artifacts.tar.gz', type: 'gzip']], credentialsId: 'Nexus', groupId: 'org.jenkins-ci.main.mec', nexusUrl: nexusRepoHostPort, nexusVersion: 'nexus3', protocol: 'http', repository: nexusRepo,version: "${env.BUILD_NUMBER}"
	sh 'rm Nexus.txt'    
	//Dirty solution ends 
	echo "UUID# ${UUID} #UUID"
}
}
}
finally {
	node {
		def EXIT_STATUS = sh(script:"sed -n -e '/openstack: Script exited with non-zero exit status/ s/.*: *//p' build.log",returnStdout: true)
		if(!UUID) {
     			if (EXIT_STATUS) {
         			echo "Packer Build command exits with Openstack: Non-Zero Exit Code: "+ EXIT_STATUS
     					 }
    			error "Packer Build Exits without spawning VM"   
		}
		else{
			/** IMAGE CREATION WAS SUCCESSFUL */
		}
	}
}
