# Java 注解

注解（Annotation）最大的好处是让代码更加简洁，具有更高的可读性。

## 三个标准注解

* @Override 表示该方法将覆盖父类的方法。只提示作用
* @Deprecated 表示该代码过期，不建议使用。
* @SuppressWarnings 告诉编译器忽略警告

## 四个元注解

元注解的意思是可以通过他们来创造更多的自定义注解。

* @Target
* @Retention
* @Document
* @Inherited

### @Target

@Target 表示该注解的作用位置，可选的位置有 

* CONSTRUCTOR 构造器
* FIELD 属性
* LOCAL_VARIABLE 也就是局部变量
* METHOD 方法
* PACKAGE 包
* PARAMETER 参数，包括方法和构造器的参数
* TYPE 类，接口，枚举

### @Retention 

@Retention 表示该注解的级别。可选的范围有：

* SOURCE : 只在源代码里出现，编译后就被丢弃了。
* CLASS : 会被编译到 class 文件里。
* RUNTIME : 在 JVM 加载 class 文件的时候，也会把注解的信息加载进去，并且参与执行。

### @Document

表示在生产 JavaDoc 文档的时候把注解信息也包含进去。对于不部分人和团队来说，呵呵～～

### @Inherited 

表示子类运行继承父类的注解。

## Android butterknife 框架的简单原理

注解的还是比较简单的，但是不好理解。重点在于反射能不能理解。关于反射，参考这个

咱们拿 butterknife 大致的思想来分析一下。

咱们一般这样使用 bufferknife : 

    public class DoctorActivity extends Activity{
      
      @InjectView(R.id.doctor_age)
      private TextView mDoctorAge;
      
      public void onCreate(Bundle saveInstanceState){
          BufferKnife.inject(this);
      }
        
    }
    
这里干了这么几件事：
1. 定义属性，并且添加了注解
2. 在 onCreate 方法里调用 BufferKnife.inject() 方法
    
然后 @InjectView 大概是这样子的：

    @Target(ElementType.FIELD)
    @Retention(RetentionPolicy.RUNTIME)
    public @interface HDFInjectView {
        int value();
    }

这个注解大概表达了以下几个意思：
1. 这个注解的作用域是变量
2. 这个注解的级别是运行时

然后 BufferKnife.inject() 大概是这样子的：

    public static void inject(Activity activity){
        Class clazz = activity.getClass();
        Log.i(TAG,"class = " + clazz);
        Field[] fields = clazz.getDeclaredFields();
        Log.i(TAG, "Fields is = " + fields);
        for(Field field : fields){
            HDFInjectView inject = field.getAnnotation(HDFInjectView.class);
            Log.i(TAG,"inject = " + inject + " field = " + field);
            if(inject != null){
                int id = inject.value();
                View view = activity.findViewById(id);
                Log.i(TAG,"view = " + view);
                try {
                    field.setAccessible(true);
                    field.set(activity,view);
                } catch (IllegalAccessException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    
主要的步骤是这样的：

1. 获得该 Activity 的 class 对象
2. 获得该 Activity 定义的所有变量，也就是 Field
3. 遍历所有 Field
4. 获得声明了 HDFInjectView 注解的 Field，也就是咱们定义的各种 View
5. 获得该 Field 的注解的值，也就是在 View 的注解上声明的 id
6. 通过 Activity.findViewById() 的方法，获得已经加载到内存的 View
7. 给 Field 赋值

所有这些等于：

    mDoctorAge = (TextView)findViewById(R.id.doctor_age);
