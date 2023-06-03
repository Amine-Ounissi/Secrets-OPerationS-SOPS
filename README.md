# SOPS Configuration for Windows
Current Version used
```
SOPS: v3.7.3
age: v1.1.1
```
## Prerequisite
Age Encryption Method

We use the age project as encrypting method. So to use SOPS, you will need a key pair from age
To get a key pair :

you can ask the admin to get the team keys

you can create your own pair using age-keygen > key.txt

After that you can delete age and age-keygen binary
## Install SOPS binary
You can download SOPS from Github Official Repository
Or simply use the following command :
```
curl -LO https://github.com/mozilla/sops/releases/download/v3.7.3/sops-v3.7.3.exe
TIP : I advocate to use Git Bash over Powershell, it will made secret editing workflow easier
```
Save your age key file in an easy to find path (for exemple ```$HOME/.sops/key.txt```)

Rename ```sops-*.exe``` binary to ```sops.exe```

Add your local bin directory to your User Path using the following command in PowerShell:
```
$UserPath = [Environment]::GetEnvironmentVariable("Path", [EnvironmentVariableTarget]::User)
$NewPath = "$UserPath;%USERPROFILE%\bin"
[Environment]::SetEnvironmentVariable("Path", $NewPath, [EnvironmentVariableTarget]::User)
```
Move ```sops.exe``` inside local bin directory (```$HOME/bin```)

You can add some functions to your ```$HOME/.bashrc``` to encrypt and decrypt easily :
```
function encrypt {
        filename=$(basename -- "$1")
        extension="${filename##*.}"
        filename="${filename%.*}"
        sops --encrypt --age $(cat ~/.sops/key.txt |grep -oP "public key: \K(.*)") $2 $3 $1 > "$filename.enc.$extension"
}
```
```
function decrypt {
        filename=$(basename -- "$1")
        extension="${filename##*.}"
        filename="${filename%%.*}"
        sops --decrypt --age $(cat ~/.sops/key.txt |grep -oP "public key: \K(.*)") $2 $3 $1 > "$filename.$extension"
}
```
## CLI Usage
Now you can use ```encrypt your-file.yaml``` and ```decrypt your-file.enc.yaml```

You can also live edit encrypted file in bash using ```sops your-file.enc.yaml```
## VS Code Extension
You can use the ```signageos/vscode-sops``` extension to work smoothly with SOPS in your daylife
## Custom Extension
The extension was not working on Windows so we fix it and made a Pull Request on the official project. To avoid waiting the PR to be merge we can use a local modified version of the extension using ```*.vsix``` file. You can find it on ```koeos-share``` and install it using the following command:
```
code --install-extension signageos-vscode-sops-0.7.1.vsix
```
When you installed the extension I advise you to use at least the following settings :
```
"sops.defaults.ageKeyFile": "C:\\Users\\{LDAP User}\\.sops\\key.txt",
"sops.creationEnabled": true
```
Then you need to create a configuration file ```.sops.yaml``` anywhere in your filesystem

The extension fetches for ```.sops.yaml``` in the same directory of the secret file you try to encrypt and will recursively search in the parent directory while it does not find the configuration file

TIP : I preconize to create ```.sops.yaml``` in your ```$HOME``` directory or at the root of the project your working on

The configuration file supports all CLI options provide by SOPS

For exemple this a configuration to encrypt ```data``` key in Kubernetes Secret named ```*-secret.yaml``` and ```passphrase``` or ```CLIENT_SIGN_PASSWORD``` key in ```*-values.yaml``` for Helm Chart :
```
creation_rules:
  - path_regex: .*-secret\.yaml$
    encrypted_regex: ^data$
    age: <PUBLIC AGE KEY>
  - path_regex: .*-values\.yaml$
    encrypted_regex: ^(passphrase|CLIENT_SIGN_PASSWORD)$
    age: <PUBLIC AGE KEY>
```
TIP : You can use different AGE public key in each item of the YAML list

## VS Code Extension Usage
Now, everytime you save a file that match rules you specified in your ```.sops.yaml```. The VS Code extension will automaticly encrypt the file

The extension will also automaticly decrypt files when you open them.
