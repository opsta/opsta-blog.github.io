---
layout: post
title:  "ลองใช้งาน OpenStack ง่ายๆ เพียงชั่วโมงละ 5 บาท"
date:   2017-05-15 09:38:13 +0700
categories: openstack openstack-ansible
---

![OpenStack Overview Diagram]({{ site.url }}/assets/20170515/openstack-overview-diagram.svg)

ปัญหาแรกสุดของเหล่า OpenStacker มือใหม่ที่อยากจะลองเล่น [OpenStack][openstack] คือ ไม่มีเครื่องเซิร์ฟเวอร์ที่สเปคสูงพอ และไม่รู้จะเริ่มลง OpenStack ได้ยังไง วันนี้เราเลยมาเสนอทางที่ Opsta เลือกใช้ในการอบรม [Opsta OpenStack Administration Workshop][openstack-workshop] ที่เช่าใช้เซิร์ฟเวอร์ราคาถูกบน Digital Ocean และใช้ user-data script ทำให้สามารถลง OpenStack มาเล่นได้ในภายใน script เดียว

<!--more-->

สิ่งที่ควรรู้
--------------------

อย่างแรก การลง OpenStack แบบ all-in-one คือการที่เราติดตั้งโปรเจคหลักๆ ของ OpenStack ทั้งหมดลงอยู่ในเครื่องเดียว ซึ่งโปรเจคหลักๆ ของ OpenStack จะประกอบไปด้วย

- Nova
- Keystone
- Glance
- Cinder
- Neutron
- Horizon

![OpenStack Popular Project Set]({{ site.url }}/assets/20170515/openstack-popular-project-set.png)
*รูปโลโก้โปรเจคต่างๆ ของ OpenStack สุดน่ารัก*{: .center-text }

ซึ่งสังเกตได้ว่า จะมีโปรเจคที่ทำงานร่วมกันอย่างน้อยถึง 6 ตัว และแต่ละโปรเจคก็กินทรัพยากรไม่ใช่น้อย จากประสบการณ์ของทีมเราพบว่า OpenStack Ansible แบบ all-in-one ต้องการเซิร์ฟเวอร์สเปคขั้นต่ำอย่างน้อยดังนี้

- CPU 4 cores
- RAM 8GB
- SSD 60GB

ซึ่งทรัพยากรข้างต้นนั้นจะเพียงพอเพียงแค่รัน OpenStack เท่านั้น แต่ถ้าจะลองเล่นและมีการสร้าง instance  ก็ต้องมีทรัพยากรตามจำนวนของ instance ที่เราต้องการสร้างเพิ่มไปด้วย เพราะฉะนั้นถ้าต้องการทดลองเล่น OpenStack แบบเต็มรูปแบบจริงๆ แล้วเราควรจะมีแรมอย่างน้อย 16GB และมี SSD อย่างน้อย 120GB ซึ่งหมายความว่าเราจะต้องลงทุนซื้อ desktop หรือ server อย่างน้อยๆ ก็สองสามหมื่นบาทขึ้นไปเพื่อมาแค่ทดลองใช้ OpenStack งานนี้หลายๆ คนอาจจะต้องคลานเข่าเข้าไปหาหัวหน้าเลยทีเดียว

แต่จะดีกว่าไหม ถ้าเราสามารถทดลองใช้ OpenStack บนคลาวด์ได้ในราคาถูก ซึ่งตัวเลือกที่ Opsta เลือกใช้คือ [Digital Ocean][digital-ocean] ซึ่งถ้าดูจากหน้า[ราคากับสเปคที่มีให้เลือก][do-price] เครื่องแรมขนาด 8GB ราคาเพียงแค่ $0.119 หรือราคาประมาณ 4 บาทกว่าๆ ต่อชั่วโมงเท่านั้น หรือแม้กระทั่งถ้าเราเลือกเครื่องแรมขนาด 16GB ราคาก็จะตกอยู่ที่ไม่เกิน 9 บาทต่อชั่วโมงเท่านั้นเอง!

![Digital Ocean Price Hourly]({{ site.url }}/assets/20170515/do-price-hourly.png)

อีกสาเหตุนึงที่เลือก Digital Ocean เพราะว่ามีฟีเจอร์ user-data ที่ทำให้เราสามารถใส่สคริปต์สั่งให้รัน command หลังจากสร้างเครื่องเสร็จ ซึ่งหมายความว่า เราสามารถสั่งให้มันลง OpenStack หลังจากที่กดสร้างเครื่องเสร็จ และเราก็จะมีหน้าที่เดียวคือ ssh เข้าไปดูสถานะการลง และจิบกาแฟรอจนกว่า OpenStack จะลงเสร็จและเริ่มใช้งานได้ทันที

เมื่อเราแก้ปัญหาเรื่องเซิร์ฟเวอร์ได้แล้ว เราก็มาเริ่มลงกันเลยดีกว่า แต่ก่อนที่จะเริ่มลง เราต้องเข้าใจก่อนว่า OpenStack all-in-one ก็จะมีหลายค่ายให้เล่น ตั้งแต่ [ลองลงแบบ manual เอง][openstack-manual-install] (โหดเกิ๊นนน), [DevStack][devstack], [Packstack][packstack] หรือ [OpenStack Ansible][openstack-ansible] ที่ Opsta จะนำมาเป็นตัวอย่างในการติดตั้ง OpenStack all-in-one ในบล็อกนี้

![OpenStack Ansible Logo]({{ site.url }}/assets/20170515/osa-logo.png)
*รูปสัตว์โลโก้ของ OpenStack Ansible คือ Cape Buffalo*{: .center-text }

[OpenStack Ansible][openstack-ansible] หรือ OSA คือโปรเจคหนึ่งของ OpenStack ที่จะมาช่วยติดตั้ง ตั้งค่า และจัดการ OpenStack cluster ด้วย [Ansible][ansible] playbooks โดยจะสามารถติดตั้งและดูแลได้ทั้ง all-in-one หรือ OpenStack cluster ใน production ที่มีขนาดเป็นร้อยเป็นพันเครื่องได้ โดยปัจจุบัน OpenStack Ansible ได้ถูกพัฒนาจนเป็นตัวเลือกแรกที่ถูกแนะนำให้ใช้ในการติดตั้ง OpenStack ใน [OpenStack Deployment Guides][openstack-deployment-guide] และ OpenStack Foundataion เลือกเป็นตัวที่ใช้ในการเตรียม environment ที่ใช้ในการสอบ [Certified OpenStack Administrator][coa] (COA) เลยทีเดียว


เริ่มติดตั้ง OpenStack
--------------------

เกริ่นมาซะเยอะ เรามาเริ่มติดตั้ง OpenStack Ansible all-in-one กันเลยดีกว่า

- ขั้นแรกให้เราไปสมัครสมาชิก [Digital Ocean][digital-ocean] พร้อมใส่หมายเลขบัตรเครดิตให้เรียบร้อย **หรือถ้าอยากได้ credit $10 ฟรี** ให้กดสมัครผ่าน [referrer][do-referrer] ของเราอันนี้เลยครับ
- เสร็จแล้วที่หน้าหลัก ให้เราเลือก **Create Droplet** ซึ่ง Digital Ocean จะเรียก Virtual Machine ที่ถูกสร้างขึ้นว่า Droplet ครับ

![Digital Ocean Main Page]({{ site.url }}/assets/20170515/do-main.png)

- ที่หน้าจอ Create Droplets ให้เราเลือกดังนี้
  - *Choose an image* เลือกเป็น Distributions **Ubuntu 16.04.2 x64**
  - *Choose a size* ให้เลือกเป็นขนาดแรม 8GB หรือ 16GB แล้วแต่ทุนทรัพย์ของเรา
  - *Add block storage* ไม่ต้องทำอะไร ปล่อยว่างไว้
  - *Choose a datacenter region* ปกติเราจะเลือก **Singapore** ที่อยู่ใกล้ไทยมากที่สุด
  - *Select additional options* ให้ติ๊กถูกที่ **User data** และใส่ script ดังข้างล่างนี้ลงไป

    ```bash
    #!/usr/bin/env bash

    set -e -x -u

    # Update and install required packages
    apt update
    apt -y dist-upgrade
    apt install -y git vim python unzip iotop htop iftop

    # Clone OSA
    git clone -b 15.1.2 https://github.com/openstack/openstack-ansible /opt/openstack-ansible

    # Prepare machine for OSA all-in-one
    cd /opt/openstack-ansible/
    ./scripts/bootstrap-ansible.sh
    # Need to source scripts-library.sh for only in cloud user-data to get all
    # environment variables
    source scripts/scripts-library.sh
    ./scripts/bootstrap-aio.sh

    # Tweak some configuration
    cat << EOF > /etc/openstack_deploy/user_z.yml
    horizon_time_zone: Asia/Bangkok
    horizon_session_timeout: 28800
    # To make Horizon can upload via browser
    horizon_images_upload_mode: legacy
    EOF

    # Install OSA all-in-one
    bash /opt/openstack-ansible/scripts/run-playbooks.sh
    ```
  - *Add your SSH keys* ถ้าต้องการ SSH โดยไม่ต้องใส่รหัสผ่าน ก็ให้คลิ๊กที่ **New SSH Key** แล้วใส่ public key ของเราลงไป แต่ถ้าไม่ใส่ หลังจากสร้าง droplet เสร็จ จะมีรหัสผ่านที่ถูกสุ่มขึ้นมาส่งมาให้ทางอีเมลของเราครับ
  - *Finalize and create* อันนี้จะเป็นการเลือกจำนวน และตั้งชื่อ droplet ที่เราจะสร้างขึ้นมาครับ เสร็จแล้วให้กดปุ่ม **Create** เพื่อสร้าง droplet ได้เลย

![Digital Ocean Create Droplet]({{ site.url }}/assets/20170515/do-create-droplet.png)

- เสร็จแล้วรอให้มันสร้าง droplet สักครู่ หลังจากสร้างเสร็จจะมีไอพีแสดงขึ้นมา ให้เรา copy ไอพีเก็บไว้

![Digital Ocean Create Droplet]({{ site.url }}/assets/20170515/do-droplet.png)

- เสร็จแล้วให้เรา SSH เข้าไปยัง droplet ที่เราเพิ่งสร้างขึ้น

```
ssh root@your-ip-address
```

- ตอนนี้ droplet ที่เราเพิ่งสร้างขึ้นจะกำลังติดตั้ง OpenStack Ansible all-in-one อยู่ ซึ่งเราสามารถดูสถานะการทำงานจาก log file */var/log/cloud-init-output.log* โดยจะใช้เวลาติดตั้งทั้งหมดประมาณ 2-4 ชั่วโมงครับ

```
tail -f /var/log/cloud-init-output.log
```

![Digital Ocean Create Droplet]({{ site.url }}/assets/20170515/osa-installing.png)

- ซึ่งหลังจากติดตั้งเสร็จเรียบร้อยแล้ว เราจะสามารถเข้าหน้าจอล็อคอินของ OpenStack Horizon ได้ทันทีผ่าน https://your-ipaddress โดย username คือ admin และรหัสผ่านจะถูกสุ่มขึ้นมาโดยสามารถดูได้จาก command

```
cat ~/openrc | grep OS_PASSWORD
```

![Digital Ocean Create Droplet]({{ site.url }}/assets/20170515/openstack-login.png){: .center-image }


ช่วงขายของ
--------------------

ติดตั้ง OpenStack เสร็จแล้ว จะเล่นยังไงต่อดี? ตอนนี้ Opsta เปิดอบรม [Opsta OpenStack Administration Workshop][openstack-workshop] ซึ่งเราจะเน้นที่การทำแล็ป OpenStack ในการใช้งานจริง ไปพร้อมกับทำความเข้าใจการทำงานร่วมกันระหว่างโปรเจคต่างๆ ของ OpenStack ซึ่งจะทำให้คนที่เข้าอบรมสามารถเข้าใจ และสามารถดูแลระบบของ OpenStack ได้ รวมถึงสามารถกลับเอาไปทดลองเล่นต่อที่บ้านหรือบริษัทของท่านได้ทันที รีบสมัครกันเข้ามา https://training.opsta.io ก่อนที่ที่นั่งจะเต็มหมดก่อนนะครับ ตอนนี้เหลืออยู่อีกไม่กี่ที่แล้ว



[openstack]: https://www.openstack.org
[openstack-workshop]: https://training.opsta.io
[digital-ocean]: https://www.digitalocean.com
[do-price]: https://www.digitalocean.com/pricing
[devstack]: https://docs.openstack.org/developer/devstack/
[openstack-manual-install]: https://docs.openstack.org/ocata/install-guide-ubuntu/
[packstack]: https://www.rdoproject.org/install/quickstart/
[openstack-ansible]: https://docs.openstack.org/developer/openstack-ansible/
[openstack-deployment-guide]: https://docs.openstack.org/project-deploy-guide/ocata/
[coa]: https://www.openstack.org/coa/
[ansible]: https://www.ansible.com/
[do-referrer]: https://m.do.co/c/7a8e05c5cc09
