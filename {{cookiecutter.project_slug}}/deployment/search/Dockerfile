# ---------- STEP 1 ----------
# Index the project files
FROM vshn/antora-indexer-cli:1.6 as indexer

WORKDIR /build
COPY . /build

RUN \
    mkdir /index && \
    antora-indexer --antora /build/docs --write files --output /index/files.json && \
    antora-indexer --antora /build/docs --write index --output /index/lunr.json

# ---------- STEP 2 ----------
# Docker image only containing the search engine and the freshly built index
FROM vshn/embedded-search-engine:1.3

# Copy the index of the search engine
COPY --from=indexer /index/files.json /site/index/files.json
COPY --from=indexer /index/lunr.json /site/index/lunr.json

