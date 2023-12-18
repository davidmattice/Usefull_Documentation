# Terraform setup

Setting up terraform and using it in a home lab

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
${SUDO} apt update && ${SUDO} apt install -y terraform
```

## Git Tagging Helper

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

Add simple alias to make using workspaces in terraform easier
```
function tf {
    local tfvars_file=${2##*/}
    local wksp=${tfvars_file%%.*}
    local tf_state_dir="/mnt/terraform/state"

    #echo "${2}, ${tfvars_file}, ${wksp}"
    if [ -n "${TF_CMD}" ]; then
        echo "Set TF_CMD environment variable before using!"
        exit 1
    fi
    if [ -n "${TF_STATE_DIR}" ]; then
        echo "Set TF_STATE_DIR environment variable before using!"
        exit 1
    fi
    if [ -d ${tf_state_dir} ]; then
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
        echo "Terraform state directory is not available [${TF_STATE_DIR}]"
    fi
}
```
You can now run ...
```
tf plan <tfvars-file-name> [terraform options]
tf apply <tfvars-file-name> [terraform options]
```
