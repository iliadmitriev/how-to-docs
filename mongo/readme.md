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
db.getCollectionNames();
db.getCollectionInfos({ name: "memo" });
```

## find records

<details>
	<summary>simple select (all, by id)</summary>
	
```js
db.getCollection('memo').find({});
db.getCollection('memo').find({'_id': 'fdbf3b6008b064101a6bb0edcf58e67a'});
```
</details>

<details>
	<summary>select with projection (expand)</summary>

```js
db.getCollection("memo").find(
  {
    _id: {
      $in: [
        "fdbf3b6008b064101a6bb0edcf58e67a",
        "66e225a7ada99fe8fd9d15f938a94ce3",
      ],
    },
    "data.goods.type": "fmcg",
    "data.goods.goodId": "6dd6d7a8c1e282f486fe7877486c7f82",
  },
  { "data.goods.goodId": 1, "data.goods.price": 1 },
);
```

</details>

### aggregate records

<details>
	<summary>count</summary>
	
```js
db.getCollection('memo').find({'_id': 12345678}).itcount()
```
</details>

<details>
	<summary>pipeline with filter(match), unwind (denormalize), project</summary>

```js
db.getCollection("memo").aggregate([
  {
    $match: {
      _id: {
        $in: [
          "fdbf3b6008b064101a6bb0edcf58e67a",
          "2c5be738c6102ba9a33b4d4729e24eff",
        ],
      },
    },
  },

  {
    $unwind: {
      path: "$data.goods",
    },
  },

  {
    $match: {
      "data.goods.available": false,
    },
  },

  {
    $project: {
      _id: 1,
      "data.goods.goodId": 1,
      "data.goods.name": 1,
      "data.goods.price": 1,
    },
  },
]);
```

</details>

## insert

<details>
	<summary>simple insert</summary>

```js
db.getCollection("memo").insert({ _id: 12345678 });
db["memo"].insert({ _id: 12345678 });
db.memo.insert({ _id: 12345678 });
```

</details>

<details>
  <summary>insert one record <code>db.collections.insertOne(documnent, options)</code></summary>

```js
db.getCollection("memo").insertOne({
  _id: "fdbf3b6008b064101a6bb0edcf58e67a",
  data: {
    goods: [
      {
        goodId: "6dd6d7a8c1e282f486fe7877486c7f82",
        type: "fmcg",
        name: "milk",
        price: 10,
        available: true,
      },
      {
        goodId: "d1c8034bdf0733018fceb39ca251e1f5",
        type: "fmcg",
        name: "potato",
        price: 15,
        available: false,
      },
      {
        goodId: "d1c8034bdf0733018fceb39ca251e1f5",
        type: "fmcg",
        name: "soy milk",
        price: 9,
        available: false,
      },
    ],
  },
});
```

</details>

<details>
	<summary>insert many records <code>db.collection.insertMany(documents, options)</code></summary>
  
```js
db.getCollection('memo').insertMany([
{
  '_id': '2c5be738c6102ba9a33b4d4729e24eff',
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
},
{
  '_id': '66e225a7ada99fe8fd9d15f938a94ce3',
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
}
]);
```
</details>

## update

<details>
	<summary>update multiple nested arrays with filter applied</summary>

```js
db.getCollection("memo").update(
  {
    _id: {
      $in: [
        "fdbf3b6008b064101a6bb0edcf58e67a",
        "2c5be738c6102ba9a33b4d4729e24eff",
      ],
    },
    "data.goods.goodId": "6dd6d7a8c1e282f486fe7877486c7f82",
  },
  {
    $set: { "data.goods.$.available": true },
  },
  { multi: true },
);
```

</details>

## delete

<details>
	<summary>simple delete by math</summary>
	
```js
db.getCollection('memo').remove({'_id': 12345678})
```
</details>

# other

get connection string

```js
db.getMongo();
```
