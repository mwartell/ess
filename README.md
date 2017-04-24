# ESS Test Scenario

Submitted by matt.wartell@gmail.com in response to a test provided by ESS.


## Resource Topology

The test specification calls for 99.999% uptime and dispersed geographic
zones.  In order to acheive High Availabiliy, multi-region application
servers need to be used as does database mirroring and failover.
The Elastic Load Balancer fronting the web servers allows for automatic
expansion of capacity by spawning addition web servers as request latency
increases.

There is an orchestration host which is responsible for hosting and
executing all provisioning for the target services and software. This
gives the collection of resources a "home base" which will require
special provisioning.

There are three regional webserver resource collections. These were
chosen somewhat abitrarily for global dispersion: us-east-1 (Virginia),
eu-west-1 (Ireland), and ap-northeast-1 (Tokyo). The MediaWiki database is
localized to us-east-1 and is MultiAZ Failover configured. For disaster
recovery, Read Replicas could be created in the other regions but
I have chosen not to add that complexity to the implementation.

### Levels of Description

When there are three logical layers as there is in this design it
is important to keep them separated as much as possible. Naming the
layers helps keep them distinct:

  *  Infrastructure - the virtual machine rooms that will host
     the various services. Everything that translates into an AWS-API
     call is considered part of infrastructure; examples are VPC, ELB,
     Security Groups, routing tables, AMI selection, and so on.
  *  Application - the web applications running on the provisioned
     infrastructure. The primary components here are Apache and PHP.
     Given the use of RDS, the database is notably infrastructure.
     The application layer is also where the Linux permissions
     structure operates.
  *  Development - the software run by the application layer. In the
     case of this scenario, that role belongs to MediaWiki.

### Orchestration Ordering

  1. Create the "home" vpc
  2. create the orchestration vm
  3. clone this repository onto orchestration
  4. execute the aws Infrastructure provisioning
  5. provision multiple Application servers
  6. install Apache, PHP, and a test page on Application servers
  7. install MediaWiki on Application servers

## Permissions and Credentials

The scenario specification calls for the definition of three role groups
but leaves the definition explicitly unspecified. The general
semantics of the groups shall be:

  1. administrator - granted Administrator role across the account. This
     is effectively root access to all resources.
  2. developer - granted full permissions to the Application servers to
     allow modification of Application code. This role does not have
     database management access nor can it instantiate new Application
     servers
  3. tester - granted no access but through the public facing web server
     interfaces thus forcing black-box testing

Of course, the permissions system allows nearly infinite granularity
to access system resources. In my experience, placing restrictions on
developers will often prompt them to seek administrator credentials,
so it is better to provide considerable lattitude.

Having dealt with arcane access mechanisms, I find simple clarity to
be easier to manage and easier for users to understand. In this
implementation, for instance, I have declined to establish firewalls
or bastion hosts which provide defense in depth at the cost of excess
complexity for this demonstration.

## Design Choices

There are three primary orchestration tool choices: Ansible,
CloudFormation, and AWS-CLI. These can be combined in various ways such
as Ansible orchestrating stack creation specified by CloudFormation as
accessed through the CLI. There is rarely an optimal choice of mechanism,
and obvious trade-offs between them; the Ansible use of declarative
mechanisms has distinct advantages for orchestration, but it does pay
for it in terms of concision and lack of readability.

Other design choices include:

* LAMP is specified but RDS (MySQL flavor) is used in preference to MySQL.
  The primary reason for this is to more conveniently handle
  database failures at the cost of greater provider
  dependency.
* HTTPS is not specified and not implemented to reduce
  installation complexity.
* Despite significant static content in MediaWiki, CloudFront
  is not used.
* IPv6 is studiously ignored.
* Cross-region routing over the public internet should be shrouded
  with a VPN but is not for implementation simplicity.
* aws-cli, git, ansible and boto are assumed present on the orchestration host
* the sample CloudFormation template was too seductive to
  dupllicate with Ansible; the MediaWiki installation demonstrates
  Ansible
* MediaWiki uses the server local filesystem for storage of
  objects like media and image files. No attempt will be
  made to replicate this single point of failure.

## Private Cloud Migration

Although the test scenario states that "the architecture should be able
to adapt to deployment" on a private cloud, the implications of that
requirement are underspecified. In particular, without knowledge of the
target system, trying to anticipate what might be needed is folly.

For the purposes of this exercise, I have done nothing to explicitly aid
porting while attempting to minimize operational dependencies. Although
cloud platforms - including private cloud such as VMware - provide
highly overlapping services, they are not congruent. Tools like Ansible
which attempt to abstract configuration actions from the underlying
system don't even attempt to homogenize cloud interfaces, providing
idiosyncratic platform-specific modules for each hosting service. If
entire teams working on cloud deployment have not formalized an
abstraction spanning multiple hosts, one person ought not expend the
effort to create one.

## Credits

  * ansible play book derived from https://github.com/ansible/ansible-examples
    copyright Eugene Varnavsky, licence CC-BY-3.0
  * cloudformation template derived from
    http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/sample-templates-appframeworks-us-west-2.html

