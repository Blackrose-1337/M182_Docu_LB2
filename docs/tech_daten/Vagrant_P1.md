# Vagrant

## Vagrantfile

Das Vagrantfile ist eine Konfigurationsdatei in Ruby-Syntax, die von Vagrant verwendet wird, um die Eigenschaften und Einstellungen einer virtuellen Maschine (VM) zu definieren. Hier sind einige der häufigsten Konfigurationselemente im Vagrantfile:
```
- Box-Konfiguration: 
  - config.vm.box: Definiert die Basis-Box, auf der die VM basiert. Eine "Box" ist ein vorbereitetes Image mit einem bestimmten Betriebssystem.

```
```
- Netzwerkkonfiguration:
  - config.vm.network: Legt Netzwerkeinstellungen für die VM fest, wie z. B. die Art des Netzwerks (private Netzwerke, Portweiterleitungen).
```
```
- Ressourcenkonfiguration:
  - config.vm.provider: Erlaubt die Konfiguration von Provider-spezifischen Einstellungen (z.B., VirtualBox, VMware). 
  - config.vm.cpu: Definiert die Anzahl der zugewiesenen CPUs. 
  - config.vm.memory: Definiert die zugewiesene Arbeitsspeichermenge.
```
```
- Provisionierung:
  - config.vm.provision: Definiert die Art der automatisierten Provisionierung (z.B., Shell-Skripte, Chef, Puppet).
```
```
- Synced Folders:
  - config.vm.synced_folder: Ermöglicht das Teilen von Verzeichnissen zwischen dem Host und der VM.
```
```
- Vagrant Plugins:
  - config.vm.plugin: Ermöglicht die Integration von Vagrant-Plugins.
```

## Vagrantfile DetectionELK


### VM "logger":
```
Betriebssystem: Ubuntu 18.04
IP-Adresse: 192.168.38.105
Ressourcen (für VMware):
    4096 MB RAM
    2 CPUs
```

```bash
config.vm.define "logger" do |cfg|
    cfg.vm.box = "bento/ubuntu-18.04"
    cfg.vm.hostname = "logger"
    cfg.vm.provision :shell, path: "bootstrap.sh"
    cfg.vm.provision :shell, path: "ELK.sh"
    cfg.vm.network :private_network, ip: "192.168.38.105", gateway: "192.168.38.1", dns: "8.8.8.8"

    cfg.vm.provider "vmware_desktop" do |v, override|
        v.vmx["displayname"] = "logger"
        v.memory = 4096
        v.cpus = 2
        v.gui = false
    end
    cfg.vm.provider "virtualbox" do |vb, override|
          vb.gui = false
          vb.name = "logger"
          vb.customize ["modifyvm", :id, "--memory", 4096]
          vb.customize ["modifyvm", :id, "--cpus", 2]
          vb.customize ["modifyvm", :id, "--vram", "32"]
          vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
          vb.customize ["modifyvm", :id, "--clipboard", "bidirectional"]
          vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
          vb.customize ["setextradata", "global", "GUI/SuppressMessages", "all" ]
    end
end
```

### VM "dc" (Domain Controller):
```
Betriebssystem: Windows Server 2016
IP-Adresse: 192.168.38.102
Ressourcen (für VMware):
    3072 MB RAM
    2 CPUs
```
```bash
config.vm.define "dc" do |cfg|
    cfg.vm.box = "cyberdefenders/win2016"
    cfg.vm.hostname = "dc"
    cfg.vm.boot_timeout = 600
    cfg.winrm.transport = :plaintext
    cfg.vm.communicator = "winrm"
    cfg.winrm.basic_auth_only = true
    cfg.winrm.timeout = 300
    cfg.winrm.retry_limit = 20
    cfg.vm.network :private_network, ip: "192.168.38.102", gateway: "192.168.38.1"

    cfg.vm.provision "shell", path: "scripts/fix-second-network.ps1", privileged: true, args: "-ip 192.168.38.102 -dns 8.8.8.8 -gateway 192.168.38.1" 
    cfg.vm.provision "shell", path: "scripts/provision.ps1", privileged: false
    cfg.vm.provision "reload"
    cfg.vm.provision "shell", path: "scripts/provision.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/download_palantir_wef.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/install-utilities.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/install-redteam.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/install-choco-extras.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/install-osquery.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/install-sysinternals.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/install-velociraptor.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/configure-ou.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/configure-wef-gpo.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/configure-powershelllogging.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/configure-AuditingPolicyGPOs.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/configure-rdp-user-gpo.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/configure-disable-windows-defender-gpo.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/install-autorunstowineventlog.ps1", privileged: false
    cfg.vm.provision "shell", inline: 'wevtutil el | Select-String -notmatch "Microsoft-Windows-LiveId" | Foreach-Object {wevtutil cl "$_"}', privileged: false
    cfg.vm.provision "shell", inline: "Set-SmbServerConfiguration -AuditSmb1Access $true -Force", privileged: false
    cfg.vm.provision "enable-public-winrm", type: "shell", path: "scripts/enable-winrm.ps1", privileged: false
    cfg.vm.provision "shell", inline: 'cscript c:\windows\system32\slmgr.vbs /dlv', privileged: false

    cfg.vm.provider "vmware_desktop" do |v, override|
        v.vmx["displayname"] = "dc.windomain.local"
        v.memory = 3072
        v.cpus = 2
        v.gui = false
        v.enable_vmrun_ip_lookup = false
    end

    cfg.vm.provider "virtualbox" do |vb, override|
        vb.gui = false
        vb.name = "dc.windomain.local"
        vb.default_nic_type = "82545EM"
        vb.customize ["modifyvm", :id, "--memory", 3072]
        vb.customize ["modifyvm", :id, "--cpus", 2]
        vb.customize ["modifyvm", :id, "--vram", "32"]
        vb.customize ["modifyvm", :id, "--clipboard", "bidirectional"]
        vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        vb.customize ["setextradata", "global", "GUI/SuppressMessages", "all" ]
    end
end
```

### VM "wef" (Windows Event Forwarder):
```
Betriebssystem: Windows Server 2016
IP-Adresse: 192.168.38.103
Abhängigkeiten: Abhängig vom "dc" (Domain Controller) für DNS (192.168.38.102)
Ressourcen (für VMware):
    2048 MB RAM
    2 CPUs
```

```bash
config.vm.define "wef" do |cfg|
     cfg.vm.box = "cyberdefenders/win2016"
     cfg.vm.hostname = "wef"
     cfg.vm.boot_timeout = 600
     cfg.vm.communicator = "winrm"
     cfg.winrm.basic_auth_only = true
     cfg.winrm.timeout = 300
     cfg.winrm.retry_limit = 20
     cfg.vm.network :private_network, ip: "192.168.38.103", gateway: "192.168.38.1", dns: "192.168.38.102"

    cfg.vm.provision "shell", path: "scripts/fix-second-network.ps1", privileged: true, args: "-ip 192.168.38.103 -dns 8.8.8.8 -gateway 192.168.38.1" 
    cfg.vm.provision "shell", path: "scripts/provision.ps1", privileged: false
    cfg.vm.provision "reload"
    cfg.vm.provision "shell", path: "scripts/provision.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/download_palantir_wef.ps1", privileged: false
    cfg.vm.provision "shell", inline: 'wevtutil el | Select-String -notmatch "Microsoft-Windows-LiveId" | Foreach-Object {wevtutil cl "$_"}', privileged: false
    cfg.vm.provision "shell", path: "scripts/install-wefsubscriptions.ps1", privileged: false
    # cfg.vm.provision "shell", path: "scripts/install-splunkuf.ps1", privileged: false       Auskommentiert!
    # cfg.vm.provision "shell", path: "scripts/install-windows_ta.ps1", privileged: false     Auskommentiert! 
    cfg.vm.provision "shell", path: "scripts/install-utilities.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/install-redteam.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/install-choco-extras.ps1", privileged: false
    cfg.vm.provision "install-winlogbeat", type: "shell", path: "scripts/install-winlogbeat.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/install-osquery.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/install-sysinternals.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/install-velociraptor.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/configure-pslogstranscriptsshare.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/install-autorunstowineventlog.ps1", privileged: false
    cfg.vm.provision "shell", inline: "Set-SmbServerConfiguration -AuditSmb1Access $true -Force", privileged: false
    cfg.vm.provision "shell", path: "scripts/install-microsoft-ata.ps1", privileged: false
    cfg.vm.provision "enable-public-winrm", type: "shell", path: "scripts/enable-winrm.ps1", privileged: false
    cfg.vm.provision "shell", inline: 'cscript c:\windows\system32\slmgr.vbs /dlv', privileged: false

     cfg.vm.provider "vmware_desktop" do |v, override|
       v.vmx["displayname"] = "wef.windomain.local"
       v.memory = 2048
       v.cpus = 2
       v.gui = false
       v.enable_vmrun_ip_lookup = false
     end

     cfg.vm.provider "virtualbox" do |vb, override|
       vb.gui = false
       vb.name = "wef.windomain.local"
       vb.default_nic_type = "82545EM"
       vb.customize ["modifyvm", :id, "--memory", 2048]
       vb.customize ["modifyvm", :id, "--cpus", 2]
       vb.customize ["modifyvm", :id, "--vram", "32"]
       vb.customize ["modifyvm", :id, "--clipboard", "bidirectional"]
       vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
       vb.customize ["setextradata", "global", "GUI/SuppressMessages", "all" ]
     end
   end
```

### VM "win10" (Windows 10):
```
Betriebssystem: Windows 10
IP-Adresse: 192.168.38.104
Abhängigkeiten: Abhängig vom "dc" (Domain Controller) für DNS (192.168.38.102)
Ressourcen (für VMware):
    2048 MB RAM
    1 CPU (bitte beachten: Unterschiedliche CPU-Anzahl im Vergleich zu den anderen VMs)
```
```bash
config.vm.define "win10" do |cfg|
    cfg.vm.box = "cyberdefenders/win10"
    cfg.vm.hostname = "win10"
    cfg.vm.boot_timeout = 1200
    cfg.vm.communicator = "winrm"
    cfg.winrm.basic_auth_only = true
    cfg.winrm.timeout = 1200
    cfg.winrm.retry_limit = 20
    cfg.vm.network :private_network, ip: "192.168.38.104", gateway: "192.168.38.1", dns: "192.168.38.102"

    cfg.vm.provision "shell", path: "scripts/fix-second-network.ps1", privileged: true, args: "-ip 192.168.38.104 -dns 8.8.8.8 -gateway 192.168.38.1" 
    cfg.vm.provision "shell", path: "scripts/MakeWindows10GreatAgain.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/provision.ps1", privileged: false
    cfg.vm.provision "reload"
    cfg.vm.provision "shell", path: "scripts/provision.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/download_palantir_wef.ps1", privileged: false
    cfg.vm.provision "shell", inline: 'wevtutil el | Select-String -notmatch "Microsoft-Windows-LiveId" | Foreach-Object {wevtutil cl "$_"}', privileged: false
    cfg.vm.provision "shell", path: "scripts/install-utilities.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/install-redteam.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/install-choco-extras.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/install-osquery.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/install-sysinternals.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/install-velociraptor.ps1", privileged: false
    cfg.vm.provision "shell", path: "scripts/install-autorunstowineventlog.ps1", privileged: false
    cfg.vm.provision "enable-public-winrm", type: "shell", path: "scripts/enable-winrm.ps1", privileged: false
    cfg.vm.provision "shell", inline: 'cscript c:\windows\system32\slmgr.vbs /dlv', privileged: false
    cfg.vm.provision "shell", inline: 'cscript c:\windows\system32\slmgr.vbs /rearm', privileged: false
    cfg.vm.provision "reload"

    cfg.vm.provider "vmware_desktop" do |v, override|
        v.vmx["displayname"] = "win10.windomain.local"
        v.vmx["gui.fullscreenatpoweron"] = "FALSE"
        v.vmx["gui.viewModeAtPowerOn"] = "windowed"
        v.memory = 2048
        v.cpus = 1
        v.gui = true
        v.enable_vmrun_ip_lookup = false
    end

    cfg.vm.provider "virtualbox" do |vb, override|
        vb.gui = false
        vb.name = "win10.windomain.local"
        vb.default_nic_type = "82545EM"
        vb.customize ["modifyvm", :id, "--memory", 2048]
        vb.customize ["modifyvm", :id, "--cpus", 2]
        vb.customize ["modifyvm", :id, "--vram", "32"]
        vb.customize ["modifyvm", :id, "--clipboard", "bidirectional"]
        vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        vb.customize ["setextradata", "global", "GUI/SuppressMessages", "all" ]
    end
end
```
### Gemeinsame Konfigurationen:
```bash 
- Alle VMs verwenden eine private Netzwerkkonfiguration mit einer festen IP-Adresse, einem Gateway und DNS-Einstellungen.
- Die VMware-Provider-Konfigurationen enthalten spezifische Einstellungen für VMware Workstation (z. B. Anzahl der CPUs, RAM, etc.).
    cfg.vm.provider "vmware_desktop" do |v, override|
- Die VirtualBox-Provider-Konfigurationen enthalten spezifische Einstellungen für VirtualBox (z. B. Anzahl der CPUs, RAM, etc.).
    cfg.vm.provider "virtualbox" do |vb, override|
```
### Bereitstellungsskripte:
```bash
- Alle VMs verwenden verschiedene Bereitstellungsskripte (bootstrap.sh, ELK.sh, provision.ps1, usw.), um Software zu installieren und Konfigurationen durchzuführen.
- Die Skripte sind vielfältig und reichen von der Konfiguration von Netzwerkeinstellungen bis zur Installation von Sicherheits- und Überwachungstools.
    cfg.vm.provision "shell", path:
```
### Weitere Hinweise:
```
Die "dc" VM fungiert als Domain Controller und führt verschiedene Bereitstellungsschritte durch, darunter die Installation von Windows-Funktionen, das Herunterladen von Tools und die Konfiguration von Gruppenrichtlinien.
Die "wef" VM ist als Windows Event Forwarder konfiguriert und installiert verschiedene Tools zur Überwachung und Sicherheitsanalyse.
Die "win10" VM ist für Windows 10 und hat spezifische Bereitstellungsschritte für die Installation von Tools und Konfigurationen.
```
