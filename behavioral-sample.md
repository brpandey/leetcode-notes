- anchoring to status, accomplishments (e.g. principal engineer)
- compare levels - salary, responsibilities (e.g. levels.fyi )
- hedge against risk (don't get down-leveled)

- anchoring story - status challenge
  - if challenge contains small potatoes => not a senior
  - else possibly a senior

- inherited software
  - e.g. provides a lot of data, but:
    - buggy, lot of operational data
    - uses a deprecated service that can be a security risk
    - need to move clients off service

  - simple solutions
    - aligned with charter of team (no security vulnerable software or 3rd party dependencies)
    - unit tests/ integration tests
    - when process crashes, people manually restart process (deep dive)
    - decommission service
    - create a pass thru service

- summary: on balance => imperfect success


- owned a backend service
  - prototype:
    - latency is prohibitive above threshhold
    - api should be synchronous
  - makes deadline more challenging but better prepared

- when folks push back
  - influence with data driven arguments
  - create a quick design document and tech spec and flesh out the roles

- bias toward fixing later
  - realize api has high latency
  - do a spike on how api can be rewritten

- start to plan roadmap of how to rewrite it
- set direction, really opinionated, be good

* temptation to embrace an immediate gain can offset a future direction
