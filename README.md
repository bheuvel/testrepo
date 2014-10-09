dsc_script code (attribute usage) does not support (e.g.) Resource Kit DSC resources

When using DSC resources which are not present by default, such as the DSC Resource Kit Waves, within the code attribute, LCM gives back errors. E.g. when using xADDomain:
```
ERROR: DSC operation failed: Powershell Cmdlet failed: PSDesiredStateConfiguration\node :
The term 'xADDomain' is not recognized as the name of a cmdlet, function, script file, or
operable program. Check the spelling of the name, or if a path was included, verify that
the path is correct and try again.
```

Reason for this is that the respective module needs to be imported into the respective session, by using the keyword `Import-DscResource`.

* This keyword is only allowed at a specific scope within the DSC Configuration, namely outside/before the `node` definition.
* Unfortunately dsc_script places the code contained in the `code` attribute, WITHIN a `node` definition
