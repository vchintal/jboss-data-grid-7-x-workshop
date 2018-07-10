# Persistence

In this section we will explore how to back up the cache with persitence store such that:

1. We achieve durability of the entries
2. We use the store as an overflow storage in case we cannot fit all of the entries in the memory 

In this lab we will try to different combination of cache configuration invoving persistence:

1. No eviction enabled
2. Eviction enabled
3. Eviction enabled with passivation

