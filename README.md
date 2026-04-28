# ubuntu-steps-download

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

**KVM INSTALL**

Step 1: Update System

    # Update and upgrade your system packages
sudo apt update && sudo apt upgrade -y

Step 2: Install KVM and All Required Packages

    # Install KVM and all required packages
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager virtinst cpu-checker

Step 3: Verify KVM Installation

    # Check if KVM is supported by your CPU
sudo kvm-ok

Step 4: Enable and Start libvirt Service

    # Enable and start the libvirt service
sudo systemctl enable --now libvirtd

    # Check if libvirtd is running
sudo systemctl status libvirtd

Step 5: Add User to KVM Groups

    # Add your user to libvirt and kvm groups
sudo usermod -aG libvirt $USER
sudo usermod -aG kvm $USER

    # Apply group changes immediately
newgrp libvirt

Step 6: Verify Everything is Working

    # List all virtual machines (should return empty list)
virsh list --all

Step 7: Open Virtual Machine Manager

    # Launch the Virtual Machine Manager GUI
virt-manager

**creating a VM in virt-manager:**

Step 1 — Open virt-manager
bash

virt-manager

This opens the Virtual Machine Manager GUI window

Step 2 — Create a new VM

    Click the monitor + sparkle icon (top left)
    This starts the new VM creation wizard

Step 3 — Choose installation source

    Select "Local install media (ISO image)"
    Click Forward
    We use an ISO file we downloaded instead of a physical CD/DVD

Step 4 — Select your ISO file

    Click Browse → Browse Local
    Navigate to your home folder and select:

  ubuntu-22.04.5-live-server-amd64.iso

    This is the OS installer that will boot in the VM

