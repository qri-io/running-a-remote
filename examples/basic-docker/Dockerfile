# here we're using the "HEAD" dev version of qri. work on remotes is under 
# active development
FROM qriio/qri-auto:latest

# qri setup provisions a qri instance
RUN qri setup -a

# configure this node as a remote
RUN qri config set \
  remote.enabled true \
  remote.acceptsizemax 10000000 \
  remote.allowremoves true

# Expose ports 
# 2503 : Qri API
# 4001 : IPFS Swarm TCP
# 4002/udp : IPFS Swarm udp
# 5001 IPFS API
# 8080 IPFS Gateway
# 8081 IPFS Swarm Websockets
EXPOSE 2503 4001 4002/udp 5001 8080 8081

# Set binary as entrypoint
# the setup flag initalizes ipfs & qri repos if none is mounted
# the migrate flag automatically executes any necessary migrations
# the no-prompt flag disables asking for any user input, reverting to defaults
CMD ["qri", "connect", "--setup", "--migrate", "--no-prompt"]
