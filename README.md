# ubuntu-personal-step-download

**Problem: Nested VT-x/AMD-V was grayed out in VirtualBox**
Solution:
Step 1 — Opened PowerShell as Administrator on Windows

Press Windows + X → Windows PowerShell (Admin)

Step 2 — Checked Hyper-V features (all were already disabled)
# Check if any Hyper-V or Virtualization features are enabled
# that might be blocking VirtualBox's nested virtualization
Get-WindowsOptionalFeature -Online | Where-Object {$_.FeatureName -like "*Hyper*" -or $_.FeatureName -like "*Virtual*"} | Select-Object FeatureName, State
# Result: All were Disabled, so Hyper-V was NOT the cause

Step 3 — Disabled Memory Integrity / Credential Guard via registry
# Memory Integrity (HypervisorEnforcedCodeIntegrity) was the real culprit
# It uses a hypervisor internally, which conflicts with VirtualBox
# This command disables it via the Windows registry
reg add "HKLM\SYSTEM\CurrentControlSet\Control\DeviceGuard\Scenarios\HypervisorEnforcedCodeIntegrity" /v "Enabled" /t REG_DWORD /d 0 /f

Step 4 — Rebooted Windows
# A reboot is required for the registry change to take effect
shutdown /r /t 0

Step 5 — After reboot, opened VirtualBox

# Power off the Ubuntu VM first (not suspend)
# Then: Settings → System → Processor
# Check "Enable Nested VT-x/AMD-V" 
# The checkbox is now clickable because Memory Integrity is disabled
