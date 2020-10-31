---
title: "Java In Action"
date: 2020-09-20
---

#Part 1
##Lambda
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

##Method references
```java
ToIntFunction<String> stringToInt = (String s) -> Integer.parseInt(s);
ToIntFunction<String> stringToInt = Integer::parseInt;

BiPredicate<List<String>, String> contains = (list, element) -> list.contains(element);
BiPredicate<List<String>, String> contains = List::contains;

Predicate<String> startsWithNumber = (String s) -> this.startsWithNumber(s);
Predicate<String> startsWithNumber = this::startsWithNumber;
```

##Useful methods to compose lambda expressions
- Comparators
```java
Comparator<Apple> c = Comparator.comparing(Apple::getWeight);
inventory.sort(comparing(Apple::getWeight).reversed());
```
Chaining Comparators
```java
inventory.sort(comparing(Apple::getWeight).reversed().thenComparing(Apple::getCountry));
```

- Composing Predicates
```java
Predicate<Apple> redAndHeavyAppleOrGreen = 
    redApple.and(apple -> apple.getWeight() > 150)
            .or(apple -> GREEN.equals(a.getColor()));
```

- Composing Functions
```java
    Function<Integer, Integer> f = x -> x + 1;
    Function<Integer, Integer> g = x -> x * 2;
    Function<Integer, Integer> h = f.andThen(g);
    int res = h.apply(1);
    
    Function<Integer, Integer> z = f.compose(g);
    int res2 = z.apply(2);
```
andThen means g(f(x))  
compose means f(g(x))

```java
public class Letter {
    public static String addHeader(String text) {
        return "header " + text;
    }

    public static String addFooter(String text) {
        return text + " footer";
    }

    public static String checkSpelling(String text) {
        return text.replaceAll("labda", "lambda");
    }
}

Function<String, String> addHeader = Letter::addHeader;
        Function<String, String> transformationPipeline =
                addHeader.andThen(Letter::addFooter)
                    .andThen(Letter::checkSpelling);
        String res = transformationPipeline.apply("shuai");
```
##Part 2



    
    