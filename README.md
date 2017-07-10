Splunk-Pickaxe
==============

A tool for syncing your repo of Splunk objects with a Splunk instance(s).

This provides a development workflow for Splunk components (i.e. dashboards,
alerts, reports, etc) and an easy way to apply them consistently.

Getting Started
---------------

Install the gem,

    gem install splunk-pickaxe

Create your repo,

    mkdir my-splunk-repo
    cd my-splunk-repo
    pickaxe init

Update the `.pickaxe.yml` file with your app name from Splunk. This should
be the name from the URL when you visit your Splunk app. (i.e. if your url is
`https://my-splunk.com/en-US/app/my_splunk_app/search` my app is `my_splunk_app`).

You also need to add your Splunk environment(s). So your `.pickaxe.yml` should
look like this,

```yaml
namespace:
  # The application in which to create the Splunk knowledge objects
  app: MY_SPLUNK_APP

environments:
  ENVIRONMENT_NAME: SPLUNK_API_URL (i.e. https://search-head.my-splunk.com:8089)
```

Add some Splunk objects. See [example repo](example-repo) or below for format.

Sync your repo with Splunk,

    pickaxe sync ENVIRONMENT_NAME

Where `ENVIRONMENT_NAME` is the name of one of the environments configured in
your `.pickaxe.yml`. These map to different Splunk instances.

By default this command assumes the user running the command has a Splunk account
and access to make these changes in the configured Splunk application. Your
password will be requested when run. Alternatively you can make use of the
options `--user` and `--password`.

Splunk Objects
--------------

Currently the following objects are supported,

 * [alerts](http://docs.splunk.com/Documentation/Splunk/latest/RESTREF/RESTsearch#saved.2Fsearches)
 * [dashboards](http://docs.splunk.com/Documentation/Splunk/latest/RESTREF/RESTknowledge#data.2Fui.2Fviews.2F.7Bname.7D)
 * [eventtypes](http://docs.splunk.com/Documentation/Splunk/latest/RESTREF/RESTknowledge#saved.2Feventtypes)
 * [reports](http://docs.splunk.com/Documentation/Splunk/latest/RESTREF/RESTsearch#saved.2Fsearches)
 * [tags](http://docs.splunk.com/Documentation/Splunk/latest/RESTREF/RESTknowledge#search.2Ftags.2F.7Btag_name.7D)
 * [field_extractions](http://docs.splunk.com/Documentation/Splunk/latest/RESTREF/RESTknowledge#data.2Fprops.2Fextractions)

### Alerts

To add a new alert to the repo simply create a new `ALERT.yml` file under `alerts`.
This YAML file should contain the following at minimum,

```yaml
name: ALERT NAME
config:
  # Search query of events used to trigger alert
  search: >
    MY SEARCH
```

It will be populated with the alert defaults which includes,

 * Running hourly
 * Emailing everyone listed in your `.pickaxe.yml`
 * Alerting if the search results in any events

You can override these defaults or any other property by specifying the property
under the `config` section. [This doc](http://docs.splunk.com/Documentation/Splunk/latest/RESTREF/RESTsearch#saved.2Fsearches.2F.7Bname.7D)
contains all the properties for an alert.

#### Common Overrides

 * To tweak the schedule set `cron_schedule` in the `config` section. By default its setup to run every hour. This should be a cron value (i.e. `0 10 * * 1` or run every Monday at 10am)
 * To tweak the length of the search or how far back the set `dispatch.earliest_time` in the `config` section. By default this is setup to search over the last hour. You could change this to `-7d@h` to search the last 7 days

### Dashboards

To add a new dashboard create a new `DASHBOARD.xml` file under `dashboards`.
The name of the file should be the name of the dashboard. If the file is
`my_dashboard.xml`, this means the name is `my_dashboard`.

The file should contain the XML source of your dashboard. You can get this
by selecting your dashboard in Splunk, then Edit -> Edit Source. Copy the XML
and paste it into the file.

### Eventtypes

To add a new eventtype create a new `eventtype.yml` file under `eventtypes`.
The name of the file should be the name of the eventtype. If the file is
`my_eventtype.yml`, this means the name is `my_eventtype`.

```yaml
name: EVENTTYPE NAME
config:
  disabled: <1|0> Set to 1 to disable
  search: index=my_index more search things
  priority: <1-10> 1 is highest priority and 10 is lowest
  tags: <string> Enter a comma-separated list of tags
  ...
```

### Reports

To add a new report to the repo simply create a new `REPORT.yml` file under `reports`.
This YAML file should contain the following at minimum,

```yaml
name: REPORT NAME
config:
  # Search query of events in the report
  search: >
    MY SEARCH
```

It will be populated with the alert defaults which includes,

 * Running hourly
 * Emailing everyone listed in your `.pickaxe.yml`

You can override these defaults or any other property by specifying the property
under the `config` section. [This doc](http://docs.splunk.com/Documentation/Splunk/latest/RESTREF/RESTsearch#saved.2Fsearches.2F.7Bname.7D)
contains all the properties for an report.

#### Common Overrides

 * To tweak the schedule set `cron_schedule` in the `config` section. By default its setup to run every hour. This should be a cron value (i.e. `0 10 * * 1` or run every Monday at 10am)
 * To tweak the length of the search or how far back the set `dispatch.earliest_time` in the `config` section. By default this is setup to search over the last hour. You could change this to `-7d@h` to search the last 7 days

### Tags

To add a new tag to the repo simply create a new `TAG.yml` file under `tags`.
This YAML file should contain the following at minimum,

```yaml
name: TAG
fields:
  - source::/some/file
  - sourcetype::my_type
  ...
```

The important config here is the `fields` key which contains the array of fields
you want in your tag. The value should be of the form `FIELD::VALUE` where `FIELD`
can be things like `source`, `sourcetype`, `eventtype` or any other Splunk field.

### Field Extractions

To add a new field extraction to the repo simply create a new `field_extraction.yml`
file under `field_extractions`. This YAML file should contain the following,

```yaml
name: my field extraction
config:
  # The props.conf stanza to which this field extraction applies, e.g. the
  # sourcetype or source that triggers this field extraction
  stanza: See Comment Above
  # If using EXTRACT type this is the regular expression
  # If using REPORT type specify a comma- or space-delimited list of transforms.conf
  # stanza names that define the field transformations to apply.
  value: See Comment Above
  # Use EXTRACT for inline regular expressions. Use REPORT for a transforms.conf stanza
  type: EXTRACT (or) REPORT
```

### Generic Configuration

Most Splunk objects offer the following additional configuration,

 * `envs`: The list of environments the object should be deployed to

```yaml
name: MY_OBJECT_NAME
envs:
  # Only deploy to dev environment
  - dev
```

By default if `envs` is not provided the object will be imported to all
environments.

Config
------

The `.pickaxe.yml` file contains the config for your Splunk resources. You can add,

* `environments` : A hash of environment names (key) to Splunk url
* `namespace`:
    * `app`: The name of your Splunk application to deploy objects to
    * `sharing`: The sharing settings for the Splunk resources (default=`app`)
* `emails`: An array of emails used for all reports and alerts (default=`[]`)

Contributing
------------

See [CONTRIBUTING.md](CONTRIBUTING.md)

LICENSE
-------

Copyright 2015 Cerner Innovation, Inc.

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at

[http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0) Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.