# log service java sdk
java sdk 是对所有log service 提供的API的封装，通过该sdk，可以调用所有log service。部分API文档请参考[文档中心](https://help.aliyun.com/document_detail/29007.html)。 

### sample 1 : 构建client
```

		String accessId = "your_access_id";
		String accessKey = "your_access_key";
		String host = "cn-hangzhou-staging-intranet.sls.aliyuncs.com";
		Client client = new Client(host, accessId, accessKey);

```

### sample 2 : 创建logstore
```

        String project = "your_project_name";
        String logstore = "your_logstore";
        int ttl_in_day = 3;
        int shard_count = 10;
        LogStore store = new LogStore(logstore, ttl_in_day, shard_count);
		CreateLogStoreResponse res = client.CreateLogStore(project, store);

```

### sample 3 : 写数据
```

		int log_group_num = 10;
		/**
		 * 向log service发送一个日志包，每个日志包中，有2行日志
		 */
		for (int i = 0; i < log_group_num; i++) {
			Vector<LogItem> logGroup = new Vector<LogItem>();
			LogItem logItem = new LogItem((int) (new Date().getTime() / 1000));
			logItem.PushBack("level", "info");
			logItem.PushBack("name", String.valueOf(i));
			logItem.PushBack("message", "it's a test message");

			logGroup.add(logItem);

			LogItem logItem2 = new LogItem((int) (new Date().getTime() / 1000));
			logItem2.PushBack("level", "error");
			logItem2.PushBack("name", String.valueOf(i));
			logItem2.PushBack("message", "it's a test message");
			logGroup.add(logItem2);

			try {
				client.PutLogs(project, logStore, topic, logGroup, "");
			} catch (LogException e) {
				System.out.println("error code :" + e.GetErrorCode());
				System.out.println("error message :" + e.GetErrorMessage());
				System.out.println("error requestId :" + e.GetRequestId());
				throw e;
			}

		}

```

### sample 4 : 读取数据
```

        int shardId = 0;  // 只读取0号shard的数据
		GetCursorResponse res;
		try {
		    // 获取最近1个小时接收到的第一批日志的cursor位置
			long fromTime = (int)(System.currentTimeMillis()/1000.0 - 3600);
			res = client.GetCursor(project, logStore, shardId, fromTime);
			System.out.println("shard_id:" + shardId + " Cursor:" + res.GetCursor());
		} catch (LogException e) {
			e.printStackTrace();
		}
		
		String cursor = res.GetCursor();
		while(true) {
			BatchGetLogResponse logDataRes = client.BatchGetLog(
							project, logStore, shardId, 100, cursor);
			// 读取到的数据
            List<LogGroupData> logGroups = logDataRes.GetLogGroups();
			
			String next_cursor = logDataRes.GetNextCursor();  // 下次读取的位置
			System.out.print("The Next cursor:" + next_cursor);
			if (cursor.equal(next_cursor)) {
						break;
			}
			cursor = next_cursor;
		}

```

## Maven配置
```
<dependency>
  <groupId>com.aliyun.openservices</groupId>
  <artifactId>aliyun-log</artifactId>
  <version>0.6.6</version>
</dependency>
```
