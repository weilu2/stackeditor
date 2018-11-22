```
package com.oolong;  
  
import com.mongodb.MongoClient;  
import com.mongodb.client.FindIterable;  
import com.mongodb.client.MongoCollection;  
import com.mongodb.client.MongoCursor;  
import com.mongodb.client.MongoDatabase;  
import org.bson.Document;  
  
import java.util.ArrayList;  
import java.util.List;  
  
/**  
 * Hello world! * */public class App   
{  
    public static void main( String[] args ) {  
  
        try{  
            // 连接到 mongodb 服务  
  MongoClient mongoClient = new MongoClient( "192.168.0.125" , 27017 );  
  
  // 连接到数据库  
  MongoDatabase mongoDatabase = mongoClient.getDatabase("medicalqa_7");  
  System.out.println("Connect to database successfully");  
  
  MongoCollection<Document> collection = mongoDatabase.getCollection("questions_2");  
  
  Document condition = new Document("_id", new Document("$gt", "00001866-b22b-11e8-add5-b82a72fc006c"));  
  
  // temp 1  
  long startTime = System.currentTimeMillis(); //获取开始时间  
  
  FindIterable<Document> findIterable = collection.find(condition).projection(new Document("_id", 1)).limit(5);  
  MongoCursor<Document> mongoCursor = findIterable.iterator();  
  
  List<Document> res = new ArrayList<>();  
  
 while(mongoCursor.hasNext()){  
                res.add(mongoCursor.next());  
  }  
  
            // TEMP 2  
  long endTime = System.currentTimeMillis(); //获取结束时间  
  System.out.println("程序运行时间：" + (endTime - startTime) + "ms");  
  
  String id = res.get(res.size()-1).get("_id").toString();  
  
  condition = new Document("_id", new Document("$gt", id));  
  findIterable = collection.find(condition).projection(new Document("_id", 1)).limit(1000000);  
  mongoCursor = findIterable.iterator();  
  
 while(mongoCursor.hasNext()){  
                res.add(mongoCursor.next());  
  }  
  
            long end2Time = System.currentTimeMillis(); //获取结束时间  
  System.out.println("程序运行时间：" + (end2Time - endTime) + "ms");  
  
  }catch(Exception e){  
            System.err.println( e.getClass().getName() + ": " + e.getMessage() );  
  }  
        System.out.println( "Hello World!" );  
  }  
}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIwNTg4MzE5OTFdfQ==
-->