# sieci

```
1. Wyłączanie interfejsu eth0:

sudo ifdown eth0

2. Konfigurowanie interfejsu eth2 z adresem IP 192.168.g.k1/24:

sudo ifconfig eth2 192.168.g.k1 netmask 255.255.255.0 up

3. Konfigurowanie routingu domyślnego przez router 192.168.g.k2:

sudo route add default gw 192.168.g.k2

4. Udostępnianie dostępu zdalnego przez SSH:

sudo apt-get install openssh-server

SSH powinno automatycznie startować po instalacji. Możesz sprawdzić status SSH:

sudo service ssh status

5. ontrolowanie dostępu do internetu, routera, komputerów w sąsiednich grupach:

Sprawdź połączenie z internetem:
ping google.com
Sprawdź dostęp do routera:
ping 192.168.g.k2
Sprawdź dostęp do komputerów w sąsiednich grupach.
ping 192.168.g'.k'


1. Konfigurowanie interfejsu eth2 z adresem IP 192.168.g.k2/24:

sudo ifconfig eth2 192.168.g.k2 netmask 255.255.255.0 up

2. Udostępnianie połączenia internetowego dla komputerów z sieci 192.168.g.0/24 (SNAT lub MASQUERADE):

sudo iptables -t nat -A POSTROUTING -s 192.168.g.0/24 -o ethX -j MASQUERADE

3. Zapewnianie dostępu do usługi SSH na komputerze 192.168.g.k1 (DNAT):

sudo iptables -t nat -A PREROUTING -i eth2 -p tcp --dport 22 -j DNAT --to-destination 192.168.g.k1:22

4. Zabronić komputerom z sieci 192.166.g.0/24 dostępu do wybranych serwisów internetowych:

sudo iptables -A FORWARD -s 192.166.g.0/24 -p tcp --dport 80 -j DROP
sudo iptables -A FORWARD -s 192.166.g.0/24 -p tcp --dport 443 -j DROP

5. Udostępnianie dostępu zdalnego do routera przez SSH wyłącznie z sieci wewnętrznej 192.168.g.0/24:

sudo iptables -A INPUT -i eth2 -p tcp --dport 22 -s 192.168.g.0/24 -j ACCEPT

6. Uruchomienie przekazywania datagramów (routingu) IPv4:

echo 1 > /proc/sys/net/ipv4/ip_forward
lub
sysctl -w net.ipv4.ip_forward=1

To umożliwia routowaniu pakietów między interfejsami.
```

```
1.Utworzenie pliku konfiguracyjnego fw.conf z nagłówkiem i instrukcją flush:
echo "# fw.conf" > fw.conf
echo "nft flush ruleset" >> fw.conf

2. Dodanie reguł do pliku fw.conf:
# fw.conf
nft flush ruleset

table inet my_filter {
    chain input {
        type filter hook input priority 0; policy drop;
        # Dodaj swoje reguły dla chain input
    }

    chain forward {
        type filter hook forward priority 0; policy drop;
        # Dodaj swoje reguły dla chain forward
    }

    chain output {
        type filter hook output priority 0; policy drop;
        # Dodaj swoje reguły dla chain output
    }
}
3. Wprowadzenie nowych reguł do nftables:

sudo nft -f fw.conf

4.3.1 (counter)
# fw.conf
nft flush ruleset

table inet my_filter {
    counter input_counter
    counter output_counter
    counter reject_counter

    chain input {
        type filter hook input priority 0; policy drop;
        counter packets input_counter bytes
        # Dodaj swoje reguły dla chain input
    }

    chain forward {
        type filter hook forward priority 0; policy drop;
        counter packets output_counter bytes
        # Dodaj swoje reguły dla chain forward
    }

    chain output {
        type filter hook output priority 0; policy drop;
        counter packets output_counter bytes
        # Dodaj swoje reguły dla chain output
    }

    chain reject {
        type filter hook input priority 0; policy drop;
        counter packets reject_counter bytes
        # Dodaj swoje reguły dla chain reject
    }
}
4.3.2

# fw.conf
nft flush ruleset

table inet my_filter {
    counter input_counter
    counter output_counter
    counter reject_counter

    chain input {
        type filter hook input priority 0; policy drop;
        counter packets input_counter bytes

        # Zezwól na przychodzące datagramy ping tylko z hostów sieci własnej grupy
        ip protocol icmp icmp type echo-request accept
        ip saddr 192.168.G.0/24 icmp type echo-request accept
    }

    chain forward {
        type filter hook forward priority 0; policy drop;
        counter packets output_counter bytes
        # Dodaj swoje reguły dla chain forward
    }

    chain output {
        type filter hook output priority 0; policy drop;
        counter packets output_counter bytes
        # Dodaj swoje reguły dla chain output
    }

    chain reject {
        type filter hook input priority 0; policy drop;
        counter packets reject_counter bytes
        # Dodaj swoje reguły dla chain reject
    }
}
4.3.3
# fw.conf
nft flush ruleset

table inet my_filter {
    counter input_counter
    counter output_counter
    counter reject_counter

    chain input {
        type filter hook input priority 0; policy drop;
        counter packets input_counter bytes

        # Zezwól na przychodzące nowe połączenia WWW (porty TCP 80 i 443)
        tcp dport {80, 443} ct state new accept
    }

    chain forward {
        type filter hook forward priority 0; policy drop;
        counter packets output_counter bytes
        # Dodaj swoje reguły dla chain forward
    }

    chain output {
        type filter hook output priority 0; policy drop;
        counter packets output_counter bytes
        # Dodaj swoje reguły dla chain output
    }

    chain reject {
        type filter hook input priority 0; policy drop;
        counter packets reject_counter bytes
        # Dodaj swoje reguły dla chain reject
    }
}

4.3.4 i 4.3.5

# fw.conf
nft flush ruleset

table inet my_filter {
    counter input_counter
    counter output_counter
    counter reject_counter
    counter kti_gda_pl_counter

    chain input {
        type filter hook input priority 0; policy drop;
        counter packets input_counter bytes
        # Dodaj swoje reguły dla chain input
    }

    chain forward {
        type filter hook forward priority 0; policy drop;
        counter packets output_counter bytes
        # Dodaj swoje reguły dla chain forward
    }

    chain output {
        type filter hook output priority 0; policy drop;
        counter packets output_counter bytes

        # Dodaj łańcuch "outgoing" z punktem przechowywania output i polityką accept
        jump outgoing
    }

    chain reject {
        type filter hook input priority 0; policy drop;
        counter packets reject_counter bytes
        # Dodaj swoje reguły dla chain reject
    }

    # Dodaj łańcuch "outgoing" z punktem przechowywania output i polityką accept
    chain outgoing {
        type filter hook output priority 0; policy accept;
        counter packets output_counter bytes

        # Zakaz korzystania z serwisu kti.gda.pl z licznikiem prób połączenia
        tcp dport {80, 443} ip daddr kti.gda.pl drop
        counter packets kti_gda_pl_counter bytes
    }
}

5.1.1
# fw.conf
nft flush ruleset

table inet my_filter {
    counter input_counter
    counter output_counter
    counter reject_counter

    chain input {
        type filter hook input priority 0; policy drop;
        counter packets input_counter bytes
        # Dodaj swoje reguły dla chain input
    }

    chain forward {
        type filter hook forward priority 0; policy drop;
        counter packets output_counter bytes

        # Współdzielenie łącza internetowego dla całej sieci lokalnej 192.168.G.0/24
        ip saddr 192.168.G.0/24 oifname "eth0" accept
    }

    chain output {
        type filter hook output priority 0; policy drop;
        counter packets output_counter bytes
        # Dodaj swoje reguły dla chain output
    }

    chain reject {
        type filter hook input priority 0; policy drop;
        counter packets reject_counter bytes
        # Dodaj swoje reguły dla chain reject
    }
}

table inet my_nat {
    chain postrouting {
        type nat hook postrouting priority 0; policy accept;

        # Masquerade dla współdzielenia łącza internetowego
        ip saddr 192.168.G.0/24 oifname "eth0" masquerade
    }
}
5.1.2
# fw.conf
nft flush ruleset

table inet my_filter {
    counter input_counter
    counter output_counter
    counter reject_counter

    chain input {
        type filter hook input priority 0; policy drop;
        counter packets input_counter bytes
        # Dodaj swoje reguły dla chain input
    }

    chain forward {
        type filter hook forward priority 0; policy drop;
        counter packets output_counter bytes

        # Współdzielenie łącza internetowego dla całej sieci lokalnej FD00:G::/64
        ip6 saddr fd00:G::/64 oifname "eth0" accept
    }

    chain output {
        type filter hook output priority 0; policy drop;
        counter packets output_counter bytes
        # Dodaj swoje reguły dla chain output
    }

    chain reject {
        type filter hook input priority 0; policy drop;
        counter packets reject_counter bytes
        # Dodaj swoje reguły dla chain reject
    }
}

table inet my_nat {
    chain postrouting {
        type nat hook postrouting priority 0; policy accept;

        # NPTv6 dla współdzielenia łącza internetowego
        ip6 saddr fd00:G::/64 oifname "eth0" npt 2001:db8:1::/48
    }
}
5.2.1 .2 .3
# fw.conf
nft flush ruleset

table inet my_filter {
    counter input_counter
    counter output_counter
    counter reject_counter

    chain input {
        type filter hook input priority 0; policy drop;
        counter packets input_counter bytes
        # Dodaj swoje reguły dla chain input
    }

    chain forwarding {
        type filter hook forward priority 0; policy drop;
        counter packets output_counter bytes

        # Punkt przechwytywania forward
        jump forward
    }

    chain output {
        type filter hook output priority 0; policy drop;
        counter packets output_counter bytes
        # Dodaj swoje reguły dla chain output
    }

    chain reject {
        type filter hook input priority 0; policy drop;
        counter packets reject_counter bytes
        # Dodaj swoje reguły dla chain reject
    }
}

table inet my_forward {
    chain forward {
        type filter hook forward priority 0; policy drop;
        counter packets output_counter bytes

        # Zapewnienie nieograniczonego dostępu do Internetu dla hostów z sieci własnej grupy (stateful)
        ip saddr 192.168.G.0/24 ip6 saddr fd00:G::/64 ct state established,related accept
    }
}

table inet my_nat {
    chain prerouting {
        type nat hook prerouting priority 0; policy accept;
    }

    chain postrouting {
        type nat hook postrouting priority 0; policy accept;
    }
}


5.3
# fw.conf
nft flush ruleset

table inet my_filter {
    counter input_counter
    counter output_counter
    counter reject_counter

    chain input {
        type filter hook input priority 0; policy drop;
        counter packets input_counter bytes
        # Dodaj swoje reguły dla chain input
    }

    chain forwarding {
        type filter hook forward priority 0; policy drop;
        counter packets output_counter bytes
        # Dodaj swoje reguły dla chain forwarding
    }

    chain output {
        type filter hook output priority 0; policy drop;
        counter packets output_counter bytes
        # Dodaj swoje reguły dla chain output
    }

    chain reject {
        type filter hook input priority 0; policy drop;
        counter packets reject_counter bytes
        # Dodaj swoje reguły dla chain reject
    }
}

table inet my_nat {
    chain prerouting {
        type nat hook prerouting priority 0; policy accept;

        # Przekierowanie przychodzących połączeń z portu 2022 routera K1 na port 22 hosta K2
        tcp dport 2022 dnat to :22
    }

    chain postrouting {
        type nat hook postrouting priority 0; policy accept;
    }
}
```
