# haqq



@cryptonik_space
24 августа
Как установить ноду HAQQ / Islamic Coin? Подробный гайд.

HAQQ - это топовый блокчейн проект на ранней стадии. Я делал обзор экосистемы Islamic Coin / HAQQ у себя на Youtube канале. Но давайте я вкратце напомню основные моменты:

Islamic Coin (Исламская монета) стремится привлечь исламский финансовый сектор в криптовалюту, причем на 2020 год объем денежных средств там составлял 2.88 триллиона долларов, к 2024 году ожидается увеличение до 3.69 триллиона долларов;
Islamic Coin / HAQQ получил фетву (решение о том, что продукт соответствует Шариату), изданную ведущими мировыми экспертами в области Ислама. То есть 2 миллиарда мусульман официально могут использовать эту валюту, и она не противоречит религиозным принципам;
Масштабируемый и быстрый EVM (виртуальная машина Эфириума) совместимый блокчейн;
Есть возможность работать с экосистемой Cosmos;
Над проектом работают очень опытные финансисты, которые управляли миллиардными оборотами;
Блокчейн HAQQ уже привлек 200 миллионов долларов инвестиций в приватном seed раунде.
Ранние последователи экосистемы HAQQ могут получить огромные аирдропы в монетах $ISLM, поэтому я советую Вам влетать во все события, которые проходят в проекте. Установка ноды играет ключевое значение в любом тестнете.  

Подготовка
Вам нужен будет сервер Ubuntu с такими характеристиками

4 Сore CPU | 8 GB RAM | 100 GB SSD/HDD
В Дискорде админы указали следующие характеристики:


Но многие нодеры берут те характеристики, которые я написал. Этого, вроде, хватает на данный момент. 

Арендовать сервер можно, например, на сайте Hetzner. Если ваши карты не работают для оплаты, то можете воспользоваться виртуальной картой PayWithMoon или BitFree.

Установка и настройка ноды HAQQ
Отступление: спасибо телеграм каналу @mmsnodes за инструкцию, которую я использовал. Но в моей инструкции есть некоторые дополнения, без которых у меня не получалось установить ноду! 

Копируем IP адрес сервера, в Hetzner он приходит на почту после аренды. Открываем терминал и коннектимся к серверу (вместо 000.000.000.000 вписываем IP адрес).

ssh root@000.000.000.000
Вводим пароль, он также должен прийти на почту, если используете Hetzner. Имейте в виду, что пароль не нужно копировать и вставлять (скорее всего, он не вставится)! Нужно вводить самостоятельно! Во время ввода ничего не будет отображаться, это нормально.

Затем Вас попросят придумать новый пароль и ввести его повторно.

Обновляем пакеты:

sudo apt update && sudo apt upgrade -y
Устанавливаем дополнительные пакеты:

sudo apt install curl build-essential git wget jq make gcc tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
Устанавливаем GO и проверяем версию:

ver="1.18.2" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
Скачиваем и устанавливаем бинарники с Github:

git clone https://github.com/haqq-network/haqq.git
cd haqq
make install
chmod +x /root/go/bin/haqqd && sudo mv /root/go/bin/haqqd /usr/local/bin/haqqd
cd $HOME
Задаем переменные, вместо your_name придумываем свое имя (пишем латинскими буквами!):

MONIKER="your_name"
CHAIN="haqq_53211-1"
WALLET_NAME="your_name"
Добавляем все в баш профиль:

echo 'export MONIKER='${MONIKER} >> $HOME/.bash_profile
echo 'export CHAIN='${CHAIN} >> $HOME/.bash_profile
echo 'export WALLET_NAME='${WALLET_NAME} >> $HOME/.bash_profile
source $HOME/.bash_profile
Инициализируем ноду:

haqqd init $MONIKER --chain-id $CHAIN
Прописываем имя сети в Config:

haqqd config chain-id $CHAIN
Скачиваем генезис файл и проверяем его:

curl -OL https://storage.googleapis.com/haqq-testedge-snapshots/genesis.json
mv genesis.json $HOME/.haqqd/config/genesis.json
haqqd validate-genesis
Сбрасываем состояние валидатора:

haqqd tendermint unsafe-reset-all --home $HOME/.haqqd
Настраиваем синхронизацию:

curl -OL https://raw.githubusercontent.com/haqq-network/testnets/main/TestEdge/state_sync.sh
chmod +x state_sync.sh ./state_sync.sh
Настраиваем прунинг:

pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="50" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.haqqd/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.haqqd/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.haqqd/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.haqqd/config/app.toml
Выключаем индексер:

indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.haqqd/config/config.toml
Создаем сервис файл:

sudo tee /etc/systemd/system/haqqd.service > /dev/null <<EOF
[Unit]
Description=haqqd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which haqqd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
Запускаем сервис и ждем, пока нода синхронизируется:

sudo systemctl daemon-reload && \
sudo systemctl enable haqqd && \
sudo systemctl restart haqqd
Чтобы ускорить процесс синхронизации:

sudo systemctl stop haqqd
haqqd tendermint unsafe-reset-all --home $HOME/.haqqd
SEEDS="8f7b0add0523ec3648cb48bc12ac35357b1a73ae@195.201.123.87:26656,899eb370da6930cf0bfe01478c82548bb7c71460@34.90.233.163:26656,f2a78c20d5bb567dd05d525b76324a45b5b7aa28@34.90.227.10:26656,4705cf12fb56d7f9eb7144937c9f1b1d8c7b6a4a@34.91.195.139:26656"
PEERS=""; \
sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.haqqd/config/config.toml
wget -O $HOME/.haqqd/config/addrbook.json "https://raw.githubusercontent.com/StakeTake/guidecosmos/main/haqq/haqq_53211-1/addrbook.json"
SNAP_RPC="http://haqq.stake-take.com:36657"
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.haqqd/config/config.toml
sudo systemctl restart haqqd && journalctl -u haqqd -f -o cat
По желанию! Можете посмотреть логи (чтобы потом выйти - жмем CTR+C):

sudo journalctl -u haqqd -f -o cat
Проверяем статус синхронизации. Нам нужно, чтобы в разделе catching_up было написано false.

curl localhost:26657/status
Когда синхронизируется, создаем кошелек и записываем все данные в надежное место, особенно seed фразу внизу из 24 слов. Если хотите восстановить уже существующий кошелек: haqqd keys add $WALLET_NAME --recover. Но я рекомендую создавать новый.

haqqd keys add $WALLET_NAME

Добавляем переменную с адресом кошелька:

WALLET_ADDRESS=$(haqqd keys show $WALLET_NAME -a)
Добавляем переменную в баш профиль:

echo 'export WALLET_ADDRESS='${WALLET_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
Теперь идем СЮДА и запрашиваем тестовые токены $ISLM. Вам нужно будет войти с помощью профиля Github! Подключаться нужно будет кошельком, который Вы только что создали. Поэтому можете создать новую личность в Chrome, установить туда MetaMask, импортировать seed фразу, которую получили в командной строке. А затем подключить кошелек к сайту и запросить тестовые монеты. 

Выполняем команду и получаем данные кошелька, вместо адрес_кошелька_haqq вставляем свой адрес:

haqqd debug addr адрес_кошелька_haqq
Сохраняем все данные куда-нибудь в файл. Валопер адрес (начинается с haqqvaloper) понадобится Вам в будущем, чтобы получать делегированные монеты.


Проверяем баланс:

haqqd query bank balances $WALLET_ADDRESS
Должно быть 10*(10^18) - это 10 тестовых монет $ISLM.


Если средства поступили, то создаем валидатора. Сумму (amount) можете указать свою. Я укажу 7 монет $ISLM.

haqqd tx staking create-validator \
  --amount=7000000000000000000aISLM \
  --pubkey=$(haqqd tendermint show-validator) \
  --moniker=$MONIKER \
  --chain-id=haqq_53211-1 \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1000000" \
  --gas="auto" \
  --from=$WALLET_NAME \
  --node https://rpc.tm.testedge.haqq.network:443
Задаем переменную с адресом валидатора:

VALOPER=$(haqqd keys show $WALLET_ADDRESS --bech val -a)
Добавляем ее в баш профиль:

echo 'export VALOPER='${VALOPER} >> $HOME/.bash_profile
source $HOME/.bash_profile
Проверка статуса валидатора:

haqqd query staking validator $VALOPER

Делегируем себе 8 монет:

haqqd tx staking delegate $VALOPER 8000000000000000000aISLM --from $WALLET_NAME --chain-id haqq_53211-1
Для дополнительного бонуса можете заделегировать мне одну монету. Это будет плюсом и Вам, и мне :)

haqqd tx staking delegate haqqvaloper15rqsx96hma5706xcqy6c5c6f4fzedhxgdru89z 1000000000000000000aISLM --from $WALLET_NAME --chain-id haqq_53211-1
Готово!

Где смотреть транзакции и валидаторов?

1) https://pingpub.explorer.testedge.haqq.network/haqq/staking
2) https://explorer.nodestake.top/haqq-testedge/staking
