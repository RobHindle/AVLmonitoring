# AVL Monitoring of many types of Capabilities
<# put list of monitoring items into MonitorList.csv
    Target,Type,Port,Credential,Description,ExpectedState
   put your Email needs in MonitoringNotice.csv
   if necessaary you can put Credential to get to your SMTP is Monitor MailP.csv

   No Args is Manually prompting
   Args0 is PROMPT, HEARTBEAT or PERIODCHECK
   Args1 is Monitorlist.csv default or what you state
   Args2 is integer 1 to N which represent hours within which a random start of process

    Machine = PING
    Machine = Core Connections (PING, RDP, SMB, HTTP, WINRM) = Test-Connection
    Machine ports = Protocol and Port# = Test-NetConnection
    Machine Folders and Files = Test-Path
    WebSites = Invoke-WebRequest
    Service = Session + Get-Service
    Processes = Session + Get-Process
    SchedTask = Session + Get-ScheduledTask
    #>

Function Monitor-AVLmany {
    Param( $tdy,
           $basepath = $PSScriptRoot,
           $srcfile  = "MonitorList.csv",
           $outmedia = "FILE",
           $outfile,
           $outScope = "ACTION",
           $outnote  = "NO"
           )
          if ($tdy.length -le 0) {$tdy = get-date -Format "yyyyMMddHHmm"}
          if ($outfile.length -le 0) {$outfile= "$basepath\Results\AVLmonitor$tdy.txt"}

$body = @()

########### Read in the targets to test
$targets = Import-Csv -Delimiter "," -Path "$basepath\$srcfile" 

########### Target testing done here by Type
Foreach ($target in $targets){
  $testrslt = "Negative"
  #$target
  #$target.testtype
  if (($($target.TestType) -eq "PING") -or ($($target.TestType) -eq "machine")){
    $rslt = Test-Connection -ComputerName $target.target -Count 1 -ErrorAction SilentlyContinue
    if ($($rslt.statuscode) -eq 0) {$testrslt = "Positive"}
  }
  elseif (($target.TestType -eq "RDP") -or ($target.TestType -eq "SMB") -or ($target.TestType -eq "WINRM") -or ($target.TestType -eq "HTTP")){
    $rslt = Test-NetConnection -ComputerName $target.target -CommonTCPPort $target.TestType -InformationLevel Detailed -ErrorAction SilentlyContinue
    if ($rslt.TCPTestSucceeded -eq "True") {$testrslt = "Positive"}
  }
  elseif ($target.TestType -eq "PORT") {
    $rslt = Test-NetConnection -ComputerName $target.target -Port $target.port -InformationLevel Detailed
    if ($rslt.TCPTestSucceeded -eq "True") {$testrslt = "Positive"}
  }
  elseif ($target.TestType -eq "PATH") {
    if ($target.Credential -eq "None") {
    $rslt = Test-Path -Path $target.target
    }
    else {
       #get $cred please 
    $rslt = Test-Path -Path $target.target -Credential $cred
    }
    if ($rslt -eq "True") {$testrslt = "Positive"}
  }
  elseif ($target.TestType -eq "WEB") {
    $rslt = Invoke-WebRequest -Uri $target.target -UseDefaultCredentials -DisableKeepAlive 
    if ($rslt.statusCode -eq "200") {$testrslt = "Positive"}
  }
  elseif ($target.TestType -eq "SERVICE") {
     $Tpieces = $target.target.split("&")
     $rslt = Get-Service -Name $Tpieces[1] -ComputerName $Tpieces[0] -ErrorAction SilentlyContinue
     #$rslt = get-wmiobject -Class Win32_Service -ComputerName $tpieces[0] -Credential $credSVC 
     if ($rslt.status -eq "Running") {$testrslt = "Positive"}
  }
  elseif ($target.TestType -eq "PROCESS") {
     $Tpieces = $target.target.split("&")
     $rslt = Get-Process -Name $Tpieces[1]  -ErrorAction SilentlyContinue
     #$rslt = Get-Process -Name $Tpieces[1] -ComputerName $Tpieces[0] -ErrorAction SilentlyContinue
     #$rslt = Invoke-Command -ComputerName $Tpieces[0] -Credential $credPROC -ScriptBlock {Get-Process -Name $Tpieces[1]} 
     if ($rslt.id.count -le $Tpieces[2]) {$testrslt = "Positive"}
  }
  elseif ($target.TestType -eq "SCHEDTASK") {     
     $Tpieces = $target.target.split("&")
     $rslt = "Stopped"
     #$tpath = "\\$($Tpieces[0])\c$\Windows\System32\Tasks"
     #$tasks = get-childitem -Recurse -Path $tpath -File
     $GSTs = Get-ScheduledTask -TaskName "Remove Transient Files"
     $GSTIs = Get-ScheduledTaskInfo -TaskName "Remove Transient Files"
       "$($GSTs.taskname),$($Gsts.description),$($Gsts.date), $($Gsts.state)"
       "$($GSTIs.taskname),$($GSTIs.LastRunTime),$($GSTIs.NextRunTime)"
       if (($GSTS.state -eq "Ready") -or ($GSTS.state -eq "Running")) {$rslt = "Running"}
     #
     <#
     foreach ($task in $tasks) {
        if ($task.name -eq $tpieces[1]) { $rslt = "Running" #}
        "$($TPieces[0]),$($task.name),$($task.fullname)"
        $temp = [xml](get-content -Path $task.fullname)
        $temp.Task.RegistrationInfo.Description
        $temp.Task.Actions.Exec
        $temp.Task.Principals.Principal
        $temp.Task.Settings.Enabled
        $temp.Task.Triggers.CalendarTrigger.Enabled
        $temp.Task.Triggers.CalendarTrigger.Repetition
        }
     } # each task in tasks
     #>
     #$rslt = Get-ScheduledTask -TaskName $Tpieces[1] -TaskPath $Tpieces[0] 
     if ($rslt.status -eq "Running") {$testrslt = "Positive"}
  }
  
  ########### Expected and Outputs
  $expected = "No"
  If ($testrslt -eq $target.ExpState) { $expected ="Yes" }
  $output = "No"
  if (($outscope -eq "ALL") -or ($expected -eq "No")) { $output = "Yes" }

  ########### Build up the Body of Results
  if ($output -eq "Yes") {
  $body += "$testrslt,$($target.target),$($target.TestType),$($Target.metadata)"
  }

} # for each target line

########## Build Output Header
$HDR = @()
$HDR += " Monitoring for $tdy"
$HDR += "*****************************"
$HDR += "$outScope"
$HDR += "Result, Target, Test, Details"

######### Output the results
#$outmedia
if (($outmedia -eq "SCREEN") -or ($outmedia -eq "BOTH")) {
   $outfile
   $HDR
   $body
   }
if (($outmedia -eq "FILE") -or ($outmedia -eq "BOTH") -or ($outNote -eq "YES") -or ($outNote -eq "YESIF")) {
   $HDR >> $outfile
   $body >> $outfile
   }

######### Send Notifications
if (($outnote -eq "YES") -or (($outNote -eq "YESIF" ) -and ($body.length -gt 0 ))) {
   $ICSV = Import-Csv -Delimiter "," -Path "$PSScriptRoot\MonitorNotice.csv" 

   $PSEmailServer = $ICSV.Server 
   $EmailID  = $ICSV.EmailID 
   $ToList   = $ICSV.To  
   $FromList = $ICSV.From
   $CCList   = $ICSV.CC
   $Port     = $ICSV.Port
   $Cert     = $ICSV.Cert
   #"$PSEmailServer,$ToList,$FromList,$Port,$EmailID"

   #NOTE:  Internal SMTP servers may not require any ID Password to receive a necessary Connection 
   $userPassword = UNProtect-CMSmessage -To "$Cert" -Path "$PSScriptRoot\MonitorMailP.txt" 
   [securestring]$secStringPassword = ConvertTo-SecureString $userPassword -AsPlainText -Force
   [pscredential]$credObject = New-Object System.Management.Automation.PSCredential ($EmailID, $secStringPassword)

   If ($CCList.length -le 0) {
      Send-MailMessage -Attachments $outfile  -Subject "Monitoring for $tdy" `
         -From $FromList `
         -To $tolist -Body "Action on attached Results of 'Negative' needed !!!"`
         -SmtpServer $PSEmailServer  -Port $Port -Credential $credObject -UseSsl
      }
      Else {
       Send-MailMessage -Attachments $outfile  -Subject "Monitoring for $tdy" `
         -From $FromList `
         -To $tolist -Body "Action on attached Results of 'Negative' needed !!!"`
         -Cc $cclist `
         -SmtpServer $PSEmailServer  -Port $Port -Credential $credObject -UseSsl
      }
}
} #Function Monitor-AVLmany

############# MAIN ###################
# If provided with Parameter Arguments then it processes directly from those indicators
# If no Arguments then prompts are triggered.
$paramvals = @("PROMPT","HEARTBEAT","PERIODCHECK")
$tdy = get-date -Format "yyyyMMddHHmm"

$basepath = $PSscriptRoot

# if there is an arguement is it in accepted list
if(($args.count -gt 0) -and ($paramvals -contains $args[0])) { 
$paramsource = $args[0]
  # if 2 orr more arguements the take monitorlist as the 2ns agruement
  if ($args.count -gt 1) {
  $srcfile = $args[1]
  if (test-path -Path $PSscriptRoot\$srcfile) {}
     else {
     $srcfile = "MonitorList.csv"
     }
  }
  else {
    $srcfile = "MonitorList.csv"
  }
}
else {
$paramsource = "PROMPT" # or HEARTBEAT or PERIODCHECK
$srcfile = "MonitorList.csv"
}

if($paramsource -eq "PROMPT") {
############ Get the Prompts for format  Screen output,All states,No Notification
$outmedia = read-host -Prompt "SCREEN, FILE or BOTH"
if ($outmedia -eq "") {$outmedia = "Screen"}
$outscope = read-host -Prompt "ACTION items or ALL"
if ($outscope -eq "") {$outscope = "ALL"}
$outnote = read-host -Prompt "Notify on YES, YESIF actions NO"
if ($outNote -eq "") {$outnote = "NO"}
}
elseif($paramsource -eq "HEARTBEAT") { #Heartbeat - 1-2x/day - All and always report
$outmedia = "FILE"
$outScope = "ALL"
$outnote = "YES"                   
}
elseif($paramsource -eq "PERIODCHECK") { #PeriodChk - Hours when no Heartbeat and only show if something is amiss
$outmedia = "FILE"                     
$outScope = "ACTION"
$outnote = "YESIF"                    
}

if (($outmedia -eq "FILE") -or ($outmedia -eq "BOTH")) {
   $outfile = "$basepath\Results\AVLmonitor$tdy.txt"
}

if ($args[2].length -gt 0) {
      $seed = Get-Date -Format "HHss"
      $maxSeconds = $($([int]$args[2] * 3600)-300) # (Number hours  * 3600 s/hr) less 5 minutes
      $SSeconds = Get-Random -Maximum $maxSeconds -Minimum 180 -SetSeed $seed  # minimum 3 minute wait
   }
   else {
     $SSeconds = 0
   }

Start-Sleep -Seconds $SSeconds  # add some randomness to when review actually done

Monitor-AVLmany -srcfile $srcfile -outScope $outscope -outmedia $Outmedia -outfile $outfile -outnote $outnote -basepath $basepath


######################## TEST BED ###################################
# you might have to do something like this if you need credentials for your SMTP connection
If ($TestParm -eq "makeCSMHolding"){
   $ICSV = Import-Csv -Delimiter "," -Path "$PSScriptRoot\MonitorNotice.csv" 
   #$ICSV = Import-Csv -Delimiter "," -Path "c:\users\owner\documents\WinPwrShlPractice\OPSTestURLS\MonitorNotice.csv" 
   $Cert     = $ICSV.Cert

   $pswd = read-host -Prompt "Password" -AsSecureString
   Protect-CMSmessage -content $pswd -to "$Cert" -OutFile "$PSScriptRoot\MonitorMailP.txt" 
   #Protect-CMSmessage -content $pswd -to "$Cert" -OutFile "c:\users\owner\documents\WinPwrShlPractice\OPSTestURLS\MonitorMailP.txt" 

}