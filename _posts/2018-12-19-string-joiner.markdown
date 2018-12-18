---
layout: post
title:  "String Joiner using StringUtils and Streams"
author: Abhay
date:   2018-12-19 00:31:21 +0530
categories: java
comments: true
---

Several times we are in a dilemma with the approach to join collection of Strings using a particular separator in java.
In this post I'm going to demonstrate and compare two ways of doing this, it will be a simple function to convert a 
collection to Comma Separated Values. This might seem like a very naive topic with the advent of Streams in Java, 
but trust me you will shocked to see the performance difference in the following CSV utility.

<h1>String Joiner Using Streams</h1>

{% highlight java %}
public static String getCSV(Collection<String> coll) {
        if (CollectionUtils.isEmpty(coll)) {
            return null;
        }
        return coll.stream().collect(Collectors.joining(","));
    }
{% endhighlight %}

<h1>String Joiner Using StringUtils</h1>

{% highlight java %}
public static String getCSV(Collection<String> coll) {
        if (CollectionUtils.isEmpty(coll)) {
            return null;
        }
        return StringUtils.join(coll, ",");
    }
{% endhighlight %}

<h1>Let's Benchmark!</h1>
Now that we have seen the implementation using both the ways, let's try and benchmark both the implementations and 
see which is fast and is more suited for a production environment.

{% highlight java %}
public class Main {
    private static final int listSize = 100;
    private static final int collSize = 10000;
    private static final int WARMUP_ITERATIONS = 10000;
    public static void main(String[] args) {
        jvmWarmup();
        List<Collection<String>> randomSets = Lists.newArrayListWithCapacity(listSize);
        for (int i = 0; i < listSize; i++) {
            Collection<String> tempSet = Sets.newHashSet();
            for (int j = 0; j < collSize; j++) {
                tempSet.add(UUID.randomUUID().toString());
            }
            randomSets.add(tempSet);
        }
        //Using Streams Joiner
        long total = 0;
        for (Collection<String> collection : randomSets) {
            long start = System.nanoTime();
            String out = StreamsUtility.getCSV(collection);
            total += (System.nanoTime() - start);
        }
        System.out.println("Using Streams Joiner : " + total / listSize);
        //Using StringUtils Joiner
        total = 0;
        for (Collection<String> collection : randomSets) {
            long start = System.nanoTime();
            String out = StringUtilsUtility.getCSV(collection);
            total += (System.nanoTime() - start);
        }
        System.out.println("Using StringUtils Joiner : " + total / listSize);
    }
    private static void jvmWarmup() {
        for (int i = 0; i < WARMUP_ITERATIONS; i++) {
            UUID.randomUUID().toString();
        }
    }
}
{% endhighlight %}


<h1>Benchmark Results</h1>
The above code was ran with different collection sizes, and run times were noted down.
The code was run using `2.2 GHz Intel Core i7` CPU with `16 GB 1600 MHz DDR3` memory 
and `-Xmx3g -Xms3g -XX:+PrintCompilation -verbose:gc` jvm args.
Please note the time unit is nano seconds.

| Collection Size | StringUtils | Streams |
|-------|--------|---------|
| 10 | 56365 | 679153 |
| 100 | 117285 | 922402 |
| 1000 | 513523 | 1125027 |
| 10000 | 918946 | 2369459 |
| 100000 | 11471419 | 19090185 |

As you can see, a StringUtils join outperforms Streams API by far! With growing collection size,
the difference starts coming closer but still the StringUtils join beats Streams join by a huge margin.

Wondering why such a huge difference?! A simple answer to that is a lot of objects as well as lambdas are created and 
StringUtils uses native code like iterators and StringBuilder which has been highly optimized over the years.

Check out the [Code][code] for more info.

[code]: https://github.com/aksonuabhay/java-samples/tree/master/string-joiner

{% if page.comments %}
{% include disqus.html %}
{% endif %}