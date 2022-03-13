**Guía en construcción -  No está terminada**

**IMPORTANTE:** Consideraciones a tener en cuenta antes de modificar el Pledge.
- Asegúrate de tener la misma versión de **cardano-cli** tanto en el entorno caliente como en el frío.

**Máquina Offline**
```bash
cardano-cli stake-pool registration-certificate \
    --cold-verification-key-file $HOME/cold-keys/node.vkey \
    --vrf-verification-key-file vrf.vkey \
    --pool-pledge 3500000000 \
    --pool-cost 340000000 \
    --pool-margin 0 \
    --pool-reward-account-verification-key-file hw-stake.vkey \
    --pool-owner-stake-verification-key-file stake.vkey \
    --pool-owner-stake-verification-key-file hw-stake.vkey \
    --mainnet \
    --single-host-pool-relay <dns based relay, example ~ relaynode1.myadapoolnamerocks.com> \
    --pool-relay-port 6000 \
    --metadata-url <url where you uploaded poolMetaData.json> \
    --metadata-hash $(cat poolMetaDataHash.txt) \
    --out-file pool.cert
```
En el anterior ejemplo, estamos usando un margin del 0%, coste fijo de 340 ADA y un pledge de 3500 ADA.  
Como referencia: 1 ADA = 1000000 Lovelace

Copia el archivo **pool.cert** al entorno caliente.  
  
**Máquina productor de bloques** 
```bash
currentSlot=$(cardano-cli query tip --mainnet | jq -r '.slot')
echo Current Slot: $currentSlot
```
```bash
cardano-cli query utxo \
    --address $(cat payment.addr) \
    --mainnet > fullUtxo.out

tail -n +3 fullUtxo.out | sort -k3 -nr > balance.out

cat balance.out

tx_in=""
total_balance=0
while read -r utxo; do
    in_addr=$(awk '{ print $1 }' <<< "${utxo}")
    idx=$(awk '{ print $2 }' <<< "${utxo}")
    utxo_balance=$(awk '{ print $3 }' <<< "${utxo}")
    total_balance=$((${total_balance}+${utxo_balance}))
    echo TxHash: ${in_addr}#${idx}
    echo ADA: ${utxo_balance}
    tx_in="${tx_in} --tx-in ${in_addr}#${idx}"
done < balance.out
txcnt=$(cat balance.out | wc -l)
echo Total ADA balance: ${total_balance}
echo Number of UTXOs: ${txcnt}
```
```bash
cardano-cli transaction build-raw \
    ${tx_in} \
    --mary-era \
    --tx-out $(cat payment.addr)+${total_balance} \
    --invalid-hereafter $(( ${currentSlot} + 10000)) \
    --fee 0 \
    --certificate-file pool.cert \
    --out-file tx.tmp
```
```bash
fee=$(cardano-cli transaction calculate-min-fee \
    --tx-body-file tx.tmp \
    --tx-in-count ${txcnt} \
    --tx-out-count 1 \
    --mainnet \
    --witness-count 4 \
    --byron-witness-count 0 \
    --protocol-params-file params.json | awk '{ print $1 }')
echo fee: $fee
```
```bash
txOut=$((${total_balance}-${fee}))
echo txOut: ${txOut}
```
```bash
cardano-cli transaction build-raw \
    ${tx_in} \
    --mary-era \
    --tx-out $(cat payment.addr)+${txOut} \
    --invalid-hereafter $(( ${currentSlot} + 10000)) \
    --fee ${fee} \
    --certificate-file pool.cert \
    --out-file tx-pool.raw
```
Copia el archivo **tx-pool.raw** a la **máquina offline**.

**Guía en construcción -  No está terminada**
