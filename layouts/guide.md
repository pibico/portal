# pibi... Guide

Small description pibi...

## Description
Long description of the app without including all the details of its functional features, which are described in another section.

## Requirements
Requires a Frappe server instance (refer to [https://github.com/frappe/frappe](https://github.com/frappe/frappe)), and ...

## Compatibility
Pibi... has been tested on Frappe/ERPNext version-15 ...

## Installation
From the frappe-bench folder, execute
```
$ bench get-app pibicut https://github.com/pibico/pibi....git
$ bench install-app pibi...
```
If you are using a multi-tenant environment, use the following command
```
$ bench --site site_name install-app pibi...
```
### Setting 
Optional subsection for apps that maintain parameters. Remove if no settings are available.

## Update
Run updates with
```
$ bench update
```
In case you update from the sources and observe errors, make sure to update dependencies with
```
$ bench update --requirements
```

## Features
Functional features described in detail. Images may be included and should be located in docs/apps/pibi.../images/

![Pibi... Example](images/pibi..._exe0.png)

## License

MIT
