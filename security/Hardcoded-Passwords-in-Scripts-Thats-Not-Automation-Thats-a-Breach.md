> 📖 **Original article:** [Hardcoded Passwords in Scripts: That's Not Automation, That's a Breach](https://www.valtersit.com/guides/security/Hardcoded-Passwords-in-Scripts-Thats-Not-Automation-Thats-a-Breach/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

There is a special place in infrastructure hell for sysadmins who write "automation" scripts with `$AdminPassword = "CompanyAdmin2026!"` right at the top. 

You usually find these masterpieces sitting on a globally readable network share like `\\fs01\IT_Scripts\NewUserSetup.ps1`, or worse, pushed to a public GitHub repository because someone didn't understand how `.gitignore` works. 

Let’s get one thing straight: if your script contains a plain-text password, you didn't write an automation script. You wrote a self-service portal for threat actors. You are doing the attacker's job for them.

### The Mechanics: The String Search

When a threat actor or an automated worm breaches a low-level workstation on your network, they don't immediately start dropping zero-days. They live off the land. One of the very first enumeration steps is mounting available network shares and running a recursive search for files ending in `.ps1`, `.sh`, `.bat`, or `.py`. 

Once they find those files, they just grep for strings like `password`, `pwd`, `credential`, or `api_key`. 

If anyone with read access to that file share can read the script, anyone can own the domain. It doesn't matter how complex the password is if you leave it written on a digital Post-it note attached to the execution file. 

### The Fix: DPAPI and Secret Vaults

Stop hardcoding secrets. If you need a script to run unattended, you have to decouple the credential from the code.

For enterprise environments, the correct answer is a dedicated secrets engine like HashiCorp Vault, Azure Key Vault, or AWS Secrets Manager. Your script authenticates via a managed identity or machine certificate, pulls the secret dynamically into memory, uses it, and dumps it.

If you don't have a vault, you can still secure local automation using the Windows Data Protection API (DPAPI). PowerShell's `Export-Clixml` cmdlet can serialize a credential object and encrypt it using a key tied specifically to the Windows user account and the machine executing the script. If an attacker copies the XML file to another machine, or tries to read it as a different user, it fails to decrypt.

### The Code

Here is the script that guarantees your Domain Controller gets encrypted by Friday:

```powershell
# THE BAD WAY (A breach waiting to happen)
# Anyone who can read this file owns the hypervisor

$Username = "admin@corp.local"
$Password = "SuperSecretAdmin123!"
$Cred = New-Object System.Management.Automation.PSCredential($Username, (ConvertTo-SecureString $Password -AsPlainText -Force))

Connect-VMHost -Server vcenter.corp.local -Credential $Cred
```

Here is the Senior Sysadmin approach using local DPAPI encryption. 

First, as a one-time setup step logged in as the service account running the automation, you prompt for the password and encrypt it to disk:

```powershell
# ONE-TIME SETUP (Run as the Service Account on the execution machine)
Get-Credential | Export-Clixml -Path "C:\SecureScripts\Credentials\vcenter_cred.xml"
```

Then, your actual automation script just imports that encrypted object. No plain-text secrets in the code, no hardcoded strings for malware to scrape.

```powershell
# THE REAL ENGINEER'S WAY (Decoupled and Encrypted)
# The credential can only be decrypted by the specific Service Account on this specific machine.

$CredPath = "C:\SecureScripts\Credentials\vcenter_cred.xml"

if (-Not (Test-Path $CredPath)) {
    Write-Error "Credential file missing. Cannot authenticate."
    exit 1
}
```

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/security/Hardcoded-Passwords-in-Scripts-Thats-Not-Automation-Thats-a-Breach/](https://www.valtersit.com/guides/security/Hardcoded-Passwords-in-Scripts-Thats-Not-Automation-Thats-a-Breach/)**
