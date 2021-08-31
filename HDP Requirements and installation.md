<em>Last updated August 26, 2021</em>
##### Table of Contents  
[Installation Requirements](#installation-requirements)  
[Installation Instructions](#installation-instructions) 
<a name="headers"/>

## Installation Requirements

The Historical Data Platform (HDP) allows you to visualize and investigate your infrastructure change over time. These are the minimum installation requirements. 

#### Instance
The first versions of the HDP should be installed on non-production environments. 

#### Puppet Enterprise
The HDP can be installed on PE version 2019.8.7 (LTS) or above.

#### Operating system

The HDP can be installed on any Linux operating system. 

|Operating system     |Versions |Architecture   |
|----------------------|---------|-------------------- |
|Enterprise Linux, CentOS, Oracle Linux, Red Hat Enterprise Linux, Scientific Linux|7, 8|x86_64, FIPS 140-2 compliant RHEL version 7|
|SUSE Linux Enterprise Server|12|X86_64|
|Ubuntu (General Availability kernels)|18.04|amd64|


#### Nodes

The HDP can support up to 1,000 nodes. These nodes report to a single HDP node, which is the recommended setup. If your architecture contains more than 1,000 nodes contact Puppet professional services to talk about optimizing your setup for your specific requirements. 

#### Storage

These are the server storage requirements for 1 HDP node.


|Nodes      |HDP Nodes|1 Month Storage|1 Year Storage|
|-----------|---------|-------------- |------------- |
|1-100      |1        |  20 GB        | 260 GB       |
|101 - 500  |1        | 100 GB        | 1.3 TB       |
|500 - 1,000|1        | 200 GB        | 2.6 TB       |

#### Memory requirements

A minimum of 8 GB of memory is required.

#### CPU requirements

A minimum of 2 CPUs is required.

#### Follow-up with Puppet

We highly value your feedback about using the historical platform and will schedule a follow-up meeting once every two weeks or more often if necessary. We also request that you notify us regarding any unexpected behavior in the Historical Data Platform so that we can swiftly investigate issues. 


## Installation Instructions

#### Preparation

1. To get started, designate one node in your environment on the host on which you will run the HDP. We will refer to this node as the HDP server. 
    - The HDP should not be located on the same host as the Primary Server.
    - If you have Docker installed on the HDP server uninstall it. The correct Docker version will be applied via the module.
2. Navigate to the HDP module in the [GitHub repository](https://github.com/puppetlabs/puppetlabs-hdp).

#### Configure the Module for the HDP

Now we will configure the HDP module for your host environment.

3. Navigate to the environment where you store your Puppet control repository, this may be stored in github, gitlab, etc.
4. Add the HDP module to your Puppet repository.
5. Create an HDP profile in your environment by modifying the ca_server. In standard Puppet installations this will be your Primary server.
    - The ca_server should not be ‘localhost’ for deployments using docker.
6. Designate the dns_name which your Primary server or compilers will use to contact the HDP. This is also the URL you will use to access the user interface. Note the following:
    - For security best practices the dns_name of the HDP should be different from the name of the agent certificate for PE to prevent collisions.
    - The HDP will request a certificate from the Puppet CA with this dns_name. 

```    
    class { 'hdp::app_stack':
        ca_server        => '<https://your_ca_server:8140>',
        dns_name         => '<hdp.your_dns.io>',
        image_repository => 'gcr.io/hdp-gcp-316600',
        hdp_version      => 'latest',
        ui_use_tls => true,
        ui_cert_files_puppet_managed => false,
    }
```


#### Configure the Module for the Data Processor
Now we will configure the data processor for your host environment to ensure that your node data is also sent to the HDP instance.

7. For the Primary server and any compilers include this class in the node’s role. The fqdn name should be the same dns_name that was used in step 6. Note that by default:
    - The Primary server and any compilers will contact the HDP host on port 9091. 
    - The user interface will be served on ports 80, 443, and 9092.
```
    class { 'hdp::data_processor':
       hdp_url =>  'https://<hdp.your_dns.io>:9091',
    }   
```
8. Initiate a Puppet run to apply the module to the HDP Server and any compilers.

#### PE Authorization

Now we will sign the certificate for the HDP in the PE Console. If you have auto sign-on enabled you will not need to perform steps 9-12.

9. Access the PE Platform
10. In the PE platform you will notice that there are new certificates to be authorized in the Certificates tab.
11. Click “accept” to approve the certificate for the HDP.
12. The HDP is now authorized by the Primary Server and you may review your node data in the Historical Data Platform.


#### Authentication

To limit access to the dns_name you may add authentication to the HDP.

13. Designate the hdp_query_username and hdp_query_password which you will use to access the HDP. For security best practices we support pre-hashed passwords using the posix crypt (c) format. 
14. When accessing the HDP these credentials must be provided.

```
    class { 'hdp::app_stack':
        ca_server        => '<https://your_ca_server:8140>',
        dns_name         => '<hdp.your_dns.io>',
        image_repository => 'gcr.io/hdp-gcp-316600',
        hdp_version      => 'latest',
        hdp_query_username => '<user>'
        hdp_query_password => '<password>'
    }
```

#### Configuration

After authorizing PE data to be sent to the HDP you can now configure the HDP.
 
15. Navigate to the HDP user interface which is located at the dns_name which was used in step 6, 
```
<https://hdp.your_dns.io>
```
16. A welcome modal will appear listing the integrated nodes from your network.
17. Follow the steps in the modal to select the facts to be displayed in the historical reports and which facts should send a notification when changed. 
18. You are now ready to use the visual platform.





