
#Fragment的创建方法

1. xml中直接定义:

        <?xml version="1.0" encoding="utf-8"?>
        <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical" >
        
            <fragment
                android:name="com.example.android.FooFragment"
                android:id="@+id/fooFragment"
                android:layout_width="match_parent" 
                android:layout_height="match_parent" />
        
        </LinearLayout>

2. 动态生成:
        
        <?xml version="1.0" encoding="utf-8"?>
        <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical" >
        
          <FrameLayout
               android:id="@+id/your_placeholder"
               android:layout_width="match_parent"
               android:layout_height="match_parent">
          </FrameLayout>
        
        </LinearLayout>

        
        FragmentTransaction ft = getSupportFragmentManager().beginTransaction();
        ft.replace(R.id.your_placeholder, new FooFragment());
        ft.commit();

# Fragment 的生命周期:
    
onAttach() Fragment被绑定到Activity (一般用于获得activity的传递的值)

onCreate() 开始创建Fragment,实例化成员变量

onCreateView() 开始创建界面,实例化布局(给当前的fragment绘制UI布局)

onViewCreated() 界面创建结束(一般用于实例化控件)

onActivityCreated() 表示activity执行oncreate方法完成了的时候会调用此方法

onStart() Fragment准备显示到屏幕

onResume() 开始交互

onPause() 暂停交互

onDestroyView() Fragment的布局开始销毁,此时Fragment仍然处于前台

onDestroy() Fragment被销毁. 该Fragment不再被使用

onDetach() 从Activity解除绑定

生命周期图:
![https://i.imgur.com/0EVReuq.png](https://i.imgur.com/0EVReuq.png)
    
使用注意点:
onAttach() 用来绑定监听等
onCreate() 传递数据,同时尽可能多的实例化和UI无关的对象,例如Adapter
onCreateView() 仅仅用来实例化布局
onViewCreated() 示例话和UI控件,完成绑定,添加监听事件等
onDetach() 用来解除监听

        public class SomeFragment extends Fragment {
            ThingsAdapter adapter;
            FragmentActivity listener;
                
            @Override
            public void onAttach(Activity activity) {
                super.onAttach(activity);
                this.listener = (FragmentActivity) activity;
            }
               
            @Override
            public void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                ArrayList<Thing> things = new ArrayList<Thing>();
                adapter = new ThingsAdapter(getActivity(), things);
            }
        
            @Override
            public View onCreateView(LayoutInflater inflater, ViewGroup parent, Bundle savedInstanceState) {
                return inflater.inflate(R.layout.fragment_some, parent, false);
            }
        	
            @Override
            public void onViewCreated(View view, Bundle savedInstanceState) {
                ListView lv = (ListView) view.findViewById(R.id.lvSome);
                lv.setAdapter(adapter);
            }
                
            @Override
            public void onActivityCreated(Bundle savedInstanceState) {
                super.onActivityCreated(savedInstanceState);
            }
        }
        

#如何获取一个Fragment
1. 通过ID获取 
        
        public class MainActivity extends AppCompatActivity {
            @Override
            public void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                if (savedInstanceState == null) {
                  DemoFragment fragmentDemo = (DemoFragment) 
                      getSupportFragmentManager().findFragmentById(R.id.fragmentDemo);
                }
            }
        }
        
2. 通过Tag获取

        public class MainActivity extends AppCompatActivity {
            @Override
            public void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                if (savedInstanceState == null) {
                  // Let's first dynamically add a fragment into a frame container
                  getSupportFragmentManager().beginTransaction(). 
                      replace(R.id.flContainer, new DemoFragment(), "SOMETAG").
                      commit();
                  // Now later we can lookup the fragment by tag
                  DemoFragment fragmentDemo = (DemoFragment) 
                      getSupportFragmentManager().findFragmentByTag("SOMETAG");
                }
            }
        }
        
3. 从Pager获取

        adapterViewPager.getRegisteredFragment(0);
         
#Fragment 与 Activity 之间通信

1. Bundle

适用于Fragment初始化时, Activity向Fragment传递数据.

Fragment中定义:

        public class DemoFragment extends Fragment {
            
            public static final String ARGUMENT_SOME_INT = "someInt"
            public static final String ARGUMENT_SOME_TITLE = "someTitle"
            
            public static DemoFragment newInstance(int someInt, String someTitle) {
                DemoFragment fragmentDemo = new DemoFragment();
                Bundle args = new Bundle();
                args.putInt(ARGUMENT_SOME_INT, someInt);
                args.putString(ARGUMENT_SOME_TITLE, someTitle);
                fragmentDemo.setArguments(args);
                return fragmentDemo;
            }
        }
        
        public class DemoFragment extends Fragment {
           @Override
           public void onCreate(Bundle savedInstanceState) {
               super.onCreate(savedInstanceState);
               if (getArguments() != null) {
                    int SomeInt = getArguments().getInt(ARGUMENT_SOME_INT, 0);	
                    String someTitle = getArguments().getString(ARGUMENT_SOME_Title, "");
               }
               	
           }
        }

Activity中实例化:

        FragmentTransaction ft = getSupportFragmentManager().beginTransaction();
        DemoFragment fragmentDemo = DemoFragment.newInstance(5, "my title");
        ft.replace(R.id.your_placeholder, fragmentDemo);
        ft.commit();
        

2. Methods

适用于Activity向Fragment通信

Fragment中暴露出方式:
        
        public class DemoFragment extends Fragment {
          public void doSomething(String param) {
              // do something in fragment
          }
        }
        
Activity中直接调用:
        
        fragmentDemo.doSomething("some param");
        
3. Listener

适用于Fragment向Activity通信

        public class MyListFragment extends Fragment {
        
            private OnItemSelectedListener listener;
    
            @Override
            public void onAttach(Activity activity) {
                super.onAttach(activity);
                if (activity instanceof OnItemSelectedListener) {
                    listener = (OnItemSelectedListener) activity;
                } else {
                    throw new ClassCastException(activity.toString()
                            + " must implement MyListFragment.OnItemSelectedListener");
                }
            }
    
            public void onSomeClick(View v) {
                listener.onRssItemSelected("some link");
            }
    
            @Override
            public void onDetach() {
                super.onDetach();
                listener = null;
            }
    
    
            public interface OnItemSelectedListener {
                public void onRssItemSelected(String link);
            }
        }
    
#FragmentManager

FragmentManager 负责管理( adding, removing, hiding, showing)Fragment, 同时负责在某个Activity中寻找Fragment.
FragmentManager 主要包含以下方法:

    addOnBackStackChangedListener : 监听 fragment back stack 变化
    beginTransaction()            : 开始事务
    findFragmentById(int id)      : 通过 Id 找到 Fragment
    findFragmentByTag(String tag) :	通过 Tag 找到 Fragment
    popBackStack()                : 从回退栈找弹出fragment
    executePendingTransactions()  : 强制提交事务
    
#管理Fragment的Backstack

压栈 : addToBackstack

        FragmentTransaction fts = getSupportFragmentManager().beginTransaction();
        fts.replace(R.id.flContainer, new FirstFragment());	
        fts.addToBackStack("optional tag");
        fts.commit();
        
出栈 : popBackStack
        
        FragmentManager fragmentManager = getSupportFragmentManager();
        if (fragmentManager.getBackStackEntryCount() > 0) {
            fragmentManager.popBackStack();
        }

#Hide VS Replace        
调用 show 方法之前, 需要先确保该fragment已经 add 了.  

        protected void displayFragmentA() {
            FragmentTransaction ft = getSupportFragmentManager().beginTransaction();
            if (fragmentA.isAdded()) {
                ft.show(fragmentA);
            } else {
                ft.add(R.id.flContainer, fragmentA, "A");
            }
            if (fragmentB.isAdded()) { ft.hide(fragmentB); }
            if (fragmentC.isAdded()) { ft.hide(fragmentC); }          
            ft.commit();
        }
                
#Fragment 中嵌套 Fragment
子 Fragment 中应该在 父Fragment 的 onViewCreated 方法中被添加

        public class ParentFragment extends Fragment {
            @Override
            public View onCreateView(LayoutInflater inflater, ViewGroup container,
                Bundle savedInstanceState) {
                return inflater.inflate(R.layout.fragment_parent, container, false);
            }
        
            @Override
            public void onViewCreated(View view, Bundle savedInstanceState) {
                insertNestedFragment();
            }
          
           private void insertNestedFragment() {
               Fragment childFragment = new ChildFragment();
               FragmentTransaction transaction = getChildFragmentManager().beginTransaction();
               transaction.replace(R.id.child_fragment_container, childFragment).commit();
           }
        }

#DialogFragment 的一般写法及调用方式

1. 自定义 Dialog View

        public class EditNameDialog extends DialogFragment {
        
            private EditText mEditText;
        
            public EditNameDialog() {
                // Empty constructor is required for DialogFragment
                        // Make sure not to add arguments to the constructor
                        // Use `newInstance` instead as shown below
            }
            
            public static EditNameDialog newInstance(String title) {
                EditNameDialog frag = new EditNameDialog();
                Bundle args = new Bundle();
                args.putString("title", title);
                frag.setArguments(args);
                return frag;
            }
        
            @Override
            public View onCreateView(LayoutInflater inflater, ViewGroup container,
                    Bundle savedInstanceState) {
                return inflater.inflate(R.layout.fragment_edit_name, container);
            }
        
            @Override
            public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
                super.onViewCreated(view, savedInstanceState);
                // Get field from view
                mEditText = (EditText) view.findViewById(R.id.txt_your_name);
                // Fetch arguments from bundle and set title
                String title = getArguments().getString("title", "Enter Name");
                getDialog().setTitle(title);
                // Show soft keyboard automatically and request focus to field
                mEditText.requestFocus();
                getDialog().getWindow().setSoftInputMode(
                    WindowManager.LayoutParams.SOFT_INPUT_STATE_VISIBLE);
            }
        }
        
        public class DialogDemoActivity extends AppCompatActivity {
          @Override
          public void onCreate(Bundle savedInstanceState) {
              super.onCreate(savedInstanceState);
              setContentView(R.layout.main);
              showEditDialog();
          }
          
          private void showEditDialog() {
              FragmentManager fm = getSupportFragmentManager();
              EditNameDialog editNameDialog = EditNameDialog.newInstance("Some Title");
              editNameDialog.show(fm, "fragment_edit_name");
          }
        }
        
2. 构建标准 Dialog
    
        class MyAlertDialogFragment extends DialogFragment {
            public MyAlertDialogFragment() {
                  // Empty constructor required for DialogFragment
            }
            
            public static MyAlertDialogFragment newInstance(String title) {
                MyAlertDialogFragment frag = new MyAlertDialogFragment();
            	Bundle args = new Bundle();
            	args.putString("title", title);
            	frag.setArguments(args);
            	return frag;
            }
            
            @Override
            public Dialog onCreateDialog(Bundle savedInstanceState) {
                String title = getArguments().getString("title");
                AlertDialog.Builder alertDialogBuilder = new AlertDialog.Builder(getActivity());
                alertDialogBuilder.setTitle(title);
                alertDialogBuilder.setMessage("Are you sure?");
                alertDialogBuilder.setPositiveButton("OK",  new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        // on success
                    }
                });
                alertDialogBuilder.setNegativeButton("Cancel", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        dialog.dismiss();
                    }
                });
        
                return alertDialogBuilder.create();
            }
        }
        
        
        public class DialogDemoActivity extends AppCompatActivity {
          @Override
          public void onCreate(Bundle savedInstanceState) {
              super.onCreate(savedInstanceState);
              setContentView(R.layout.main);
              showAlertDialog();
          }
        
          private void showAlertDialog() {
              FragmentManager fm = getSupportFragmentManager();
              MyAlertDialogFragment alertDialog = MyAlertDialogFragment.newInstance("Some title");
              alertDialog.show(fm, "fragment_alert");
          }
        }
        
        
#移除 Dialog 的 TitleBar
        
        @Override
        public Dialog onCreateDialog(Bundle savedInstanceState) {
            Dialog dialog = super.onCreateDialog(savedInstanceState);
            // request a window without the title
            dialog.getWindow().requestFeature(Window.FEATURE_NO_TITLE);
            return dialog;
        }

#透明效果
        
        <style name="AppDialogTheme" parent="Theme.AppCompat.Light.Dialog">
            <item name="android:windowIsTranslucent">true</item>
            <item name="android:windowBackground">@android:color/transparent</item>
            <!-- ...other stuff here... -->
        </style>
        
第三方开源库: [https://github.com/afollestad/material-dialogs](https://github.com/afollestad/material-dialogs)




注意点:

1. startActivityForResult
 
    调用Fragment#startActivityForResult方法时，只有直接添加到activity里的fragment能收到onActivityResult回调，child fragment是收不到onActivityResult的。
    推荐使用LocalBroadcastManager.
       
2. 构造函数

    一定要提供默认构造函数。不能用构造函数传递参数！不要写带参数的构造函数。
    原因：Fragment会被重新销毁（Activity销毁的时候它里面的Fragment就被销毁了，可能因为内存不足，手机配置发生变化，横竖屏切换)。在重新创建的时候系统调用的是无参构造函数。
    
3. 关于getActivity()

    Fragment的方法getActivity()只在被关联到Activity之后才会得到结果，也就是在onAttach和onDetach两个生命周期之间会非空（此时isAdded返回值是true）。其他时候不应该使用Activity！如果要使用，那说明你设计得不好。
    使用所依附的Activity时应该判断getActivity是否为空或者isAdded是否为true。
    Fragment依附的Activity随时可能被destroy掉！很多时候是在不经意间。

4. Activity引用

    Fragment里不应该保留Activity引用！需要用到的时候通过getActivity()获取，因为那个引用不仅会导致内存泄露，而且在你用的时候，那个Activity可能已经不是正在显示的那个Activity了，这个Fragment也可能已经不是正在显示的那个Fragment了。

5. hide和show

    通过add添加Fragment会触发Fragment生命周期，hide和show不会触发生命周期。
    像微信那样，一个Activity里有四个Fragment，下面四个按钮，点击一个按钮显示其中一个Fragment。这种情况下，为了优化性能，你可以这样做：
    起初只add第一个Fragment，其余三个Fragment都不添加。点击某个标签的时候先看对应的Fragment是否已经添加，没有则new一个并add这个Fragment，隐藏其他所有Fragment；如果已经添加了就直接show这个Fragment，隐藏其他所有Fragment。

6. 数据保存
7. repalce VS hide/show

    replace: 每次切换的时候，Fragment都会重新实例化，重新加载一遍数据，这样非常消耗性能和用户的数据流量。
8. Fragment重叠
   
   其实是由Activity被回收后重启所导致的Fragment重复创建和重叠的问题。
   在Activity onCreate()中添加Fragment的时候一定不要忘了检查一下savedInstanceState. savedInstanceState 中没有才新建一个Fragment的实例,否则找出原有的实例.
   获取:

        if (savedInstanceState != null) {
            menuFragment = (MenuFragment) getSupportFragmentManager().getFragment(savedInstanceState, MenuFragment.LOG_TAG);
            productListFragment = (ProductListFragment) getSupportFragmentManager().getFragment(savedInstanceState, ProductListFragment.LOG_TAG);
            scanCodeFragment = (ScanCodeFragment) getSupportFragmentManager().getFragment(savedInstanceState, ScanCodeFragment.LOG_TAG);
        } else {
            menuFragment = new MenuFragment();
            productListFragment = new ProductListFragment();
            scanCodeFragment = new ScanCodeFragment();
        }
    保存:

        @Override
        protected void onSaveInstanceState(Bundle outState) {
            if (menuFragment != null) {
                getSupportFragmentManager().putFragment(outState, MenuFragment.LOG_TAG, menuFragment);
            }
            if (scanCodeFragment != null) {
                getSupportFragmentManager().putFragment(outState, ScanCodeFragment.LOG_TAG, scanCodeFragment);
            }
            if (productListFragment != null) {
                getSupportFragmentManager().putFragment(outState, ProductListFragment.LOG_TAG, productListFragment);
            }
            super.onSaveInstanceState(outState);
        }

