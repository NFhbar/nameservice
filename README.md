# COSMOS Nameservice App
Nameservice app built on [Cosmos SDK](https://cosmos.network/docs/) that allows for buing and selling of names.

## Requirements
Latest version of [go](https://golang.org/)

## Install
Install dependencies:

```
$ make get_tools
$ dep ensure -v
```

Build the app:
```
# Update dependencies to match the constraints and overrides above
dep ensure -update -v

# Install the app into your $GOBIN
make install

# Now you should be able to run the following commands:
nsd help
nscli help
```

## Run the App
```
# Initialize configuration files and genesis file
$ nsd init --chain-id testchain

# Copy the `Address` output here and save it for later use
$ nscli keys add jack

# Copy the `Address` output here and save it for later use
$ nscli keys add alice

# Add both accounts, with coins to the genesis file
$ nsd add-genesis-account $(nscli keys show jack --address) 1000mycoin,1000jackCoin
$ nsd add-genesis-account $(nscli keys show alice --address) 1000mycoin,1000aliceCoin
```

To run the app:
```
$ nsd start
```

## Interact with APP
With the app running you can run the following commands:
```
# First check the accounts to ensure they have funds
nscli query account $(nscli keys show jack --address) \
    --indent --chain-id testchain
nscli query account $(nscli keys show alice --address) \
    --indent --chain-id testchain

# Buy your first name using your coins from the genesis file
nscli tx nameservice buy-name jack.id 5mycoin \
    --from     $(nscli keys show jack --address) \
    --chain-id testchain

# Set the value for the name you just bought
nscli tx nameservice set-name jack.id 8.8.8.8 \
    --from     $(nscli keys show jack --address) \
    --chain-id testchain

# Try out a resolve query against the name you registered
nscli query nameservice resolve jack.id --chain-id testchain
# > 8.8.8.8

# Try out a whois query against the name you just registered
nscli query nameservice whois jack.id --chain-id testchain
# > {"value":"8.8.8.8","owner":"cosmos1l7k5tdt2qam0zecxrx78yuw447ga54dsmtpk2s","price":[{"denom":"mycoin","amount":"5"}]}

# Alice buys name from jack
nscli tx nameservice buy-name jack.id 10mycoin \
    --from     $(nscli keys show alice --address) \
    --chain-id testchain
```

## REST Server
With the client running:
```
$ nscli rest-server --chain-id testchain --trust-node
```

Queries:
```
# Get the sequence and account numbers for jack to construct the below requests
$ curl -s -k https://localhost:1317/auth/accounts/cosmos127qa40nmq56hu27ae263zvfk3ey0tkapwk0gq6
# > {"type":"auth/Account","value":{"address":"cosmos127qa40nmq56hu27ae263zvfk3ey0tkapwk0gq6","coins":[{"denom":"jackCoin","amount":"1000"},{"denom":"mycoin","amount":"1010"}],"public_key":{"type":"tendermint/PubKeySecp256k1","value":"A9YxyEbSWzLr+IdK/PuMUYmYToKYQ3P/pM8SI1Bxx3wu"},"account_number":"0","sequence":"1"}}

# Get the sequence and account numbers for alice to construct the below requests
$ curl -s -k https://localhost:1317/auth/accounts/cosmos1h7ztnf2zkf4558hdxv5kpemdrg3tf94hnpvgsl
# > {"type":"auth/Account","value":{"address":"cosmos1h7ztnf2zkf4558hdxv5kpemdrg3tf94hnpvgsl","coins":[{"denom":"aliceCoin","amount":"1000"},{"denom":"mycoin","amount":"980"}],"public_key":{"type":"tendermint/PubKeySecp256k1","value":"Avc7qwecLHz5qb1EKDuSTLJfVOjBQezk0KSPDNybLONJ"},"account_number":"1","sequence":"1"}}

# Buy another name for jack
$ curl -XPOST -s -k https://localhost:1317/nameservice/names --data-binary '{"base_req":{"name":"jack","password":"foobarbaz","chain_id":"testchain","sequence":"2","account_number":"0"},"name":"jack1.id","amount":"5mycoin","buyer":"cosmos127qa40nmq56hu27ae263zvfk3ey0tkapwk0gq6"}'
# > {"check_tx":{"gasWanted":"200000","gasUsed":"1242"},"deliver_tx":{"log":"Msg 0: ","gasWanted":"200000","gasUsed":"2986","tags":[{"key":"YWN0aW9u","value":"YnV5X25hbWU="}]},"hash":"098996CD7ED4323561AC9011DEA24C70C8FAED2A4A10BC8DE2CE35C1977C3B7A","height":"23"}

# Set the data for that name that jack just bought
$ curl -XPUT -s -k https://localhost:1317/nameservice/names --data-binary '{"base_req":{"name":"jack","password":"foobarbaz","chain_id":"testchain","sequence":"3","account_number":"0"},"name":"jack1.id","value":"8.8.4.4","owner":"cosmos127qa40nmq56hu27ae263zvfk3ey0tkapwk0gq6"}'
# > {"check_tx":{"gasWanted":"200000","gasUsed":"1242"},"deliver_tx":{"log":"Msg 0: ","gasWanted":"200000","gasUsed":"1352","tags":[{"key":"YWN0aW9u","value":"c2V0X25hbWU="}]},"hash":"B4DF0105D57380D60524664A2E818428321A0DCA1B6B2F091FB3BEC54D68FAD7","height":"26"}

# Query the value for the name jack just set
$ curl -s -k https://localhost:1317/nameservice/names/jack1.id
# 8.8.4.4

# Query whois for the name jack just bought
$ curl -s -k https://localhost:1317/nameservice/names/jack1.id/whois
# > {"value":"8.8.8.8","owner":"cosmos127qa40nmq56hu27ae263zvfk3ey0tkapwk0gq6","price":[{"denom":"STAKE","amount":"10"}]}

# Alice buys name from jack
$ curl -XPOST -s -k https://localhost:1317/nameservice/names --data-binary '{"base_req":{"name":"alice","password":"foobarbaz","chain_id":"testchain","sequence":"1","account_number":"1"},"name":"jack1.id","amount":"10mycoin","buyer":"cosmos1h7ztnf2zkf4558hdxv5kpemdrg3tf94hnpvgsl"}'
# > {"check_tx":{"gasWanted":"200000","gasUsed":"1264"},"deliver_tx":{"log":"Msg 0: ","gasWanted":"200000","gasUsed":"4509","tags":[{"key":"YWN0aW9u","value":"YnV5X25hbWU="}]},"hash":"81A371392B52F703266257D524538085F8C749EE3CBC1C579873632EFBAFA40C","height":"70"}
```

To get the addresses:
```
$ nscli keys show jack --address
$ nscli keys show alice --address
```

Request schemas:

`POST /nameservice/names`
```JSON
{
  "base_req": {
    "name": "string",
    "password": "string",
    "chain_id": "string",
    "sequence": "number",
    "account_number": "number",
    "gas": "string,not_req",
    "gas_adjustment": "string,not_req",
  },
  "name": "string",
  "amount": "string",
  "buyer": "string"
}
```

`PUT /nameservice/names`
```JSON
{
  "base_req": {
    "name": "string",
    "password": "string",
    "chain_id": "string",
    "sequence": "number",
    "account_number": "number",
    "gas": "string,not_req",
    "gas_adjustment": "strin,not_reqg"
  },
  "name": "string",
  "value": "string",
  "owner": "string"
}
```