---
layout: layout.pug
navigationTitle:  V2 Pool Reference
title: V2 Pool Reference
menuWeight: 85
excerpt:

enterprise: false
---

A reference of all Edge-LB pool configurations options in the V2 API.

# V2 Pool Reference

The tables below describe all possible configuration options. The majority of fields have sensible defaults and should be modified with caution.

## Configuration Guidelines

- If a default is not set, it will be left empty, even for objects.
- Set defaults in the object that is furthest from the root object.
- Always set a default for arrays.
- The purpose of "nullable" is to allow the output JSON field to be set to the golang "zero value". Without "nullable", the field will be removed altogether from the resulting JSON.
- Actual validation is done in the code, not expressed in swagger.
- Since an empty boolean is interpreted as "false", don't set a default.
- CamelCase.
- Swagger will only do enum validation if it is a top level definition.

<a name="pool"></a>
# pool
The pool contains information on resources that the pool needs. Changes made to this section will relaunch the tasks.
| Key                         | Type     |  Nullable   |  Properties     | Description    |
| --------------------------- | -------- | ----------- | --------------  | -------------- |
| apiVersion                  | string   |             |                 | The api / schema version of this pool object. Should be V2 for new pools. |
| name                        | string   |             |                 | The pool name. |
| namespace                   | string   |  true       |                 | The DC/OS space (sometimes also referred to as a "group"). |
| packageName                 | string   |             |                 |                |
| packageVersion              | string   |             |                 |                |
| role                        | string   |             |                 | Mesos role for load balancers. Defaults to "slave_public" so that load balancers will be run on public agents. Use "*" to run load balancers on private agents. Read more about Mesos roles at http://mesos.apache.org/documentation/latest/roles/.  |
| cpus                        | number   |             |                 |                |
| cpusAdminOverhead           | number   |             |                 |                |
| mem                         | int32    |             |                 | Memory requirements (in MB). |
| memAdminOverhead            | int32    |             |                 | Memory requirements (in MB). |
| disk                        | int32    |             |                 | Disk size (in MB). |
| count                       | integer  | true        |                 | Number of load balancer instances in the pool. |
| constraints                 | string   | true        |                 | Marathon style constraints for load balancer instance placement. |
| ports                       | array    |             |                 | <ul><li>Override ports to allocate for each load balancer instance.</li><li>Defaults to {{haproxy.frontend.objs[].bindPort}} and {{haproxy.stats.bindPort}}.</li><li>Use this field to pre-allocate all needed ports with or without the frontends present. For example: [80, 443, 9090].</li><li>If the length of the ports array is not zero, only the ports specified will be allocated by the pool scheduler.</li></ul> |
| items                       | int32    |             |                 |                |
| secrets                     | array    |             | <ul><li>[secret](#secrets-prop)</li><li>[file](#secrets-prop)</li></ul> | DC/OS secrets. |
| environmentVariables        | object   |             | [additionalProperties](#env-var) | Environment variables to pass to tasks. Prefix with `ELB_FILE_` and it will be written to a file. For example, the contents of `ELB_FILE_MYENV` will be written to `$ENVFILE/ELB_FILE_MYENV`. |
| autoCertificate             | boolean  |             |                 | Autogenerate a self-signed SSL/TLS certificate. It is not generated by default. It will be written to `$AUTOCERT`. |
| virtualNetworks             | array    |             | <ul><li>[name](#vn-prop)</li><li>[labels](#vn-prop)</li></ul> | Virtual networks to join. |
| haproxy                     |          |             |                 |                 |

<a name="secrets-prop"></a>
## pool.secrets

| Key           | Type        | Description |
| ------------- | ----------- | ----------- |
| secret        | object      |             |

### pool.secrets.secret

| Key           | Type        | Description |
| ------------- | ----------- | ----------- |
| secret        | string      | Secret name. |
| file          | string      | File name.<br />The file `myfile` will be found at `$SECRETS/myfile`. |

<a name="env-var"></a>
## pool.environmentVariables

| Key                   | type        | Description |
| --------------------- | ----------- | ----------- |
| additionalProperties  | string      | Environment variables to pass to tasks.<br />Prefix with "ELB_FILE_" and it will be written to a file. For example, the contents of "ELB_FILE_MYENV" will be written to "$ENVFILE/ELB_FILE_MYENV". |

<a name="vn-prop"></a>
## pool.virtualNetworks

| Key           | Type        | Description |
| ------------- | ----------- | ----------- |
| name          | string      | The name of the virtual network to join. |
| labels        | string      | Labels to pass to the virtual network plugin. |

<a name="haproxy-prop"></a>
# pool.haproxy

| Key             | Type    | Description         |
| --------------- | ------- | ------------------- |
| stats           |         |                     |
| frontends       | array   | Array of frontends. |
| backends        | array   | Array of backends.  |

<a name="stats-prop"></a>
# pool.haproxy.stats

| Key            | Type     |
| -------------- | -------- |
| bindAddress    | string   |
| bindPort       | int 32   |

<a name="frontend-prop"></a>
# pool.haproxy.frontend

| Key             | Type    | Properties     | Description    | x-nullable | Format |
| --------------- | ------- | -------------- | -------------- | ---------- | ------ |
| name            | string  |                | Defaults to `frontend_{{bindAddress}}_{{bindPort}}`.  |   |   | bindAddress     | string  |                | Only use characters that are allowed in the frontend name. Known invalid frontend name characters include `*`, `[`, and `]`.  |   |   |
| bindPort        | integer |                | The port (e.g. 80 for HTTP or 443 for HTTPS) that this frontend will bind to.  |   | int32  |
| bindModifier    | string  |                | Additional text to put in the bind field   |   |   |
| certificates    | array   |                | SSL/TLS certificates in the load balancer.<br /><br />For secrets, use `$SECRETS/my_file_name`<br />For environment files, use `$ENVFILE/my_file_name`<br />For autoCertificate, use `$AUTOCERT`.<br />type: string  |   |   |
| redirectToHttps | object  | <ul><li>[except](#redirect-https-prop)</li><li>[items](#redirect-https-prop)</li></ul>  | Setting this to the empty object is enough to redirect all traffic from HTTP (this frontend) to HTTPS (port 443). Default: except: [] |   |   |
| miscStrs        | array of strings  |   | Additional template lines inserted before use_backend  |   |   |
| protocol        |   |   | The frontend protocol is how clients/users communicate with HAProxy.  |   |   |
| linkBackend     | object  | <ul><li>defaultBackend</li><li>map</li></ul>  | This describes what backends to send traffic to. This can be expressed with a variety of filters such as matching on the hostname or the HTTP URL path.<br />Default: map: []   |   |   |

<a name="redirect-https-prop"></a>
## pool.haproxy.frontend.redirectToHttps

| Key             | Type    | Properties  | Description     |
| --------------- | ------- | ----------- | --------------- |
| except          | array   |             | You can additionally set a whitelist of fields that must be matched to allow HTTP.  |
| items           | object  | <ul><li>[host](#items-prop)</li><li>[pathBeg](#items-prop)</li></ul> | Boolean AND will be applied with every selected value. |

<a name="items-prop"></a>
### pool.frontend.redirectToHttps.items

| Key             | Type    | Description |
| --------------- | ------- | ----------- |
| host            | string  | Match on host. |
| pathBeg         | string  | Math on path.  |

## pool.haproxy.frontend.linkBackend

| Key             | Type    | Properties | Description |
| --------------- | ------- | ---------- | ----------- |
| defaultBackend  | string  |            | This is default backend that is routed to if none of the other filters are matched.  |
| map             | array   | <ul><li>[backend](#map-prop)</li><li>[hostEq](#map-prop)</li><li>[hostReg](#map-prop)</li><li>[pathBeg](#map-prop)</li><li>[pathEnd](#map-prop)</li><li>[pathReg](#map-prop)</li></ul> | This is an optional field that specifies a mapping to various backends. These rules are applied in order.<br />"Backend" and at least one of the condition fields must be filled out. If multiple conditions are filled out, they will be combined with a boolean "AND". |

<a name="map-prop"></a>
### pool.frontend.linkBackend.map

| Key             | Type    | Description |
| --------------- | ------- | ----------- |
| backend         | string  |             |
| hostEq          | string  | Must be all lowercase. |
| hostReg         | string  | Must be all lowercase. It is possible for a port (e.g. `foo.com:80`) to be in this regex.  |
| pathBeg         | string  |             |
| pathEnd         | string  |             |
| pathReg         | string  |             |

<a name="backend-prop"></a>
# pool.haproxy.backend

| Key             | Type    | Properties     | Description    |
| --------------- | ------- | -------------- | -------------- |
| name            | string  |                | The name the frontend refers to. |
| protocol        | string  |                | The backend protocol is how HAProxy communicates with the servers it is load balancing. |
| rewriteHttp     |         |                | Manipulate HTTP headers. There is no effect unless the protocol is either HTTP or HTTPS. |
| balance         | string  |                | Load balancing strategy. E.g., roundrobin, leastconn, etc. |
| customCheck     | object  | <ul><li>[httpchk](#customCheck-prop)</li><li>[httpchkMiscStr](#customCheck-prop)</li><li>[sslHelloChk](#customCheck-prop)</li><li>[miscStr](#customCheck-prop)</li></ul>  | Specify alternate forms of healthchecks.  |
| miscStrs        | array of strings |       | Additional template lines inserted before servers  |
| services        | array   |                | Array of backend service selectors.  |

<a name="customCheck-prop"></a>
## pool.haproxy.backend.customCheck

| Key            | Type     |
| -------------  | -------- |
| httpchk        | boolean  |
| httpchkMiscStr | string   |
| sslHelloChk    | boolean  |
| miscStr        | string   |

<a name="#rewrite-prop"></a>
# pool.haproxy.backend.rewriteHttp

| Key             | Type    | Properties     | Description    |
| --------------- | ------- | -------------- | -------------- |
| host            | string  |                | Set the host header value. |
| path            | object  | <ul><li>[fromPath](#path-prop)</li><li>[toPath](#path-prop)</li></ul>  | Rewrite the HTTP URL path. All fields required, otherwise it's ignored.  |
| request         |         |                |                |
| response        |         |                |                |
| sticky          | object  | <ul><li>[enabled](#sticky-prop)</li><li>[customStr](#sticky-prop)</li></ul>  | Sticky sessions via a cookie.<br />To use the default values (recommended), set this field to the empty object.  |

<a name="path-prop"></a>
## pool.haproxy.backend.rewriteHttp.path

| Key             | Type    |
| --------------- | ------- |
| fromPath        | string  |
| toPath          | string  |

<a name="sticky-prop"></a>
## pool.haproxy.backend.rewriteHttp.sticky

| Key             | Type    | nullable   |
| --------------- | ------- | ---------- |
| enabled         | boolean | true       |
| customStr       | string  |            |

<a name="rewrite-req-prop"></a>
# pool.haproxy.backend.rewriteHttp.request

| Key                         | Type       | nullable   |
| --------------------------- | ---------- | ---------- |
| forwardfor                  | boolean    | true       |
| xForwardedPort              | boolean    | true       |
| xForwardedProtoHttpsIfTls   | boolean    | true       |
| setHostHeader               | boolean    | true       |
| rewritePath                 | boolean    | true       |

<a name="rewrite-resp-prop"></a>
# pool.haproxy.backend.rewriteHttp.response

| Key             | Type       | nullable   |
| --------------- | ---------- | ---------- |
| rewriteLocation | boolean    | true       |

<a name="service-prop)"></a>
# pool.haproxy.backend.service

| Key             | Type       |
| --------------- | ---------- |
| marathon        | object     |
| mesos           | object     |
| endpoint        | object     |

<a name="service-marathon-prop)"></a>
# pool.haproxy.backend.service.marathon

| Key                  | Type      | Description                                                       |
| -----------          | --------- | -----------                                                       |
| serviceID            | string    | Marathon pod or application ID.                                   |
| serviceIDPattern     | string    | serviceID as a regex pattern.                                     |
| containerName        | string    | Marathon pod container name, optional unless using Marathon pods. |
| containerNamePattern | string    | containerName as a regex pattern.                                 |

<a name="service-mesos-prop)"></a>
# pool.haproxy.backend.service.mesos

| Key                  | Type      | Description                       |
| -----------          | --------- | -----------                       |
| frameworkName        | string    | Mesos framework name.             |
| frameworkNamePattern | string    | frameworkName as a regex pattern. |
| frameworkID          | string    | Mesos framework ID.               |
| frameworkIDPattern   | string    | frameworkID as a regex pattern.   |
| taskName             | string    | Mesos task name.                  |
| taskNamePattern      | string    | taskName as a regex pattern.      |
| taskID               | string    | Mesos task ID.                    |
| taskIDPattern        | string    | taskID as a regex pattern.        |

<a name="service-endpoint-prop)"></a>
# pool.haproxy.backend.service.endpoint

| Key         | Type      | Description                                                                                   |
| ----------- | --------- | -----------                                                                                   |
| type        | string    | Enum field, can be `AUTO_IP`, `AGENT_IP`, `CONTAINER_IP`, or `ADDRESS`. Default is `AUTO_IP`. |
| miscStr     | string    | Append arbitrary string to add to the end of the "server" directive.                          |
| check       | object    | Enable health checks. These are by default TCP health checks. For more options see "customCheck". These are required for DNS resolution to function properly. |
| address     | string    | Server address override, can be used to specify a cluster internal address such as a VIP. Only allowed when using type `ADDRESS`. |
| port   | integer | Port number.                                                                                  |
| portName | integer | Name of port.                                                                      |
| allPorts  | boolean | Selects all ports defined in service when `true`.                     |

<a name="service-endpoint-check-prop)"></a>
# pool.haproxy.backend.service.endpoint.check

| Key         | Type      |
| ----------- | --------- |
| enabled     | boolean   |
| customStr   | string    |

<a name="error-prop"></a>
# error

| Key             | Type        |
| --------------- | ----------- |
| code            | int32       |
| message         | string      |
