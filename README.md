# kiosk-demo-electron
[![Build status](https://ci.appveyor.com/api/projects/status/um6ul6dbwjrw913m/branch/master?svg=true)](https://ci.appveyor.com/project/syedhassaanahmed/kiosk-demo-electron/branch/master)

This Electron App demonstrates multi-screen Kiosk mode experience by creating an .msi package which executes [PowerShell scripts](https://github.com/syedhassaanahmed/kiosk-demo-electron/blob/master/tools/scripts/Install-ShellLauncher.ps1). Scripts toggle [Windows 10 Shell Launcher](https://docs.microsoft.com/en-us/windows-hardware/customize/enterprise/shell-launcher) as well as set the kiosk user to [AutoLogon](https://docs.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/microsoft-windows-shell-setup-autologon). Creating an .msi makes sure we can distribute our app to multiple kiosks via MDM e.g [Microsoft Intune](https://docs.microsoft.com/en-us/intune/apps-add).

## Create Installer
`npm run dist` will build everything and create the .msi in `dist` folder.

## Configure
Kiosk parameters are passed to the installer like this: `KioskDemoElectron.msi KIOSK_USERNAME=<kiosk user> KIOSK_PASSWORD=<kiosk password>`. All params have a default value as can be seen in [product.wxs](https://github.com/syedhassaanahmed/kiosk-demo-electron/blob/master/tools/product.wxs).

## How it works
- First we package the app using [electron-packager](https://github.com/electron-userland/electron-packager)
- Then we harvest binaries produced by electron-packager using WiX's [Heat tool](http://wixtoolset.org/documentation/manual/v3/overview/heat.html).
- We also copy our PowerShell script into the root of electron-packager output. We could let the harvester take care of PowerShell files as well, but we wanted to explicitly specify them in `product.wxs` with [Custom Actions](http://wixtoolset.org/documentation/manual/v3/wixdev/extensions/authoring_custom_actions.html).
- Output of harvest tool is `heat.wxs` which contains a [Fragment](https://www.firegiant.com/wix/tutorial/upgrades-and-modularization/fragments/) with list of files. We take that as well as `product.wxs` and pass it to the [Candle tool](http://wixtoolset.org/documentation/manual/v3/overview/candle.html). Candle is responsible for preprocessing .wxs files and generates compiled `.wixobj` files.
- Finally we use the [Light tool](http://wixtoolset.org/documentation/manual/v3/overview/light.html) to generate msi from .wixobj.
- PowerShell Scripts are executed from [WiX Custom Actions](https://damienbod.com/2013/09/01/wix-installer-with-powershell-scripts/).

## Troubleshoot
Msi logging can be enabled by executing the installer like this: `msiexec /i "setup.msi" /l*v "msi.log" PARAM=VALUE`. PowerShell log will be located at `C:\Windows\SysWOW64\powershell.log`.

## Telemetry
The solution uses [Application Insights](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-nodejs) to collect basic telemetry data from the app. To enable it, please create an environment variable named `APPINSIGHTS_INSTRUMENTATIONKEY` and set it to the Instrumentation Key obtained from Azure portal.