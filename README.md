Ubuntu , Debian iptables config

iptables OPTION

  --append  -A chain            Append to chain
  --check   -C chain            Check for the existence of a rule
  --delete  -D chain            Delete matching rule from chain
  --delete  -D chain rulenum
                                Delete rule rulenum (1 = first) from chain
  --insert  -I chain [rulenum]
                                Insert in chain as rulenum (default 1=first)
  --replace -R chain rulenum
                                Replace rule rulenum (1 = first) in chain
  --list    -L [chain [rulenum]]
                                List the rules in a chain or all chains
  --list-rules -S [chain [rulenum]]
                                Print the rules in a chain or all chains
  --flush   -F [chain]          Delete all rules in  chain or all chains
  --zero    -Z [chain [rulenum]]
                                Zero counters in chain or all chains
  --new     -N chain            Create a new user-defined chain
  --delete-chain
            -X [chain]          Delete a user-defined chain
  --policy  -P chain target
                                Change policy on chain to target
  --rename-chain
            -E old-chain new-chain
                                Change chain name, (moving any references)
Options:
    --ipv4      -4              Nothing (line is ignored by ip6tables-restore)
    --ipv6      -6              Error (line is ignored by iptables-restore)
[!] --protocol  -p proto        protocol: by number or name, eg. `tcp'
[!] --source    -s address[/mask][...]
                                source specification
[!] --destination -d address[/mask][...]
                                destination specification
[!] --in-interface -i input name[+]
                                network interface name ([+] for wildcard)
 --jump -j target
                                target for rule (may load target extension)
  --goto      -g chain
                              jump to chain with no return
  --match       -m match
                                extended match (may load extension)
  --numeric     -n              numeric output of addresses and ports
[!] --out-interface -o output name[+]
                                network interface name ([+] for wildcard)
  --table       -t table        table to manipulate (default: `filter')
  --verbose     -v              verbose mode
  --wait        -w [seconds]    maximum wait to acquire xtables lock before give up
  --wait-interval -W [usecs]    wait time to try to acquire xtables lock
                                default is 1 second
  --line-numbers                print line numbers when listing
  --exact       -x              expand numbers (display exact values)
[!] --fragment  -f              match second or further fragments only
  --modprobe=<command>          try to insert modules using this command
  --set-counters PKTS BYTES     set the counter during insert/append
[!] --version   -V              print package version.

1. iptable Filter

Chain
INPUT : 서버로 들어오는 것들을 정의하는 정책
FORWARD : 서버를 통해서 나가는 , FORWARDING 되어 나가는 것들을 정의하는 정책
OUTPUT : 서버에서 나가는 것들을 정의하는 정책

Example )
접근제어
iptable -A INPUT -s 192.168.0.0/24 -j DROP // 출발지 주소가 192.168.0.0 대역을 가진 IP주소들은 서버로 접근이 불가능함 , Ping, Service 등 연결이 제한됨
iptables -A FORWARD -s 10.10.10.0/24 -i enp0s25 -j ACCEPT // 출발지 주소가 10.10.10.0 대역인 IP 주소들은 enp0s25 인터페이스를 통해 지나갈 수 있음
iptables -A OUTPUT -s 192.168.0.100 -d 8.8.8.8 -j ACCEPT // 출발지 주소가 192.168.0.100 IP일 경우 8.8.8.8 주소로 가는 것들을 허용

port 제어
iptable -A INPUT -s 192.168.0.0/24 -p udp --dport 53 -j DROP // 출발지가 192.168.0.0/24인 대역에서 udp 53번 포트로 요청이 올 경우 DROP 시키는 정책

-p tcp --dport 80  // -p udp --dport 53 등 다양하게 포트 제어 가능

2. iptable NAT

Chain
OUTPUT : 서버에서 나가는 패킷에 대한 정책
PREROUTING(DNAT) : 서버로 들어오는 패킷의 주소를 변경
POSTROUTING(SNAT) : 서버에서 나가는 패킷의 주소를 변경
Local IP 192.168.0.100 -> Linux Server IP 152.43.23.10 -> 외부 순서로 패킷이 전송될 경우 Local의 IP가 Linux Server를 거치면서 152.43.23.10의 주소로 외부와 통신하여 데이터를 주고받음

Example )
iptables -t nat -A PREROUTING -s 10.10.10.0/24 -p tcp -i enp0s25 --dport 80 -j DNAT --to 192.168.0.100 // 10.10.10.0 대역의 IP에서 tcp 연결 요청이 올 경우 192.168.0.100의 80번 포트로 포트포워딩 되도록 설정

iptables -t nat -A POSTROUTING -s 192.168.2.0/24 -o enp0s25 -j MASQUERADE // 192.168.2.0/24 대역의 IP는 enp0s25 인터페이스 주소로 NAT되어 외부와 통신되도록 설정 MASQUERADE가 외부와 연결이 가능하도록 설정해주는 기능


iptables-save or iptables -L // 정책 설정 확인

iptables-persistent 패키지 사용 시 /etc/iptables/rule.v4,6 에 추가됨
dpkg-reconfigure iptables-persistent로 저장
이 후에 재부팅 하여도 설정한 정책들이 적용됨

