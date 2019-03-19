任何应用程序中最好的类是那些做的东西：BarcodeDecoder，KoopaPhysicsEngine和AudioStreamer。 这些类有依赖关系; 也许是BarcodeCameraFinder，DefaultPhysicsEngine和HttpStreamer。

相比之下，任何应用程序中最差的类都是占用空间而没有做太多的类：BarcodeDecoderFactory，CameraServiceLoader和MutableContextWrapper。 这些类是笨拙的胶带，将有趣的东西连接在一起。

Dagger是这些FactoryFactory类的替代品，它们实现了依赖注入设计模式，而没有编写样板文件的负担。 它允许您专注于有趣的类。声明依赖项，指定如何满足它们，并发送您的应用程序。

通过构建标准的javax.inject注释（JSR 330），每个类都很容易测试。 您不需要一堆样板来将RpcCreditCardService换成FakeCreditCardService。

依赖注入不仅仅用于测试。它还可以轻松创建可重复使用的可互换模块。您可以在所有应用中共享相同的AuthenticationModule。您可以在开发期间运行DevLoggingModule，在生产中运行ProdLoggingModule以在每种情况下获得正确的行为。

### 为什么Dagger2不同

依赖注入框架已存在多年，其中包含各种用于配置和注入的API。那么，为什么重新发明轮子呢？Dagger2是第一个用生成的代码实现完整堆栈的人。指导原则是生成代码，该代码模仿用户可能手写的代码，以确保依赖注入尽可能简单，可追踪和高效。 有关设计的更多背景，请观看+ Gregory Kick的演讲（幻灯片）。

## 使用Dagger

我们将通过建立咖啡机来展示依赖注入和Dagger。有关可以编译和运行的完整示例代码，请参阅Dagger的咖啡示例。

### 声明依赖关系

Dagger构造应用程序类的实例并满足它们的依赖关系。它使用javax.inject.Inject批注来标识它感兴趣的构造函数和字段。

使用@Inject注释Dagger应该用来创建类实例的构造函数。 当请求新实例时，Dagger将获取所需的参数值并调用此构造函数。

```
class Thermosiphon implements Pump {
  private final Heater heater;

  @Inject
  Thermosiphon(Heater heater) {
    this.heater = heater;
  }
  ...
}
```

Dagger可以直接注入字段。 在此示例中，它获取heater字段的Heater实例和pump字段的Pump实例。

```
class CoffeeMaker {
  @Inject Heater heater;
  @Inject Pump pump;

  ...
}
```

如果您的类具有@Inject-annotated字段但没有@Inject-annotated构造函数，则Dagger将在请求时注入这些字段，但不会创建新实例。 添加带有@Inject批注的无参数构造函数，以指示Dagger也可以创建实例。

Dagger也支持方法注入，但通常首选构造函数或字段注入。

缺少@Inject注释的类不能由Dagger构造。

### 满足依赖性

默认情况下，Dagger通过构造所请求类型的实例来满足每个依赖关系，如上所述。 当您请求CoffeeMaker时，它将通过调用新的CoffeeMaker()并设置其可注入字段来获取一个。

但@Inject无处不在：

- 无法构造接口。
- 无法注释第三方类。
- 必须配置可配置对象！

对于@Inject不足或笨拙的情况，请使用@Provide-annotated方法来满足依赖关系。方法的返回类型定义了它满足的依赖关系。

For example, provideHeater() is invoked whenever a Heater is required:

```
@Provides static Heater provideHeater() {
  return new ElectricHeater();
}
```
It’s possible for @Provides methods to have dependencies of their own. This one returns a Thermosiphon whenever a Pump is required:

```
@Provides static Pump providePump(Thermosiphon pump) {
  return pump;
}
```
All @Provides methods must belong to a module. These are just classes that have an @Module annotation.

```
@Module
class DripCoffeeModule {
  @Provides static Heater provideHeater() {
    return new ElectricHeater();
  }

  @Provides static Pump providePump(Thermosiphon pump) {
    return pump;
  }
}
```

By convention, @Provides methods are named with a provide prefix and module classes are named with a Module suffix.

### 构建图表

The @Inject and @Provides-annotated classes form a graph of objects, linked by their dependencies. Calling code like an application’s main method or an Android Application accesses that graph via a well-defined set of roots. In Dagger 2, that set is defined by an interface with methods that have no arguments and return the desired type. By applying the @Component annotation to such an interface and passing the module types to the modules parameter, Dagger 2 then fully generates an implementation of that contract.

```
@Component(modules = DripCoffeeModule.class)
interface CoffeeShop {
  CoffeeMaker maker();
}
```

The implementation has the same name as the interface prefixed with Dagger. Obtain an instance by invoking the builder() method on that implementation and use the returned builder to set dependencies and build() a new instance.

```
CoffeeShop coffeeShop = DaggerCoffeeShop.builder()
    .dripCoffeeModule(new DripCoffeeModule())
    .build();
```
Note: If your @Component is not a top-level type, the generated component’s name will include its enclosing types’ names, joined with an underscore. For example, this code:

```
class Foo {
  static class Bar {
    @Component
    interface BazComponent {}
  }
}
```
would generate a component named DaggerFoo_Bar_BazComponent.

Any module with an accessible default constructor can be elided as the builder will construct an instance automatically if none is set. And for any module whose @Provides methods are all static, the implementation doesn’t need an instance at all. If all dependencies can be constructed without the user creating a dependency instance, then the generated implementation will also have a create() method that can be used to get a new instance without having to deal with the builder.

```
CoffeeShop coffeeShop = DaggerCoffeeShop.create();
```
Now, our CoffeeApp can simply use the Dagger-generated implementation of CoffeeShop to get a fully-injected CoffeeMaker.

```
public class CoffeeApp {
  public static void main(String[] args) {
    CoffeeShop coffeeShop = DaggerCoffeeShop.create();
    coffeeShop.maker().brew();
  }
}
```
Now that the graph is constructed and the entry point is injected, we run our coffee maker app. Fun.

```
$ java -cp ... coffee.CoffeeApp
~ ~ ~ heating ~ ~ ~
=> => pumping => =>
 [_]P coffee! [_]P
```

### Bindings in the graph

The example above shows how to construct a component with some of the more typical bindings, but there are a variety of mechanisms for contributing bindings to the graph. The following are available as dependencies and may be used to generate a well-formed component:

- Those declared by @Provides methods within a @Module referenced directly by @Component.modules or transitively via @Module.includes
- Any type with an @Inject constructor that is unscoped or has a @Scope annotation that matches one of the component’s scopes
- The component provision methods of the component dependencies
- The component itself
- Unqualified builders for any included subcomponent
- Provider or Lazy wrappers for any of the above bindings
- A Provider of a Lazy of any of the above bindings (e.g., Provider<Lazy<CoffeeMaker>>)
- A MembersInjector for any type

## Singletons and Scoped Bindings

Annotate an @Provides method or injectable class with @Singleton. The graph will use a single instance of the value for all of its clients.

```
@Provides @Singleton static Heater provideHeater() {
  return new ElectricHeater();
}
```
The @Singleton annotation on an injectable class also serves as documentation. It reminds potential maintainers that this class may be shared by multiple threads.

```
@Singleton
class CoffeeMaker {
  ...
}
```
Since Dagger 2 associates scoped instances in the graph with instances of component implementations, the components themselves need to declare which scope they intend to represent. For example, it wouldn’t make any sense to have a @Singleton binding and a @RequestScoped binding in the same component because those scopes have different lifecycles and thus must live in components with different lifecycles. To declare that a component is associated with a given scope, simply apply the scope annotation to the component interface.

```
@Component(modules = DripCoffeeModule.class)
@Singleton
interface CoffeeShop {
  CoffeeMaker maker();
}
```
Components may have multiple scope annotations applied. This declares that they are all aliases to the same scope, and so that component may include scoped bindings with any of the scopes it declares.

## Reusable scope

Sometimes you want to limit the number of times an @Inject-constructed class is instantiated or a @Provides method is called, but you don’t need to guarantee that the exact same instance is used during the lifetime of any particular component or subcomponent. This can be useful in environments such as Android, where allocations can be expensive.

For these bindings, you can apply @Reusable scope. @Reusable-scoped bindings, unlike other scopes, are not associated with any single component; instead, each component that actually uses the binding will cache the returned or instantiated object.

That means that if you install a module with a @Reusable binding in a component, but only a subcomponent actually uses the binding, then only that subcomponent will cache the binding’s object. If two subcomponents that do not share an ancestor each use the binding, each of them will cache its own object. If a component’s ancestor has already cached the object, the subcomponent will reuse it.

There is no guarantee that the component will call the binding only once, so applying @Reusable to bindings that return mutable objects, or objects where it’s important to refer to the same instance, is dangerous. It’s safe to use @Reusable for immutable objects that you would leave unscoped if you didn’t care how many times they were allocated.

```
@Reusable // It doesn't matter how many scoopers we use, but don't waste them.
class CoffeeScooper {
  @Inject CoffeeScooper() {}
}

@Module
class CashRegisterModule {
  @Provides
  @Reusable // DON'T DO THIS! You do care which register you put your cash in.
            // Use a specific scope instead.
  static CashRegister badIdeaCashRegister() {
    return new CashRegister();
  }
}

@Reusable // DON'T DO THIS! You really do want a new filter each time, so this
          // should be unscoped.
class CoffeeFilter {
  @Inject CoffeeFilter() {}
}
```

### Lazy injections 懒惰注射

Sometimes you need an object to be instantiated lazily. For any binding T, you can create a Lazy<T> which defers instantiation until the first call to Lazy<T>’s get() method. If T is a singleton, then Lazy<T> will be the same instance for all injections within the ObjectGraph. Otherwise, each injection site will get its own Lazy<T> instance. Regardless, subsequent calls to any given instance of Lazy<T> will return the same underlying instance of T.

```
class GrindingCoffeeMaker {
  @Inject Lazy<Grinder> lazyGrinder;

  public void brew() {
    while (needsGrinding()) {
      // Grinder created once on first call to .get() and cached.
      lazyGrinder.get().grind();
    }
  }
}
```

### Provider injections 提供者注射

Sometimes you need multiple instances to be returned instead of just injecting a single value. While you have several options (Factories, Builders, etc.), one option is to inject a Provider<T> instead of just T. A Provider<T> invokes the binding logic for T each time .get() is called. If that binding logic is an @Inject constructor, a new instance will be created, but a @Provides method has no such guarantee.

```
class BigCoffeeMaker {
  @Inject Provider<Filter> filterProvider;

  public void brew(int numberOfPots) {
  ...
    for (int p = 0; p < numberOfPots; p++) {
      maker.addFilter(filterProvider.get()); //new filter every time.
      maker.addCoffee(...);
      maker.percolate();
      ...
    }
  }
}
```
Note: Injecting Provider<T> has the possibility of creating confusing code, and may be a design smell of mis-scoped or mis-structured objects in your graph. Often you will want to use a factory or a Lazy<T> or re-organize the lifetimes and structure of your code to be able to just inject a T. Injecting Provider<T> can, however, be a life saver in some cases. A common use is when you must use a legacy architecture that doesn’t line up with your object’s natural lifetimes (e.g. servlets are singletons by design, but only are valid in the context of request-specfic data).

### Qualifiers 限定符

Sometimes the type alone is insufficient to identify a dependency. For example, a sophisticated coffee maker app may want separate heaters for the water and the hot plate.

In this case, we add a qualifier annotation. This is any annotation that itself has a @Qualifier annotation. Here’s the declaration of @Named, a qualifier annotation included in javax.inject:

```
@Qualifier
@Documented
@Retention(RUNTIME)
public @interface Named {
  String value() default "";
}
```
You can create your own qualifier annotations, or just use @Named. Apply qualifiers by annotating the field or parameter of interest. The type and qualifier annotation will both be used to identify the dependency.

```
class ExpensiveCoffeeMaker {
  @Inject @Named("water") Heater waterHeater;
  @Inject @Named("hot plate") Heater hotPlateHeater;
  ...
}
```
Supply qualified values by annotating the corresponding @Provides method.

```
@Provides @Named("hot plate") static Heater provideHotPlateHeater() {
  return new ElectricHeater(70);
}

@Provides @Named("water") static Heater provideWaterHeater() {
  return new ElectricHeater(93);
}
```
Dependencies may not have multiple qualifier annotations.

### 可选绑定

If you want a binding to work even if some dependency is not bound in the component, you can add a @BindsOptionalOf method to a module:

```
@BindsOptionalOf abstract CoffeeCozy optionalCozy();
```
That means that @Inject constructors and members and @Provides methods can depend on an Optional<CoffeeCozy> object. If there is a binding for CoffeeCozy in the component, the Optional will be present; if there is no binding for CoffeeCozy, the Optional will be absent.

Specifically, you can inject any of the following:

- Optional< CoffeeCozy> (unless there is a @Nullable binding for CoffeeCozy; see below)
- Optional< Provider < CoffeeCozy >>
- Optional< Lazy < CoffeeCozy >>
- Optional< Provider < Lazy < CoffeeCozy >>>

(You could also inject a Provider or Lazy or Provider of Lazy of any of those, but that isn’t very useful.)

If there is a binding for CoffeeCozy, and that binding is @Nullable, then it is a compile-time error to inject Optional<CoffeeCozy>, because Optional cannot contain null. You can always inject the other forms, because Provider and Lazy can always return null from their get() methods.

An optional binding that is absent in one component can be present in a subcomponent if the subcomponent includes a binding for the underlying type.

You can use either Guava’s Optional or Java 8’s Optional.

### 绑定实例

通常，在构建组件时，您可以获得数据。 例如，假设您有一个使用命令行参数的应用程序; 您可能希望在组件中绑定这些args。

也许你的应用程序只需要一个参数来表示你想要注入的用户名@UserName String。 您可以将注释@BindsInstance的方法添加到组件构建器，以允许将该实例注入组件中。

```
@Component(modules = AppModule.class)
interface AppComponent {
  App app();

  @Component.Builder
  interface Builder {
    @BindsInstance Builder userName(@UserName String userName);
    AppComponent build();
  }
}
```
Your app then might look like

```
public static void main(String[] args) {
  if (args.length > 1) { exit(1); }
  App app = DaggerAppComponent
      .builder()
      .userName(args[0])
      .build()
      .app();
  app.run();
}
```
In the above example, injecting @UserName String in the component will use the instance provided to the Builder when calling this method. Before building the component, all @BindsInstance methods must be called, passing a non-null value (with the exception of @Nullable bindings below).

If the parameter to a @BindsInstance method is marked @Nullable, then the binding will be considered “nullable” in the same way as a @Provides method is nullable: injection sites must also mark it @Nullable, and null is an acceptable value for the binding. Moreover, users of the Builder may omit calling the method, and the component will treat the instance as null.

@BindsInstance methods should be preferred to writing a @Module with constructor arguments and immediately providing those values.

### 编译时验证

The Dagger annotation processor is strict and will cause a compiler error if any bindings are invalid or incomplete. For example, this module is installed in a component, which is missing a binding for Executor:

```
@Module
class DripCoffeeModule {
  @Provides static Heater provideHeater(Executor executor) {
    return new CpuHeater(executor);
  }
}
```
When compiling it, javac rejects the missing binding:

```
[ERROR] COMPILATION ERROR :
[ERROR] error: java.util.concurrent.Executor cannot be provided without an @Provides-annotated method.
```
Fix the problem by adding an @Provides-annotated method for Executor to any of the modules in the component. While @Inject, @Module and @Provides annotations are validated individually, all validation of the relationship between bindings happens at the @Component level. Dagger 1 relied strictly on @Module-level validation (which may or may not have reflected runtime behavior), but Dagger 2 elides such validation (and the accompanying configuration parameters on @Module) in favor of full graph validation.

### 编译时代码生成

Dagger’s annotation processor may also generate source files with names like CoffeeMaker_Factory.java or CoffeeMaker_MembersInjector.java. These files are Dagger implementation details. You shouldn’t need to use them directly, though they can be handy when step-debugging through an injection. The only generated types you should refer to in your code are the ones Prefixed with Dagger for your component.

### 在你的构建中使用Dagger

您需要在应用程序的运行时导入dagger-2.X.jar。为了激活代码生成，您需要在编译时在构建中包含dagger-compiler-2.X.jar。有关更多信息，请参阅 README 文件。

### License 许可

```
Copyright 2012 The Dagger Authors

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```