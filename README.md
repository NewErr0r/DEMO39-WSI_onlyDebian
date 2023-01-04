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

<h2>Сетевая связность</h2>

<p>В рамках данного модуля требуется обеспечить сетевую связность между регионами работы приложения, а также обеспечить выход ВМ в имитируемую сеть “Интернет”.</p>

<ul>
    <li>Сети, подключенные к ISP, считаются внешними:</li>
    <ul>
      <li>Запрещено прямое попадание трафика из внутренних сетей во внешние и наоборот;</li>
        <h4>ISP</h4>
        <pre>echo net.ipv4.ip_forward=1 >> /etc/sysctl.conf<br> sysctl -p</pre>
    </ul>
    <li>Платформы контроля трафика, установленные на границах регионов, должны выполнять трансляцию трафика, идущего из соответствующих внутренних сетей во внешние сети стенда и в сеть Интернет.</li>
    <ul>
      <li>Трансляция исходящих адресов производится в адрес платформы, расположенный во внешней сети.</li>
        <h4>ISP</h4>
        <pre>apt install -y firewalld<br>systemctl enable --now firewalld</pre>
        <pre>firewall-cmd --permanent --zone=trusted --add-interface=enp0s{8,9,10}<br>firewall-cmd --permanent --add-masquerade<br>firewall-cmd --reload</pre>
        <h4>RTR-L</h4>
        <pre>echo net.ipv4.ip_forward=1 >> /etc/sysctl.conf<br> sysctl -p</pre>
        <pre>echo nameserver 77.88.8.8 > /etc/resolv.conf<br>apt install -y firewalld<br>systemctl enable --now firewalld</pre>
        <pre>firewall-cmd --permanent --zone=trusted --add-interface=enp0s8<br>firewall-cmd --permanent --add-masquerade<br>firewall-cmd --reload</pre>
        <h4>RTR-R</h4>
        <pre>echo net.ipv4.ip_forward=1 >> /etc/sysctl.conf<br> sysctl -p</pre>
        <pre>echo nameserver 77.88.8.8 > /etc/resolv.conf<br>apt install -y firewalld<br>systemctl enable --now firewalld</pre>
        <pre>firewall-cmd --permanent --zone=trusted --add-interface=enp0s8<br>firewall-cmd --permanent --add-masquerade<br>firewall-cmd --reload</pre>
    </ul>
</ul>

<ul>
    <li>Между платформами должен быть установлен защищенный туннель, позволяющий осуществлять связь между регионами применением внутренних адресов.</li>
    <ul>
    <li>Трафик, проходящий по данному туннелю, должен быть защищен:</li>
        <ul>
        <li>Платформа ISP не должна иметь возможности просматривать содержимое пакетов, идущих из одной внутренней сети в другую.</li>
        </ul>
    <li>Туннель должен позволять защищенное взаимодействие между платформами управления трафиком по их внутренним адресам</li>
        <ul>
        <li>Взаимодействие по внешним адресам должно происходит без применения туннеля и шифрования</li>
        </ul>
    <li>Трафик, идущий по туннелю между регионами по внутренним адресам, не должен транслироваться.</li>
    <h4>RTR-L</h4>
    <pre>apt install wireguard -y</pre>
    <pre>mkdir /etc/wireguard/keys<br>cd /etc/wireguard/keys<br>wg genkey | tee srv-sec.key | wg pubkey > srv-pub.key<br>wg genkey | tee cli-sec.key | wg pubkey > cli-pub.key</pre>
    <pre>cat srv-sec.key cli-pub.key >> /etc/wireguard/wg0.conf<br>vi /etc/wireguard/wg0.conf<br>
      [Interface]
      Address = 10.20.30.1/30
      ListenPort = 12345
      PrivateKey = srv-sec.key<br>  
      [Peer]
      PublicKey = cli-pub.key
      AllowedIPs = 10.20.30.0/30, 172.16.100.0/24</pre>
    <pre>systemctl enable --now wg-quick@wg0</pre>
    <pre>firewall-cmd --permanent --add-port=12345/{tcp,udp}<br>firewall-cmd --permanent --zone=trusted --add-interface=wg0<br>firewall-cmd --reload</pre>
    <pre>vi /etc/ssh/sshd_config<br>...<br>PermitRootLogin yes<br>...</pre>
    <pre>systemctl restart sshd</pre>
    <h4>RTR-R</h4>
    <pre>apt install wireguard -y</pre>
    <pre>mkdir /etc/wireguard/keys<br>cd /etc/wireguard/keys<br>scp root@4.4.4.100:/etc/wireguard/cli-sec.key ./<br>scp root@4.4.4.100:/etc/wireguard/srv-pub.key ./</pre>
    <pre>cat cli-sec.key srv-pub.key >> /etc/wireguard/wg0.conf<br>vi /etc/wireguard/wg0.conf<br>
      [Interface]
      Address = 10.20.30.2/30
      PrivateKey = cli-sec.key<br>  
      [Peer]
      PublicKey = srv-pub.key
      Endpoint = 4.4.4.100:12345
      AllowedIPs = 10.20.30.0/30, 192.168.100.0/24
      PersistentKeepalive = 10</pre>
    <pre>systemctl enable --now wg-quick@wg0</pre>
    <pre>firewall-cmd --permanent --zone=trusted --add-interface=wg0<br>firewall-cmd --reload</pre>
    </ul>
</ul>
<ul>
    <li>Платформа управления трафиком RTR-L выполняет контроль входящего трафика согласно следующим правилам:</li>
    <ul>
        <li>Разрешаются подключения к портам DNS, HTTP и HTTPS для всех клиентов;</li>
        <ul>
            <li>Порты необходимо для работы настраиваемых служб</li>
        </ul>
        <li>Разрешается работа выбранного протокола организации защищенной связи;</li>
        <ul>
            <li>Разрешение портов должно быть выполнено по принципу “необходимо и достаточно”</li>
        </ul>
        <li>Разрешается работа протоколов ICMP;</li>
        <li>Разрешается работа протокола SSH;</li>
        <li>Прочие подключения запрещены;</li>
        <li>Для обращений в платформам со стороны хостов, находящихся внутри регионов, ограничений быть не должно;</li>
        <h4>RTR-L</h4>
        <pre>firewall-cmd --permanent --add-service={dns,http,https}<br>firewall-cmd --permanent --add-protocol=icmp<br>firewall-cmd --reload</pre> 
    </ul>
    <li>Платформа управления трафиком RTR-R выполняет контроль входящего трафика согласно следующим правилам:</li>
    <ul>
        <li>Разрешаются подключения к портам HTTP и HTTPS для всех клиентов;</li>
        <ul>
            <li>Порты необходимо для работы настраиваемых служб</li>
        </ul>
        <li>Разрешается работа выбранного протокола организации защищенной связи;</li>
        <ul>
            <li>Разрешение портов должно быть выполнено по принципу “необходимо и достаточно”</li>
        </ul>
        <li>Разрешается работа протоколов ICMP;</li>
        <li>Разрешается работа протокола SSH;</li>
        <li>Прочие подключения запрещены;</li>
        <li>Для обращений в платформам со стороны хостов, находящихся внутри регионов, ограничений быть не должно;</li>
        <h4>RTR-R</h4>
        <pre>firewall-cmd --permanent --add-service={http,https}<br>firewall-cmd --permanent --add-protocol=icmp<br>firewall-cmd --reload</pre> 
    </ul>
</ul>
<ul>
    <li>Обеспечьте настройку служб SSH региона Left:</li>
    <ul>
        <li>Подключения со стороны внешних сетей по протоколу к платформе управления трафиком RTR-L на порт 2222 должны быть перенаправлены на ВМ Web-L;</li>
        <li>Подключения со стороны внешних сетей по протоколу к платформе управления трафиком RTR-R на порт 2244 должны быть перенаправлены на ВМ Web-R;</li>
        <h4>RTR-L</h4>
        <pre>firewall-cmd --permanent --add-forward-port=port=2222:proto=tcp:toport=22:toaddr=192.168.100.100<br>firewall-cmd --reload</pre>
        <h4>RTR-R</h4>
        <pre>firewall-cmd --permanent --add-forward-port=port=2244:proto=tcp:toport=22:toaddr=172.16.100.100<br>firewall-cmd --reload</pre>
        <h4>WEB-L</h4>
        <pre>vi /etc/ssh/sshd_config<br>...<br>PermitRootLogin yes<br>...</pre>
        <pre>systemctl restart sshd</pre>
        <h4>WEB-R</h4>
        <pre>vi /etc/ssh/sshd_config<br>...<br>PermitRootLogin yes<br>...</pre>
        <pre>systemctl restart sshd</pre>
    </ul>
</ul>

<h2>Инфраструктурные службы</h2>

<p>В рамках данного модуля необходимо настроить основные инфраструктурные службы и настроить представленные ВМ на применение этих служб для всех основных функций.</p>

<h1>Таблица 2. DNS-записи зон</h1>

![Image alt](https://github.com/NewErr0r/39-WSI/blob/main/table_dns.png?raw=true)

<ul>
    <li>Выполните настройку первого уровня DNS-системы стенда:</li>
    <ul>
        <li>Используется ВМ ISP;</li>
        <li>Обслуживается зона demo.wsr.</li>
        <ul>
            <li>Наполнение зоны должно быть реализовано в соответствии с Таблицей 2;</li>
        </ul>
        <li>Сервер делегирует зону int.demo.wsr на SRV;</li>
        <ul>
            <li>Поскольку SRV находится во внутренней сети западного региона, делегирование происходит на внешний адрес маршрутизатора данного региона.</li>
            <li>Маршрутизатор региона должен транслировать соответствующие порты DNS-службы в порты сервера SRV.</li>
        </ul>
        <li>Внешний клиент CLI должен использовать DNS-службу, развернутую на ISP, по умолчанию;</li>
        <h4>ISP</h4>
        <pre>apt install bind9 -y</pre>
        <pre>mkdir /opt/dns<br>cp /etc/bind/db.local /opt/dns/demo.db<br>chown -R bind:bind /opt/dns</pre>
        <pre>vi /etc/apparmor.d/usr.sbin.named<br>...<br>/opt/dns/** rw,<br>...</pre>
        <pre>systemctl restart apparmor.service</pre>
        <pre>vi /etc/bind/named.conf.options<br>options {
            directory "/var/cache/bind";
            dnssec-validation no;
            allow-query { any; };
            listen-on-v6 { any; };
        };</pre>
        <pre>vi /etc/bind/named.conf.default-zones<br>zone "demo.wsr" {
            type master;
            allow-transfer { any; };
            file "/opt/dns/demo.db";
        };</pre>
        <pre>vi /opt/dns/demo.db</pre>
        
![Image alt](https://github.com/NewErr0r/39-WSI/blob/main/demo_db.png?raw=true)

</pre>
<pre>systemctl restatr bind9</pre>
        <h4>RTR-L</h4>
        <pre>firewall-cmd --permanent --add-forward-port=port=53:proto={tcp,udp}:toport=53:toaddr=192.168.100.200<br>firewall-cmd --reload</pre>
</ul>


