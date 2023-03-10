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
</pre>
</ul>
</ul>
   <ul>
        <li>Выполните настройку второго уровня DNS-системы стенда;</li>
        <ul>
            <li>Используется ВМ SRV;</li>
            <li>Обслуживается зона int.demo.wsr;</li>
            <ul>
                <li>Наполнение зоны должно быть реализовано в соответствии с Таблицей 2;</li>
            </ul>
            <li>Обслуживаются обратные зоны для внутренних адресов регионов</li>
            <ul>
                <li>Имена для разрешения обратных записей следует брать из Таблицы 2;</li>
            </ul>
            <li>Сервер принимает рекурсивные запросы, исходящие от адресов внутренних регионов;</li>
            <ul>
                <li>Обслуживание клиентов(внешних и внутренних), обращающихся к зоне int.demo.wsr, должно производиться без каких-либо ограничений по адресу источника;</li>
            </ul>
            <li>Внутренние хосты регионов (равно как и платформы управления трафиком) должны использовать данную DNS-службу для разрешения всех запросов имен;</li>
            <h4>SRV</h4>
            <pre>apt install bind9 -y</pre>
            <pre>mkdir /opt/dns<br>cp /etc/bind/db.local /opt/dns/int.demo.db<br>cp /etc/bind/db.127 /opt/dns/100.168.192.in-addr.arpa.db<br>cp /etc/bind/db.127 /opt/dns/100.16.172.in-addr.arpa.db<br>chown -R bind:bind /opt/dns</pre>
            <pre>vi /etc/apparmor.d/usr.sbin.named<br>...<br>/opt/dns/** rw,<br>...</pre>
            <pre>systemctl restart apparmor.service</pre>
            <pre>vi /etc/bind/named.conf.options<br>options {
            directory "/var/cache/bind";
            dnssec-validation no;
            allow-query { any; };
            listen-on-v6 { any; };
        };</pre>
            <pre>vi /etc/bind/named.conf.default-zones<br>
        zone "int.demo.wsr" {
            type master;
            allow-transfer { any; };
            file "/opt/dns/int.demo.db";
        };<br>        
        zone "100.168.192.in-addr.arpa" {
            type master;
            allow-transfer { any; };
            file "/opt/dns/100.168.192.in-addr.arpa.db";
        };<br>  
        zone "100.16.172.in-addr.arpa" {
            type master;
            allow-transfer { any; };
            file "/opt/dns/100.16.172.in-addr.arpa.db";
        };</pre>  
            </pre>
 <pre>vi /opt/dns/int.demo.db</pre>
            
![Image alt](https://github.com/NewErr0r/39-WSI/blob/main/int_demo_db.png?raw=true)
            
 <pre>vi /opt/dns/100.168.192.in-addr.arpa.db</pre>
            
![Image alt](https://github.com/NewErr0r/39-WSI/blob/main/arpa_1.png?raw=true)
            
 <pre>vi /opt/dns/100.16.172.in-addr.arpa.db</pre>
            
![Image alt](https://github.com/NewErr0r/39-WSI/blob/main/arpa_2.png?raw=true)
            
 <pre>systemctl restatr bind9</pre>
</ul></ul>
    </pre>
        <ul>
        <li>Выполните настройку первого уровня системы синхронизации времени:</li>
        <ul>
            <li>Используется сервер ISP.</li>
            <li>Сервер считает собственный источник времени верным, stratum=4;</li>
            <li>Сервер допускает подключение только через внешний адрес соответствующей платформы управления трафиком;</li>
            <ul>
                <li>Подразумевается обращение SRV для синхронизации времени;</li>
            </ul>
            <li>Клиент CLI должен использовать службу времени ISP;</li>
            <h4>ISP</h4>
            <pre>apt install -y chrony</pre>
            <pre>vi /etc/chrony/chrony.conf<br>
            local stratum 4
            allow 4.4.4.0/24
            allow 3.3.3.0/24</pre>
            <pre>systemctl restart chronyd</pre>
            <h4>CLI</h4>
            <pre>New-NetFirewallRule -DisplayName "NTP" -Direction Inbound -LocalPort 123 -Protocol UDP -Action Allow</pre>
            <pre>Start-Service W32Time<br>w32tm /config /manualpeerlist:4.4.4.1 /syncfromflags:manual /reliable:yes /update<br>Restart-Service W32Time</pre>
            <pre>Set-Service -Name W32Time -StartupType Automatic</pre>
        </ul>
    </ul>
    
<ul>
    <li>Выполните конфигурацию службы второго уровня времени на SRV.</li>
    <ul>
        <li>Сервер синхронизирует время с хостом ISP;</li>
        <ul>
            <li>Синхронизация с другими источникам запрещена;</li>
        </ul>
        <li>Сервер должен допускать обращения внутренних хостов регионов, в том числе и платформ управления трафиком, для синхронизации времени;</li>
        <li>Все внутренние хосты(в том числе и платформы управления трафиком) должны синхронизировать свое время с SRV;</li>
        <h4>SRV</h4>
        <pre>apt install -y chrony</pre>
        <pre>vi /etc/chrony/chrony.conf<br>
        #pool 2.debian.pool.ntp.org iburst
        pool 4.4.4.1 iburst
        allow 192.168.100.0/24
        allow 172.16.100.0/24
        </pre>
        <pre>systemctl restart chronyd</pre>
        <h4>WEB-L</h4>
        <pre>apt install -y chrony</pre>
        <pre>vi /etc/chrony/chrony.conf<br>
        #pool 2.debian.pool.ntp.org iburst
        pool ntp.int.demo.wsr iburst
        allow 192.168.100.0/24
        </pre>
        <pre>systemctl restart chronyd</pre>
        <h4>WEB-R</h4>
        <pre>apt install -y chrony</pre>
        <pre>vi /etc/chrony/chrony.conf<br>
        #pool 2.debian.pool.ntp.org iburst
        pool ntp.int.demo.wsr iburst
        allow 172.16.100.0/24
        </pre>
        <pre>systemctl restart chronyd</pre>
        <h4>RTR-L</h4>
        <pre>apt install -y chrony</pre>
        <pre>vi /etc/chrony/chrony.conf<br>
        #pool 2.debian.pool.ntp.org iburst
        pool ntp.int.demo.wsr iburst
        allow 192.168.100.0/24
        </pre>
        <pre>systemctl restart chronyd</pre>
        <h4>RTR-R</h4>
        <pre>apt install -y chrony</pre>
        <pre>vi /etc/chrony/chrony.conf<br>
        #pool 2.debian.pool.ntp.org iburst
        pool ntp.int.demo.wsr iburst
        allow 172.16.100.0/24
        </pre>
    </ul>
</ul>

<ul>
    <li>Реализуйте файловый SMB-сервер на базе SRV</li>
    <ul>
        <li>Сервер должен предоставлять доступ для обмена файлами серверам WEB-L и WEB-R;</li>
        <li>Сервер, в зависимости от ОС, использует следующие каталоги для хранения файлов:</li>
        <ul>
            <li>/mnt/storage для система на базе Linux;</li>
            <li>Диск R:\ для систем на базе Windows;</li>
        </ul>
        <li>Хранение файлов осуществляется на диске (смонтированном по указанным выше адресам), реализованном по технологии RAID типа “Зеркало”;</li>
        <h4>SRV</h4>
        <pre>apt install mdadm -y</pre>
        <pre>mdadm --zero-superblock --force /dev/sd{b,c}<br>wipefs --all --force /dev/sd{b,c}<br>mdadm --create --verbose /dev/md0 -l 1 -n 2 /dev/sd{b,c}<br>mkfs.ext4 /dev/md0</pre>
        <pre>mkdir /mnt/storage<br>chmod 777 /mnt/storage<br>echo /dev/md0 /mnt/storage ext4 defaults 1 2 >> /etc/fstab<br>mount -a</pre>
        <pre>apt install samba smbclient cifs-utils -y</pre>
        
<pre>vi /etc/samba/smb.conf</pre>
        
![Image alt](https://github.com/NewErr0r/39-WSI/blob/main/smb_conf.png?raw=true)
     
 <pre>systemctl restart smbd</pre>

 </ul></ul>
 
 <ul>
    <li>Сервера WEB-L и WEB-R должны использовать службу, настроенную на SRV, для обмена файлами между собой:</li>
    <ul>
        <li>Служба файлового обмена должна позволять монтирование в виде стандартного каталога Linux;</li>
        <ul>
            <li>Разделяемый каталог должен быть смонтирован по адресу /opt/share;</li>
        </ul>
        <li>Каталог должен позволять удалять и создавать файлы в нем для всех пользователей;</li>
        <h4>WEB-L</h4>
        <pre>apt install -y cifs-utils</pre>
        <pre>mkdir /opt/share<br>chmod 777 /opt/share<br>echo //srv.int.demo.wsr/storage /opt/share cifs guest 0 0 >> /etc/fstab<br>mount -a</pre>
        <h4>WEB-R</h4>
        <pre>apt install -y cifs-utils</pre>
        <pre>mkdir /opt/share<br>chmod 777 /opt/share<br>echo //srv.int.demo.wsr/storage /opt/share cifs guest 0 0 >> /etc/fstab<br>mount -a</pre>
    </ul>
</ul>
 
<ul>
    <li>Выполните настройку центра сертификации на базе SRV:</li>
    <ul>
        <li>В случае применения решения на базе Linux используется центр сертификации типа OpenSSL и располагается по адресу /var/ca;</li>
        <li>Выдаваемые сертификаты должны иметь срок жизни не менее 500 дней;</li>
        <li>Параметры выдаваемых сертификатов:</li>
        <ul>
            <li>Страна RU;</li>
            <li>Организация DEMO.WSR;</li>
            <li>Прочие поля (за исключением CN) должны быть пусты;</li>
        </ul>
        <h4>SRV</h4>
        <pre>mkdir /var/ca<br>cd /var/ca</pre>
        <pre>openssl req -newkey rsa:4096 -keyform PEM -keyout ca.key -x509 -days 3650 -outform PEM -out ca.cer</pre>
    
![Image alt](https://github.com/NewErr0r/39-WSI/blob/main/Root_CA.png?raw=true)
        
</ul></ul>

<h2>Инфраструктура веб-приложения</h2>

<p>Данный блок подразумевает установку и настройку доступа к веб-приложению, выполненному в формате контейнера Docker.</p>

<ul>
    <li>Образ Docker (содержащий веб-приложение) расположен на ISO-образе (WEB-L|R -  /root/AppDocker0) дополнительных материалов;</li>
    
![Image alt](https://github.com/NewErr0r/39-WSI/blob/main/AppDocker0.png?raw=true)    
    
<ul>
        <li>Выполните установку приложения AppDocker0;</li>
    </ul>
    <li>Пакеты для установки Docker расположены на дополнительном ISO-образе;</li>
    <li>Инструкция по работе с приложением расположена на дополнительном ISO-образе (WEB-L/R /root/AppDocker0);</li>
    <h4>WEB-L</h4>
    <pre>apt update<br>apt install ca-certificates curl gnupg lsb-release -y <br>curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg<br>echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null<br>apt update<br>apt install -y docker-ce<br>systemctl enable --now docker</pre>
    <pre>cd /root/AppDocker0<br>docker build -t app .<br>docker run --name app  -p 8080:80 -d app</pre>
    <h4>WEB-R</h4>
    <pre>apt update<br>apt install ca-certificates curl gnupg lsb-release -y <br>curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg<br>echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null<br>apt update<br>apt install -y docker-ce<br>systemctl enable --now docker</pre>
    <pre>cd /root/AppDocker0<br>docker build -t app .<br>docker run --name app  -p 8080:80 -d app</pre>
</ul>

<ul>
    <li>Необходимо реализовать следующую инфраструктуру приложения.</li>
    <ul>
        <li>Клиентом приложения является CLI (браузер Edge);</li>
        <li>Хостинг приложения осуществляется на ВМ WEB-L и WEB-R;</li>
        <li>Доступ к приложению осуществляется по DNS-имени www.demo.wsr;</li>
        <ul>
            <li>Имя должно разрешаться во “внешние” адреса ВМ управления трафиком в обоих регионах;</li>
            <li>При необходимости, для доступа к к приложению допускается реализовать реверс-прокси или трансляцию портов;</li>
        </ul>
        <li>Доступ к приложению должен быть защищен с применением технологии TLS;</li>
        <ul>
            <li>Необходимо обеспечить корректное доверие сертификату сайта, без применения “исключений” и подобных механизмов;</li>
        </ul>
        <li>Незащищенное соединение должно переводиться на защищенный канал автоматически;</li>
        </ul>
    <li>Необходимо обеспечить отказоустойчивость приложения;</li>
    <ul>
        <li>Сайт должен продолжать обслуживание (с задержкой не более 25 секунд) в следующих сценариях:</li>
        <ul>
            <li>Отказ одной из ВМ Web</li>
            <li>Отказ одной из ВМ управления трафиком.</li>
        </ul>
        <h4>RTR-L</h4>
        <pre>apt install nginx -y<br>systemctl enable --now nginx</pre>
        <pre>mkdir cert<br>cd cert<br>openssl genrsa -out app.key 4096<br>openssl req -new -key app.key -out app.req -sha256</pre>
        <h4>SRV</h4>
        <pre>cd /var/ca<br>scp root@192.168.100.254:/root/cert/app.req ./<br>openssl x509 -req -in app.req -CA ca.cer -CAkey ca.key -set_serial 100 -extentions app -days 1460 -outform PEM -out app.cer -sha256<br>scp app.cer root@192.168.100.254:/root/cert<br>rm -f app.</pre>
        <h4>RTR-R</h4>
        <pre>apt install nginx -y<br>systemctl enable --now nginx</pre>
        <pre>mkdir cert<br>cd cert<br>openssl genrsa -out app.key 4096<br>openssl req -new -key app.key -out app.req -sha256</pre>
        <pre>vi /etc/ssh/sshd_config<br>...<br>PermitRootLogin yes<br>...</pre>
        <pre>systemctl restart sshd</pre>
        <h4>SRV</h4>
        <pre>cd /var/ca<br>scp root@172.16.100.254:/root/cert/app.req ./<br>openssl x509 -req -in app.req -CA ca.cer -CAkey ca.key -set_serial 100 -extentions app -days 1460 -outform PEM -out app.cer -sha256<br>scp app.cer root@172.16.100.254:/root/cert<br>rm -f app.</pre>
        <h4>RTR-L</h4>
        <pre>mkdir -p /etc/ssl/nginx/private<br>cp app.cer /etc/pki/nginx/<br>cp app.key /etc/pki/nginx/private</pre>
        <pre>vi /etc/nginx/sites-available.d/proxy.conf</pre>
        
![Image alt](https://github.com/NewErr0r/39-WSI/blob/main/proxy_l.png?raw=true)        
             
    ln -s /etc/nginx/sites-available.d/proxy.conf /etc/nginx/sites-enabled.d/proxy.conf
    nginx -t
    systemctl restart nginx 
        
 <h4>RTR-R</h4>
 <pre>mkdir -p /etc/ssl/nginx/private<br>cp app.cer /etc/pki/nginx/<br>cp app.key /etc/pki/nginx/private</pre>
 <pre>vi /etc/nginx/sites-available.d/proxy.conf</pre>
        
![Image alt](https://github.com/NewErr0r/39-WSI/blob/main/proxy2.png?raw=true)        
        
    ln -s /etc/nginx/sites-available.d/proxy.conf /etc/nginx/sites-enabled.d/proxy.conf
    nginx -t
    systemctl restart nginx 
        
  <h4>SRV</h4>
        <pre>cp /var/ca/ca.cer /mnt/storage</pre>
  <h4>CLI</h4>
        <pre>scp -P 2222 'root@4.4.4.100:/opt/share/ca.cer' C:\Users\user\Desktop\</pre>
        <pre>Import-Certificate -FilePath "C:\Users\user\Desktop\ca.cer" -CertStoreLocation cert:\CurrentUser\Root</pre>
  
![Image alt](https://github.com/NewErr0r/39-WSI/blob/main/site.png?raw=true)
        
  </ul>    
</ul>
