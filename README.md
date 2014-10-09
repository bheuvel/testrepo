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

The ConfigurationGenerator ([configuration_generator.rb Chef 11.16.0] (https://github.com/opscode/chef/blob/11.16.0/lib/chef/util/dsc/configuration_generator.rb)) uses the following code:
```
def configuration_code(code, configuration_name)
    "$ProgressPreference = 'SilentlyContinue';Configuration '#{configuration_name}'\n{\n\tnode 'localhost'\n{\n\t#{code}\n}}\n"
end
```


Possible solution could be to leave out the `localhost` definition, which MAY require the user to implement this, but will allow usage of the `Import-DscResource` keyword. Without this functionality, the code attribute is basically only usable for the (about) 12 natively provided resources, instead of the (about) 80 resources in the DSC Resource Kit Wave 7.

Leaving out the `localhost' definition will also make the `code` and `command` attribute behave more similar; providing, or complete control, DSC script code.
