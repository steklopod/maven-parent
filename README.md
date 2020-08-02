# Корневой POM для микросервисов
Регламентирует версии используемых библиотек. Задаются следующее:
* properties
* dependencyManagement
* build.pluginManagement

# Особенности
Автоматически подключен плагин 
```xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>versions-maven-plugin</artifactId>
</plugin>
```

# Подключение
## Корневой pom сервиса
1.  Указывается parent проект
```xml
<parent>
    <groupId>by.steklopod</groupId>
    <artifactId>microservice-parent</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</parent>
```
2. Убирается тег `<groupId>by.steklopod</groupId>`
3. Убираются версии библиотек `dependencyManagement`
4. Убирается плагин `versions-maven-plugin`
5. В теге `properties` указываются версии сервисов/библиотек платформы
6. В `dependencyManagement` указываются редко/специфичные для данного модуля зависимости

## Подключение impl-части сервиса
1. Версии сторонних библиотек должны контролироваться в microservice-parent
2. Прописываются версии компонентов платформы из корневого pom
3. В секции `<plugins>` остается следующее:
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
        </plugin>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-deploy-plugin</artifactId>
        </plugin>
        <plugin>
            <artifactId>maven-surefire-plugin</artifactId>
        </plugin>
        <plugin>
            <artifactId>maven-failsafe-plugin</artifactId>
        </plugin>
        <plugin>
            <groupId>org.sonarsource.scanner.maven</groupId>
            <artifactId>sonar-maven-plugin</artifactId>
        </plugin>
        <plugin>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

## Подключение **api**-части сервиса
1. Указывается зависимость:
```xml
<dependency>
    <groupId>by.steklopod</groupId>
    <artifactId>microservice-swagger-resource</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</dependency>
```

2. Указываются генераторы
```xml
<build>
    <plugins>
        <plugin>
            <artifactId>maven-dependency-plugin</artifactId>
        </plugin>
        <plugin>
            <groupId>org.openapitools</groupId>
            <artifactId>openapi-generator-maven-plugin</artifactId>
            <executions>
                <!-- Генерация интерфейсов и модели для внешних взаимодействий -->
                <execution>
                    <id>generate-external</id>
                    <goals>
                        <goal>generate</goal>
                    </goals>
                    <configuration>
                        <generatorName>spring</generatorName>
                        <inputSpec>${project.basedir}/src/main/resources/external/openapi.yml</inputSpec>
                        <generateSupportingFiles>false</generateSupportingFiles>
    
                        <!-- Generate API -->
                        <apiPackage>${project.groupId}.collection.api</apiPackage>
                        <generateApiDocumentation>false</generateApiDocumentation>
                        <generateApis>true</generateApis>
                        <generateApiTests>false</generateApiTests>
    
                        <!-- Generate Model -->
                        <modelPackage>${project.groupId}.collection.model</modelPackage>
                        <generateModelDocumentation>false</generateModelDocumentation>
                        <generateModels>true</generateModels>
                        <generateModelTests>false</generateModelTests>
    
                        <!-- Generate only interfaces, without server stubs -->
                        <configOptions>
                            <interfaceOnly>true</interfaceOnly>
                            <useTags>true</useTags>
                            <hideGenerationTimestamp>true</hideGenerationTimestamp>
                        </configOptions>
    
                        <!-- Use custom class for date fields -->
                        <importMappings>
                            <importMapping>java.time.OffsetDateTime=java.time.LocalDateTime</importMapping>
                        </importMappings>
                        <typeMappings>
                            <typeMapping>OffsetDateTime=LocalDateTime</typeMapping>
                        </typeMappings>
                        <!--Use custom templates.templates to get ride of default methods in generated interfaces -->
                        <!--Only api.mustache template had changed -->
                        <templateDirectory>${biometrics.swagger.template}</templateDirectory>
                    </configuration>
                </execution>
                <!-- Генерация интерфейсов и модели для внутренних взаимодействий -->
                <execution>
                    <id>generate-internal</id>
                    <goals>
                        <goal>generate</goal>
                    </goals>
                    <configuration>
                        <generatorName>spring</generatorName>
                        <inputSpec>${project.basedir}/src/main/resources/internal/openapi.yml</inputSpec>
                        <generateSupportingFiles>false</generateSupportingFiles>
    
                        <!-- Generate API -->
                        <apiPackage>${project.groupId}.collection.api.internal</apiPackage>
                        <generateApiDocumentation>false</generateApiDocumentation>
                        <generateApis>true</generateApis>
                        <generateApiTests>false</generateApiTests>
    
                        <!-- Generate Model -->
                        <modelPackage>${project.groupId}.collection.model.internal</modelPackage>
                        <generateModelDocumentation>false</generateModelDocumentation>
                        <generateModels>true</generateModels>
                        <generateModelTests>false</generateModelTests>
    
                        <!-- Generate only interfaces, without server stubs -->
                        <configOptions>
                            <interfaceOnly>true</interfaceOnly>
                            <useTags>true</useTags>
                            <hideGenerationTimestamp>true</hideGenerationTimestamp>
                        </configOptions>
    
                        <!-- Use custom class for date fields -->
                        <importMappings>
                            <importMapping>java.time.OffsetDateTime=java.time.LocalDateTime</importMapping>
                            <importMapping>Page=org.springframework.data.domain.Page</importMapping>
                        </importMappings>
                        <typeMappings>
                            <typeMapping>OffsetDateTime=LocalDateTime</typeMapping>
                        </typeMappings>
                        <!--Use custom templates.templates to get ride of default methods in generated interfaces -->
                        <!--Only api.mustache template had changed -->
                        <templateDirectory>${biometrics.swagger.template}</templateDirectory>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```
### Проверка последних версий библиотек
```shell script
mvn versions:display-dependency-updates
```
