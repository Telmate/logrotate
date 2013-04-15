Description
===========

Manages the logrotate package and provides a definition to manage
application specific logrotate configuration.

Requirements
============

Should work on any platform that includes a 'logrotate' package and
writes logrotate configuration to /etc/logrotate.d. Tested on Ubuntu,
Debian and Red Hat/CentOS.

Recipes
=======

## logrotate::global

Generates and controls a global `/etc/logrotate.conf` file that will include
additional files generated by the `logrotate_app` definition (see below). The
contents of the configuration file is controlled through node attributes under
`node['logrotate']['global']`. The default attributes are based on the
configuration from the Ubuntu logrotate package.

To define a valueless directive (e.g. `compress`, `copy`) simply add an attribute
named for the directive with a truthy value :

```ruby
node['logrotate']['global']['compress'] = 'any value here'
```

Note that defining a valueless directive with a falsey value will not make it
false, but will remove it:

```ruby
# Removes a defaulted 'compress' directive; does not add a 'nocompress' directive.
node.override['logrotate']['global']['compress'] = false
```

To fully overrride a booleanish directive like `compress`, you should probably
remove the positive form and add the negative form:

```ruby
node.override['logrotate']['global']['compress'] = false
node.override['logrotate']['global']['nocompress'] = true
```

The same is true of frequency directives; to be certain the frequency directive
you want is included in the global configuration, you should override the ones
you don't want as false:

```ruby
%w[ daily, weekly, yearly ].do |freq|
    node.override['logrotate']['global'][freq] = false
end
node.override['logrotate']['global']['monthly'] = true
```

To define a parameter with a value (e.g. `create`, `mail`) add an attribute
with the desired value:

```ruby
node['logrotate']['global']['create'] = '0644 root adm'
```

To define a path stanza in the global configuration (generally unneeded because
of the `logrotate_app` definition) just add an attribute with the path as the
name and a hash containing directives and parameters as described above:

```ruby
node['logrotate']['global']['/var/log/wtmp'] = {
    'missingok' => true,
    'monthly' => true,
    'create' => '0660 root utmp',
    'rotate' => 1
}
```

`firstaction`, `prerotate`, `postrotate`, and `lastaction` scripts can be defined either
as arrays of the lines to put in the script or multiline strings:

```ruby
node['logrotate']['global']['/var/log/foo/*.log'] = {
    'missingok' => true,
    'monthly' => true,
    'create' => '0660 root adm',
    'rotate' => 1,
    'prerotate' => [ 'service foo start_rotate', 'logger started foo service log rotation' ],
    'postrotate' => <<-EOF
        service foo end_rotate
        logger completed foo service log rotation
    EOF
}
```

Definitions
===========

## logrotate\_app

This definition can be used to drop off customized logrotate config
files on a per application basis.

The definition takes the following params:

* path: specifies a single path (string) or multiple paths (array)
  that should have logrotation stanzas created in the config file. No
  default, this must be specified.
* enable: true/false, if true it will create the template in
  /etc/logrotate.d.
* frequency: sets the frequency for rotation. Default value is
  'weekly'. Valid values are: daily, weekly, monthly, yearly, see the
  logrotate man page for more information.
* size: Log files are rotated when they grow bigger than size bytes.
* template: sets the template source, default is "logrotate.erb".
* cookbook: select the template source from the specified cookbook. By
  default it will use the cookbook where the definition is used.
* create: creation parameters for the logrotate "create" config,
  follows the form "mode owner group". This is an optional parameter,
  and is nil by default.
* postrotate: lines to be executed after the log file is rotated
* prerotate: lines to be executed after the log file is rotated
* sharedscripts: if true, the sharedscripts options is specified which
  makes sure prescript and postscript commands are run only once (even
  if multiple files match the path)
* lastaction: lines to be executed once after all log files that match
  the wildcarded pattern are rotated, after postrotate script is run 
  and only if at least one log is rotated

See USAGE below.

USAGE
====

The default recipe will ensure logrotate is always up to date.

To create application specific logrotate configs, use the
`logrotate_app` definition. For example, to rotate logs for a tomcat
application named myapp that writes its log file to
/var/log/tomcat/myapp.log:

    logrotate_app "tomcat-myapp" do
      cookbook "logrotate"
      path "/var/log/tomcat/myapp.log"
      frequency "daily"
      rotate 30
      create "644 root adm"
    end

To rotate multiple logfile paths, specify the path as an array:

    logrotate_app "tomcat-myapp" do
      cookbook "logrotate"
      path [ "/var/log/tomcat/myapp.log", "/opt/local/tomcat/catalina.out" ]
      frequency "daily"
      create "644 root adm"
      rotate 7
    end

To specify which logrotate options, specify the options as an array:

    logrotate_app "tomcat-myapp" do
      cookbook "logrotate"
      path "/var/log/tomcat/myapp.log"
      options ["missingok", "delaycompress", "notifempty"]
      frequency "daily"
      rotate 30
      create "644 root adm"
    end

License and Author
==================

- Author:: Scott M. Likens (<scott@likens.us>)
- Author:: Joshua Timberman (<joshua@opscode.com>)
- Copyright:: 2009, Scott M. Likens
- Copyright:: 2011-2012, Opscode, Inc.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
