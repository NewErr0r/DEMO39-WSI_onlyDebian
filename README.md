<h1>DEMO2023</h1>

<h1>Образец задания</h1>

<p>Образец задания для демонстрационного экзамена по комплекту оценочной документации.</p>

<h1>Описание задания</h1>

![Image alt](https://github.com/NewErr0r/39-WSI/blob/main/demo_topology.drawio.png?raw=true)

<h1>Таблица 1. Характеристики ВМ</h1>

![Image alt](https://github.com/NewErr0r/39-WSI/blob/main/table.png?raw=true)

<h2>Виртуальные машины и коммутация</h2>

<p>Необходимо выполнить создание и базовую конфигурацию виртуальных машин.</p>

<ul>
    <li>На основе предоставленных ВМ или шаблонов ВМ создайте отсутствующие виртуальные машины в соответствии со схемой</li>
    <ul>
      <li>Характеристики ВМ установите в соответствии с Таблицей 1;</li>
      <li>Коммутацию (если таковая не выполнена) выполните в соответствии со схемой сети.</li>
    </ul>
    <li>Имена хостов в созданных ВМ должны быть установлены в соответствии со схемой.</li>
    <li>Адресация должна быть выполнена в соответствии с Таблицей 1;</li>
    <li>Обеспечьте ВМ дополнительными дисками, если таковое необходимо в соответствии с Таблицей 1;</li>
</ul>

<h4>ISP</h4>

<pre>
hostnamectl set-hostname ISP
</pre>
<pre>
vi /etc/network/interfaces      # enp0s3 NAT-адаптер ( доступ в Интернет )
    
    auto enp0s3 
    iface enp0s3 inet dhcp
    
    auto enp0s8
    iface enp0s8 inet static
    address 3.3.3.1 
    netmask 255.255.255.0
    
    auto enp0s9
    iface enp0s9 inet static
    address 4.4.4.1 
    netmask 255.255.255.0
    
    auto enp0s10
    iface enp0s10 inet static
    address 5.5.5.1 
    netmask 255.255.255.0    
</pre>
<pre>
systemctl restart networking
</pre>

<h4>RTR-L</h4>

<pre>
hostnamectl set-hostname RTR-L
</pre>
<pre>
vi /etc/network/interfaces
    
    auto enp0s3 
    iface enp0s3 inet static
    address 4.4.4.100
    netmask 255.255.255.0
    gateway 4.4.4.1
    
    auto enp0s8
    iface enp0s8 inet static
    address 192.168.100.254 
    netmask 255.255.255.0  
    dns-nameservers 192.168.100.200
</pre>
<pre>
systemctl restart networking
</pre>

<h4>RTR-R</h4>

<pre>
hostnamectl set-hostname RTR-R
</pre>
<pre>
vi /etc/network/interfaces
    
    auto enp0s3 
    iface enp0s3 inet static
    address 5.5.5.100
    netmask 255.255.255.0
    gateway 5.5.5.1
    
    auto enp0s8
    iface enp0s8 inet static
    address 172.16.100.254 
    netmask 255.255.255.0  
    dns-nameservers 192.168.100.200
</pre>
<pre>
systemctl restart networking
</pre>

<h4>SRV</h4>

<pre>
hostnamectl set-hostname SRV
</pre>
<pre>
vi /etc/network/interfaces
    
    auto enp0s3 
    iface enp0s3 inet static
    address 192.168.100.200
    netmask 255.255.255.0
    gateway 192.168.100.254
    dns-nameservers 4.4.4.1 192.168.100.200
</pre>
<pre>
systemctl restart networking
</pre>

<p> Доп. диски </p>

![Image alt](https://github.com/NewErr0r/39-WSI/blob/main/disks.png?raw=true)

<h4>WEB-L</h4>

<pre>
hostnamectl set-hostname WEB-L
</pre>
<pre>
vi /etc/network/interfaces
    
    auto enp0s3 
    iface enp0s3 inet static
    address 192.168.100.100
    netmask 255.255.255.0
    gateway 192.168.100.254
    dns-nameservers 192.168.100.200
</pre>
<pre>
systemctl restart networking
</pre>

<h4>WEB-R</h4>

<pre>
hostnamectl set-hostname WEB-R
</pre>
<pre>
vi /etc/network/interfaces
    
    auto enp0s3 
    iface enp0s3 inet static
    address 172.16.100.100
    netmask 255.255.255.0
    gateway 172.16.100.254
    dns-nameservers 192.168.100.200
</pre>
<pre>
systemctl restart networking
</pre>

<h4>CLI</h4>

<pre>
Rename-Computer -NewName CLI
Restart-Computer
</pre>

![Image alt](https://github.com/NewErr0r/39-WSI/blob/main/ipv4_cli.png?raw=true)
