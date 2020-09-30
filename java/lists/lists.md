#### 分割list

```java
public class testList {
 
    @Test
    public void  test(){
        List<Integer> numList = Lists.newArrayList(1, 2, 3, 4, 5, 6, 7, 8);
 
        List<List<Integer>> lists=Lists.partition(numList,3);
        System.out.println(lists);//[[1, 2, 3], [4, 5, 6], [7, 8]]
 
    }
 
}
```

