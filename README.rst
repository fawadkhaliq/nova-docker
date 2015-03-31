===============================
nova-docker
===============================

Docker driver for OpenStack Nova.

Free software: Apache license

----------------------------
Installation & Configuration
----------------------------

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
1. Install the python modules.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For example::

  $ python setup.py install

Note: There are better and cleaner ways of managing Python modules, such as using distribution packages or 'pip'. The setup.py file and Debian's stdeb, for instance, may be used to create Debian/Ubuntu packages.

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
2. Enable the driver in Nova's configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In nova.conf::

  compute_driver=novadocker.virt.docker.DockerDriver

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
3. Optionally tune site-specific settings.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In nova.conf::

  [docker]
  # Commented out. Uncomment these if you'd like to customize:
  ## vif_driver=novadocker.virt.docker.vifs.DockerGenericVIFDriver
  ## snapshots_directory=/var/tmp/my-snapshot-tempdir

--------------------------
Uploading Images to Glance
--------------------------

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
1. Enable the driver in Glance's configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In glance-api.conf::

  container_formats=ami,ari,aki,bare,ovf,ova,docker

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
2. Save docker images to Glance
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Images may now be saved directly to Glance::

  $ docker pull busybox
  $ docker save busybox | glance image-create --is-public=True --container-format=docker --disk-format=raw --name busybox

**Note:** At present, only administrators should be allowed to manage images.  With devstack you can make yourself administrator by sourcing openrc as::

  source openrc admin

Once done you can go back to a user::

  source openrc demo

The name of the image in Glance should be explicitly set to the same name as the image as it is known to Docker. In the example above, an image has been tagged in Docker as 'busybox'. Matching this is the '--name busybox' argument to *glance image-create*. If these names do not align, the image will not be bootable.

^^^^^^^^^^^^^^^^^^^^^
3. Generate a keypair
^^^^^^^^^^^^^^^^^^^^^

You can optionally create a keypair to use in your docker images::

  nova keypair-add mykey > mykey.pem

^^^^^^^^^^^^^^^^^^^^^
4. Start a container
^^^^^^^^^^^^^^^^^^^^^

Start a new container.  This uses the key created above::

  nova boot --flavor m1.small --image cirros --key-name mykey test1

^^^^^^^^^^^^^^^^^^^^^
5. ssh into container
^^^^^^^^^^^^^^^^^^^^^

You can check the IP address of the container by using::

  nova list

And then ssh into it::

  ssh -i ../devstack/mykey.pem cirros@<IP ADDRESS>

-----
Notes
-----

* Earlier releases of this driver required the deployment of a private docker registry. This is no longer required. Images are now saved and loaded from Glance.
* Images loaded from Glance may do bad things. Only allow administrators to add images. Users may create snapshots of their containers, generating images in Glance -- these images are managed and thus safe.

----------
Contact Us
----------
Join us in #nova-docker on Freenode IRC

--------
Features
--------

* TODO


----------------
DevStack Install
----------------

^^^^^^^^^^^^^^^^
1. Prerequisites
^^^^^^^^^^^^^^^^

a VM with ubuntu 14.04
download server in software & update center set to main server

^^^^^^^^^^^^^^^^
2. Install DevStack
^^^^^^^^^^^^^^^^

git clone https://git.openstack.org/openstack-dev/devstack
cd devstack
cp ./samples/local.conf local.conf (using the default configuration file)
./stack.sh
if everything works fine unstack devstack using ./unstack.sh before moving forward

^^^^^^^^^^^^^^
3. Install Docker
^^^^^^^^^^^^^^

sudo apt-get update
sudo apt-get install docker.io

^^^^^^^^^^^^^^^^^^^^
4. Install Nova-Docker
^^^^^^^^^^^^^^^^^^^^

git clone https://git.openstack.org/stackforge/nova-docker /opt/stack/nova-docker
cd /opt/stack/nova-docker
sudo python setup.py install

^^^^^^^^^^^^^^^^^^^^
5. Prepare DevStack
^^^^^^^^^^^^^^^^^^^^

export INSTALLDIR={DEVSTACK_PARENT_DIR (IN THIS CASE $HOME)}
./opt/stack/nova-docker/contrib/devstack/prepare_devstack.sh
make sure DATABASE_PASSWORD is set to the same value in localrc ~/devstack/localrc as is in ~/devstack/localconf
also to make things easy for later change ADMIN_PASSWORD=admin in ~/devstack/localrc

localrc
following is the content in ~/devstacl/lcoalrc

#disable nova net 
disable_service n-net

#enable neutron services
enable_service q-svc
enable_service q-agt
enable_service q-dhcp
enable_service q-l3
enable_service q-meta
enable_service neutron


export VIRT_DRIVER=docker
export DEFAULT_IMAGE_NAME=cirros
export NON_STANDARD_REQS=1
export IMAGE_URLS=" "
export VIRT_DRIVER=docker
export DEFAULT_IMAGE_NAME=cirros
export NON_STANDARD_REQS=1
export IMAGE_URLS=" "
DATABASE_PASSWORD=stackdb
RABBIT_PASSWORD=55085e3154bfdf2c6f0f
SERVICE_TOKEN=32f531a405fc1031bebb
SERVICE_PASSWORD=b8223cf92af4627f7c8d
ADMIN_PASSWORD=admin

^^^^^^^^^^^^^^^^^^^^^^^^^^^
6. Install a Docker Filter
^^^^^^^^^^^^^^^^^^^^^^^^^^^

 sudo cp /opt/stack/nova-docker/etc/nova/rootwrap.d/docker.filters \
        /etc/nova/rootwrap.d/

^^^^^^^^^
7. Stack!
^^^^^^^^^

./stack.sh

