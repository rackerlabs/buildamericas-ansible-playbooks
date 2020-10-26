# Radius server playbook
This playbook was created to deploy the several components needed for a FreeRadius server with default configuration. The purpose for this was a POC requested by a customer to support MFA on AWS Directory Services (Windows implementation).
The playbook requires 2 parameters as inputs:

 - **vpc_cidr**: VPC CIDR for the implementation. This will allow the request sent from the Directory Service ENI's to be handled by the Radius server
 - **shared_secret**: The shared secret set to authenticate through PAP protocol. The string must be set on both Directory service and Radius server

To run this playbook, the easiest path is to copy this playbook to a S3 bucket (making sure the role/instance profile associated to the EC2 instance has read permission on the bucket) and run the automation SSM Run Command on document AWS-ApplyAnsiblePlaybooks, specifying the proper path to the playbook:

```HCL
{
      action = "aws:runDocument",
      inputs = {
        documentPath = "AWS-ApplyAnsiblePlaybooks",
        documentParameters = {
          SourceType = "S3"
          SourceInfo = jsonencode(
            {
              "path" : "https://s3.amazonaws.com/rackspace-buildamericas-test/radius.yaml"
            }
          )
          InstallDependencies = "True"
          PlaybookFile        = "radius.yaml"
          ExtraVariables      = "SSM=True vpc_cidr=10.103.0.0/16 shared_secret=${random_string.shared_secret.result}"
          Check               = "False"
          Verbose             = "-vv"
        },
        documentType = "SSMDocument"
      },
      name           = "AnsibleRadiusPlaybook",
      timeoutSeconds = 300
    }
```