# template-centos-7-prod

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

## Jenkins

* At the bottom of the Jenkinsfile, ensure you change the email address to your PTA 
administrator's address so they get failure notifications.

* Ensure inventory.py has been committed to your git repo with execute permissions.