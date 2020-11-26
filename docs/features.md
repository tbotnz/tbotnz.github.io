# features

- Speaks REST and JSON RPC northbound, then CLI over SSH or Telnet or NETCONF/RESTCONF southbound to your network devices
- Turns any Python script into a easy to consume, asynchronous and documented API with webhook support
- Large amount of supported network device vendors thanks to [napalm](https://github.com/napalm-automation/napalm), [netmiko](https://github.com/ktbyers/netmiko),  [ncclient](https://github.com/ncclient/ncclient) and [requests](https://github.com/psf/requests)
- Built in multi-level abstraction interface for network service lifecycle functions for create, retrieve and delete and validate
- In band service inventory
- Ability to write your own [service models and templates](https://github.com/tbotnz/netpalm/tree/master/netpalm/backend/plugins/extensibles/j2_service_templates) using your own existing [jinja2 templates](https://github.com/tbotnz/netpalm/tree/master/netpalm/backend/plugins/extensibles/custom_scripts)
- Well documented API with [postman collection](https://documenter.getpostman.com/view/2391814/T1DqgwcU?version=latest#33acdbb8-b5cd-4b55-bc67-b15c328d6c20) full of examples and every instance gets it own openAPI 3 and self documenting for your service templates and scripts
- Supports pre and post checks accross CLI devices raising exceptions and not deploying config as required
- Multiple ways to queue jobs to devices, either pinned strict task by task to each device or pooled first in first out
- Modern, container based scale out architecture supported by every component
- Highly [configurable](https://github.com/tbotnz/netpalm/blob/master/config/config.json) for all aspects of the platform
- Leverages an ecrypted Redis layer providing caching and queueing of jobs to and from devices

## caching
* Supports the following per-request configuration (`/getconfig` routes only for now)
    * permit the result of this request to be cached (default: false), and permit this request to return cached data
    * hold the cache for 30 seconds (default: 300.  Should not be set above `redis_task_result_ttl` which defaults to 500)
    * do NOT invalidate any existing cache for this request (default: false)
    ```json
      {
        "cache": {
          "enabled": true,
          "ttl": 30,
          "poison": false
        }
      }
     ```
  
* Supports the following global configuration:
    * Enable/Disable caching: `"redis_cache_enabled": true`
        for caching to apply it must be enabled BOTH globally and in the request itself
    * Default TTL:  `"redis_cache_default_timeout": 300`

* Any change to the request payload will result in a new cache key EXCEPT:
    * JSON formatting.  `{ "x": 1, "y": 2 } == {"x":1,"y":2}`
    * Dictionary ordering:  `{"x":1,"y":2} == {"y":2,"x"1}`
    * changes to cache configuration (e.g. changing the TTL, etc)
    * `fifo` vs `pinned` queueing strategy

* Any call to any `/setconfig` route for a given host:port will poison ALL cache entries for that host:port
    * Except `/setconfig/dry-run` of course 
