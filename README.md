# Spring Cloud Config Server Demo

## Summary

* Centralizing configuration
* Decoupling changes/deployment of code and configuration
* Control and audit configuration changes
* Configuration as Code

## Deployment

```puml
@startuml
left to right direction
node node1 {
    component app_a_instance_1
    component app_b_instance_1
}
node node2 {
    component app_a_instance_2
    component app_b_instance_2
}



[app_a_instance_1] ..> REST: GET
[app_b_instance_1] ..> REST: GET
[app_a_instance_2] ..> REST: GET
[app_b_instance_2] ..> REST: GET

node node3 {
    REST <-- [config_server]: provided
}
database git_repo
git_repo <. config_server: read
@enduml
```

Configuration is wrote in property and yaml files, and stored in Git repository. Config server is reading content of Git repository, and expose content as read only REST API. Application read specific configuration by request corresponding URL to config server.

## REST API

Expose two types of resources:

1. Profile
2. File

**Profile**

HTTP Method|URI Pattern                        |Response MIME
-----------|-----------------------------------|-------------
GET        |/{application-name}/{profile-name} |application/json

**File**

HTTP Method|URI Pattern                        |Response MIME
-----------|-----------------------------------|-------------
GET        |/{application-name}-{profile-name}.{suffix} |text/plain


## Application Read Configuration

```puml
@startuml
participant app_a_instance_1
participant config_server
database git_repo

activate config_server
activate app_a_instance_1

app_a_instance_1 -> config_server: GET(/{application-name}/{profile-name})
config_server -> git_repo: pull
app_a_instance_1 <-- config_server: json
note left
{  
   "name":"app1",
   "profiles":[  
      "default"
   ],
   "label":null,
   "version":"40215a6f5b25f2315a4d2fa3f62f67d031aacbff",
   "state":null,
   "propertySources":[  
      {  
         "name":"git@github.com:rscai/config-repo-dev.git/app1.yml",
         "source":{  
            "datasource.driverClass":"com.oracle.jdbc.Driver",
            "datasource.url":"jdbc:thin://hostname:port/serviceName",
            "pool.max":10,
            "pool.min":2
         }
      }
   ]
}
end note
@enduml
```

## Change Configuration

```puml
@startuml
|Maker|
start
:create feature branch from master;
:commit changes to feature branch;
:create pull request from feature branch to master;
|Checker|
:review pull request;
if (approve?) then (yes)
    :merge into master;
else (no)
    :reject;
endif
end
@enduml
```

Config server is only reading master branch. 
Maker must not commit configuration changes to master branch directly. Instead, Maker should create a feature branch from master branch, then commit configuration changes to the feature branch.
After all configuration changes is ready on feature branch, then maker create a pull request from feature branch to master branch.
Checker review the pull request and decide if accept these changes.

## Promote Configuration Changes

All code changes must go through mature verification process. Same for configuration changes.

```puml
@startuml
database dev_repo
database sit_repo
database uat_repo
database prod_repo

[dev_repo] .> [sit_repo]: pull request
[sit_repo] .> [uat_repo]: pull request
[uat_repo] .> [prod_repo]: pull request
@enduml
``` 

Maker create pull request from lower environment configuration repository (master branch) to higher environment configuration repository (master branch).
Checker review pull request and decide if accept changes.