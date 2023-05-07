# Powershell-Script
Handpicked removal of a user profile and all data.
----------------Start of script---------------------------------
$numberOfRuns = 2
for ($i = 1; $i -le $numberOfRuns; $i++) {
    # ADD USERS TO DELETE AND EXCLUDE HERE
$usersToDelete = "user1","user2","user3"
$excludedUserNames = "Administrator", "DefaultAccount", "Guest", "vboxuser"

foreach ($user in $usersToDelete) {
    $userPath = "C:\Users\$user"
    Write-Output "`nTaking ownership of the folder for $user"
    TAKEOWN /F $userPath /A /R /D "Y" 1>$null
    Write-Output "`nUpdating permissions for $user"
    ICACLS $userPath /grant Administrators:F /T 1>$null
}
foreach ($user in $usersToDelete) {
    Get-LocalUser $user | Remove-LocalUser
}

Get-WmiObject -Class Win32_UserProfile | Where-Object { $_.LocalPath -match "^C:\\Users\\($([string]::Join('|', $usersToDelete)))$" -and $_.SID -notmatch "^S-1-5-18$" -and $_.SID -notmatch "^S-1-5-19$" -and $_.SID -notmatch "^S-1-5-20$" } | ForEach-Object {
    $sid = $_.SID
    $username = $_.LocalPath -replace "^C:\\Users\\"
    if ($username -notin $excludedUserNames) {
        Remove-WmiObject -Class Win32_UserProfile -Filter "SID='$sid'" -ErrorAction SilentlyContinue
        $userPath = "C:\Users\$username"
        if (Test-Path $userPath) { Remove-Item $userPath -Recurse -Force }
        Remove-Item "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList\$sid" -Recurse -Force -ErrorAction SilentlyContinue
        Remove-Item -Path "C:\Users\$username" -Recurse -Force -ErrorAction SilentlyContinue
        
        foreach ($user in $usersToDelete) {
            Get-LocalUser $user | Remove-LocalUser
        }
    
    }
    
}

foreach ($user in $usersToDelete) {
    $userPath = "C:\Users\$user"
    if (Test-Path $userPath) {
        Remove-Item -Recurse -Force $userPath 2>&1
    }
}
}
----------------------END OF SCRIPT----------------------------------------
