# Ginkou

1. Determine if the current player exists based on their wallet address. If the player does not exist, they must be created. 

> The `keys` parameter is used in the query to filter specific entities. The parameter takes an array with two elements:
> 
> 1. The first element represents the game ID, which has a default value of 1.
> 2. The second element is the wallet address of the player being added.

By providing these values, the query retrieves information about the player with the specified wallet address within the context of the game with the specified game ID.

```bash
curl '{torii_host}/graphql' \
  --data-raw '{"operationName":"FilteredEntities","variables":{"keys":["0x1","{YOUR_WALLET_ACCOUNT}"]},"query":"query FilteredEntities($keys: [String\u0021]\u0021) {\n  entities(keys: $keys) {\n    edges {\n      node {\n        model_names\n        models {\n          ... on Player {\n            player_id\n            name\n          }\n        }\n      }\n    }\n  }\n}\n"}' \
  --compressed
```

2. Assuming the player exists, you can obtain the player_id. Using the player_id, you can retrieve the amount of gold and silver resources currently available.

```bash
curl '{torii_host}' \
  --data-raw '{"operationName":"FilteredEntities","variables":{"keys":["0x1","0x1"]},"query":"query FilteredEntities($keys: [String\u0021]\u0021) {\n  entities(keys: $keys) {\n    edges {\n      node {\n        model_names\n        models {\n          ... on PlayerInfo {\n            gold\n            silver\n          }\n        }\n      }\n    }\n  }\n}\n"}' \
  --compressed
```

3. If there are resources available, you can proceed with the withdrawal. Retrieve the corresponding ERC20 token 

> The `keys` parameter: 
> 1. game_id
> 2. player_id
> 3. represents the resource type: Gold=0x1, Silver=0x2
> 4. the number of tokens to withdraw

```bash
sozo execute $GINKOU_ACTION_ADDR withdraw --calldata 0x1,0x1,0x1,0x2
```

4. After withdrawal, you can swap the tokens (by Ekubo) for another resource.

5. Call the approve method of the ERC20 token you want to exchange. Assuming that Gold will be swapped for Silver:

```bash
starkli invoke $WSILVER_ERC20_ADDR approve $GINKOU_ACTION_ADDR 0x8 0
```

6. Deposit the new resource tokens in the dojo to obtain the corresponding in-game resources:

> The `keys` parameter: 
> 1. game_id
> 2. player_id
> 3. represents the resource type: Gold=0x1, Silver=0x2
> 4. the number of tokens to deposit

```bash
sozo execute $GINKOU_ACTION_ADDR deposit --calldata  0x1,0x1,0x2,0x8
```

7. The updated data state can be viewed using the query from step 2.
