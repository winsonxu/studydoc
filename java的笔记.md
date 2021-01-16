编译跳过测试工程
mvn clean package -Dmaven.test.skip=true

后台运行
nohup java -jar yourapp.jar --server.port=9901 &