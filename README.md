# hello-lambda
Simple Java Hello World serverless function to be deployed in AWS Lambda

# Lambda Deploy

## 前提条件
```
C:\>node -v
v8.9.4
C:\>npm -v
5.8.0
C:\>aws --version
aws-cli/1.11.76 Python/2.7.9 Windows/8 botocore/1.5.39
```


## ツールセットアップ

1. Serverless Frameworkインストール
   ```
   C:\>npm install -g serverless
   ・・・
   C:\>sls -v
   1.27.3
   ```
2. AWS CLI セットアップ
   ```
   PS E:\awscli> aws configure
   AWS Access Key ID [****************BG4A]: 自分のAWS Access Key IDを入力し、Enterキー押下
   AWS Secret Access Key [****************BsCe]: 自分のAWS Secret Access Keyを入力し、Enterキー押下
   Default region name [us-west-2]: us-east-1
   Default output format [None]: json
   ```

## Javaのアプリケーション作成
1. Serverless Frameworkを使用して、Java開発用ディレクトリを自動生成する
   ```
   C:\>cd workspace
   C:\workspace>mkdir serverlesssample
   C:\workspace>cd serverlesssample
   C:\workspace\serverlesssample>serverless create --template aws-java-maven
   ・・・
   ```
   ※上記で作成されたディレクトリを用いて開発を始めても問題ない。しかし、すでにAWS Lambda向けのJavaアプリケーション開発用ディレクトリが存在する場合は、上記で生成されたserverless.ymlをコピーして、pom.xmlと同じディレクトリに配置すればよい。
2. Jarファイルのパッケージング
   AWS Lambdaにアップロードするjarファイルは依存するjarを含めて固める必要がある。
   そこで、pom.xmlに以下のpluginの記述をする。
   ```pom.xml
    <plugins>
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-shade-plugin</artifactId>
          <version>2.3</version>
          <configuration>
            <createDependencyReducedPom>false</createDependencyReducedPom>
          </configuration>
          <executions>
            <execution>
              <phase>package</phase>
              <goals>
                <goal>shade</goal>
              </goals>
            </execution>
          </executions>
        </plugin>
    </plugins>
   ```
3. 上記1,2を実施したJavaアプリケーションの取得
   本プロジェクトを `git clone`する。
4. ビルド
   ```
   C:\workspace\serverlesssample>mvn package shade:shade
   ```
   ビルド、パッケージングが成功するとtargetディレクトリ配下にjarファイルが生成される。

## AWS Lambdaへのデプロイ
1. serverless.ymlの修正
   ```
   service: aws-java-maven # NOTE: update this with your service name
   provider:
     name: aws
     runtime: java8
   
   # you can overwrite defaults here
     stage: dev
     region: us-west-2
   
   # you can add packaging information here
   package:
     artifact: target/hello-lambda-1.0.0-SNAPSHOT.jar
   
   functions:
     hello:
       handler: com.github.chrishantha.lambda.example.HelloWorldRequestHandler
   ```
2. Serverlessを使用したデプロイ
   ```
   C:\workspace\serverlesssample>serverless deploy -v