# Powershell

## WSUS 

Below code base on the WSUS group name and appove all security update for computer needed.


```
$WsussServer = get-wsusserver 

$GroupName = "Production"

$GroupID = ($WsussServer.GetComputerTargetGroups() | ?{$_.name -eq $GroupName}).id

$AllComputer = $WsussServer.GetComputerTargets() | ?{$_.ComputerTargerGroupIDs -contains $GroupID}

$AllPatch =@()

foreach ($Computer in $AllComputer)
{
	$ComputerPatch = $Computer.GetUpdateInstallationInfoperupdate() |?{$_.Updateinstallationstate -eq "Notinstall"}| ?{$_.updaterApprovalAction -eq "NotApproved"}
	$AllPatch += $ComputerPatch
}

foreach($patch in $AllPatch)
{
	$tmp = get-wsusupdate -updateid $patch.updateid
	if($tmp.Classification -eq "Security Updates")
	{
		$tmp | approve-wsusupdate -targetGroupName $GroupName -Action Install
		Write-host "$($patch.updateid) Had been approved on $GroupName."
		
	}
}
```