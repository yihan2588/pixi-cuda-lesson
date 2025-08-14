FROM ghcr.io/prefix-dev/pixi:noble as build

ARG CUDA_VERSION=12
ARG ENVIRONMENT=gpu

COPY ./pixi.toml /app/
COPY ./pixi.lock /app/
COPY ./.gitignore /app/

WORKDIR /app

RUN export CONDA_OVERRIDE_CUDA=${CUDA_VERSION} && \
    pixi info && \
    pixi install --locked --environment ${ENVIRONMENT} && \
    echo "#!/bin/bash" > /app/entrypoint.sh && \
    pixi shell-hook --environment ${ENVIRONMENT} -s bash >> /app/entrypoint.sh && \
    echo 'exec "$@"' >> /app/entrypoint.sh

FROM ghcr.io/prefix-dev/pixi:noble as final

ARG ENVIRONMENT=gpu

COPY --from=build /app/.pixi/envs/${ENVIRONMENT} /app/.pixi/envs/${ENVIRONMENT}
COPY --from=build /app/pixi.toml /app/pixi.toml
COPY --from=build /app/pixi.lock /app/pixi.lock
COPY --from=build /app/.gitignore /app/.gitignore
# The ignore files are needed for 'pixi run' to work in the container
COPY --from=build /app/.pixi/.gitignore /app/.pixi/.gitignore
COPY --from=build /app/.pixi/.condapackageignore /app/.pixi/.condapackageignore
COPY --from=build /app/entrypoint.sh /app/entrypoint.sh

WORKDIR /app

RUN pixi info && \
    chmod +x /app/entrypoint.sh

ENTRYPOINT ["/app/entrypoint.sh"]
