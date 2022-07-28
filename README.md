## Установка HERMES relayer
<!--

cd $HOME
mkdir -p $HOME/.hermes/bin

''''verison="v0.15.0"
wget "https://github.com/informalsystems/ibc-rs/releases/download/$verison/hermes-$verison-x86_64-unknown-linux-gnu.tar.gz"
tar -C $HOME/.hermes/bin/ -vxzf hermes-$verison-x86_64-unknown-linux-gnu.tar.gz''''

rm hermes-$verison-x86_64-unknown-linux-gnu.tar.gz

echo "export PATH=$PATH:$HOME/.hermes/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
 
НАСТРОЙКА КОНФИГА ГЕРМЕС

Задать переменные
MEMO="zuka"
CH1_RPC="78.107.234.44"
CH1_RPC_PORT="25657"
CH1_GRPC_PORT="9192"
CH1_CHAIN_ID="STRIDE-TESTNET-2"
CH1_ACC_PREFIX="stride"
CH1_DENOM="ustrd"
CH1_REL_WALLET="CH1_REL_WALLET"

CH2_RPC="78.107.234.44"
CH2_RPC_PORT="36657"
CH2_GRPC_PORT="9797"
CH2_CHAIN_ID="GAIA"
CH2_ACC_PREFIX="cosmos"
CH2_DENOM="uatom"
CH2_REL_WALLET="CH2_REL_WALLET"


# сохранить переменные одной командой

echo "
export MEMO=${MEMO}
export CH1_CHAIN_ID=${CH1_CHAIN_ID}
export CH1_DENOM=${CH1_DENOM}
export CH1_REL_WALLET=${CH1_REL_WALLET}
export CH2_CHAIN_ID=${CH2_CHAIN_ID}
export CH2_DENOM=${CH2_DENOM}
export CH2_REL_WALLET=${CH2_REL_WALLET}
" >> $HOME/.bash_profile

source $HOME/.bash_profile

# СОЗДАЕМ КОНФИГ ФАИЛ ГЕРМЕСА (все скопировать и в терминал)

echo "[global]
log_level = 'info'
[mode]
[mode.clients]
enabled = true
refresh = true
misbehaviour = true
[mode.connections]
enabled = false
[mode.channels]
enabled = false
[mode.packets]
enabled = true
clear_interval = 100
clear_on_start = true
tx_confirmation = true
[rest]
enabled = true
host = '0.0.0.0'
port = 3000
[telemetry]
enabled = true
host = '0.0.0.0'
port = 3001
[[chains]]
id = '$CH1_CHAIN_ID'
rpc_addr = 'http://$CH1_RPC:$CH1_RPC_PORT'
grpc_addr = 'http://$CH1_RPC:$CH1_GRPC_PORT'
websocket_addr = 'ws://$CH1_RPC:$CH1_RPC_PORT/websocket'
rpc_timeout = '10s'
account_prefix = '$CH1_ACC_PREFIX'
key_name = '$CH1_REL_WALLET'
store_prefix = 'ibc'
max_tx_size = 100000
max_gas = 20000000
gas_price = { price = 0.001, denom = '$CH1_DENOM' }
gas_adjustment = 0.1
max_msg_num = 30
clock_drift = '5s'
trusting_period = '10h 29m'
trust_threshold = { numerator = '1', denominator = '3' }
memo_prefix = '$MEMO'
[[chains]]
id = '$CH2_CHAIN_ID'
rpc_addr = 'http://$CH2_RPC:$CH2_RPC_PORT'
grpc_addr = 'http://$CH2_RPC:$CH2_GRPC_PORT'
websocket_addr = 'ws://$CH2_RPC:$CH2_RPC_PORT/websocket'
rpc_timeout = '10s'
account_prefix = '$CH2_ACC_PREFIX'
key_name = '$CH2_REL_WALLET'
store_prefix = 'ibc'
default_gas = 100000
max_gas = 2000000
gas_price = { price = 0.01, denom = '$CH2_DENOM' }
gas_adjustment = 0.1
max_msg_num = 30
max_tx_size = 2097152
clock_drift = '5s'
max_block_time = '30s'
trusting_period = '10h 29m'
trust_threshold = { numerator = '1', denominator = '3' }
address_type = { derivation = 'cosmos' }" > $HOME/.hermes/config.tomlэ

# ПРОВЕРИТЬ КОНФИГ

hermes config validate

пример вывода
# Success: "configuration is valid"


проверить работоспособность сетей

hermes health-check

Создание кошельков:

# заисать переменные

CH1_REL_WALLET="CH1_REL_WALLET"
CH2_REL_WALLET="CH2_REL_WALLET"
CH1_CHAIN_ID=$CH1_CHAIN_ID
CH2_CHAIN_ID=$CH2_CHAIN_ID

********************
ИМПОРТИРОВАТЬ КОШЕЛЕК ПЕРВОЙ СЕТИ!

hermes keys restore $CH1_CHAIN_ID -n $CH1_REL_WALLET -m "встваить мнемонику"


ИМПОРТИРОВАТЬ КОШЕЛЕК ВТОРОЙ СЕТИ

hermes keys restore $CH2_CHAIN_ID -n $CH2_REL_WALLET -m  "вставить мнемонику"


*********************




# ПРОВЕРИТЬ СОСТОЯНИЕ КАНАЛОВ

hermes --json query channel end $CH1_CHAIN_ID transfer $CN_NUM_1 | jq


hermes --json query channel end $CH2_CHAIN_ID transfer $CN_NUM_2 | jq

*********************************
СОЗДАТЬ СЕРВИСНЫЙ ФАИЛ:

sudo tee /etc/systemd/system/hermesd.service > /dev/null <<EOF
[Unit]
Description=HERMES
After=network.target
[Service]
Type=simple
User=$USER
ExecStart=$(which hermes) start
Restart=on-failure
RestartSec=10
LimitNOFILE=4096
[Install]
WantedBy=multi-user.target
EOF

# Start service

sudo systemctl daemon-reload
sudo systemctl enable hermesd
sudo systemctl restart hermesd && journalctl -u hermesd -f

отправить транзакции

hermes tx raw ft-transfer   $CH2_CHAIN_ID   $CH1_CHAIN_ID   transfer   $CN_NUM_1   10000   -d $CH1_DENOM   -k $CH1_REL_WALLET   -r sei1zv62we39aqhj2g5mghznuj3k0hcg9ztqw4aax0 -n 1   -t 60   -o 100 

hermes tx raw ft-transfer   $CH1_CHAIN_ID  $CH2_CHAIN_ID   transfer   $CN_NUM_2  10000   -d $CH2_DENOM   -k $CH2_REL_WALLET   -r tori1zv62we39aqhj2g5mghznuj3k0hcg9ztqpdmzm7  -t 60   -o 100

