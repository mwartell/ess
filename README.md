# ESS Test Scenario

Submitted by matt.wartell@gmail.com


## Resource Topology

The test specification specifies 99.999% uptime and dispersed geographic
zones.  In order to acheive High Availabiliy, multi-region application
servers need to be used as does database mirroring and failover.

Using multiple regions entails a plurality of VPCs which, for the sake
of simplicity will be allocated one per region. Multiple VPC entails
multiple interconnected subnets. Subnet numbering could be automated,
but for this case it is easiest to specify regional subnets manually. The
subnets will be in th 10.0.0.0/24 block with network 10.1.0.0/24 for
the home vpc and 10.2.0.0/24 for the next region and so on. There
are some capabilities that are foregone by keeping the VPC map
simple,such as unreachability of the database VPC from the public
internet.

There is an orchestration host which is responsible for hosting and
executing all provisioning for the target services and software. This
gives the collection of resources a "home base" which will require
special provisioning. The order of provisioning also is important: for
instance, one cannot create a Multi-AZ DB instance unless the secondary
VPC already exists.

### Levels of Description

When there are three logical layers as there is in this design it
is important to keep them separated as much as possible. Naming the
layers helps keep them distinct:

  a. Infrastructure — the virtual machine rooms that will host
     the various services. Everything that translates into a AWS-API
     call is considered part of infrastructure; examples are VPC, ELB,
     Security Groups, routing tables, AMI selection, and so on.
  b. Application - the web applications running on the provisioned
     infrastructure. The primary components here are Apache and PHP.
     Given the use of RDS, the database is notably infrastructure.
     The application layer is also where the Linux permissions
     structure operates.

### Orchestration Ordering

  1. Create the "home" vpc
  2. create the orchestration vm
  3. clone this repository onto orchestration
  4. execute the aws resource provisioning
  5. instantiate multiple web hosts with LAMP loading

## Private Cloud Migration

Although the test scenario states that "the architecture should be able
to adapt to deployment" on a private cloud, the implications of that
requirement are underspecified. In particular, without knowledge of the
target system, trying to anticipate what might be needed is folly.

For the purposes of this exercise, I have done nothing to explicitly aid
porting while attempting to minimize operational dependencies. Although
cloud platforms — including private cloud such as VMware — provide
highly overlapping services, they are not congruent. Tools like Ansible
which attempt to abstract configuration actions from the underlying
system don't even attempt to homogenize cloud interfaces, providing
idiosyncratic platform-specific modules for each hosting service.


## Console Actions

There are a few actions needed to bootstrap this environment. Although
possibly scriptable, the merit of scripting is minimal:

  * create an AWS non-root user with SSH keys and Administrator policy

## Design Choices

There are three primary orchestration tool choices: Ansible,
CloudFormation, and AWS-CLI. These can be combined in various ways such
as Ansible orchestrating stack creation specified by CloudFormation as
accessed through the CLI. There is rarely an optimal choice of mechanism,
and obvious trade-offs between them; the Ansible use of declarative
mechanisms has distinct advantages for orchestration, but it does pay
for it in terms of concision and lack of readability.

Other design choices include:

    * LAMP is specified but RDS is used in preference to MySQL.
      The primary reason for this is to more conveniently handle
      database failures at the cost of greater infrastructure
      dependency.
    * HTTPS is not specified and not implemented. This will reduce
      installation complexity.
    * Despite significant static content in MediaWiki, CloudFront
      is not used.
    * IPv6 is studiously ignored.
    * Cross-region routing over the public internet should be shrouded
      with a VPN but is not for implementation simplicity.

    * aws-cli, git, ansible and boto are assumed present


## Credits

  * ansible play book derived from https://github.com/ansible/ansible-examples
    copyright Eugene Varnavsky, licence CC-BY-3.0

