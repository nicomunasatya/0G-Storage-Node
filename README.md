# 0G-Storage-Node
Running 0G storage v3 node using VPS

Requirements
- Memory: 32 GB RAM
- CPU: 8 Cores
- Disk: 500GB / 1TB NVME SSD
- Bandwidth: 100 Mbps (Download / Upload)

1. Install dependencies for building from source
    ```
    sudo apt-get update
    sudo apt-get install clang cmake build-essential openssl pkg-config libssl-dev
    ```
2. install go
    ```
    cd $HOME && \
    ver="1.22.0" && \
    wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
    sudo rm -rf /usr/local/go && \
    sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
    rm "go$ver.linux-amd64.tar.gz" && \
    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
    source ~/.bash_profile && \
    go version
    ```
3. install rustup
    ```
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
    ```
    ```
    . "$HOME/.cargo/env"
    ```
4. Download and install binary
   If you want remove the node and install from scratch:
    ```
    cd $HOME
    rm -rf 0g-storage-node
    git clone https://github.com/0glabs/0g-storage-node.git
    cd 0g-storage-node
    git checkout v0.8.4
    git submodule update --init
    ```
5. Config:
   You can download my sample config
   https://github.com/nicomunasatya/0G-Storage-Node/blob/main/config-testnet-turbo.toml
    ```
    wget -O $HOME/0g-storage-node/run/config-testnet-turbo.toml https://raw.githubusercontent.com/nicomunasatya/0G-Storage-Node/main/config-testnet-turbo.toml
    ```
6. Set your miner key:
   Define varible The input will be hidden from the screen for security
    ```
    printf '\033[34mEnter your private key: \033[0m' && read -s PRIVATE_KEY
    ```
7. Verifying Configuration Changes
   To check if the configuration has been updated correctly
    ```
    grep -E "^(network_dir|network_enr_address|network_enr_tcp_port|network_enr_udp_port|network_libp2p_port|network_discovery_port|rpc_listen_address|rpc_enabled|db_dir|log_config_file|log_contract_address|mine_contract_address|reward_contract_address|log_sync_start_block_number|blockchain_rpc_endpoint|auto_sync_enabled|find_peer_timeout)" $HOME/0g-storage-node/run/config-testnet-turbo.toml
    ```
8. create service
    ```
    sudo tee /etc/systemd/system/zgs.service > /dev/null <<EOF
    [Unit]
    Description=ZGS Node
    After=network.target

    [Service]
    User=$USER
    WorkingDirectory=$HOME/0g-storage-node/run
    ExecStart=$HOME/0g-storage-node/target/release/zgs_node --config $HOME/0g-storage-node/run/config.toml
    Restart=on-failure
    RestartSec=10
    LimitNOFILE=65535

    [Install]
    WantedBy=multi-user.target
    EOF
    ```
9. Start node
    ```
    sudo systemctl daemon-reload && \
    sudo systemctl enable zgs && \
    sudo systemctl restart zgs && \
    sudo systemctl status zgs
    ```
10. Check log
    Full log:
    ```
    tail -f ~/0g-storage-node/run/log/zgs.log.$(TZ=UTC date +%Y-%m-%d)
    ```
    ![Full Log](https://raw.githubusercontent.com/nicomunasatya/0G-Storage-Node/main/Check%20full%20log.png)
    
    tx_seq log:
    ```
    tail -f ~/0g-storage-node/run/log/zgs.log.$(TZ=UTC date +%Y-%m-%d) | grep tx_seq
    ```
    ![tx_seq log](https://raw.githubusercontent.com/nicomunasatya/0G-Storage-Node/main/Tx_seq%20log.png)
    
    Monitoring 0g-storage-node Logs without Network and Peer Messages:
    ```
    tail -f ~/0g-storage-node/run/log/zgs.log.$(date +%Y-%m-%d) | grep -v "discv5\|network\|router\|Peer"
    ```
    ![Monitoring 0g-storage-node Logs without Network and Peer Messages](https://raw.githubusercontent.com/nicomunasatya/0G-Storage-Node/main/Tx_seq%20log.png)
    
    logSync & Peers through RPC:
    ```
    while true; do
        response=$(curl -s -X POST http://localhost:5678 -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","method":"zgs_getStatus","params":[],"id":1}')
        logSyncHeight=$(echo $response | jq '.result.logSyncHeight')
        connectedPeers=$(echo $response | jq '.result.connectedPeers')
        echo -e "logSyncHeight: \033[32m$logSyncHeight\033[0m, connectedPeers: \033[34m$connectedPeers\033[0m"
        sleep 5;
    done
    ```
    ![Log Sync & Peers](https://raw.githubusercontent.com/nicomunasatya/0G-Storage-Node/main/logSync%20%26%20Peers%20through%20RPC.png)
    
12. Stop & Delete node
    ```
    sudo systemctl stop zgs
    ```
    ```
    sudo systemctl disable zgs
    sudo rm /etc/systemd/system/zgs.service
    rm -rf $HOME/0g-storage-node
    ```

OR You Can Running 0G Storage V3 using one line code
```
wget https://raw.githubusercontent.com/nicomunasatya/0G-Storage-Node/main/0g_storage_node_v3.sh && chmod +x 0g_storage_node_v3.sh && ./0g_storage_node_v3.sh
```
#UPDATE 0G Storage V3 using one line code
```
wget https://raw.githubusercontent.com/nicomunasatya/0G-Storage-Node/main/storage_node_update_v3.sh && chmod +x storage_node_update_v3.sh && ./storage_node_update_v3.sh
```


