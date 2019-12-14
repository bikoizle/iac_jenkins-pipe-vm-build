def BUILD_STATUS = 0;
def DATE = Calendar.getInstance().getTime().format('YYYYMMdd-hhmmss',TimeZone.getTimeZone('CET'));

def GIT_CREDS_ID = "70c6a9da-bbb3-45b8-8565-d34f227696d9";

def GIT_VMBUILD_PBK_TAG = "0.2.2";
def GIT_GETVMINFO_PBK_TAG = "0.1.3";
def GIT_ANSIBLE_ROLE_TAG = "0.2.1";

def ANSIBLE_ROLE_NAME = "iac_ansible-role-server";

def GIT_URL_VMBUILD = "https://github.com/bikoizle/iac_ansible-playbook-vmbuild.git";
def GIT_URL_VMDELETE = "https://github.com/bikoizle/iac_ansible-playbook-vmdelete.git";
def GIT_URL_GETVMINFO = "https://github.com/bikoizle/iac_ansible-playbook-getvminfo.git";
def GIT_URL_ANSIBLE_ROLE;

def VMBUILD_PBK_DIR = "vmbuild";
def GETVMINFO_PBK_DIR = "getvminfo";

def VMBUILD_PBK = "vmbuild.yml";
def GETVMINFO_PBK = "getvminfo.yml";

def OS_URL = "http://bender.lan:5000/v3";
def OS_PROJECT = "admin";
def OS_PROJECT_DOMAIN = "Default";
def OS_USER_DOMAIN = "Default";
def OS_IMAGE_NAME = "CustomOS-20191214-030039";
def OS_VM_NAME = "customos" + "-" + "$DATE";
def OS_VM_BUILD_TIMEOUT = "200";
def OS_VM_FLAVOUR = "lab.small";
def OS_VM_VOL_SIZE = "20";
def OS_VM_NET = "private_network";

def OS_VM_INFO;
def OS_VM_IP_ADDRESS;

def ANSIBLE_WORKSPACE_DIR = "ansible_workspace";
def ANSIBLE_ROLE_DIR;
def ANSIBLE_USER = "root";

node {

    properties(
      [
        parameters(
          [
           choice(choices: ['iac_ansible-role-server', 'iac_ansible-role-apache', 'iac_ansible-role-mariadb'], description: 'Role to apply when creating the virtual machine.', name: 'ANSIBLE_ROLE_NAME'), 
           string(defaultValue: "$GIT_ANSIBLE_ROLE_TAG", description: '''MANDATORY: Git tag with the version of the role to download.
Each role tag can be obtained from its repository in https://github.com/bikoizle/''', name: 'GIT_ANSIBLE_ROLE_TAG', trim: false),
           string(defaultValue: "$OS_IMAGE_NAME", description: 'Image name to use when building the virtual machine.', name: 'OS_IMAGE_NAME', trim: false),
           string(defaultValue: "$OS_VM_NAME", description: 'Virtual machine name.', name: 'OS_VM_NAME', trim: false),
           string(defaultValue: "$OS_VM_VOL_SIZE", description: 'Virtual machine volume size in GB.', name: 'OS_VM_VOL_SIZE', trim: false),
           choice(choices: ['lab.small', 'lab.medium', 'lab.large'], description: '''Virtual machine configuration flavour:
 - lab.small: 1 vCPU + 2 GB RAM
 - lab.medium: 2 vCPU + 4 GB RAM
 - lab.large: 4 vCPU + 8 GB RAM''', name: 'OS_VM_FLAVOUR')
          ]
        )
      ]
    )

     try{

        stage("Fetch files"){
    
         ANSIBLE_ROLE_NAME = "${params.ANSIBLE_ROLE_NAME}"
         GIT_ANSIBLE_ROLE_TAG = "${params.GIT_ANSIBLE_ROLE_TAG}"
         OS_IMAGE_NAME = "${params.OS_IMAGE_NAME}"
         GIT_URL_ANSIBLE_ROLE = "https://github.com/bikoizle/${ANSIBLE_ROLE_NAME}.git"
         ANSIBLE_ROLE_DIR = "$ANSIBLE_WORKSPACE_DIR/roles/$ANSIBLE_ROLE_NAME"
         OS_VM_NAME = "${params.OS_VM_NAME}"
         OS_VM_VOL_SIZE = "${params.OS_VM_VOL_SIZE}"
         OS_VM_FLAVOUR = "${params.OS_VM_FLAVOUR}"

         echo "Fetching vmbuild playbook"
    
         checkout([$class: 'GitSCM',
                branches: [[name: "refs/tags/$GIT_VMBUILD_PBK_TAG"]],
                userRemoteConfigs: [[
                    credentialsId: "$GIT_CREDS_ID",
                    url: "$GIT_URL_VMBUILD"]],
                extensions: [[$class: "RelativeTargetDirectory", relativeTargetDir: "$VMBUILD_PBK_DIR"]],
            ])
    
         echo "Fetching getvminfo playbook"
    
         checkout([$class: 'GitSCM',
                branches: [[name: "refs/tags/$GIT_GETVMINFO_PBK_TAG"]],
                userRemoteConfigs: [[
                    credentialsId: "$GIT_CREDS_ID",
                    url: "$GIT_URL_GETVMINFO"]],
                extensions: [[$class: "RelativeTargetDirectory", relativeTargetDir: "$GETVMINFO_PBK_DIR"]],
            ])
 
         echo "Fetching Ansible role"

         checkout([$class: 'GitSCM',
                branches: [[name: "refs/tags/$GIT_ANSIBLE_ROLE_TAG"]],
                userRemoteConfigs: [[
                    credentialsId: "$GIT_CREDS_ID",
                    url: "$GIT_URL_ANSIBLE_ROLE"]],
                extensions: [[$class: "RelativeTargetDirectory", relativeTargetDir: "$ANSIBLE_ROLE_DIR"]],
            ])


        }
    
        stage("Create VM"){
    
         echo "Creating VM"
    
         withCredentials([string(credentialsId: "vault_path", variable: "vault_file")]) {
    
           ansiblePlaybook(
               playbook: "$VMBUILD_PBK_DIR/$VMBUILD_PBK",
               vaultCredentialsId: "vault_creds",
               extraVars: [
                   vault: "$vault_file",
                   url: "$OS_URL",
                   project: "$OS_PROJECT",
                   project_domain: "$OS_PROJECT_DOMAIN",
                   user_domain: "$OS_USER_DOMAIN",
                   image_name: "$OS_IMAGE_NAME",
                   vm_name: "$OS_VM_NAME",
                   timeout: "$OS_VM_BUILD_TIMEOUT",
                   vm_flavour: "$OS_VM_FLAVOUR",
                   vm_vol_size: "$OS_VM_VOL_SIZE",
                   vm_net: "$OS_VM_NET"
               ])
          }
    
        }
    
        stage("Get VM info"){
    
         echo "Getting VM info"
    
         withCredentials([string(credentialsId: "vault_path", variable: "vault_file")]) {
    
           ansiblePlaybook(
               playbook: "$GETVMINFO_PBK_DIR/$GETVMINFO_PBK",
               vaultCredentialsId: "vault_creds",
               extraVars: [
                   vault: "$vault_file",
                   url: "$OS_URL",
                   project: "$OS_PROJECT",
                   project_domain: "$OS_PROJECT_DOMAIN",
                   user_domain: "$OS_USER_DOMAIN",
                   vm_name: "$OS_VM_NAME",
                   timeout: "$OS_VM_BUILD_TIMEOUT",
               ])
          }

         echo "Loading VM info JSON file"

         OS_VM_INFO = readJSON file: "$GETVMINFO_PBK_DIR/output/vminfo.json"

        }

        stage("Prepare Ansible workspace"){

          echo "Preparing Ansible workspace"

          OS_VM_IP_ADDRESS = OS_VM_INFO['accessIPv4'][0]

          if (fileExists("$ANSIBLE_WORKSPACE_DIR/roles/$ANSIBLE_ROLE_NAME/.requirements.yml")) {

            echo "Downloading role dependencies with Ansible Galaxy"

            sh "ansible-galaxy install -r $ANSIBLE_WORKSPACE_DIR/roles/$ANSIBLE_ROLE_NAME/.requirements.yml -p $ANSIBLE_WORKSPACE_DIR/roles -f"

          }

          sh "cp $ANSIBLE_WORKSPACE_DIR/roles/$ANSIBLE_ROLE_NAME/tests/* $ANSIBLE_WORKSPACE_DIR"
          sh "sed -i 's|localhost|$OS_VM_IP_ADDRESS|g' $ANSIBLE_WORKSPACE_DIR/inventory $ANSIBLE_WORKSPACE_DIR/test.yml"

          echo "Removing old VM IP address ssh fingerprint"

          sh "ssh-keygen -R $OS_VM_IP_ADDRESS"

        }

        stage("Wait for VM"){

          echo "Waiting for VM to be ready"

          timeout(time: 1, unit: 'HOURS'){
             waitUntil{
                status = sh(returnStatus: true, script: "ansible -i '$OS_VM_IP_ADDRESS,' all -m ping")
                return status == 0
             }
         }

        }

        stage("Configure VM"){

          echo "Applying Ansible Role"

          sh "ansible-playbook -u $ANSIBLE_USER -i $ANSIBLE_WORKSPACE_DIR/inventory $ANSIBLE_WORKSPACE_DIR/test.yml"

        }
    

    }
    catch(error){
      BUILD_STATUS = 1 
      println error.getMessage()
    }

    stage("Clean up"){

     if (BUILD_STATUS == 1){

         echo "Build finished with errors!"

       }

    }

}
