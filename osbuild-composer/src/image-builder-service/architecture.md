## Current architecture

This diagram shows the overall architecture, and each sub-section goes into details about the three main
components, which are run as independent services.

```plantuml
@startuml

skinparam defaultTextAlignment center

!define SPRITESURL https://raw.githubusercontent.com/plantuml-stdlib/gilbarbara-plantuml-sprites/v1.0/sprites
!includeurl SPRITESURL/openshift.puml

!include <aws/common>
!include <aws/Compute/AmazonEC2/AmazonEC2>
!include <aws/Compute/AmazonEC2/instances/instances>
!include <aws/Database/AmazonRDS/AmazonRDS.puml>
!include <aws/Database/AmazonRDS/PostgreSQLinstance/PostgreSQLinstance>

skinparam linetype ortho

actor actor

component "<$openshift>\napi.openshift.com" as oc {
    rectangle "image-builder-composer"

}

component "<$openshift>\nconsole.redhat.com" as crc {
    rectangle "image-builder-crc"

}

AMAZONEC2(ec2) {
    INSTANCES(workerfleet, worker fleet)
}

AMAZONRDS(ibdb) {
    POSTGRESQLINSTANCE(db1, image-builder-db\ninsights account)
}

AMAZONRDS(composerdb) {
    POSTGRESQLINSTANCE(db2, composer-db\nappsre account)
}

actor -right-> [image-builder-crc]: **A**
[image-builder-crc] -down-> db1
[image-builder-composer] -down-> db2
[image-builder-crc] -right-> [image-builder-composer]: **B**
workerfleet -> [image-builder-composer]: **C**

legend right
A. User connection to the UI under crc/insights/image-builder, using RH SSO for auth.
B. ib-crc connects to composer to queue/query images, using RH SSO for auth.
C. The workers connect to composer to request jobs and post results.
endlegend

@enduml
```

The metadata defining the service for App-Interface is kept upstream and open as templates for both the [osbuild-composer](https://github.com/osbuild/osbuild-composer/blob/main/templates/composer.yml) and [image-builder components](https://github.com/osbuild/image-builder/blob/main/templates/image-builder.yml).
The tooling to operate the service is to large parts open source and publicly accessible, e.g. qontract in the form of [qontract-server](https://github.com/app-sre/qontract-server), [qontract-reconcile](https://github.com/app-sre/qontract-reconcile).
The architecture documents in this section comply with the AppSRE contract.
