---
title: 项目架构——JetPack『Room』
tags: [JetPack,Room,开源项目学习]
---

#### 项目介绍

[**官方文档翻译**]Room框架提供了一种位于SQLite之上的健壮的数据库操作层。

Room可以帮助你缓存设备中的应用数据。这里的缓存作为唯一数据源服务于应用程序，不管当前是否有网络连接，都能提供连续不中断的数据浏览体验。

#### 常用注解

##### @TypeConverters

```java
//类定义
package android.arch.persistence.room;
/**
 * 在Room中为指定的类/接口，方法/变量设置转换器。
 * 如果在{Database}上指定转换器，那么这个Database中所有的Dao和Entry都可以使用这个转换器。
 * 如果在{Dao}上指定转换器，那么所有这个Dao中的所有的方法都可以使用这个转换器。
 * 如果在{Entity}上指定转换器，那么所有这个Entity中的所有的成员变量都可以使用这个转换器。
 * 如果在{POJO}上指定转换器，那么所有这个POJO中的所有的成员变量都可以使用这个转换器。
 * 如果在{Entity的成员变量}上指定转换器，那么只有这个成员变量可以使用这个转换器。
 * 如果在{Dao的方法}上指定转换器，那么所有这个方法中的所有的参数都可以使用这个转换器。
 */
@Target({ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.FIELD})
@Retention(RetentionPolicy.CLASS)
public @interface TypeConverters {
    /**
     * 定义转换器的类的列表，如果转换器的方法不是静态的，Room会自动创建一个他们的实例对象
     */
    Class<?>[] value();
}
//举例使用

```

举例使用：

```java
//AppDatabase.java
@Database(entities = {User.class, Book.class}, version = 3)
@TypeConverters({Converters.class})
public abstract class AppDatabase extends RoomDatabase {
	...
}

//Converters.java
public class Converters {

	@TypeConverter
	public static Date fromTimestamp(Long value) {
		return value == null ? null : new Date(value);
	}

	@TypeConverter
	public static Long dateToTimestamp(Date date) {
		return date == null ? null : date.getTime();
	}
}
```

从Room自动编译生成的代码可以知道，**fromXxx在load流程中调用，dateToXxx在bind流程中使用**

<!--more-->

#### CURD

##### 清空数据

```kotlin
@Query("delete from word where id > 0")
fun clear()
```

回调方法

```
RoomDatabase.Callback
```

根据id查询数据，使用:id指代参数中的id

```kotlin
@Query("Select *from tablename where id=:id")
fun findWithId(vararg id:Long)
```

根据指定的id数组，查询

```java
@Query("SELECT * FROM user WHERE uid IN (:userIds)")
List<User> loadAllByIds(int[] userIds);
```



#### 版本更新

#### 注意事项

1. 有Compiler的引用，如果是Kotlin环境下的，需要使用kapt

   ```java
   apply plugin: 'kotlin-kapt'
   api rootProject.ext.dependencies["room"]
   api rootProject.ext.dependencies["room-rxjava"]
   kapt rootProject.ext.dependencies["room-compiler"]
   annotationProcessor rootProject.ext.dependencies["room-compiler"]
   ```

2. 需要在每个用到APT编译生成代码的**模块**中<font color="#dd0000">**都**</font>引入对Compiler的依赖，**或者可以这么理解，kapt、annotationProcessor只是一个编译插件，无法通过api传递依赖，所以需要在用到地方都引用**。

   ```java
   kapt rootProject.ext.dependencies["room-compiler"]
   annotationProcessor rootProject.ext.dependencies["room-compiler"]
   ```


#### 集成使用

1. 结合LiveData+Paging+ViewModel使用

   这种用法比较常规，关于LiveData的详细介绍可以查看笔者的另一篇博客：JetPack『LiveData』

   首先在DAO中定义返回DataSource.Factory的方法：

   ```java
   @Query("SELECT * FROM cheese ORDER BY name COLLATE NOCASE ASC")
   DataSource.Factory<Integer, Cheese> allCheesesByName();
   ```

   其次在ViewModel中创建LiveData对象：

   ```java
   public class CheeseViewModel extends AndroidViewModel {
   
       private final CheeseDao mCheeseDao;
       public final LiveData<PagedList<Cheese>> listLiveData;
   
       public CheeseViewModel(@NonNull Application application) {
           super(application);
           mCheeseDao = CheeseDb.getDatabase(application).cheeseDao();
           LiveData<PagedList<Cheese>> listLiveData = new LivePagedListBuilder<>(mCheeseDao.allCheesesByName(),new PagedList.Config.Builder()
            .setPageSize(30).setEnablePlaceholders(true).build())
            .build();
       }
   }
   
   ```

   最后为LiveData添加Observer：

   ```java
   CheeseViewModel viewModel = ViewModelProviders.of(this).get(CheeseViewModel.class);
   viewModel.listLiveData.observe(this, new Observer<PagedList<Cheese>>() {
               @Override
               public void onChanged(@Nullable PagedList<Cheese> cheeses) {
                   adapter.submitList(cheeses);
               }
           });
   ```

2. 结合RxJava使用

   一度给我造成一点麻烦，始终不能返回Observable\<T>，后来发现是因为是版本问题，<font color="#dd0000">**注意：自2.0.0-beta01版本开始才支持返回Observable**</font>，而Room 2.0+又需要将Support升级到AndroidX才行。

   升级AndroidX：

   1. 升级Android Studio 到3.2及以上
   2. 升级CompileSdkVersion到28及以上，升级targetSdkVersion到28及以上，升级buildToolVersion到28.0.0及以上。
   3. 在AS中选择Refactor > Migrate to androidx，将现有代码中support的依赖升级到Androidx

   升级好AndroidX之后，在升级Room的版本到2.0.0以上，这里使用最新的2.2.0-rc01。现在可以在Dao中返回Observable类型数据了：

   ```java
   @Dao
   public interface UserDao {
   	@Query("SELECT * from user")
   	Observable<List<User>> loadUser();
   }
   ```

   <font color="#dd0000">**这里有一个坑坑坑！上面返回Observable<List\<T>>可以正常编译，如果改成Observable<ArrayList\<T>>就会报错！！**</font>

   支持Observable之后，查询得到的Observable对象也能像LiveData一样， 监听到数据库中数据的改变，因此每次插入/更新数据，都会触发Observable的刷新。

   