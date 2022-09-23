---
title: Java求解数组最值
excerpt: 学习一下Java一些常用工具类
date: 2022-08-19 20:33:58
tags: [Java]
categories: Java学习
banner_img: /img/banner_img/background2.jpg
index_img: /img/index_img/2.png
---
## 求解数组中的最大值

<p class="note note-primary">
使用Math工具类可以求解一些简单的数学运算，但是不能求解数组的最值问题。
</p>

使用Java的一些工具类可以快捷的实现


```Java
Integer[] a = {};
int mi = Collections.min(Arrays.asList(a));
```

### Collections.max

```Java
public static <T> T max(Collection<? extends T> coll,
                        Comparator<? super T> comp)
```
返回给定集合的最大元素，根据指定的比较器诱导的顺序。集合中的所有元素必须相互比较由指定的比较器。

需要读取的数据使用上界通配符（Upper Bounds Wildcards）`<? extends T>`

这表示只能存T的子类。

需要写入的数据使用下界通配符（Lower Bounds Wildcards）`<? super T>`

这表示只能存T的父类。

### Array.asList

```Java
public static <T> List<T> asList(T... a)

```
Arrays.asList()返回由指定数组支持的一个固定大小的列表(ArrayList)。List继承了接口Collection

## Java排序

<p class="note note-primary">
使用sort和匿名类可以很方便的排序，类去实现compare接口让类的实例可以排序。
数组用Arrays，集合用Collections
</p>

### 数组排序
逆序排列
```Java
Integer[] arrays = {1, 2, 3};
Arrays.sort(arrays, Collections.reverseOrder());
```

使用匿名函数（类）
```Java

Arrays.sort(arrays, new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {
        return o2 - o1;
        //如果返回正数则交换
        //当o2大的时候交换，也就是o2需要小，那么是降序
    }
});

```
### 集合排序

使用匿名函数（类）
```Java


List<Student> students = new ArrayList<Student>(Arrays.asList(studentWang, studentZhang, studentGou, studentZhao, studentLi));
Collections.sort(students, new Comparator<Student>() {
    public int compare(Student o1, Student o2) {
        if(null == o1.getAge()) {
            return -1;
        }
        if(null == o2.getAge()) {
            return 1;
        }
        return o1.getAge().compareTo(o2.getAge());
    }
});
```

实现接口
```Java

public class StudentAsc implements Comparable<StudentAsc> {
    private String name;
 
    private Integer age;
 
    public StudentAsc(String name, Integer age) {
        this.name = name;
        this.age = age;
    }
 
    public String getName() {
        return name;
    }
 
    public void setName(String name) {
        this.name = name;
    }
 
    public Integer getAge() {
        return age;
    }
 
    public void setAge(Integer age) {
        this.age = age;
    }
 
    public int compareTo(StudentAsc o) {
        if(null == this.age) {
            return -1;
        }
        if(null == o.getAge()) {
            return 1;
        }
        return this.age.compareTo(o.getAge());
    }
 
    @Override
    public String toString() {
        return "StudentAsc{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
 
}
//方法调用
List<StudentAsc> studentAscs = new ArrayList<StudentAsc>(Arrays.asList(studentWang, studentZhang, studentGou, studentZhao, studentLi));
Collections.sort(studentAscs);
```