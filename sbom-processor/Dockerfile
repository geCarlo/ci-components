FROM debian:12-slim

RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    ca-certificates \
    git \
    jq \
    && rm -rf /var/lib/apt/lists/*

# Add any extra tooling your SBOM processing needs here, for example:
#   python3, a vulnerability database client, a reporting tool, etc.
#
# Example (python3 + pandas for CSV processing):
#   RUN apt-get update && apt-get install -y --no-install-recommends python3 python3-pip \
#       && pip3 install pandas --no-cache-dir

COPY scripts/ /usr/local/bin/
RUN chmod +x /usr/local/bin/*.sh

ENTRYPOINT ["/bin/bash"]
