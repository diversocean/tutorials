# WSL2, 외부 네트워크와 연결하기!

## Point
- wsl2 주소로 ssh 연결이 외부에서 안되요!

## Solve
- 포트포워딩

**in wsl2**
```bash
$ ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.x.x.x
```
  
**in PowerShell**
```bash
$ ipconfig
IPv4 주소 . . . . . . . . . : 192.168.0.x
```

## Port Forward
local external ip -> local internal ip -> wsl adapter ip
  
  
**in wsl2**  
```bash  
$ touch wsl-connect-external.ps1  
```  
  
**wsl-connect-external.ps1**  
```
$remoteport = bash.exe -c "ifconfig eth0 | grep 'inet '"
$found = $remoteport -match '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}';

if( $found ){
  $remoteport = $matches[0];
} else{
  echo "The Script Exited, the ip address of WSL 2 cannot be found";
  exit;
}

#[Ports]

#All the ports you want to forward separated by coma
$ports=@(22,80,443,10000,3000,5000);


#[Static ip]
#You can change the addr to your ip config to listen to a specific address
$addr='0.0.0.0';
$ports_a = $ports -join ",";


#Remove Firewall Exception Rules
iex "Remove-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' ";

#adding Exception Rules for inbound and outbound Rules
iex "New-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' -Direction Outbound -LocalPort $ports_a -Action Allow -Protocol TCP";
iex "New-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' -Direction Inbound -LocalPort $ports_a -Action Allow -Protocol TCP";

for( $i = 0; $i -lt $ports.length; $i++ ){
  $port = $ports[$i];
  iex "netsh interface portproxy delete v4tov4 listenport=$port listenaddress=$addr";
  iex "netsh interface portproxy add v4tov4 listenport=$port listenaddress=$addr connectport=$port connectaddress=$remoteport";
}
```  
  
  
### Troubleshooting  
Windows를 재시작 할때마다 WSL2의 IP는 변경되기에  
매번 이 작업을 수행하기 매우 번거롭다. 더군다나 포트마다 설정해줘야 한다.  
이를 아래 커맨드를 통해 쉽게 해결 할 수 있었다.  
  
**in PowerShell(administrator)**  
```
> PowerShell.exe -ExecutionPolicy Bypass -File .\wsl-forward-server.ps1
```
  
  
여기까지 되었다면 PowerShell이나 cmd에서 ipconfig를 입력했을 때 나타나는  
ipv4 주소로 wsl어댑터 주소가 포트포워딩 되었을것이다.  
이제 내부 ipv4 주소를 공유기 외부IP, 포트로 포트포워딩 해주면 끗  
