# grafana-compose

Grafana, Loki, Syslog-ng Logging service.

Ingress control:

- I use HAProxy as ingress gateway to Grafana (3000/tcp) and Loki (3100/tcp) services
- Syslog-ng has it's own filters

```json
filter f_network {
    match("^10\\.3\\.(1[0-9]|20)\\." value("SOURCEIP")) or
    match("^172\\.16\\.1([0-4][0-9]|50)\\." value("SOURCEIP"));
};
```

Clone repository and initialize systemd service.

```bash
git clone https://github.com/artyomtsybulkin/grafana-compose.git \
    /var/mnt/compose
sudo ln -s /var/mnt/compose/podman-compose.service \
    /etc/systemd/system/podman-compose.service
sudo systemctl enable --now podman-compose.service
sudo systemctl daemon-reload
```

Compose file validation

```bash
podman compose -f podman-compose.yaml config
```

Testing syslog delivery using `nc`.

```bash
echo "<134>Debug test syslog by UDP" | nc -v -u -w1 <syslog_server_ip> 514
echo "<134>Debug test syslog by TCP" | nc -v <syslog_server_ip> 514
```

Testing syslog delivery using `powershell`.

```powershell
$SyslogServer = "172.23.32.135"   # Change to your syslog server IP
$SyslogPort = 514
$Message = "<134>$(Get-Date -Format 'MMM dd HH:mm:ss') $(hostname) Test syslog tcp+udp message from PowerShell"

# UDP
$UdpClient = New-Object System.Net.Sockets.UdpClient
$UdpClient.Connect($SyslogServer, $SyslogPort)
$Bytes = [System.Text.Encoding]::ASCII.GetBytes($Message)
$UdpClient.Send($Bytes, $Bytes.Length)
$UdpClient.Close()

# TCP
$client = New-Object System.Net.Sockets.TcpClient
$client.Connect($SyslogServer, $SyslogPort)
$stream = $client.GetStream()
$writer = New-Object System.IO.StreamWriter($stream)
$writer.AutoFlush = $true
$writer.WriteLine($Message)
$writer.Close()
$client.Close()

Write-Output "Message sent to $SyslogServer"
```
