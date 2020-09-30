### stream流分割list

```java
/**
 *
 * @param list 要分割的list
 * @param maxNumber 分割大小
 * @param <T> 类型
 * @return
 */
public <T> List<List<T>> splitList(List<T> list,Integer maxNumber){
    int limit = (list.size() + maxNumber - 1) / maxNumber;
    List<List<T>> splitList = new ArrayList<>();
    Stream.iterate(0, n -> n + 1).limit(limit).forEach(i -> {
        splitList.add(list.stream().skip(i * maxNumber).limit(maxNumber).collect(Collectors.toList()));
    });
    return splitList;
}
```

### stream流循环处理list

```java
/**
* 将List<Integer>转换为List<String>
**/
@Test
public void method1(){
    List<Integer> intList = new ArrayList<>();
    intList.add(1);
    intList.add(2);
    intList.add(3);
    intList.add(4);
    intList.add(5);

    List<String> stringList = intList.stream().map(a -> {
        return a + "";
    }).collect(Collectors.toList());
    System.out.println(stringList.size());

    List<Integer> collect = intList.stream().skip(2).collect(Collectors.toList());
    System.out.println(collect);

}
```

### stream流list转map

```java
@Test
    public void  method2() {
        List<Map<String, Object>> mapList = new ArrayList<>();
        Map<String, Object> map = new HashMap<>();
        map.put("dataq_asset_action", "CREATE");
        map.put("count", 5L);
        mapList.add(map);

        Map<String, Object> map2 = new HashMap<>();
        map2.put("dataq_asset_action", "DELETE");
        map2.put("count", 4L);
        mapList.add(map2);

        Map<String, Object> map3 = new HashMap<>();
        map3.put("dataq_asset_action", "UPDATE");
        map3.put("count", 3L);
        mapList.add(map3);

        Map<Object, Object> collect = mapList.stream().collect(Collectors.toMap(p -> {
            return (Object) (p.get("dataq_asset_action"));
        }, k -> {
            return (Long) (k.get("count"));
        }));

        System.out.println(collect.size());

    }
```

