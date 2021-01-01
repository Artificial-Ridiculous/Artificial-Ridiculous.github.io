---
layout: post
title:  "HashMap的简单Java实现"
date:   2019-09-02 15:51:00 +0800
category: java
---

Hashmap可以做到常数时间的查找。主要思想是维护一个数组，数组中的每个元素是一条链表。

Value类-Employee：

```java
class Employee{
    public int id;
    public String name;
    public Employee next;

    public Employee(int id, String name) {
        this.id = id;
        this.name = name;

    }

    @Override
    public String toString() {
        return "Employee{" +
                "id=" + id +
                ", name='" + name + '\'' +
                '}';
    }
}
```

---

链表定义：

```java
class EmployeeList{
    private Employee head;

    public void add(Employee emp){
        if(head == null){
            head = emp;
            return;
        }
        Employee p = head;
        while(p.next != null){
            p = p.next;
        }
        p.next = emp;
    }

    public void traversal(int i){
        if(head == null){
            System.out.println("第"+(i+1)+"条链表为空.");
            return;
        }
        System.out.print("第"+(i+1)+"条链表信息为:");
        Employee p = head;
        while(p != null){
            System.out.print("->"+p);
            p = p.next;
        }
        System.out.println();
    }

    public Employee findEmpById(int id){
        boolean found = false;
        Employee p = head;
        while(p != null){
            if(p.id == id){
                found = true;
                break;
            }else{
                p = p.next;
            }
        }
        if(found){
            return p;
        }else {
            return null;
        }
    }
}
```

---

HashMap逻辑：

```java
class HashTable{

    private EmployeeList[] table;
    private int size;

    public HashTable(int size){
        // 重要! 此时只是创建了一个数组 但每个元素都是null
        table = new EmployeeList[size];
        this.size=size;
        for (int i = 0; i < size; i++) {
            //此时才把数组中每一个链表初始化了
            table[i] = new EmployeeList();
        }
    }

    public void add(Employee emp){
        int index = getIndex(emp);
        table[index].add(emp);
    }

    public int getIndex(Employee emp){
        return emp.id%size;
    }

    public void traversal(){
        for (int i = 0; i < size; i++) {
            table[i].traversal(i);
        }
    }

    public Employee findEmpById(int id){
        return table[id%size].findEmpById(id);
    }
}
```

