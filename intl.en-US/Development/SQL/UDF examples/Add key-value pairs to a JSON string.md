---
keyword: adding key-value pairs to JSON strings
---

# Add key-value pairs to a JSON string

This topic describes how to use a Java UDF to add one or more key-value pairs to a basic JSON string or overwrite the pairs. Nested JSON strings support this function.

## UDF used to add key-value pairs to a JSON string

```
PutJsonValues(String jsonBase,String Key1,String Value1,String Key2,String Value2...)
```

-   Description: used to add one or more key-value pairs to a basic JSON string or overwrite the pairs. Nested JSON strings support this function, and the number of parameters can be an odd value. This UDF can work with the `ToJsonObject` function in [Convert data types to JSON STRING](/intl.en-US/Development/SQL/UDF examples/Convert data types to JSON STRING.md). You must register the `ToJsonObject` function first.
-   Parameters:
    -   jsonBase: the basic JSON string to which you want to add key-value pairs.
    -   Key1: Key1 you want to add, which is of the STRING type.
    -   Value1: Value1 that corresponds to Key1. It is of the STRING type.
    -   Key2: Key2 you want to add, which is of the STRING type.
    -   Value2: Value2 that corresponds to Key2. It is of the STRING type.

## UDF example

-   Resource upload

    Due to sandbox limits, to run this function in Alibaba Cloud MaxCompute, you need to upload the fastjson.JSON and UDF JAR packages to MaxCompute as resources and reference the two packages when you create a function in DataWorks. You can visit [MVN](https://mvnrepository.com/artifact/com.alibaba/fastjson) to download the fastjson.JSON package.

-   Function registration

    After PutJsonValues.java passes the test, register it as a function.

    In this example, PutJsonValues.jar is the JAR package generated by packaging the example code of the UDF used to add key-value pairs to a JSON string. ODPSUDF-1.0-SNAPSHOT2.jar is the JAR package generated by packaging `ToJsonString` in the previous topic. fastjson-1.2.28.odps.jar is a fastjson.JSON package.

    **Note:**

    -   For more information about the procedure in this example.
    -   To publish a UDF to a server for production use, the UDF needs to go through packaging, uploading, and registration. You can use the one-click publish function to complete these steps. MaxCompute Studio allows you to run the `mvn clean package` command, upload a JAR package, and register the UDF in sequence. For more information, see [t12133.md\#](/intl.en-US/Tools and Downloads/MaxCompute Studio/Developing Java/Package, upload, and register a Java program.md).
    -   For more information about how to upload resources by using common commands on the client, see [Resource operations](/intl.en-US/Development/Common commands/Resource operations.md).
-   Example

    After the UDF is registered, execute the following statement:

    ```
    SELECT tojson(NULL)
    ,  tojson(concat('123', chr(3)))
    ,  tojson(123)
    ,  tojson(12.3)
    ,  tojson(true)
    ,  tojson(CAST(123  AS  DECIMAL))
    ,  putJsonValuesTest('{}', 'a', '123', 'b', tojson('abc'), 'c', '{"c1":567}')
    FROM dual;
    ```

    The result is as follows:

    ```
    null
    "123\u0003"
    123
    12.3
    true
    123
    {"a":123,"b":"abc","c":{"c1":567}}
    ```

    **Note:**

    -   `null` indicates the NULL value. `a` is a numeric value without quotation marks. `b` is a string which needs to be converted to a JSON string by using `tojson`. `{" c1":567}` is a valid JSON element and can be imported directly.
    -   `putJsonValues` is function that you register by using DataWorks or [IntelliJ IDEA](/intl.en-US/Tools and Downloads/MaxCompute Studio/Developing Java/Package, upload, and register a Java program.md).

## UDF code example

```
import com.alibaba.fastjson.JSON; // Alibaba Cloud UDF.
import com.alibaba.fastjson.JSONObject; // The fastjson.JSON class is introduced.
import com.aliyun.odps.udf.UDF;

public class PutJsonValues extends UDF {
// Data of the STRING type in MaxCompute is consistent with that in Java UDFs. The String... keyValues field is a changeable formal parameter, which indicates that the number of parameters is uncertain.
    public  PutJsonValues(){
    }

    public  String evaluate(String jsonBase, String... keyValues) {
        if (jsonBase == null) {
            return null;
        }
        JSONObject outputObj = JSON.parseObject(jsonBase);
        for (int i = 1; i < keyValues.length; i = i + 2) {
            String key = keyValues[i - 1];
            String value = keyValues[i];
            if (key == null || value == null) {
                continue;
            }
            Object valueObj = JSON.parse(value);
            outputObj.put(key, valueObj);
        }
        return JSON.toJSONString(outputObj);
    }
}
```

## UDF unit testing

```
// The following code is used to check whether the data type conversion takes effect. You can comment out the code when you use the function.
public static void main(String[] args) {
    PutJsonValues putJsonValues=new PutJsonValues();
    System.out.println(putJsonValues.hashCode());

    String js=putJsonValues.evaluate("{}", "k", "null");
    System.out.println(js);
    System.out.println(new PutJsonValues().evaluate("{}", "a", "123", "b", "234", "c", "345"));
    System.out.println(new PutJsonValues().evaluate("{}", "k", "\"abc\""));
    System.out.println(new PutJsonValues().evaluate("{}", "k", "{\"kk\":123}"));
}
```
