# VirtIO automation
virtio-automation is a set of ansible playbooks that can set up a number of virtio environments to be used for benchmarking and debugging purposes.

## Architecture


The architecture of this benchmark is composed of 3 elements:

* **The KVM host**: The main target.
* **Guest**: The guest VM where the NFV application will run. Port forwarding will be configured
so that the guest is available on port 2222 in the host's control interface
* **T-REX** Another host connected via two independent NICs to the target where the [t-rex packet generator](https://trex-tgn.cisco.com/) will run.




              +------------------------------------+
              |                                    |
              |     Guest                          |
              |                  +---------------+ |
              |                  |   dpdk        | |
              |  22              +---------------+ |
              +---+--------------------------------+
                  ^                |    |   |   |
              +------------------------------------+
              |   |   KVM host   +---------------+ |
              +-----------+      | dpdk          | |
         2222 +---+       |      +---------------+ |
              |ctrl iface |       +----+   +----+  |
              +-----------+       |NIC1|   |NIC2|  |
              +----------------------+--------+----+
                                     |        |
                                     |        |
                                     |        |
                                     |        |
                                     |        |
                                     |        |
              +----------------------+--------+---+
              |                   |NIC1|   |NIC2| |
              +-----------+       +----+   +----+ |
              |ctrl iface |                       |
              +-----------+                       |
              |                                   |
              |                    TREX injector  |
              +-----------------------------------+






## Usage
In order to use this scripts you need to modify the given example inventory to match your needs:

    all:
      hosts:

      children:
        # kvm-hosts: The main target
        kvm-hosts:
          my-kvm-host:
            ansible_host: kvmhost.example.com
            ansible_user: root
            ansible_port: 22
            remote_image: http://download.eng.brq.redhat.com/rhel-7/rel-eng/RHEL-7/latest-RHEL-7.6/compose/Server/x86_64/images/rhel-guest-image-7.6-210.x86_64.qcow2
            nics:
              - "0000:04:00.0"
              - "0000:04:00.1"

        # guests: The guest VM. The ansible playbook will add port-forwarding
        # rules to the host so the guest is accesible at port 2222, so the
        # "ansible_host" should be the same the kvm-host and "ansbile_port" 2222
        # Modify the ansible_ssh_private_keyfile to the apropriate keyfile
        guests:
          my-guest:
            ansible_host: kvmhost.example.com
            ansible_user: root
            ansible_port: 2222
            ansible_ssh_private_key_file: guest_id_rsa

        # injectors: This host will be used to inject traffic into the target
        # It should be physically connected to the target via two different NICs
        injectors:
          virtlab218:
            ansible_host: trex.example.com
            ansible_ssh_user: root
            nics:
              - "0000:04:00.0"
              - "0000:04:00.1"




Note that a guest_id_rsa ssh key is needed to be able to access the guest once it's booted, so create it:

    $ ssh-keygen -t rsa -N "" -f guest_id_rsa


Now you can run the playbooks:

To set the host up:

    $ ansible-playbook -i ./your_inventory.yml host-setup.yml

To tear the host down:

    $ ansible-playbook -i ./your_inventory.yml host-teardown.yml

To set the guest up:

    $ ansible-playbook -i ./your_inventory.yml guest-setup.yml


To install t-rex:

    $ ansible-playbook -i ./your_inventory.yml trex-setup.yml


Note: If you use --ask-pass, you will need to `export ANSIBLE_HOST_KEY_CHECKING=False` to run guest-related playbooks
(at least the first time)


## TODO
* Support vIO-MMU architecture
* Support SR-IOV architecture
* Support vDPA architecture
* Be able to select vhost-user vs vhost-net
* Be able to select virtio-net vs virtio-pmd (dpdk-based)
* Be able to run a user-specified version of dpdk, qemu, vfio ...

