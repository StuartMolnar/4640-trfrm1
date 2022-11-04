<b>Assumptions:</b>
- terraform is installed 
- SSH key named "my_key" attached to DigitalOcean account (or edit line 21 in main.tf)
- project named 4640_labs exists

<h4>First</h4>

Initialize a git repository in "~/" with the following files:
- .gitignore
- main.tf
- .env

<h4>Next</h4>

Create a DigitalOcean API Token and keep track of its authentication string.

<h4>Next</h4>

Open your .env file at ```~/<gitrepo>``` and put:
```
export TF_VAR_do_token=<your token authentication string>
```

<h4>Next</h4>

In the command line at ```~/<gitrepo>``` enter: ```source .env```

<h4>Next</h4>

In command line at ```~/<gitrepo>``` enter: ```terraform init```

<h4>Next</h4>

Open your main.tf file at ```~/<gitrepo>``` and put:
```
terraform {
  required_providers {
    digitalocean = {
      source  = "digitalocean/digitalocean"
      version = "~> 2.0"
    }
  }
}

# Set the variable value in *.tfvars file
# or using -var="do_token=..." CLI option
variable "do_token" {}

# Configure the DigitalOcean Provider
provider "digitalocean" {
  token = var.do_token
}

# Set the SSH key used
data "digitalocean_ssh_key" "my_key" {
  name = "my_key"
}

# Set the project used
data "digitalocean_project" "lab_project" {
  name = "4640_labs"
}

# Create a new tag
resource "digitalocean_tag" "do_tag" {
  name = "Web"
}

# Create a new vpc
resource "digitalocean_vpc" "web_vpc" {
  name   = "web"
  region = "sfo3"
}

# Create a new droplet
resource "digitalocean_droplet" "web" {
  image    = "rockylinux-9-x64"
  name     = "web-1"
  tags     = [digitalocean_tag.do_tag.id]
  region   = "sfo3"
  size     = "s-1vcpu-512mb-10gb"
  vpc_uuid = digitalocean_vpc.web_vpc.id
  ssh_keys = [data.digitalocean_ssh_key.my_key.id]
}

# Add new web-1 droplet to existing 4640_labs project
resource "digitalocean_project_resources" "project_attach" {
    project = data.digitalocean_project.lab_project.id
    resources = [
        digitalocean_droplet.web.urn
    ]    
}

output "server_ip" {
    value = digitalocean_droplet.web.ipv4_address
}
```
<h4>Next</h4>

In ```~/<gitrepo>``` enter: ```terraform apply```



