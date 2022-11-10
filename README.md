# Vaporum Coin pos64splitter

An automated staker for PoS assetchains. Please see https://docs.komodoplatform.com/komodo/assetchain-params.html#ac-staked for details on pos64 POS implementation. 

This is a work in progress. We aim to make this easy to use and as "set and forget" as possible. Please feel free to contribute code and ideas. 

Currently, this will maintain a static number of UTXOs. This is important because a staking wallet can become very bloated over time. The block reward of any staked blocks will be combined with the UTXO used to stake the block.

## Dependencies
```shell
sudo apt-get install python3-dev
sudo apt-get install python3 libgnutls28-dev libssl-dev
sudo apt-get install python3-pip
pip3 install setuptools
pip3 install wheel
pip3 install base58 slick-bitcoinrpc
```

[komodod](https://github.com/StakedChain/komodo) installed with your assetchain running.

## How to Use

The following examples will use VPRM. Replace VPRM with the chain you are using.

`git clone https://github.com/StakedChain/pos64staker`

`cd pos64staker`

`./genaddresses.py`
```shell
Please specify chain:VPRM
```

This will create a `list.json` file in the current directory. **THIS FILE CONTAINS PRIVATE KEYS. KEEP IT SAFE.**
Copy this file to the directory `komodod` is located. 

`cp list.json ~/komodo/src/list.json`

`./sendmany64.py`
```shell
Please specify chain:VPRM
Balance: 1000000.77
Please specify the size of UTXOs:10
Please specify the amount of UTXOs to send to each segid:10
```
Please take note of what this is actually asking for. The above example will send 6400 coins total. It will send 100 coins in 10 UTXOs to each of the 64 segids. Will throw error if your entered amounts are more than your balance. Will tell you how much avalible you have for each segid.

You now need to start the daemon with -blocknotify and -pubkey set.

Fetch a pubkey from your `list.json` and place it in your start command. For example:

`./komodod -ac_name=VPRM -ac_supply=0 -ac_eras=6 -ac_blocktime=30 -ac_reward=5000000000,2500000000,1250000000,625000000,312500000,156250000 -ac_end=1000000,3500000,8500000,18500000,38500000,166500000 -ac_staked=50 -ac_sapling=1 -ac_cbmaturity=1 -ac_cc=0 -addnode=68.3.67.21 -addnode=167.172.130.118 -addnode=157.230.90.81 '-blocknotify=/home/<USER>/pos64staker/staker.py %s VPRM'`

NOTE the VPRM in -blocknotify make sure you change this to the correct chain name you are using also note the single quotes.

After the daemon has started and is synced simply do `komodo-cli -ac_name=VPRM setgenerate true 0` to begin staking. 


### How the staker.py works

on block arrival:

getinfo for -pubkey 

setpubkey for R address 

check coinbase -> R address 

if yes check segid of block.

if -1 send PoW mined coinbase to :

        listunspent call ... 

        sort by amount -> smallest at top and then by confirms -> lowest to top. (we want large and old utxos to maximise staking rewards.)

        select the top txid/vout

        add this txid to txid_list

        get last segid stakes 1440 blocks (last24H)

        select all segids under average stakes per segid in 24H

        randomly choose one to get segid we will send to.        

if segid >= 0 :

    fetch last transaction in block

    check if this tx belongs to the node

    if yes, use alrights code to combine this coinbase utxo with the utxo that staked it.
    
    
### Withdraw 

Withdraw script is for withdrawing funds from a staking node, without messing up utxo distribution. Works like this:

    Asks for percentage you want locked (kept). 
    
    It then counts how many utxo per segid. 
    
    Locks the largest and oldest utxos in each segid up to the % you asked.
    
    Gives balance of utxos remaning that are not locked.  These should be the smallest and newest utxo's in each segid. The least likely to stake.
    
    Then lets you send some coins to an address. 
    
    Unlocks utxos again.
