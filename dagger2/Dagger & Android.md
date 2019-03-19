与其他大多数依赖注入框架相比，Dagger 2的主要优点之一是其严格生成的实现（无反射）意味着它可以在Android应用程序中使用。但是，在Android应用程序中使用Dagger时仍有一些注意事项。

## Philosophy

虽然为Android编写的代码是Java源代码，但它在样式方面通常有很大不同。 通常，存在这样的差异以适应移动平台的独特性能考虑。

But many of the patterns commonly applied to code intended for Android are contrary to those applied to other Java code. Even much of the advice in Effective Java is considered inappropriate for Android.
但是，通常应用于Android代码的许多模式与应用于其他Java代码的模式相反。甚至有效Java中的大部分建议都被认为不适合Android。

In order to achieve the goals of both idiomatic and portable code, Dagger relies on ProGuard to post-process the compiled bytecode. This allows Dagger to emit source that looks and feels natural on both the server and Android, while using the different toolchains to produce bytecode that executes efficiently in both environements. Moreover, Dagger has an explicit goal to ensure that the Java source that it generates is consistently compatible with ProGuard optimizations.
为了实现惯用和可移植代码的目标，Dagger依靠ProGuard对已编译的字节码进行后处理。这允许Dagger在服务器和Android上发出外观和感觉自然的源，同时使用不同的工具链来生成在两个环境中有效执行的字节码。此外，Dagger有一个明确的目标，即确保它生成的Java源始终与ProGuard优化兼容。

当然，并非所有问题都能以这种方式解决，但它是提供Android特定兼容性的主要机制。

tl;dr

Dagger假设Android上的用户将使用ProGuard。

## 推荐的ProGuard设置

Watch this space for ProGuard settings that are relevant to applications using Dagger.
观看此空间以获取与使用Dagger的应用程序相关的ProGuard设置。

### dagger.android

使用Dagger编写Android应用程序的一个主要困难是，许多Android框架类都由操作系统本身实例化，如Activity和Fragment，但如果Dagger可以创建所有注入的对象，则效果最佳。 相反，您必须在生命周期方法中执行成员注入。这意味着许多类最终看起来像：

```
public class FrombulationActivity extends Activity {
  @Inject Frombulator frombulator;

  @Override
  public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // DO THIS FIRST. Otherwise frombulator might be null!
    ((SomeApplicationBaseType) getContext().getApplicationContext())
        .getApplicationComponent()
        .newActivityComponentBuilder()
        .activity(this)
        .build()
        .inject(this);
    // ... now you can write the exciting code
  }
}
```
这有一些问题：

- 复制粘贴代码使得以后很难重构。随着越来越多的开发人员复制粘贴该块，更少的人会知道它实际上做了什么。
- 更重要的是，它需要请求注入类型（FrombulationActivity）来了解其注入器。即使这是通过接口而不是具体类型完成的，它也打破了依赖注入的核心原则：类不应该知道它是如何注入的。

dagger.android中的类提供了一种简化此模式的方法。

### 注入Activity对象

- 在应用程序组件中安装AndroidInjectionModule，以确保这些基本类型所需的所有绑定都可用。

- 首先编写实现AndroidInjector <YourActivity>的@Subcomponent，并使用扩展AndroidInjector.Builder <YourActivity>的@Subcomponent.Builder：

```
@Subcomponent(modules = ...)
public interface YourActivitySubcomponent extends AndroidInjector<YourActivity> {
  @Subcomponent.Builder
  public abstract class Builder extends AndroidInjector.Builder<YourActivity> {}
}
```

- 定义子组件后，通过定义绑定子组件构建器的模块并将其添加到注入应用程序的组件，将其添加到组件层次结构中：

```
@Module(subcomponents = YourActivitySubcomponent.class)
abstract class YourActivityModule {
  @Binds
  @IntoMap
  @ClassKey(YourActivity.class)
  abstract AndroidInjector.Factory<?>
      bindYourActivityInjectorFactory(YourActivitySubcomponent.Builder builder);
}

@Component(modules = {..., YourActivityModule.class})
interface YourApplicationComponent {}
```

专业提示：如果您的子组件及其构建器没有其他方法或超类型，而不是步骤＃2中提到的方法或超类型，则可以使用@ContributesAndroidInjector为您生成它们。 而不是第2步和第3步，添加一个返回活动的抽象模块方法，使用@ContributesAndroidInjector对其进行注释，并指定要安装到子组件中的模块。 如果子组件需要范围，则还要将范围注释应用于该方法。

```
@ActivityScope
@ContributesAndroidInjector(modules = { /* modules to install into the subcomponent */ })
abstract YourActivity contributeYourActivityInjector();
```

- 接下来，使您的Application实现HasActivityInjector和从activityInjector()方法返回@Inject DispatchingAndroidInjector <Activity>：

```
public class YourApplication extends Application implements HasActivityInjector {
  @Inject DispatchingAndroidInjector<Activity> dispatchingActivityInjector;

  @Override
  public void onCreate() {
    super.onCreate();
    DaggerYourApplicationComponent.create()
        .inject(this);
  }

  @Override
  public AndroidInjector<Activity> activityInjector() {
    return dispatchingActivityInjector;
  }
}
```

- 最后，在Activity.onCreate()方法中，在调用super.onCreate()之前调用AndroidInjection.inject(this)：

```
public class YourActivity extends Activity {
  public void onCreate(Bundle savedInstanceState) {
    AndroidInjection.inject(this);
    super.onCreate(savedInstanceState);
  }
}
```

- 恭喜！

### How did that work?

AndroidInjection.inject() gets a DispatchingAndroidInjector<Activity> from the Application and passes your activity to inject(Activity). The DispatchingAndroidInjector looks up the AndroidInjector.Factory for your activity’s class (which is YourActivitySubcomponent.Builder), creates the AndroidInjector (which is YourActivitySubcomponent), and passes your activity to inject(YourActivity).

AndroidInjection.inject()从Application获取DispatchingAndroidInjector <Activity>并将您的活动传递给inject（Activity）。 DispatchingAndroidInjector为您的活动类（即YourActivitySubcomponent.Builder）查找AndroidInjector.Factory，创建AndroidInjector（即YourActivitySubcomponent），并将您的活动传递给inject（YourActivity）。

### Injecting Fragment objects

注入Fragment就像注入一个Activity一样简单。以相同的方式定义子组件，并将HasActivityInjector替换为HasFragmentInjector。

不像在Activity类型中那样注入onCreate()，而是将Fragments注入onAttach()。

与为Activitys定义的模块不同，您可以选择在何处安装Fragments模块。 您可以将Fragment组件作为另一个Fragment组件，Activity组件或Application组件的子组件 - 这一切都取决于Fragment所需的其他绑定。 在确定组件位置后，使相应的类型实现HasFragmentInjector。 例如，如果您的Fragment需要来自YourActivitySubcomponent的绑定，那么您的代码将如下所示：

```
public class YourActivity extends Activity implements HasFragmentInjector {
  @Inject DispatchingAndroidInjector<Fragment> fragmentInjector;

  @Override
  public void onCreate(Bundle savedInstanceState) {
    AndroidInjection.inject(this);
    super.onCreate(savedInstanceState);
    // ...
  }

  @Override
  public AndroidInjector<Fragment> fragmentInjector() {
    return fragmentInjector;
  }
}

public class YourFragment extends Fragment {
  @Inject SomeDependency someDep;

  @Override
  public void onAttach(Activity activity) {
    AndroidInjection.inject(this);
    super.onAttach(activity);
    // ...
  }
}

@Subcomponent(modules = ...)
public interface YourFragmentSubcomponent extends AndroidInjector<YourFragment> {
  @Subcomponent.Builder
  public abstract class Builder extends AndroidInjector.Builder<YourFragment> {}
}

@Module(subcomponents = YourFragmentSubcomponent.class)
abstract class YourFragmentModule {
  @Binds
  @IntoMap
  @ClassKey(YourFragment.class)
  abstract AndroidInjector.Factory<?>
      bindYourFragmentInjectorFactory(YourFragmentSubcomponent.Builder builder);
}

@Subcomponent(modules = { YourFragmentModule.class, ... }
public interface YourActivityOrYourApplicationComponent { ... }
```

### Base Framework Types

Because DispatchingAndroidInjector looks up the appropriate AndroidInjector.Factory by the class at runtime, a base class can implement HasActivityInjector/HasFragmentInjector/etc as well as call AndroidInjection.inject(). All each subclass needs to do is bind a corresponding @Subcomponent. Dagger provides a few base types that do this, such as DaggerActivity and DaggerFragment, if you don’t have a complicated class hierarchy. Dagger also provides a DaggerApplication for the same purpose — all you need to do is to extend it and override the applicationInjector() method to return the component that should inject the Application.

The following types are also included:

- DaggerService and DaggerIntentService
- DaggerBroadcastReceiver
- DaggerContentProvider

Note: DaggerBroadcastReceiver should only be used when the BroadcastReceiver is registered in the AndroidManifest.xml. When the BroadcastReceiver is created in your own code, prefer constructor injection instead.

###Support libraries

For users of the Android support library, parallel types exist in the dagger.android.support package.

> TODO(ronshapiro): we should begin to split this up by androidx packages

### How do I get it?

Add the following to your build.gradle:

```
dependencies {
  compile 'com.google.dagger:dagger-android:2.x'
  compile 'com.google.dagger:dagger-android-support:2.x' // if you use the support libraries
  annotationProcessor 'com.google.dagger:dagger-android-processor:2.x'
}
```

## When to inject

Constructor injection is preferred whenever possible because javac will ensure that no field is referenced before it has been set, which helps avoid NullPointerExceptions. When members injection is required (as discussed above), prefer to inject as early as possible. For this reason, DaggerActivity calls AndroidInjection.inject() immediately in onCreate(), before calling super.onCreate(), and DaggerFragment does the same in onAttach(), which also prevents inconsistencies if the Fragment is reattached.

It is crucial to call AndroidInjection.inject() before super.onCreate() in an Activity, since the call to super attaches Fragments from the previous activity instance during configuration change, which in turn injects the Fragments. In order for the Fragment injection to succeed, the Activity must already be injected. For users of ErrorProne, it is a compiler error to call AndroidInjection.inject() after super.onCreate().

## FAQ

### Scoping AndroidInjector.Factory

AndroidInjector.Factory is intended to be a stateless interface so that implementors don’t have to worry about managing state related to the object which will be injected. When DispatchingAndroidInjector requests a AndroidInjector.Factory, it does so through a Provider so that it doesn’t explicitly retain any instances of the factory. Because the AndroidInjector.Builder implementation that is generated by Dagger does retain an instance of the Activity/Fragment/etc that is being injected, it is a compile-time error to apply a scope to the methods which provide them. If you are positive that your AndroidInjector.Factory does not retain an instance to the injected object, you may suppress this error by applying @SuppressWarnings("dagger.android.ScopedInjectoryFactory") to your module method.