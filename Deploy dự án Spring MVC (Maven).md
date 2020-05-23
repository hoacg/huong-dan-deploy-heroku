# Deploy dự án Spring MVC (Maven)

## Bước 1 - Sử dụng biến môi trường

Sửa mã nguồn trong `dataSource()` như sau:

```java
@Autowired
private Environment environment;

@Bean
    public DataSource dataSource(){
        DriverManagerDataSource dataSource = new DriverManagerDataSource();

        String dbDriver = environment.getProperty("DB_DRIVER");
        String dbType = environment.getProperty("DB_TYPE");
        String dbHost = environment.getProperty("DB_HOST");
        String dbName = environment.getProperty("DB_NAME");
        String dbConfig = environment.getProperty("DB_CONFIG");
        String dbUsername = environment.getProperty("DB_USERNAME");
        String dbPassword = environment.getProperty("DB_PASSWORD");

        dataSource.setDriverClassName(dbDriver);
        String urlDatabase = dbType + dbHost + dbName + dbConfig;
        dataSource.setUrl(urlDatabase);
        dataSource.setUsername(dbUsername);
        dataSource.setPassword(dbPassword);
        return dataSource;
    }
```

## Bước 2 - Thiết lập biến môi trường trong Heroku

Cách làm:

* Vào mục `Settings` --> `Config Vars` --> `Reveal Config Vars` để có thể thêm các biến môi trường liên quan tới dự án.

Ví dụ như hình bên dưới:

![image-20200523233716061](_img/Deploy dự án Spring MVC (Maven)/image-20200523233716061.png)

## Bước 3 - Sử dụng webapp-runner trong Maven

Đầu tiên, bổ sung file `pom.xml`:

```xml
<build>
	...
	<plugins>
		...
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-dependency-plugin</artifactId>
      <executions>
        <execution>
          <phase>package</phase>
          <goals><goal>copy</goal></goals>
          <configuration>
            <artifactItems>
              <artifactItem>
              <groupId>com.heroku</groupId>
              <artifactId>webapp-runner</artifactId>
              <version>9.0.30.0</version>
              <destFileName>webapp-runner.jar</destFileName>
              </artifactItem>
            </artifactItems>
          </configuration>
        </execution>
      </executions>
    </plugin>
    ...
	</plugins>
	...
</build>
```

Sau đó, chạy lệnh `mvn package` để tải file `webapp-runner.jar` vào thư mục `target/dependency`.

## Bước 4 - Bổ sung Procfile

Tạo file Procfile ở thư mục gốc của dự án với mội dung như sau:

```properties
web: java $JAVA_OPTS -jar target/dependency/webapp-runner.jar --port $PORT target/*.war
```

## Bước 5 - Deploy

Tạo app mới trên Heroku và thực hiện lệnh sau để thêm remote heroku vào Git dự án:

```bash
$ heroku git:remote -a <tên-app-vừa-tạo>
```

Chạy lệnh `git push heroku master` để đưa các thay đổi ở các bước trên lên Heroku.

Cuối cùng, chạy lệnh sau để kích hoạt dyno web trong Heroku:

```bash
$ heroku ps:scale web=1
```


