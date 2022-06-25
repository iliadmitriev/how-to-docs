# MongoDB

# Start

1. create instance of mongodb with docker, setting user and password with exposing port 27017
```shell
docker run -d --name mongo-server \
   -e MONGO_INITDB_ROOT_USERNAME=memo \
   -e MONGO_INITDB_ROOT_PASSWORD=secret \
   -p 27017:27017 \
   mongo
```

2. install cli tool `mongosh`
```shell
brew install mongosh
```

3. connect to mongo
```shell
mongosh "mongodb://memo:secret@127.0.0.1:27017/"
```


# Databases
```shell
show databases
show dbs
use memo
```

# Collections
## show collections
```js
db.getCollectionNames()
db.getCollectionInfos({'name': 'memo'})
```

## select
```js
db.getCollection('memo')
db.getCollection('memo').find({'_id': 12345678})
```
select with projection
```js
db.getCollection('memo').find(
{
  '_id': {
    $in: [
		'fdbf3b6008b064101a6bb0edcf58e67a',
		'd52c82dd7aa3dba58e305b87b12efe02',
		'3d52aed99b80c882c3f20d6c830c5376'
		]
	},
	'data.goods.type': 'fmcg',
  'data.goods.goodId': '6dd6d7a8c1e282f486fe7877486c7f82'
});
```


### aggregate
```ja
db.getCollection('memo').find({'_id': 12345678}).itcount()
```

## insert
1. simple
```js
db.getCollection('memo').insert({'_id': 12345678})
db['memo'].insert({'_id': 12345678})
db.memo.insert({'_id': 12345678})
```
2. more complex example
```js
db.getCollection('memo').insert({
  '_id': 'fdbf3b6008b064101a6bb0edcf58e67a',
  'data': {
    'goods': [
      {
        'goodId': '6dd6d7a8c1e282f486fe7877486c7f82',
        'type': 'fmcg',
        'name': 'milk',
        'price': 10,
        'available': true
      },
      {
        'goodId': 'd1c8034bdf0733018fceb39ca251e1f5',
        'type': 'fmcg',
        'name': 'potato',
        'price': 15,
        'available': false
      },
      {
        'goodId': 'd1c8034bdf0733018fceb39ca251e1f5',
        'type': 'fmcg',
        'name': 'soy milk',
        'price': 9,
        'available': false
      }
    ]
  }
});
```


## delete
```js
db.getCollection('memo').remove({'_id': 12345678})
```

# other
get connection string
```
db.getMongo()
```
