Welcome to ReadMe.txt ReadMeAVL.txt for AVLMonitoring tools.

Disclaimer:  Use these tools at your own choosing and risk.

AVLMonitor.txt is provided if your organization is uncomfortable with transfering in .ps1 files.

AVLMonitor tools provide for crude website, web application,machine,interface, file, service or process monitoring and centralized notification on  issues:
      Can be run as a HeartBeat?
      Can be run as a PeriodChk with notificatino only if action required?
      Can also have alternate manifest to check for ad hoc, failure debugging or focus check situations?

These watch functions can be programmed into ShortCuts or ScheduledTasks like: 
%systemroot%\System32\WindowsPowerShell\v1.0\powershell.exe -executionpolicy Bypass -Windowstyle Hidden -file "<FullPath>\AVLmonitor.ps1" HEARTBEAT
	Recommended for this would be to run once every 12 or 24 hours.
%systemroot%\System32\WindowsPowerShell\v1.0\powershell.exe -executionpolicy Bypass -Windowstyle Hidden -file "<FullPath>\AVLmonitor.ps1" PERIODCHECK
	Recommended for this would be to run once every 1 or 2 hours.

%systemroot%\System32\WindowsPowerShell\v1.0\powershell.exe -executionpolicy Bypass -Windowstyle Hidden -file "<FullPath>\AVLmonitor.ps1" HEARTBEAT AcitveDirChecks.csv|ExchangeChecks.csv|...
	Recommend that more in depth specific debugging profiles can be set as ShortCuts for immediate adhoc usage.

Configuration of your email SMTP connection is required in the file MonitorNotice.csv
Provision of your email credential may be required and can go into the MonitorMailP.txt file.  DO NOT use PlainTEXT.  CMS encryption or at least securestring should be considered.

You will see a HashProfile<Date-Time>.txt that has Hash value confirmations of the untampered file in this project.
	Independantly you will see this same file published at https://web.ncf.ca/bv178/HashChecks.html
Hash Confirmations only provide comfort to mitigate tampering.