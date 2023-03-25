### redis-序列化和反序列化

下面这个方法，用于将对象序列化到redis中，并且从redis取出后，反序列化到对象上

```java
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>

private static final ObjectMapper mapper = new ObjectMapper();

//创建对象
User user = new User("hurenxiang","30");
//手动序列化
String json = mapper.writeValueAsString(user);
//写入数据
stringRedisTemplate.opsForValue().set("user:200",json);
//获取数据
String s = stringRedisTemplate.opsForValue().get("user:200");
//手动反序列化
User user1 = mapper.readValue(s, User.class);
System.out.println("user1="+ user1);
```

### 算法排序技巧

#### Collections.sort

需求：给定一个map，key和value都是Integer类型，先根据value进行升序排序，如果value相同，根据key降序排序

```java
public class ListTest {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();
        list.add(2);
        list.add(3);
        list.add(1);
        list.add(3);
        list.add(2);
        Map<Integer,Integer> map = new HashMap<>();
        for (int i = 0 ; i < list.size() ; i++){
            map.put(list.get(i),map.getOrDefault(list.get(i),0)+1);
        }
        Collections.sort(list,(a,b)->{
            int cn1 = map.get(a);
            int cn2 = map.get(b);
            return cn1 != cn2 ? cn1 - cn2 : a - b;
        });
        list.stream().forEach(System.out::println);
    }
}
```

