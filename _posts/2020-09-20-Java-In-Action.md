---
title: "Java In Action"
date: 2020-09-20
---

#Lambda
- Where and how to use lambdas  
    Use a lambda expression in the context of a functional interface.
- Functional interface  
Functional interface only has one abstract method.  
- How to use lambdas
    1. Use functional interface
    2. Parameterize the behavior which means parameterize the functional interface
    3. Execute a behavior
    4. Pass lambdas
```java
public class Solution {
    //First step
    public interface BufferedReaderProcessor {
        public abstract String process(BufferedReader b) throws IOException;
    }
    //Second and Third
    public String processFile(BufferedReaderProcessor p) throws IOException {
        try (BufferedReader br = new BufferedReader(new FileReader("...."))) {
            return p.process(br);
        }
    }
    //Fourth
    String oneLine = processFile((BufferedReader b) -> b.readLine());
}
```
- Some useful functional interface
1. Predicate
```java
public class Solution {
    public interface Predicate<T> {
        boolean test(T t);
    }
    
    public <T> List<T> filter(List<T> list, Predicate<T> p) {
        List<T> ans = new ArrayList<>();
        for (T t : list) {
            if (p.test(t)) {
                ans.add(t);
            }
        }
        return ans;
    }
    List<String> nonEmptry = filter(listOfStrings, (String s) -> !s.isEmpty());
}
```
2. Consumer
```java
public interface Consumer<T> {
    void accept(T t);
}

public <T> void forEach(List<T> list, Consumer<T> c) {
    for (T t : list) {
        c.accept(t);
    }
}

forEach(listOfStrings, (String s) -> System.out.println(s));
```

3. Function
Use the interface when you need to map information from an input to an output
```java
public interface Function<T, R> {
    R apply(T t);
}

public <T, R> List<R> map(List<T> list, Function<T, R> f) {
    List<R> result = new ArrayList<>();
    for (T t : list) {
        result.add(f.apply(t));
    }
    return result;
}
```

#Method references
    
    