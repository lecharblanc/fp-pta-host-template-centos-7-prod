# template-centos-7-prod

A terraform process that produces the 'golden' image you should base all your production systems on.

For information about PTA and how to use it please visit https://github.com/Forcepoint/fp-pta-overview/blob/master/README.md

## Setup

This can be run from Windows or Linux.

### First Run

The first time you run this should be manually on your own machine. 

1. Modify the main.tf file...
    1. Needs the appropriate values for your vsphere instance.
    1. Remove the backend section as you don't have Artifactory up and running yet. Once you do, add it back in.

1. Look at the Jenkinsfile. Set every environment variable it sets, and execute the commands listed.

    1. If you're running on windows though, you won't be able to run the ansible playbook to shutdown
the VM as ansible can't be installed on Windows. Instead, just log into vSphere and shutdown the VM
by hand.

### Jenkins

Once you've got your pta-controller Jenkins system setup, you'll want to run this job through
Jenkins itself. 

* Be sure you create all of the credential objects referred to in the Jenkinsfile.

* At the bottom of the Jenkinsfile, ensure you change the email address to your PTA 
administrator's address so they get failure notifications.

* Ensure inventory.py has been committed to your git repo with execute permissions.

## Notes

The part about Terraform that makes it more than just a nice way to clone VMs is
that it knows if the VM it cloned from changed, it will recreate the destination VM.
Since we've got the test template set to be recreated by Packer every week, we don't
want Terraform to recreate a Production application every week. To deal with this,
you have a job that "promotes" the test template to be your prod template. You would
only replace the prod template rarely. That's what this job is. It is run on demand
only when you intend to replace the VM for each Production application. For instance,
if you say changed your test template from CentOS 7 to CentOS 8, this would allow you
to handle that and TEST it extensively before deciding to run this job and 
make it the base for your production applications.
