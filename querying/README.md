# Querying

The entries within a typical JDG cache is mostly retrived via their keys, one at a time via the calls such as `cache.get(...)` but what if:
* The key is not known or unavailable
* The goal is to retrieve not just one but multiple entries based on a criteria such as `age > 50`  for, let's say, all of the `Person` entities in the cache

This is where JDG Query API would help retrieve matching entities from the cache. 

For a comprehensive list of what Querying features exist and to which modes (embedded/client-server) of usage they apply to, can be accessed via the [query comparision official documentation](https://access.redhat.com/documentation/en-us/red_hat_jboss_data_grid/7.2/html-single/developer_guide/#querying_comparison).