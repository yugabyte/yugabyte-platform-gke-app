FROM marketplace.gcr.io/google/debian10 AS build

RUN apt-get update \
    && apt-get install -y --no-install-recommends gettext

ADD chart/yugaware /tmp/chart
RUN cd /tmp && tar -czvf /tmp/yugaware.tar.gz chart

ADD apptest/deployer/yugaware /tmp/test/chart
RUN cd /tmp/test \
    && tar -czvf /tmp/test/yugaware.tar.gz chart/

ARG REGISTRY
ARG TAG

ADD schema.yaml /tmp/schema.yaml
RUN cat /tmp/schema.yaml \
    | env -i "REGISTRY=$REGISTRY" "TAG=$TAG" envsubst \
    > /tmp/schema.yaml.new \
    && mv /tmp/schema.yaml.new /tmp/schema.yaml

ADD apptest/deployer/schema.yaml /tmp/apptest/schema.yaml
RUN cat /tmp/apptest/schema.yaml \
    | env -i "REGISTRY=$REGISTRY" "TAG=$TAG" envsubst \
    > /tmp/apptest/schema.yaml.new \
    && mv /tmp/apptest/schema.yaml.new /tmp/apptest/schema.yaml

FROM gcr.io/cloud-marketplace-tools/k8s/deployer_helm

COPY --from=build /tmp/yugaware.tar.gz /data/chart/
COPY --from=build /tmp/test/yugaware.tar.gz /data-test/chart/
COPY --from=build /tmp/apptest/schema.yaml /data-test/
COPY --from=build /tmp/schema.yaml /data/
