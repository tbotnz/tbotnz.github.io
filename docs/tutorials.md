# tutorials

## using caching
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

