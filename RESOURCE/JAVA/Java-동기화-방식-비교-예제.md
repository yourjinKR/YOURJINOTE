

```java
package org.example.standard.thread;  
  
import java.util.concurrent.atomic.AtomicInteger;  
import java.util.concurrent.locks.Lock;  
import java.util.concurrent.locks.ReentrantLock;  
  
public class ConcurrencyEx2 {  
    static class Example {  
  
        public static void main(String[] args) throws InterruptedException {  
            BankAccountImpl account = new BankAccountAtomic();  
  
            Runnable task = () -> {  
                for (int i = 0; i < 100_000; i++) {  
                    account.deposit(1);  
                }  
            };  
  
            Thread t1 = new Thread(task);  
            Thread t2 = new Thread(task);  
  
            t1.start();  
            t2.start();  
  
            t1.join();  
            t2.join();  
  
            System.out.println("잔액: " + account.getBalance());  
        }  
    }  
  
    interface BankAccountImpl {  
        void deposit(int amount);  
        int getBalance();  
    }  
  
    // 동기화 되지 않음  
    static class BankAccount implements BankAccountImpl {  
        private int balance = 0;  
  
        public void deposit(int amount) {  
            balance += amount;  
        }  
  
        public int getBalance() {  
            return balance;  
        }  
    }  
  
    // synchronized 메서드  
    static class BankAccountAsync implements BankAccountImpl {  
        private int balance = 0;  
  
        public synchronized void deposit(int amount) {  
            balance += amount;  
        }  
  
        public int getBalance() {  
            return balance;  
        }  
    }  
  
    // synchronized 블럭  
    static class BankAccountAsyncBlock implements BankAccountImpl {  
        private int balance = 0;  
  
        public void deposit(int amount) {  
            synchronized (this) {  
                balance += amount;  
            }  
        }  
  
        public int getBalance() {  
            return balance;  
        }  
    }  
  
    // ReentrantLock 사용  
    static class BankAccountReentrant implements BankAccountImpl {  
        private int balance = 0;  
        private final Lock lock = new ReentrantLock();  
  
        public void deposit(int amount) {  
            lock.lock();  
  
            try {  
                balance += amount;  
            } finally {  
                lock.unlock();  
            }  
        }  
  
        public int getBalance() {  
            return balance;  
        }  
    }  
  
    // CAS 기반 Atomic Wrapper Class  
    static class BankAccountAtomic implements BankAccountImpl {  
        private AtomicInteger balance = new AtomicInteger(0);  
  
        @Override  
        public void deposit(int amount) {  
            balance.addAndGet(amount);  
        }  
  
        @Override  
        public int getBalance() {  
            return balance.get();  
        }  
    }  
}
```