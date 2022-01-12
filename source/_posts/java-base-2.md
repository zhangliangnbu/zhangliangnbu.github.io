---
title: Java基础二：生产者消费者模型
date: 2018-09-12 15:14:22
updated: 2022-01-11 10:52:48
tags:
- java
categories:
- Java
---

# 简介

生产者消费者模型是一种重要的模型，基于等待/通知机制。生产者/消费者模型描述的是有一块缓冲区作为仓库，生产者可将产品放入仓库，消费者可以从仓库中取出产品，生产者/消费者模型关注的是以下几个点：

- 生产者生产的时候消费者不能消费
- 消费者消费的时候生产者不能生产
- 缓冲区空时消费者不能消费
- 缓冲区满时生产者不能生产

生产者/模型作为一种重要的模型，它的优点在于：

- 解耦。有缓冲区，生产者和消费者不直接相互调用，就把生产者和消费者之间的强耦合解开，变为了生产者和缓冲区/消费者和缓冲区之间的弱耦合。
- 平衡生产者和消费者的处理能力。如果消费者直接从生产者这里拿数据，如果生产者生产的速度很慢，但消费者消费的速度很快，那消费者就得占用CPU的时间片白白等在那边。有了生产者/消费者模型，生产者和消费者就是两个独立的并发体，生产者把生产出来的数据往缓冲区一丢就好了，不必管消费者；消费者也是，从缓冲区去拿数据就好了，也不必管生产者，缓冲区满了就不生产，缓冲区空了就不消费，使生产者/消费者的处理能力达到一个动态的平衡。

<!--more-->

# 实现

```java
public class ProducerCustomer {
    public static void main(String[] args) {
        Barrel barrel = new Barrel();
        Producer producer1 = new Producer(barrel);
        Producer producer2 = new Producer(barrel);
        Customer customer1 = new Customer(barrel);
        Customer customer2 = new Customer(barrel);
        new Thread(producer1).start();
        new Thread(producer2).start();
        new Thread(customer1).start();
        new Thread(customer2).start();
    }

    public static class Barrel {
        // capacity
        private final Stack<Integer> stack = new Stack<>();
        private final int capacity;
        private static final int DEFAULT_CAPACITY = 4;

        public Barrel() {
            this.capacity = DEFAULT_CAPACITY;
        }

        public Barrel(int capacity) {
            this.capacity = capacity;
        }

        public void put(int val) throws Exception {
            // prepare
            Thread.sleep((long) (1000 * Math.random()));

            synchronized (this) {
                // full?
                while (stack.size() >= capacity) {
                    System.out.println("满了，等待...");
                    wait();
                }
                stack.push(val);
                System.out.println("生产：" + val + "/剩余：" + stack);
                notifyAll();
            }
        }

        public void pop() throws Exception {
            // prepare
            Thread.sleep((long) (1000 * Math.random()));
            // empty
            synchronized (this) {
                while (stack.isEmpty()) {
                    System.out.println("空了，等待...");
                    wait();
                }
                int val = stack.pop();
                System.out.println("消费：" + val + "/剩余：" + stack);
                notifyAll();
            }
        }
    }

    public static class Producer implements Runnable {
        private Barrel barrel;
        public Producer(Barrel barrel) {
            this.barrel = barrel;
        }

        @Override
        public void run() {
            for (int i = 0; i < 10; i ++) {
                try {
                    barrel.put(new Random().nextInt(100));
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static class Customer implements Runnable {
        private Barrel barrel;
        public Customer(Barrel barrel) {
            this.barrel = barrel;
        }

        @Override
        public void run() {
            for (int i = 0; i < 10; i ++) {
                try {
                    barrel.pop();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

