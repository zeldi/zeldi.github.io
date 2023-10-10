---
layout: post
title: Deploy Keycloak (quarkus version) in Docker
permalink: keycloak-docker
date: 2023-09-15
tags: ["keycloak","docker","postgresql"]
category: devops
---


# Intro
Keycloak is a widely-used, free software that helps manage identity and access management for applications, offering robust services for authentication and authorization.

# Obselete version of Keycloak.
The JBOSS version of Keycloak is obselete. Therefore, the "jboss/keycloak" Docker image we've been using has not been updated for a year, which poses security risks and means we're not benefiting from new features. To address this issue, we should transition to the latest version of Keycloak, which is based on Quarkus. We should consider using the official Keycloak image available at "quay.io/keycloak/keycloak" to ensure we have the most recent version of Keycloak.

# Challenge

One of issue with documentation of quarkus-based keycloak is insuffient howto for integrating Keycloak with external databases (i.e Postgres). Hence, it causes confusion for users trying to set up Keycloak with their chosen database system.

Moreover, when linking Keycloak to Postgres, the documentation fails to provide clear instructions for executing the initial "kc.sh build" command. This omission adds unnecessary complexity for users.

# Solution
To address the above difficulties, we can craft a custom Dockerfile that addresses the challenges and establishes a smooth Keycloak configuration. Below is an illustration of a `Dockerfile` that rectifies the concerns:

```yaml
ARG KEYCLOAK_VERSION

FROM quay.io/keycloak/keycloak:$KEYCLOAK_VERSION as builder

# Configure postgres database vendor
ENV KC_DB=postgres

ENV KC_FEATURES="token-exchange,scripts,preview"

WORKDIR /opt/keycloak

# If run the image in kubernetes, switch and active below line.
# RUN /opt/keycloak/bin/kc.sh build --cache=ispn --cache-stack=kubernetes --health-enabled=true --metrics-enabled=true
RUN /opt/keycloak/bin/kc.sh build --cache=ispn --health-enabled=true --metrics-enabled=true

FROM quay.io/keycloak/keycloak:$KEYCLOAK_VERSION

LABEL image.version=$KEYCLOAK_VERSION

COPY --from=builder /opt/keycloak/ /opt/keycloak/

# If any themes
# COPY themes/<nice-themes> /opt/keycloak/themes/<nice-themes>

# https://github.com/keycloak/keycloak/issues/19185#issuecomment-1480763024
USER root
RUN sed -i '/disabledAlgorithms/ s/ SHA1,//' /etc/crypto-policies/back-ends/java.config
USER keycloak

RUN /opt/keycloak/bin/kc.sh show-config

ENTRYPOINT ["/opt/keycloak/bin/kc.sh"]
```

The `Dockerfile` builds a Keycloak image by using the "quay.io/keycloak/keycloak" image as a base. It sets up the Postgres database vendor configuration, defines the desired Keycloak functionalities, and executes the essential `kc.sh build` command to guarantee a correct Keycloak configuration.

To build the updated Keycloak image, please execute the following command:

```bash
docker build --build-arg KEYCLOAK_VERSION=21.1.1 -t keycloak --progress=plain --no-cache .
```

Once the image is create, we can start our Keycloak version in Docker by running these commands:
```bash
docker run -d --name keycloak-v21 -p 8080:8080 \
         -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=password \
         keycloak \
         --verbose \
         start \
         --hostname-strict=false \
         --optimized \
         --http-enabled=true \
         --db-url="jdbc:postgresql://<postgresql_db_hostname>/keycloak" \
         --db-username=keycloak \
         --db-password=password

docker logs -f keycloak-v21
```

The keycloak page can be accessed via url http://<you-ip-address>:8080. Refer to https://www.keycloak.org/getting-started/getting-started-docker for detail on getting started with keycloak.

# Outro

By upgrading to the quarkus-based keycloak, we are exposed with the latest features for Keycloak. Furthermore, using "quay.io/keycloak/keycloak" image and employing a custom Dockerfile, we can effectively address the issues associated from outdated Keycloak images (JBOSS version) and inadequate documentation regarding database integration.

This approach guarantees a secure and current Keycloak environment, along with comprehensive guidance for seamlessly integrating Keycloak with Postgres or other external databases.

We the explaination above users can proficiently set up and manage Keycloak within Docker, harnessing its robust identity and access management features.


# References:
*  https://www.keycloak.org/getting-started/getting-started-docker
*  https://www.keycloak.org/server/containers


