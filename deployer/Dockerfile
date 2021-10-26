FROM marketplace.gcr.io/google/debian9 AS build

RUN apt-get update \
    && apt-get install -y --no-install-recommends gettext

ADD apptest/deployer/yugaware /tmp/test/chart
RUN cd /tmp/test \
    && tar -czvf /tmp/test/yugaware.tar.gz chart/

ARG REGISTRY
ARG TAG

ADD apptest/deployer/schema.yaml /tmp/apptest/schema.yaml
RUN cat /tmp/apptest/schema.yaml \
    | env -i "REGISTRY=$REGISTRY" "TAG=$TAG" envsubst \
    > /tmp/apptest/schema.yaml.new \
    && mv /tmp/apptest/schema.yaml.new /tmp/apptest/schema.yaml

FROM gcr.io/cloud-marketplace-tools/k8s/deployer_helm/onbuild

ARG REGISTRY
ARG TAG

COPY --from=build /tmp/test/yugaware.tar.gz /data-test/chart/
COPY --from=build /tmp/apptest/schema.yaml /data-test/
