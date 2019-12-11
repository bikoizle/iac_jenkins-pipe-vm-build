def BUILD_STATUS = 0;
def DATE = Calendar.getInstance().getTime().format('YYYYMMdd-hhmmss',TimeZone.getTimeZone('CET'));

def GIT_CREDS_ID = "70c6a9da-bbb3-45b8-8565-d34f227696d9";

def GIT_VMBUILD_PBK_TAG = "0.1.1";
def GIT_GETVMINFO_PBK_TAG = "0.1.3";
def GIT_ANSIBLE_ROLE_TAG = "0.2.0";

def ANSIBLE_ROLE_NAME = "iac_ansible-role-server";

def GIT_URL_VMBUILD = "https://github.com/bikoizle/iac_ansible-playbook-vmbuild.git";
def GIT_URL_VMDELETE = "https://github.com/bikoizle/iac_ansible-playbook-vmdelete.git";
def GIT_URL_GETVMINFO = "https://github.com/bikoizle/iac_ansible-playbook-getvminfo.git";
def GIT_URL_ANSIBLE_ROLE = "https://github.com/bikoizle/${ANSIBLE_ROLE_NAME}.git";

def VMBUILD_PBK_DIR = "vmbuild";
def GETVMINFO_PBK_DIR = "getvminfo";
def ANSIBLE_ROLE_DIR = "roles/$ANSIBLE_ROLE_NAME";

def VMBUILD_PBK = "vmbuild.yml";
def GETVMINFO_PBK = "getvminfo.yml";

def OS_URL = "http://bender.lan:5000/v3";
def OS_PROJECT = "admin";
def OS_PROJECT_DOMAIN = "Default";
def OS_USER_DOMAIN = "Default";
def OS_IMAGE_NAME = "CustomOS-20191207-091815";
def OS_VM_NAME = "customos_test" + "-" + "$DATE";
def OS_VM_BUILD_TIMEOUT = "200";
def OS_VM_FLAVOUR = "lab.small";
def OS_VM_NET = "private_network";

def OS_VM_INFO;
def OS_VM_IP_ADDRESS;

def ANSIBLE_WORKSPACE_DIR = "ansible_workspace";
def ANSIBLE_USER = "root";

node {

     try{

        stage("Fetch files"){
    
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

          if (fileExists("$ANSIBLE_WORKSPACE_DIR/roles/${env.JOB_BASE_NAME}/.requirements.yml")) {

            echo "Downloading role dependencies with Ansible Galaxy"

            sh "ansible-galaxy install -r $ANSIBLE_WORKSPACE_DIR/roles/${env.JOB_BASE_NAME}/.requirements.yml -p $ANSIBLE_WORKSPACE_DIR/roles -f"

          }

          sh "cp $ANSIBLE_WORKSPACE_DIR/roles/${env.JOB_BASE_NAME}/tests/* $ANSIBLE_WORKSPACE_DIR"
          sh "sed -i 's|localhost|$OS_VM_IP_ADDRESS|g' $ANSIBLE_WORKSPACE_DIR/inventory $ANSIBLE_WORKSPACE_DIR/test.yml"

          echo "Removing old VM IP address ssh fingerprint"

          sh "ssh-keygen -R $OS_VM_IP_ADDRESS"

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
