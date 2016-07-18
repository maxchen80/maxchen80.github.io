---
layout: post
title: MongoDB手册中文简化版
---

### 安装MongoDB
	# curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.2.0.tgz
	# tar -zxvf mongodb-linux-x86_64-3.2.0.tgz
	# mv mongodb-linux-x86_64-3.2.0 /usr/local/mongodb
	# cd /usr/local/mongodb
	# mkdir -p data/db data/log
	# vi mongod.cfg
	systemLog:
	   destination: file
	   path: /usr/local/mongodb/data/log/mongod.log
	storage:
	   dbPath: /usr/local/mongodb/data/db
	net:
	   #bindIp: 127.0.0.1
	   port: 27017
	processManagement:
	   fork: true
	# ./mongod --config ../mongod.cfg &
	# ./mongo -host 127.0.0.1 -p 27017

### MongoDB的CRUD操作
插入文档
查询文档
修改文档
删除文档

db.createUser(
    {
      user: "maxchen",
      pwd: "vH4pP9jW",
      roles: [
         { role: "dbOwner", db: "test" }
      ]
    }
)
db.updateUser(
   "maxchen",
   {
      pwd: "vH4pP9jWm"
   }
)
	# ./mongod --config ../mongod.cfg --auth &
	# ./mongo --host 127.0.0.1 --port 27017 -u maxchen -p vH4pP9jWm



	# ./mongodump -h 127.0.0.1 -p 27017 -u maxchen -p vH4pP9jWm -d test -o backup001
	# ./mongorestore -h 127.0.0.1 -p 27017 -u maxchen -p vH4pP9jWm -d test backup001/test/

	# ./mongoexport -h 127.0.0.1 -p 27017 -u maxchen -p vH4pP9jWm -d test -c magnet -o magnet_20160316.json
	# ./mongoimport -h 127.0.0.1 -p 27017 -u maxchen -p vH4pP9jWm -d test -c magnet magnet_20160316.json 
