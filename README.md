# vcd-h5-themes

VMware vCloud Director 9.x PowerShell cmdlets to assist managing HTML5 portal &amp; themes

This module provides several cmdlets to assist in managing the configurations and content of customization of the HTML5 portal in VMware vCloud Director v9.x environments. Note that none of these modules will affect branding or customization in the legacy FlexUI (Flash-based) portal present in previous releases of vCloud Director.

The table below shows the cmdlets included in this module, a brief description of each and the minimum vCloud Director version required for each.

cmdlet Name  | Function                                                  | Minimum API / vCD Version
------------ | --------------------------------------------------------- | ---------------------------
[`Get-Branding`](#Get-Branding) | Gets the currently defined HTML5 portal branding settings | 30.0 (vCloud Director v9.1)
[`Set-Branding`](#Set-Branding) | Sets the vCloud Director HTML5 portal branding settings   | 30.0 (vCloud Director v9.1)
[`Get-Theme`](#Get-Theme) | Gets the available portal themes                          | 30.0 (vCloud Director v9.1)
[`Set-Theme`](#Set-Theme) | Sets which portal theme is the current system default     | 30.0 (vCloud Director v9.1)
[`Remove-Theme`](#Remove-Theme) | Deletes a portal theme                                    | 30.0 (vCloud Director v9.1)
[`New-Theme`](#New-Theme) | Creates a new portal theme                                | 30.0 (vCloud Director v9.1)
[`Publish-Css`](#Publish-Css) | Uploads a generated CSS file for a vCloud Director theme  | 31.0 (vCloud Director v9.5)
[`Get-Css`](#Get-Css) | Downloads a CSS file from a vCloud Director theme         | 31.0 (vCloud Director v9.5)

The sections below provide documentation of each cmdlet and the parameters it takes together with example usage information.

## Get-Branding

This function retrieves the current branding settings for a vCloud Director instance. It returns the currently configured portal name, color scheme and currently selected default theme.

Parameters:

Parameter  | Type   | Default | Required | Description
---------  | ------ | ------- | -------- | -----------
vCDHost    | String | None    | No       | The FQDN of the vCloud Site (e.g. 'my.cloud.com'). Must be specified if you are connected to multiple vCD sites (Connect-CIServer) already. If only connected to a single vCD site then this will be used automatically.

Output:

A PSObject containing the configured branding settings for the Portal Name, Color (banner background), currently selected Theme and any defined customLinks entries.

Example:

```PowerShell
C:\PS> Get-Branding -vcdHost 'my.cloud.com'

portalName               portalColor selectedTheme                      customLinks
----------               ----------- -------------                      -----------
My Cloud Portal          #0C0C01     @{themeType=CUSTOM; name=MyCloud}  {}
```

## Set-Branding

This function updates the current branding settings for a vCloud Director instance. It returns a message confirming the setting has been successfully applied.

Parameters:

Parameter   | Type      | Default | Required | Description
---------   | --------- | ------- | -------- | -----------
vCDHost     | String    | None    | No       | The FQDN of the vCloud Site (e.g. 'my.cloud.com'). Must be specified if you are connected to multiple vCD sites (Connect-CIServer) already. If only connected to a single vCD site then this will be used automatically.
portalName  | String    | None    | No       | A new name for the site portal, if omitted the previous portal name is retained.
portalColor | String    | None    | No       | A new color for the site portal banner, if omitted the previous color value is retained. Must be specified in HTML hexadecimal 16-bit color values using upper-case characters (e.g. '#1A2B3C').
customLinks | Hashtable | None    | No       | A hash of custom URL keys and values to be created or updated. Note that currently this serves no purpose as the customLinks functionality is not enabled in vCloud Director v9.x. Example: `@{'about'='https://my.company.com/about';'support'='https://my.company.com/support'}`

Output:

A message confirms whether the requested changes have been successfully submitted to the vCloud API or an error. Settings can be verified by using [`Get-Branding`](#Get-Branding)

Example:

```PowerShell
C:\PS> Set-Branding -vcdHost 'my.cloud.com' -portalName 'My Cloud Portal'
Branding configuration sent successfully.
```

## Get-Theme

This function returns the list of currently configured themes in vCloud Director. It will usually show the default built-in themes ('Default' and 'Dark') as well as any custom themes created. If an optional ThemeName is supplied only themes matching that name will be returned (can be used to check if a theme already exists).

Parameters:

Parameter   | Type      | Default | Required | Description
---------   | --------- | ------- | -------- | -----------
vCDHost     | String    | None    | No       | The FQDN of the vCloud Site (e.g. 'my.cloud.com'). Must be specified if you are connected to multiple vCD sites (Connect-CIServer) already. If only connected to a single vCD site then this will be used automatically.
ThemeName   | String    | None    | No       | A theme name to match, can be used to determine if a theme already exists.

Output:

A PSObject of themes found in the vCloud instance.

Example:

```PowerShell
C:\PS> Get-Theme -vcdHost 'my.cloud.com'

themeType name
--------- ----
CUSTOM    MyTheme
BUILT_IN  Default
BUILT_IN  Dark
```

## Set-Theme

This function sets the specified theme to be the system default theme for vCloud Director to use in the HTML5 portal. Changes made here will be shown in subsequent [`Get-Branding`](#Get-Branding) requests.

Parameters:

Parameter   | Type      | Default | Required | Description
---------   | --------- | ------- | -------- | -----------
vCDHost     | String    | None    | No       | The FQDN of the vCloud Site (e.g. 'my.cloud.com'). Must be specified if you are connected to multiple vCD sites (Connect-CIServer) already. If only connected to a single vCD site then this will be used automatically.
ThemeName   | String    | None    | Yes      | The theme to be set as the new system default theme. If the theme name cannot be matched an error will be generated and no changes made.
custom      | Boolean   | true    | No       | Specifies whether the theme is a custom (user-created) theme or a built-in theme. Only needs to be specified (as 'false') if reverting to one of the VMware supplied default themes ('Default' or 'Dark').

Output:

A message indicating that the default theme configuration has been set successfully or an error.

Example 1: Set a custom theme as default:

```PowerShell
C:\PS> Set-Theme -vCDHost 'my.cloud.com' -ThemeName 'MyTheme'
Default theme configuration set successfully.
```

Example 2: Revert to default 'Dark' Theme:

```PowerShell
C:\PS> Set-Theme -vCDHost 'my.cloud.com' -ThemeName 'Dark' -custom:$false
Default theme configuration set successfully.
```

## New-Theme

This function creates a new theme with the specified name. Note that the created theme will not automatically be activated as the default system theme, that needs to be done using [`Set-Theme`](#Set-Theme) once created.

Parameters:

Parameter   | Type      | Default | Required | Description
---------   | --------- | ------- | -------- | -----------
vCDHost     | String    | None    | No       | The FQDN of the vCloud Site (e.g. 'my.cloud.com'). Must be specified if you are connected to multiple vCD sites (Connect-CIServer) already. If only connected to a single vCD site then this will be used automatically.
ThemeName   | String    | None    | Yes      | The theme to be created. If the theme already exists an error will be generated and no changes made.

Output:

A message indicating that the new theme has been created successfully or an error.

Example:

```PowerShell
C:\PS> New-Theme -vCDHost 'my.cloud.com' -ThemeName 'MyTheme'
Theme MyTheme created successfully.
```

## Remove-Theme

This function deletes a theme with the specified name.

Parameters:

Parameter   | Type      | Default | Required | Description
---------   | --------- | ------- | -------- | -----------
vCDHost     | String    | None    | No       | The FQDN of the vCloud Site (e.g. 'my.cloud.com'). Must be specified if you are connected to multiple vCD sites (Connect-CIServer) already. If only connected to a single vCD site then this will be used automatically.
ThemeName   | String    | None    | Yes      | The theme to be removed. If the theme doesn't exist an error will be generated and no changes made.

Output:

A message indicating that the theme has been removed or an error.

Example:

```PowerShell
C:\PS> Remove-Theme -vCDHost 'my.cloud.com' -ThemeName 'MyTheme'
Theme MyTheme was removed successfully.
```

## Publish-Css

This function uploads the specified .css file as the customization for the specified portal theme. Compatible .css files are generated by the VMware theme builder available in the [VMware vcd-ext-sdk](https://github.com/vmware/vcd-ext/sdk) project under the /ui/theme-builder folder.

Parameters:

Parameter   | Type      | Default | Required | Description
---------   | --------- | ------- | -------- | -----------
vCDHost     | String    | None    | No       | The FQDN of the vCloud Site (e.g. 'my.cloud.com'). Must be specified if you are connected to multiple vCD sites (Connect-CIServer) already. If only connected to a single vCD site then this will be used automatically.
ThemeName   | String    | None    | Yes      | The theme to be removed. If the theme doesn't exist an error will be generated and no changes made.
CssFile     | String    | None    | Yes      | The path and filename of a .css file to be uplaoded as the CSS content for the specified theme, generally this will be the .css outputted by the VMware theme-builder.

Output:

A message indicating whether the .css file has been successfully uploaded or not.

Example:

```PowerShell
C:\PS> Publish-Css -vCDHost 'my.cloud.com' -ThemeName 'MyTheme' -CssFile 'mytheme.css'
Theme CSS file uploaded succesfully.
```

## Get-Css

This function downloads any customization css that has been uploaded to a theme.

Parameters:

Parameter   | Type      | Default | Required | Description
---------   | --------- | ------- | -------- | -----------
vCDHost     | String    | None    | No       | The FQDN of the vCloud Site (e.g. 'my.cloud.com'). Must be specified if you are connected to multiple vCD sites (Connect-CIServer) already. If only connected to a single vCD site then this will be used automatically.
ThemeName   | String    | None    | Yes      | The theme to be removed. If the theme doesn't exist an error will be generated and no changes made.
CssFile     | String    | None    | Yes      | The path and filename of a file to be written with the downloaded css information, an existing file with the same name will be overwritten.

Output:

A message indicating whether or not the specified CSS was downloaded.

Example:

```PowerShell
C:\PS> Get-Css -vCDHost 'my.cloud.com' -ThemeName 'MyTheme' -CssFile 'mytheme.css'
Theme CSS file downloaded succesfully.
```