### Android面试知识点

[TOC]

> 感悟：
>
> 1. 选一个自己擅长的领域
> 2. 基础一定要背，常用的api
> 3. 试着去了解这个领域，市面上的技术
> 4. 如果有时间，研究一个众所周知的库的源码

#### 第一,基础知识点

##### 1.1 四大组件之一——Activity

###### 1.1.1 生命周期（正常，异常）

> 四种状态：APSK(Active, Paused, Stoped, Killed),通过Activity栈来管理Activity
>
> Active: 栈顶，可见，可交互
>
> Pause: 可见，不可交互（例如 dialog挡住，透明层挡住）
>
> Stop：不可见

**正常情况下：7个回调函数**

<img src="/Users/burt/Documents/Android面试知识点/image-20191116230640063.png" alt="image-20191116230640063" style="zoom:50%;" />

**异常情况下：内存不足，系统配置发生改变**

异常退出时保存数据：onSaveInstanceState(Bundle xxx)

重新创建时恢复数据：onRestoreInstanceState(Bundel yyy)



###### 1.1.2 Activity组件之间的通信

> 3种类型：
>
> Activity - Activity之间
>
> Activity - Service之间
>
> Activity - Fragment之间

1. Activity - Activity之间3种方式

   - Intentd/Bundle

     ```java
             // 新建用于封装数据的 bundle对象
             Bundle bundle = new Bundle();
             bundle.putString("name", "xubobo");
             bundle.putInt("age", 18);
     				// 创建Intent对象
     				Intent intent = new Intent(OneActivity.this, MainActivity.class);
             // 将bundle对象 嵌入 Intent中
             intent.putExtras(bundle);
     				// 跳转Activity
             startActivity(intent);
     
     -------------------------------------------------------------------
       			// 在第二个Activity中获取Intent
       			Intent intent = getIntent();
             String name = intent.getStringExtra("name");
             int age = intent.getIntExtra("age", 0);
     ```

     

   - 类静态变量

   - 全局变量

2. Activity - Service之间

   > 三种方式：
   >
   > 1. 使用Binder，借助于 ServiceConnection这个类
   > 2. 简单通信，使用Intent
   > 3. 使用 handler的callback

3. Activity - Fragment之间

   - Activity - Fragment

     > 两种方式：
     >
     > 使用bundle
     >
     > 在Activity中定义public方法，然后在 Fragment中调用

     ```java
     // 第一种方式，使用bundle
     				Bundle bundle = new Bundle();
             bundle.putString("key", "value is value");
             Fragment fragment = new BlankFragment();
             // 将fragment 与  bundle绑定
             fragment.setArguments(bundle);
     
            -----------------------------------
              // fragment的onStart方法，获取值
              @Override
              public void onStart() {
                 super.onStart();
     
                 if (isAdded()) {
                     String key = getArguments().getString("key");
                 }
     
         }
     	
     -----------------------------------------------------------------------------------
     // 第二种方式，在Activity中定义方法
       
     		// 定义一个public方法，给fragment传值
         public String getTitleDemo() {
     
             return "it's cool";
         }
       ---------------------------------------------
         // fragment中的onAttach(Context context)方法
         
         @Override
         public void onAttach(Context context) {
             super.onAttach(context);
           
             // 强转为Activity， 然后调用Activity中的公开方法
             String titleDemo = ((InterviewActivity) context).getTitleDemo();
     
             Toast.makeText(context,titleDemo, Toast.LENGTH_SHORT).show();
     
         }
       
     ```

     

   - Fragment - Activity

     > 使用接口回调的方式：
     >
     > 1. 在fragment中定义接口Listener，然后由Activity实现
     > 2. 在fragment的onAttach(Activity activity)中为listener赋值
     > 3. 当onDetach时，需要释放listener, 将listener置为null

     ```java
     // 第一步
     /**
          * This interface must be implemented by activities that contain this
          * fragment to allow an interaction in this fragment to be communicated
          * to the activity and potentially other fragments contained in that
          * activity.
          * <p>
          * See the Android Training lesson <a href=
          * "http://developer.android.com/training/basics/fragments/communicating.html"
          * >Communicating with Other Fragments</a> for more information.
          */
         public interface OnFragmentInteractionListener {
             // TODO: Update argument type and name
             void onFragmentInteraction(Uri uri);
         }
     	
     // 第二步
     		@Override
         public void onAttach(Context context) {
             super.onAttach(context);
             if (context instanceof OnFragmentInteractionListener) {
                 mListener = (OnFragmentInteractionListener) context;
             } else {
                 throw new RuntimeException(context.toString()
                         + " must implement OnFragmentInteractionListener");
             }
     
             // 强转为Activity， 然后调用Activity中的公开方法
             String titleDemo = ((InterviewActivity) context).getTitleDemo();
     
             Toast.makeText(context,titleDemo, Toast.LENGTH_SHORT).show();
         }
     
     // 第三步，释放listener
     		@Override
         public void onDetach() {
             super.onDetach();
             mListener = null;
         }
     ```

     

###### 1.1.3 四种启动模式

> standerd
>
> singleTop:栈顶复用——im对话框， 新闻推送打开窗-回调onNewIntent(Intent intent)
>
> singleTask：栈内复用——应用的主界面—回调onNewIntent(Intent intent)
>
> singleInstance：独占一个栈——来电显示

###### 1.1.4 源码解读startActivity

##### 1.2，四大组件之二——Service

###### 1.2.1, service和线程的区别与应用场景

###### 1.2.2, 如何管理service生命周期

###### 1.2.3, service和intentService的区别

###### 1.2.4, startService和bindService先后次序的问题

###### 1.2.5, 序列化：Parcelable 和 Serializable

###### 1.2.6, Binder机制

#### 第二，深入知识点

##### 2.1 Handler

##### 2.2 Binder

##### 2.3 Ui绘制

##### 2.4 事件分发

#### 第三，基本知识点的细节

#### 第四，系统核心机制