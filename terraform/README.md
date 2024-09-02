# Terraform setup

Setting up terraform on a Management device and using it in a home lab

## Terraform Installation

Install Terraform ...
```
if [ $UID -ne 0 ]; then
    export SUDO="sudo"
else
    export SUDO=""
fi
wget -O - https://apt.releases.hashicorp.com/gpg | ${SUDO} gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep VERSION_CODENAME /etc/os-release | cut -f 2 -d =) main" | ${SUDO} tee /etc/apt/sources.list.d/hashicorp.list
${SUDO} apt update && ${SUDO} apt install -y terraform jq
tf=$(whereis terraform | cut -d ' ' -f 2)
echo "export TF_CMD=${tf}" >>~/.bashrc
```

## Use a NFS share to store Terraform state on a NAS

Ensure the NAS volume has permissions for the Management node that will be running Terraform.

Add a mount to **/etc/fstab** so the path is automounted
```
if [ $UID -ne 0 ]; then
    export SUDO="sudo"
else
    export SUDO=""
fi
echo "<IP>:<volume> /mnt/terraform nfs defaults 0 0" | ${SUDO} tee -a /etc/fstab
${SUDO} mount -a
echo "export TF_STATE_DIR=/mnt/terraform/state" >>~/.bashrc
```
## Git Tagging Helper

Add this function to **~/.bashrc** to make it easy to add tags to terraform repos as they are committed.
```
function upd_tags {
    tag_full=$(cut -d= -f 2 version.txt)
    echo "Updating tags to: ${tag_full}"
    echo "Press Enter to continue ..."
    read junk

    tag_min=${tag_full%.*}
    tag_maj=${tag_min%.*}.x
    tag_min=${tag_min}.x

    git tag -f ${tag_full}
    git tag -f ${tag_min}
    git tag -f ${tag_maj}
    git push origin -f ${tag_full} ${tag_min} ${tag_maj}
}
```

## Terraform Planning Helper

Add this simple function to **~/.bashrc** to make using workspaces in terraform easier
```
function tf {
    local tfvars_file=${2##*/}
    local wksp=${tfvars_file%%.*}
    local tf_state_dir=$(grep workspace_dir *.tf | head -1 | cut -d = -f 2 | tr -d ' "')

    # Check if tfvars file is accessable
    if [ ! -r ${2} ]; then
        echo "Could not read [${2}]!"
        exit 1
    fi

    # Ensure we have an executable for Terraform
    if [ -z "${TF_CMD}" ]; then
        # Environment variable is not set try to find a default
        TF_CMD=$(whereis terraform | cut -d : -f 2 | tr -d " ")
        if [ -z "${TF_CMD}" ]; then
            echo "Could not find terraform command.  Set TF_CMD environment variable or ensure terraform is in the PATH before using!"
            exit 1
        fi
    fi

    if [ -z "${tf_state_dir}" ]; then
        echo "WARNING: No local backend configured, useing local state!"
        sleep 5
    fi

    #echo "${TF_CMD}, ${tfvars_file}, ${wksp}, ${tf_state_dir}"

    if [[ -z "${tf_state_dir}" ]] || ( [[ -n "${tf_state_dir}" ]] && [[ -d "${tf_state_dir}" ]] ); then
        case "$1" in
            plan)
                rm -rf .terraform 2>/dev/null
                rm -f .terraform.lock.hcl 2>/dev/null
                ${TF_CMD} init -input=false
	            if [ $? -eq 0 ]; then
	                ${TF_CMD} workspace select ${wksp}
    	            if [ $? -ne 0 ]; then
	                    ${TF_CMD} workspace new ${wksp}
        	        fi
            	    if [ $? -eq 0 ]; then
	                    ${TF_CMD} plan -input=false -out=${wksp}.plan -var-file=${2} ${*:3}
                    fi
                fi
	            ;;
            apply)
                ${TF_CMD} apply -input=false ${wksp}.plan ${*:3}
	            ;;
            *)
                ${TF_CMD} $*
	            ;;
        esac
    else
        echo "Local backend set to [${tf_state_dir}], but directory is not accessable"
    fi
}
```
You can now run ...
```
tf plan <tfvars-file-name> [terraform options]
tf apply <tfvars-file-name> [terraform options]
```
