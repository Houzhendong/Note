```powershell
$OutputEncoding = [System.Text.Encoding]::UTF8

# Query for installed Visual Studio instances using CIM
$vsInstances = Get-CimInstance -ClassName MSFT_VSInstance -Namespace root/cimv2/vs

# If there are no Visual Studio instances, output an error message
if ($vsInstances -eq $null) {
    Write-Host "No Visual Studio instances found."
    return
}

$msbuildPath = $null

# Loop through each Visual Studio instance and try to locate MSBuild
foreach ($vs in $vsInstances) {
    $vsInstallPath = $vs.InstallLocation
    $temp = Join-Path -Path $vsInstallPath -ChildPath "MSBuild\Current\Bin\MSBuild.exe"
    
    # Check if MSBuild exists in this path
    if (Test-Path $temp) {
        $msbuildPath = $temp
        break
    }
}

if($msbuildPath -ne $null) {
    Write-Host "MSBuild found at: $msbuildPath"
} else {
    # If MSBuild is not found in any of the Visual Studio instances
    Write-Host "MSBuild not found in any Visual Studio instances."
}
```