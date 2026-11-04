# syntax=docker/dockerfile:1

# Base image for the `oh-my-openagent` sandbox kit.
#
# Pre-bakes `omo-ai` so sandbox creation is fast: the `omo` binary is already
# on the image and `sbx create` pays no install step.
#
# Built and published by build-and-publish-kits.yml (see ../PUBLISHING.md) --
# no bespoke workflow needed, same as pi/aider/codex/opencode/senpi.
#
# The base is a floating tag, which is why CI also rebuilds on a schedule.
ARG BASE_IMAGE=docker.io/sbx/shell-image:latest
FROM ${BASE_IMAGE}

# Re-declared inside the stage: an ARG defined before the first FROM is a
# global build arg, visible only to FROM lines. Without this, the LABEL below
# would expand to an empty string.
ARG BASE_IMAGE

# Bun runs omo's engine, so the image needs Bun rather than Node. The
# `omo` launcher prefers Bun over Node when it is on PATH, and the package
# targets Bun 1.4+. The installer needs unzip.
USER root
RUN <<'EOF'
set -euxo pipefail
apt-get update && \
    apt-get install -y --no-install-recommends ca-certificates curl unzip && \
    rm -rf /var/lib/apt/lists/*
EOF

USER agent
WORKDIR /home/agent

# omo-ai is the OmO native package; it ships the `omo` binary and pulls
# @code-yeongyu/senpi internally, so no separate platform package is needed.
# Bun and the global install live under the agent's home, because the engine
# is prepared in place on first run and the agent must own those files.
ARG OMO_VERSION=5.1.48
RUN <<'EOF'
set -euxo pipefail
curl -fsSL https://bun.sh/install | bash
"$HOME/.bun/bin/bun" add -g "omo-ai@${OMO_VERSION}"
EOF
ENV PATH="/home/agent/.bun/bin:${PATH}"

# Inherited from the base image, but re-declared deliberately so the value is
# owned here rather than depending on inheritance.
LABEL com.docker.sandboxes.flavor="omo"

# Informational only -- nothing in sbx reads this. Worth setting because the
# base is a floating tag rebuilt nightly, so this is the one place the produced
# image records what it was actually built on.
LABEL com.docker.sandboxes.base="${BASE_IMAGE}"

USER agent
WORKDIR /home/agent
CMD [ "omo" ]
