# Refactoring Priorities (Nautilus)
**Epic Type:** Implementation/Exploratory Epic

Epic planning and implementation follows the
[Epic Planning and Marathon SOP](https://docs.ghga-dev.de/main/sops/sop001_epic_planning.html).

**Attention: Please do not put any confidential content here.**

## Scope
### Outline:
This epic aims to explore and implement some of the refactoring priorities that are simple to implement
but give especially high benefits.


### Included/Required:
- replace dependency injection framework
- make devcontainer environment more lightweight
- streamline CI and local workflows


## Additional Details:

### Replace Dependency Injection Framework

Currently, we are using the dependency_injector library and are experiencing the following problems:
- difficult debugability:
    - e.g. if arguments are not passed correctly to the constructors, often the container just freezes without
    indications on where the problem is
    - the debugger cannot enter the cython code
- initialization does not happen lazy, i.e. event consumers are also started when starting the REST API of a service
- non-idomatic to python developers, initialization using constructers feels "magic" but unusual to new developers

Alternatives to evaluate:
1. do not use a DI framework at all but define one function that does the dependency resolution per entrypoint
   (i.e. one for the REST API and one for the event consumer)
2. 



## Human Resource/Time Estimation:

Number of sprints required: \<Insert a number.\>

Number of developers required: \<Insert a number.\>
