# cisco-swim

An Ansible role to maintain the system software (check, stage, install, and clean) on various Cisco platforms

## Supported Platforms
* Cisco Nexus:
  * 5548UP
  * 9396
* Cisco IOS:
  * Catalyst 3750
  * Catalyst 3850
  * Catalyst 9410R
  * Catalyst 9300
  * ISR

## Requirements

The inventory must be properly setup for the platforms being used from the [Ansible Network Modules](https://docs.ansible.com/ansible/latest/network/index.html).  Additionally, this role uses net_put to push binary images to IOS devices which requires Ansible 2.7+.

## Cloning directly to into roles directory:

```bash
cd roles
git clone https://github.com/CiscoDevNet/ansible-cisco-swim cisco-swim
```

## Role Variables

This role uses two data structures to determine if the device is running the correct software and from where to get the correct software:

* `network_version_map`: This data structure contains the version that the platform should be running and reference into the `network_image_map` data structure to specify from which location to get the software.
* `network_image_map`: This data structure contains the location of the software, the size of the software, and the checksum for the software.  When the image URL begins with `file://`, that software is pushed from a local file.  Otherwise, that URL is used on the system to pull the software from the appropriate location.

```
really_delete_files: no

network_version_map:
  WS-C3850:
    version: '16.06.03'
    software: 'cat3k_16.6.3'
  N5K-C5548UP:
    version: '7.1(4)N1(1)'
    software: 'nxos_7.1(4)N1(1)'
  N3K-C3064PQ:
    version: '6.0(2)U6(9)'
    software: 'nxos_6.0(2)U6(9)'

network_image_map:
  'cat3k_16.6.3':
    system_image_url: file:///tmp/cat3k_caa-universalk9.16.06.03.SPA.bin
    system_image_size: 410434678
    system_image_checksum: 16fdc2dd9b8a5f7096a71e04ead0431d
  'nxos_7.3(3)N1(1)':
    system_image_url: http://1.2.3.4/n5000-uk9.7.3.3.N1.1.bin
    system_image_size: 206130057
    system_image_checksum: a828dc515a2f5ca796a8dab1d86d2eed
    kickstart_image_url: http://1.2.3.4/n5000-uk9-kickstart.7.3.3.N1.1.bin
    kickstart_image_size: 34375680
    kickstart_image_checksum: 2200d5880dc68820d9cba8083fe9f88c
  'nxos_6.0(2)U6(10)':
    system_image_url: http://1.2.3.4/n3000-uk9.6.0.2.U6.10.bin
    system_image_size: 206130057
    system_image_checksum: 98b1ba8106afbc85b83c0f985a66cd30
    kickstart_image_url: http://1.2.3.4/n3000-uk9-kickstart.6.0.2.U6.10.bin
    kickstart_image_size: 37881856
    kickstart_image_checksum: f07cbe12d2e489ce02b9577b59753335
```

## Modes of Operation

This role has 4 distinct modes of operation:

* **facts**: Just gather facts about the device and compare against the version it should be running
* **clean**: Clean unneeded images from the storage on the device to make room for new firmware.
* **stage**: Stage the software and/or check the checksum to make sure that the image is on the device storage and intact.  If the image is not present, the device storage is checked to make sure that it has enough space for the image, the image is transferred, then the image's checksum is verified after the image is transferred.  If the image is present, its checksum is verified.  If the checksum fails, the images is re-copied.
* **install**: Put the firmware into service and reboot

## Example Playbook

If the role is called without the `tasks_from` attribute, it defaults to the facts mode:
    - hosts: network
      gather_facts: no
      tasks:
        - include_role:
            name: cisco-swim

To run in a different mode, simply specify the mode in `tasks_from`:  

    - hosts: network
      gather_facts: no
      tasks:
        - include_role:
            name: cisco-swim
            tasks_from: stage

In order to pass the task in on either the command line or via an AWX template:

    - hosts: network
      gather_facts: no
      tasks:
        - include_role:
            name: cisco-swim
            tasks_from: "{{ network_software_task | default('facts') }}"

Then invoke as:

    ansible-playbook -e network_software_task=stage cisco-swim.yml

Clean mode will show you what files it will delete. It won't look to delete the previous image.
After you run clean mode you'll see the files it will delete. It won't delete the files.
Use var really_delete=yes/no

```bash
ansible-playbook your_playbook_name.yml -e really_delete_files=yes -k -vvv
```

## License

CISCO SAMPLE LICENSE
