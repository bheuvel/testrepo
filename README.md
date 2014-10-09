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

The ConfigurationGenerator ([configuration_generator.rb Chef 11.16.0] (https://github.com/opscode/chef/blob/11.16.0/lib/chef/util/dsc/configuration_generator.rb#L84)) uses the following code (I inserted 'illegal' linebreaks here for clarification of scope, and one comment line starting with '#BvdH'):
```
def configuration_code(code, configuration_name)
    "$ProgressPreference = 'SilentlyContinue';Configuration '#{configuration_name}'
    {
#BvdH: Import-DscResource should be inserted here
        node 'localhost'
        {
            #{code}
        }
    }
    "
end
```


Possible solution could be to leave out the `localhost` definition, which MAY require the user to implement this within the `code` attribute, but will also allow usage of the `Import-DscResource` keyword in the `code` attribute.

Without this functionality, the code attribute is basically only usable for the (about) 12 natively provided resources, instead of the (about) 80 resources in the DSC Resource Kit Wave 7.

Leaving out the `localhost` definition will also make the `code` and `command` attribute behave more similar; providing, or complete control, DSC script code.


Alternative solution might be to use a separate attribute which needs to list the required/used DSC modules, which would then be inserted before the `localhost` block. This would be somewhat similar to what dsc_rource does ([dsc_provider.rb#L106](https://github.com/opscode-cookbooks/dsc/blob/master/libraries/dsc_provider.rb#L106)), but as dsc_resource takes a specific named (`resource_name`) DSC resource, the required import-code is relatively easily and automatically generated ([dsc_provider.rb#L119](https://github.com/opscode-cookbooks/dsc/blob/master/libraries/dsc_provider.rb#L119)). In this case the entire code would need to be parsed and analyzed for DSC resources.


In the end, dsc_script CAN off course be used with (e.g.) DSC Resource Kit Waves provided resources, but requires usage of (creation of) an external file. This could be a cookbook file, but would rather see this "enabled" for the (inline) code attribute.

Or perhaps I grossly overlooked something...
