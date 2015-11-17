# Upgrade process

## Required Files:
**incorta-upgrade.zip** this is the upgrade package that is required to perform the upgrade.

## How to upgrade:

- unzip the package in a temp folder **don't unzip the package in the installation folder** because it will overwrite the existing installation in the wrong way.

- you might need to set execution permissions to the upgrade script.

```
chmod +x upgrade
```

- run the upgrade script.

```
./upgrade /IncortaInstallation/Incorta_V2.1

## /IncortaInstallation/Incorta_V2.1 this should be replaced by the full installation path.
```

- the upgrade process prints out the status and also writes it to a log file.


**for manual upgrade instructions** please refer to [this](https://github.com/Incorta/installer/tree/master/incorta.dbUpgrader)
