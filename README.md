# Python library for using the UNICORE REST API

This library covers part of the UNICORE REST API, making common tasks
like file access, job submission and management, workflow submission
and management more convenient, and integrating UNICORE features better
with typical Python usage.

For the full, up-to-date documentation of the REST API,
see https://sourceforge.net/p/unicore/wiki/REST_API

Development of this library was funded in part by the Human Brain Project

For more information about the Human Brain Project, see https://www.humanbrainproject.eu/

See LICENSE file for licensing information

## Getting started with pyUNICORE

Install from PyPI with

    pip install -U pyunicore

Sample code to create a client for a UNICORE site

    import pyunicore.client as unicore_client
    import json
    from base64 import b64encode
   
    base_url = "https://localhost:8080/DEMO-SITE/rest/core"

    # authenticate with username/password
    token = b64encode(b"demouser:test123").decode("ascii")
    transport = unicore_client.Transport(token, oidc=False)
    
    client = unicore_client.Client(transport, base_url)
    print(json.dumps(client.properties, indent = 2))

## Running a sample job and reading result data

    my_job = {'Executable': 'date'}
    
    job = client.new_job(job_description=my_job, inputs=[])
    print(json.dumps(job.properties, indent = 2))
    
    job.poll() # wait for job to finish
 
    work_dir = job.working_dir
    print(json.dumps(work_dir.properties, indent = 2))
    
    stdout = work_dir.stat("/stdout")
    print(json.dumps(stdout.properties, indent = 2))
  
    content = stdout.raw().read()
    print(content)

## Connecting to a Registry and listing all registered services

    import pyunicore.client as unicore_client
    import json, b64encode

    registry_url = "https://localhost:8080/REGISTRY/rest/registries/default_registry"

    # authenticate with username/password
    token = b64encode(b"demouser:test123").decode("ascii")
    transport = unicore_client.Transport(token, oidc=False)

    r = unicore_client.Registry(tr, registry_url)
    print(r.site_urls)

## Helpers

The `pyunicore.helpers` module provides a set of higher-level APIs:

- Different authentication methods:
  1. user-password (`pyunicore.helpers.UserAuthentication`)
  2. bearer token (`pyunicore.helpers.TokenAuthentication`)
- Creating a `pyunicore.client.Transport` (`pyunicore.helpers.create_transport`).
- Connecting to
  - a registry (`pyunicore.helpers.connect_to_registry`).
  - a site via a Registry URL (`pyunicore.helpers.connect_to_site_from_registry`).
  - a site via its core URL (`pyunicore.helpers.connect_to_site`).
- Defining a job description as a dataclass and easily converting to a `dict` as required
  by `pyunicore.client.Client.new_job` (`pyunicore.helpers.JobDescription`).
- All possible job statuses that may be returned by the jobs API (`pyunicore.helpers.JobStatus`).

### Creating a transport

```Python
import json
from pyunicore import helpers

authentication = helpers.UserAuthentication(user="demouser", password="test123")

transport = helpers.create_transport(authentication )
print(json.dumps(transport.properties, indent=2))
```

### Connecting to a registry

```Python
import json
from pyunicore import helpers


registry_url = "https://localhost:8080/REGISTRY/rest/registries/default_registry"

authentication = helpers.UserAuthentication(user="demouser", password="test123")

client = helpers.connect_to_registry(
    registry_url=registry_url,
    authentication=authentication,
)
print(json.dumps(client.properties, indent=2))
```

### Connecting to a site via a registry

```Python
import json
from pyunicore import helpers

registry_url = "https://localhost:8080/REGISTRY/rest/registries/default_registry"
site = "DEMO-SITE"

authentication = helpers.UserAuthentication(user="demouser", password="test123")

client = helpers.connect_to_site_from_registry(
    registry_url=registry_url,
    site_name=site,
    authentication=authentication,
)
print(json.dumps(client.properties, indent=2))
```

### Connecting to a site directly

```Python
import json
from pyunicore import helpers

site_url = "https://localhost:8080/DEMO-SITE/rest/core"

authentication = helpers.UserAuthentication(user="demouser", password="test123")

client = helpers.connect_to_site(
    site_api_url=site_url ,
    authentication=authentication,
)
print(json.dumps(client.properties, indent=2))
```

### Defining a job

```Python
from pyunicore import helpers

client = ...

resources = helpers.Resources(nodes=4)
job = helpers.JobDescription(
    executable="ls",
    project="demoproject",
    resources=resources,
)

client.new_job(job.to_dict())
```
