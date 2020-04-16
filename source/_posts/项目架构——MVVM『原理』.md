---
title: MVVM『原理』
tags: 
---

## Dagger2

### Factory

```java
/**
 * An {@linkplain Scope unscoped} {@link Provider}. While a {@link Provider} <i>may</i> apply
 * scoping semantics while providing an instance, a factory implementation is guaranteed to exercise
 * the binding logic ({@link Inject} constructors, {@link Provides} methods) upon each call to
 * {@link #get}.
 *
 * <p>Note that while subsequent calls to {@link #get} will create new instances for bindings such
 * as those created by {@link Inject} constructors, a new instance is not guaranteed by all
 * bindings. For example, {@link Provides} methods may be implemented in ways that return the same
 * instance for each call.
 */
public interface Factory<T> extends Provider<T> {
}
```

### Provider

```java
/**
 * Provides instances of {@code T}. Typically implemented by an injector. For
 * any type {@code T} that can be injected, you can also inject
 * {@code Provider<T>}. Compared to injecting {@code T} directly, injecting
 * {@code Provider<T>} enables:
 *
 * <ul>
 *   <li>retrieving multiple instances.</li>
 *   <li>lazy or optional retrieval of an instance.</li>
 *   <li>breaking circular dependencies.</li>
 *   <li>abstracting scope so you can look up an instance in a smaller scope
 *      from an instance in a containing scope.</li>
 * </ul>
 *
 * <p>For example:
 *
 * <pre>
 *   class Car {
 *     &#064;Inject Car(Provider&lt;Seat> seatProvider) {
 *       Seat driver = seatProvider.get();
 *       Seat passenger = seatProvider.get();
 *       ...
 *     }
 *   }</pre>
 */
public interface Provider<T> {

    /**
     * Provides a fully-constructed and injected instance of {@code T}.
     *
     * @throws RuntimeException if the injector encounters an error while
     *  providing an instance. For example, if an injectable member on
     *  {@code T} throws an exception, the injector may wrap the exception
     *  and throw it to the caller of {@code get()}. Callers should not try
     *  to handle such exceptions as the behavior may vary across injector
     *  implementations and even different configurations of the same injector.
     */
    T get();
}
```

<!-- more -->

### Component

```java
package dagger;

import static java.lang.annotation.ElementType.TYPE;
import static java.lang.annotation.RetentionPolicy.RUNTIME;

import java.lang.annotation.Documented;
import java.lang.annotation.Retention;
import java.lang.annotation.Target;
import javax.inject.Inject;
import javax.inject.Provider;
import javax.inject.Qualifier;
import javax.inject.Scope;
import javax.inject.Singleton;

/**
 * Annotates an interface or abstract class for which a fully-formed, dependency-injected
 * implementation is to be generated from a set of {@linkplain #modules}. The generated class will
 * have the name of the type annotated with {@code @Component} prepended with {@code Dagger}. For
 * example, {@code @Component interface MyComponent {...}} will produce an implementation named
 * {@code DaggerMyComponent}.
 *
 * <a name="component-methods"></a>
 * <h2>Component methods</h2>
 *
 * <p>Every type annotated with {@code @Component} must contain at least one abstract component
 * method. Component methods may have any name, but must have signatures that conform to either
 * {@linkplain Provider provision} or {@linkplain MembersInjector members-injection} contracts.
 *
 * <a name="provision-methods"></a>
 * <h3>Provision methods</h3>
 *
 * <p>Provision methods have no parameters and return an {@link Inject injected} or {@link Provides
 * provided} type. Each method may have a {@link Qualifier} annotation as well. The following are
 * all valid provision method declarations:
 *
 * <pre><code>
 *   SomeType getSomeType();
 *   {@literal Set<SomeType>} getSomeTypes();
 *   {@literal @PortNumber} int getPortNumber();
 * </code></pre>
 *
 * <p>Provision methods, like typical {@link Inject injection} sites, may use {@link Provider} or
 * {@link Lazy} to more explicitly control provision requests. A {@link Provider} allows the user of
 * the component to request provision any number of times by calling {@link Provider#get}. A {@link
 * Lazy} will only ever request a single provision, but will defer it until the first call to {@link
 * Lazy#get}. The following provision methods all request provision of the same type, but each
 * implies different semantics:
 *
 * <pre><code>
 *   SomeType getSomeType();
 *   {@literal Provider<SomeType>} getSomeTypeProvider();
 *   {@literal Lazy<SomeType>} getLazySomeType();
 * </code></pre>
 *
 * <a name="members-injection-methods"></a>
 * <h3>Members-injection methods</h3>
 *
 * <p>Members-injection methods have a single parameter and inject dependencies into each of the
 * {@link Inject}-annotated fields and methods of the passed instance. A members-injection method
 * may be void or return its single parameter as a convenience for chaining. The following are all
 * valid members-injection method declarations:
 *
 * <pre><code>
 *   void injectSomeType(SomeType someType);
 *   SomeType injectAndReturnSomeType(SomeType someType);
 * </code></pre>
 *
 * <p>A method with no parameters that returns a {@link MembersInjector} is equivalent to a members
 * injection method. Calling {@link MembersInjector#injectMembers} on the returned object will
 * perform the same work as a members injection method. For example:
 *
 * <pre><code>
 *   {@literal MembersInjector<SomeType>} getSomeTypeMembersInjector();
 * </code></pre>
 *
 * <h4>A note about covariance</h4>
 *
 * <p>While a members-injection method for a type will accept instances of its subtypes, only {@link
 * Inject}-annotated members of the parameter type and its supertypes will be injected; members of
 * subtypes will not. For example, given the following types, only {@code a} and {@code b} will be
 * injected into an instance of {@code Child} when it is passed to the members-injection method
 * {@code injectSelf(Self instance)}:
 *
 * <pre><code>
 *   class Parent {
 *     {@literal @}Inject A a;
 *   }
 *
 *   class Self extends Parent {
 *     {@literal @}Inject B b;
 *   }
 *
 *   class Child extends Self {
 *     {@literal @}Inject C c;
 *   }
 * </code></pre>
 *
 * <a name="instantiation"></a>
 * <h2>Instantiation</h2>
 *
 * <p>Component implementations are primarily instantiated via a generated <a
 * href="http://en.wikipedia.org/wiki/Builder_pattern">builder</a> or <a
 * href="https://en.wikipedia.org/wiki/Factory_(object-oriented_programming)">factory</a>.
 *
 * <p>If a nested {@link Builder @Component.Builder} or {@link Factory @Component.Factory} type
 * exists in the component, Dagger will generate an implementation of that type. If neither exists,
 * Dagger will generate a builder type that has a method to set each of the {@linkplain #modules}
 * and component {@linkplain #dependencies} named with the <a
 * href="http://en.wikipedia.org/wiki/CamelCase">lower camel case</a> version of the module or
 * dependency type.
 *
 * <p>In either case, the Dagger-generated component type will have a static method, named either
 * {@code builder()} or {@code factory()}, that returns a builder or factory instance.
 *
 * <p>Example of using a builder:
 *
 * <pre>{@code
 *   public static void main(String[] args) {
 *     OtherComponent otherComponent = ...;
 *     MyComponent component = DaggerMyComponent.builder()
 *         // required because component dependencies must be set
 *         .otherComponent(otherComponent)
 *         // required because FlagsModule has constructor parameters
 *         .flagsModule(new FlagsModule(args))
 *         // may be elided because a no-args constructor is visible
 *         .myApplicationModule(new MyApplicationModule())
 *         .build();
 *   }
 * }</pre>
 *
 * <p>Example of using a factory:
 *
 * <pre>{@code
 * public static void main(String[] args) {
 *     OtherComponent otherComponent = ...;
 *     MyComponent component = DaggerMyComponent.factory()
 *         .create(otherComponent, new FlagsModule(args), new MyApplicationModule());
 *     // Note that all parameters to a factory method are required, even if one is for a module
 *     // that Dagger could instantiate. The only case where null is legal is for a
 *     // @BindsInstance @Nullable parameter.
 *   }
 * }</pre>
 *
 * <p>In the case that a component has no component dependencies and only no-arg modules, the
 * generated component will also have a factory method {@code create()}. {@code
 * SomeComponent.create()} and {@code SomeComponent.builder().build()} are both valid and
 * equivalent.
 *
 * <a name="scope"></a>
 * <h2>Scope</h2>
 *
 * <p>Each Dagger component can be associated with a scope by annotating it with the {@linkplain
 * Scope scope annotation}. The component implementation ensures that there is only one provision of
 * each scoped binding per instance of the component. If the component declares a scope, it may only
 * contain unscoped bindings or bindings of that scope anywhere in the graph. For example:
 *
 * <pre><code>
 *   {@literal @}Singleton {@literal @}Component
 *   interface MyApplicationComponent {
 *     // this component can only inject types using unscoped or {@literal @}Singleton bindings
 *   }
 * </code></pre>
 *
 * <p>In order to get the proper behavior associated with a scope annotation, it is the caller's
 * responsibility to instantiate new component instances when appropriate. A {@link Singleton}
 * component, for instance, should only be instantiated once per application, while a {@code
 * RequestScoped} component should be instantiated once per request. Because components are
 * self-contained implementations, exiting a scope is as simple as dropping all references to the
 * component instance.
 *
 * <a name="component-relationships"></a>
 * <h2>Component relationships</h2>
 *
 * <p>While there is much utility in isolated components with purely unscoped bindings, many
 * applications will call for multiple components with multiple scopes to interact. Dagger provides
 * two mechanisms for relating components.
 *
 * <a name="subcomponents"></a>
 * <h3>Subcomponents</h3>
 *
 * <p>The simplest way to relate two components is by declaring a {@link Subcomponent}. A
 * subcomponent behaves exactly like a component, but has its implementation generated within a
 * parent component or subcomponent. That relationship allows the subcomponent implementation to
 * inherit the <em>entire</em> binding graph from its parent when it is declared. For that reason, a
 * subcomponent isn't evaluated for completeness until it is associated with a parent.
 *
 * <p>Subcomponents are declared by listing the class in the {@link Module#subcomponents()}
 * attribute of one of the parent component's modules. This binds the {@link Subcomponent.Builder}
 * or {@link Subcomponent.Factory} for that subcomponent within the parent component.
 *
 * <p>Subcomponents may also be declared via a factory method on a parent component or subcomponent.
 * The method may have any name, but must return the subcomponent. The factory method's parameters
 * may be any number of the subcomponent's modules, but must at least include those without visible
 * no-arg constructors. The following is an example of a factory method that creates a
 * request-scoped subcomponent from a singleton-scoped parent:
 *
 * <pre><code>
 *   {@literal @}Singleton {@literal @}Component
 *   interface ApplicationComponent {
 *     // component methods...
 *
 *     RequestComponent newRequestComponent(RequestModule requestModule);
 *   }
 * </code></pre>
 *
 * <a name="component-dependencies"></a>
 * <h3>Component dependencies</h3>
 *
 * <p>While subcomponents are the simplest way to compose subgraphs of bindings, subcomponents are
 * tightly coupled with the parents; they may use any binding defined by their ancestor component
 * and subcomponents. As an alternative, components can use bindings only from another <em>component
 * interface</em> by declaring a {@linkplain #dependencies component dependency}. When a type is
 * used as a component dependency, each <a href="#provision-methods">provision method</a> on the
 * dependency is bound as a provider. Note that <em>only</em> the bindings exposed as provision
 * methods are available through component dependencies.
 *
 * @since 2.0
 */
@Retention(RUNTIME) // Allows runtimes to have specialized behavior interoperating with Dagger.
@Target(TYPE)
@Documented
public @interface Component {
  /**
   * A list of classes annotated with {@link Module} whose bindings are used to generate the
   * component implementation. Note that through the use of {@link Module#includes} the full set of
   * modules used to implement the component may include more modules that just those listed here.
   */
  Class<?>[] modules() default {};

  /**
   * A list of types that are to be used as <a href="#component-dependencies">component
   * dependencies</a>.
   */
  Class<?>[] dependencies() default {};

  /**
   * A builder for a component.
   *
   * <p>A builder is a type with setter methods for the {@linkplain Component#modules modules},
   * {@linkplain Component#dependencies dependencies} and {@linkplain BindsInstance bound instances}
   * required by the component and a single no-argument build method that creates a new component
   * instance.
   *
   * <p>Components may have a single nested {@code static abstract class} or {@code interface}
   * annotated with {@code @Component.Builder}. If they do, then Dagger will generate a builder
   * class that implements that type. Note that a component with a {@code @Component.Builder} may
   * not also have a {@code @Component.Factory}.
   *
   * <p>Builder types must follow some rules:
   *
   * <ul>
   *   <li>There <i>must</i> be exactly one abstract no-argument method that returns the component
   *       type or one of its supertypes, called the "build method".
   *   <li>There <i>may</i> be other other abstract methods, called "setter methods".
   *   <li>Setter methods <i>must</i> take a single argument and return {@code void}, the builder
   *       type or a supertype of the builder type.
   *   <li>There <i>must</i> be a setter method for each {@linkplain Component#dependencies
   *       component dependency}.
   *   <li>There <i>must</i> be a setter method for each non-{@code abstract} {@linkplain
   *       Component#modules module} that has non-{@code static} binding methods, unless Dagger can
   *       instantiate that module with a visible no-argument constructor.
   *   <li>There <i>may</i> be setter methods for modules that Dagger can instantiate or does not
   *       need to instantiate.
   *   <li>There <i>may</i> be setter methods annotated with {@code @BindsInstance}. These methods
   *       bind the instance passed to them within the component. See {@link
   *       BindsInstance @BindsInstance} for more information.
   *   <li>There <i>may</i> be non-{@code abstract} methods, but they are ignored as far as
   *       validation and builder generation are concerned.
   * </ul>
   *
   * For example, this could be a valid {@code Component} with a {@code Builder}:
   *
   * <pre><code>
   * {@literal @}Component(modules = {BackendModule.class, FrontendModule.class})
   * interface MyComponent {
   *   MyWidget myWidget();
   *
   *   {@literal @}Component.Builder
   *   interface Builder {
   *     Builder backendModule(BackendModule bm);
   *     Builder frontendModule(FrontendModule fm);
   *     {@literal @}BindsInstance
   *     Builder foo(Foo foo);
   *     MyComponent build();
   *   }
   * }</code></pre>
   */
  @Retention(RUNTIME) // Allows runtimes to have specialized behavior interoperating with Dagger.
  @Target(TYPE)
  @Documented
  @interface Builder {}

  /**
   * A factory for a component.
   *
   * <p>A factory is a type with a single method that returns a new component instance each time it
   * is called. The parameters of that method allow the caller to provide the {@linkplain
   * Component#modules modules}, {@linkplain Component#dependencies dependencies} and {@linkplain
   * BindsInstance bound instances} required by the component.
   *
   * <p>Components may have a single nested {@code static abstract class} or {@code interface}
   * annotated with {@code @Component.Factory}. If they do, then Dagger will generate a factory
   * class that will implement that type. Note that a component with a {@code @Component.Factory}
   * may not also have a {@code @Component.Builder}.
   *
   * <p>Factory types must follow some rules:
   *
   * <ul>
   *   <li>There <i>must</i> be exactly one abstract method, which must return the component type or
   *       one of its supertypes.
   *   <li>The method <i>must</i> have a parameter for each {@linkplain Component#dependencies
   *       component dependency}.
   *   <li>The method <i>must</i> have a parameter for each non-{@code abstract} {@linkplain
   *       Component#modules module} that has non-{@code static} binding methods, unless Dagger can
   *       instantiate that module with a visible no-argument constructor.
   *   <li>The method <i>may</i> have parameters for modules that Dagger can instantiate or does not
   *       need to instantiate.
   *   <li>The method <i>may</i> have parameters annotated with {@code @BindsInstance}. These
   *       parameters bind the instance passed for that parameter within the component. See {@link
   *       BindsInstance @BindsInstance} for more information.
   *   <li>There <i>may</i> be non-{@code abstract} methods, but they are ignored as far as
   *       validation and factory generation are concerned.
   * </ul>
   *
   * For example, this could be a valid {@code Component} with a {@code Factory}:
   *
   * <pre><code>
   * {@literal @}Component(modules = {BackendModule.class, FrontendModule.class})
   * interface MyComponent {
   *   MyWidget myWidget();
   *
   *   {@literal @}Component.Factory
   *   interface Factory {
   *     MyComponent newMyComponent(
   *         BackendModule bm, FrontendModule fm, {@literal @}BindsInstance Foo foo);
   *   }
   * }</code></pre>
   *
   * <p>For a root component, if a {@code @Component.Factory} is defined, the generated component
   * type will have a {@code static} method named {@code factory()} that returns an instance of that
   * factory.
   *
   * @since 2.22
   */
  @Retention(RUNTIME) // Allows runtimes to have specialized behavior interoperating with Dagger.
  @Target(TYPE)
  @Documented
  @interface Factory {}
}
```

### DoubleCheck

```java
import static dagger.internal.Preconditions.checkNotNull;

import dagger.Lazy;
import javax.inject.Provider;

/**
 * A {@link Lazy} and {@link Provider} implementation that memoizes the value returned from a
 * delegate using the double-check idiom described in Item 71 of <i>Effective Java 2</i>.
 */
public final class DoubleCheck<T> implements Provider<T>, Lazy<T> {
  private static final Object UNINITIALIZED = new Object();

  private volatile Provider<T> provider;
  private volatile Object instance = UNINITIALIZED;

  private DoubleCheck(Provider<T> provider) {
    assert provider != null;
    this.provider = provider;
  }

  @SuppressWarnings("unchecked") // cast only happens when result comes from the provider
  @Override
  public T get() {
    Object result = instance;
    if (result == UNINITIALIZED) {
      synchronized (this) {
        result = instance;
        if (result == UNINITIALIZED) {
          result = provider.get();
          instance = reentrantCheck(instance, result);
          /* Null out the reference to the provider. We are never going to need it again, so we
           * can make it eligible for GC. */
          provider = null;
        }
      }
    }
    return (T) result;
  }

  /**
   * Checks to see if creating the new instance has resulted in a recursive call. If it has, and the
   * new instance is the same as the current instance, return the instance. However, if the new
   * instance differs from the current instance, an {@link IllegalStateException} is thrown.
   */
  public static Object reentrantCheck(Object currentInstance, Object newInstance) {
    boolean isReentrant = !(currentInstance == UNINITIALIZED
        // This check is needed for fastInit's implementation, which uses MemoizedSentinel types.
        || currentInstance instanceof MemoizedSentinel);

    if (isReentrant && currentInstance != newInstance) {
      throw new IllegalStateException("Scoped provider was invoked recursively returning "
          + "different results: " + currentInstance + " & " + newInstance + ". This is likely "
          + "due to a circular dependency.");
    }
    return newInstance;
  }

  /** Returns a {@link Provider} that caches the value from the given delegate provider. */
  // This method is declared this way instead of "<T> Provider<T> provider(Provider<T> delegate)"
  // to work around an Eclipse type inference bug: https://github.com/google/dagger/issues/949.
  public static <P extends Provider<T>, T> Provider<T> provider(P delegate) {
    checkNotNull(delegate);
    if (delegate instanceof DoubleCheck) {
      /* This should be a rare case, but if we have a scoped @Binds that delegates to a scoped
       * binding, we shouldn't cache the value again. */
      return delegate;
    }
    return new DoubleCheck<T>(delegate);
  }

  /** Returns a {@link Lazy} that caches the value from the given provider. */
  // This method is declared this way instead of "<T> Lazy<T> lazy(Provider<T> delegate)"
  // to work around an Eclipse type inference bug: https://github.com/google/dagger/issues/949.
  public static <P extends Provider<T>, T> Lazy<T> lazy(P provider) {
    if (provider instanceof Lazy) {
      @SuppressWarnings("unchecked")
      final Lazy<T> lazy = (Lazy<T>) provider;
      // Avoids memoizing a value that is already memoized.
      // NOTE: There is a pathological case where Provider<P> may implement Lazy<L>, but P and L
      // are different types using covariant return on get(). Right now this is used with
      // DoubleCheck<T> exclusively, which is implemented such that P and L are always
      // the same, so it will be fine for that case.
      return lazy;
    }
    return new DoubleCheck<T>(checkNotNull(provider));
  }
}

```

### @Scope

```java
package javax.inject;

import java.lang.annotation.Target;
import java.lang.annotation.Retention;
import java.lang.annotation.Documented;
import static java.lang.annotation.RetentionPolicy.RUNTIME;
import static java.lang.annotation.ElementType.ANNOTATION_TYPE;

/**
 * 用于标识范围(scope)的注解。一个scope注解被用在那些包含可注入(injectable)的构造函数的类上，用来控制该类的实例复用策略。默认情况下没有scope注解的类，每次回创建一个实例。如果使用了scope注解，注入器(injector)可以保留实例来在下一次进行注入时进行复用。如果有多个线程需要访问一个scope实例，那么此时需要考虑线程安全问题。Identifies scope annotations. A scope annotation applies to a class
 * containing an injectable constructor and governs how the injector reuses
 * instances of the type. By default, if no scope annotation is present, the
 * injector creates an instance (by injecting the type's constructor), uses
 * the instance for one injection, and then forgets it. If a scope annotation
 * is present, the injector may retain the instance for possible reuse in a
 * later injection. If multiple threads can access a scoped instance, its
 * implementation should be thread safe. The implementation of the scope
 * itself is left up to the injector.
 *
 * <p>In the following example, the scope annotation {@code @Singleton} ensures
 * that we only have one Log instance:
 *
 * <pre>
 *   &#064;Singleton
 *   class Log {
 *     void log(String message) { ... }
 *   }</pre>
 * 
 如果一个注入器上有多个scope注解，或者有一个它不支持的scope，则会产生一个错误。
 * <p>The injector generates an error if it encounters more than one scope
 * annotation on the same class or a scope annotation it doesn't support.
 *
 * <p>A scope annotation:
 * <ul>
 *   <li>is annotated with {@code @Scope}, {@code @Retention(RUNTIME)},
 *      and typically {@code @Documented}.</li>
 *   <li>should not have attributes.</li>
 *   <li>is typically not {@code @Inherited}, so scoping is orthogonal to
 *      implementation inheritance.</li>
 *   <li>may have restricted usage if annotated with {@code @Target}. While
 *      this specification covers applying scopes to classes only, some 
 *      injector configurations might use scope annotations
 *      in other places (on factory method results for example).</li>
 * </ul>
 *
 * <p>For example:
 *
 * <pre>
 *   &#064;java.lang.annotation.Documented
 *   &#064;java.lang.annotation.Retention(RUNTIME)
 *   &#064;javax.inject.Scope
 *   public @interface RequestScoped {}</pre>
 * 
 * <p>Annotating scope annotations with {@code @Scope} helps the injector
 * detect the case where a programmer used the scope annotation on a class but
 * forgot to configure the scope in the injector. A conservative injector
 * would generate an error rather than not apply a scope.
 *
 * @see javax.inject.Singleton @Singleton
 */
@Target(ANNOTATION_TYPE)
@Retention(RUNTIME)
@Documented
public @interface Scope {}
```

### ScopedProvider

```java
package dagger.internal;

import javax.inject.Provider;

/**
 * A {@link Provider} implementation that memoizes the result of a {@link Factory} instance.
 *
 * @author Gregory Kick
 * @since 2.0
 */
public final class ScopedProvider<T> implements Provider<T> {
  private static final Object UNINITIALIZED = new Object();

  private final Factory<T> factory;
  private volatile Object instance = UNINITIALIZED;

  private ScopedProvider(Factory<T> factory) {
    assert factory != null;
    this.factory = factory;
  }

  @SuppressWarnings("unchecked") // cast only happens when result comes from the factory
  @Override
  public T get() {
    // double-check idiom from EJ2: Item 71
    Object result = instance;
    if (result == UNINITIALIZED) {
      synchronized (this) {
        result = instance;
        if (result == UNINITIALIZED) {
          instance = result = factory.get();
        }
      }
    }
    return (T) result;
  }

  /** Returns a new scoped provider for the given factory. */
  public static <T> Provider<T> create(Factory<T> factory) {
    if (factory == null) {
      throw new NullPointerException();
    }
    return new ScopedProvider<T>(factory);
  }
}
```



## ViewModel

### ViewModelStore

```java
/**
 * Class to store {@code ViewModels}.
 * <p>
 * An instance of {@code ViewModelStore} must be retained through configuration changes:
 * if an owner of this {@code ViewModelStore} is destroyed and recreated due to configuration
 * changes, new instance of an owner should still have the same old instance of
 * {@code ViewModelStore}.
 * <p>
 * If an owner of this {@code ViewModelStore} is destroyed and is not going to be recreated,
 * then it should call {@link #clear()} on this {@code ViewModelStore}, so {@code ViewModels} would
 * be notified that they are no longer used.
 * <p>
 * {@link android.arch.lifecycle.ViewModelStores} provides a {@code ViewModelStore} for
 * activities and fragments.
 */
public class ViewModelStore {

    private final HashMap<String, ViewModel> mMap = new HashMap<>();

    final void put(String key, ViewModel viewModel) {
        ViewModel oldViewModel = mMap.put(key, viewModel);
        if (oldViewModel != null) {
            oldViewModel.onCleared();
        }
    }

    final ViewModel get(String key) {
        return mMap.get(key);
    }

    /**
     *  Clears internal storage and notifies ViewModels that they are no longer used.
     */
    public final void clear() {
        for (ViewModel vm : mMap.values()) {
            vm.onCleared();
        }
        mMap.clear();
    }
}
```

### ViewModelStores

```java
/**
 * Factory methods for {@link ViewModelStore} class.
 */
@SuppressWarnings("WeakerAccess")
public class ViewModelStores {

    private ViewModelStores() {
    }

    /**
     * Returns the {@link ViewModelStore} of the given activity.
     *
     * @param activity an activity whose {@code ViewModelStore} is requested
     * @return a {@code ViewModelStore}
     */
    @NonNull
    @MainThread
    public static ViewModelStore of(@NonNull FragmentActivity activity) {
        if (activity instanceof ViewModelStoreOwner) {
            return ((ViewModelStoreOwner) activity).getViewModelStore();
        }
        return holderFragmentFor(activity).getViewModelStore();
    }

    /**
     * Returns the {@link ViewModelStore} of the given fragment.
     *
     * @param fragment a fragment whose {@code ViewModelStore} is requested
     * @return a {@code ViewModelStore}
     */
    @NonNull
    @MainThread
    public static ViewModelStore of(@NonNull Fragment fragment) {
        if (fragment instanceof ViewModelStoreOwner) {
            return ((ViewModelStoreOwner) fragment).getViewModelStore();
        }
        return holderFragmentFor(fragment).getViewModelStore();
    }
}
```



### ViewModelProviders

```java
/**
 * Utilities methods for {@link ViewModelStore} class.
 */
public class ViewModelProviders {

    /**
     * @deprecated This class should not be directly instantiated
     */
    @Deprecated
    public ViewModelProviders() {
    }

    private static Application checkApplication(Activity activity) {
        Application application = activity.getApplication();
        if (application == null) {
            throw new IllegalStateException("Your activity/fragment is not yet attached to "
                    + "Application. You can't request ViewModel before onCreate call.");
        }
        return application;
    }

    private static Activity checkActivity(Fragment fragment) {
        Activity activity = fragment.getActivity();
        if (activity == null) {
            throw new IllegalStateException("Can't create ViewModelProvider for detached fragment");
        }
        return activity;
    }

    /**
     * Creates a {@link ViewModelProvider}, which retains ViewModels while a scope of given
     * {@code fragment} is alive. More detailed explanation is in {@link ViewModel}.
     * <p>
     * It uses {@link ViewModelProvider.AndroidViewModelFactory} to instantiate new ViewModels.
     *
     * @param fragment a fragment, in whose scope ViewModels should be retained
     * @return a ViewModelProvider instance
     */
    @NonNull
    @MainThread
    public static ViewModelProvider of(@NonNull Fragment fragment) {
        return of(fragment, null);
    }

    /**
     * Creates a {@link ViewModelProvider}, which retains ViewModels while a scope of given Activity
     * is alive. More detailed explanation is in {@link ViewModel}.
     * <p>
     * It uses {@link ViewModelProvider.AndroidViewModelFactory} to instantiate new ViewModels.
     *
     * @param activity an activity, in whose scope ViewModels should be retained
     * @return a ViewModelProvider instance
     */
    @NonNull
    @MainThread
    public static ViewModelProvider of(@NonNull FragmentActivity activity) {
        return of(activity, null);
    }

    /**
     * Creates a {@link ViewModelProvider}, which retains ViewModels while a scope of given
     * {@code fragment} is alive. More detailed explanation is in {@link ViewModel}.
     * <p>
     * It uses the given {@link Factory} to instantiate new ViewModels.
     *
     * @param fragment a fragment, in whose scope ViewModels should be retained
     * @param factory  a {@code Factory} to instantiate new ViewModels
     * @return a ViewModelProvider instance
     */
    @NonNull
    @MainThread
    public static ViewModelProvider of(@NonNull Fragment fragment, @Nullable Factory factory) {
        Application application = checkApplication(checkActivity(fragment));
        if (factory == null) {
            factory = ViewModelProvider.AndroidViewModelFactory.getInstance(application);
        }
        return new ViewModelProvider(ViewModelStores.of(fragment), factory);
    }

    /**
     * Creates a {@link ViewModelProvider}, which retains ViewModels while a scope of given Activity
     * is alive. More detailed explanation is in {@link ViewModel}.
     * <p>
     * It uses the given {@link Factory} to instantiate new ViewModels.
     *
     * @param activity an activity, in whose scope ViewModels should be retained
     * @param factory  a {@code Factory} to instantiate new ViewModels
     * @return a ViewModelProvider instance
     */
    @NonNull
    @MainThread
    public static ViewModelProvider of(@NonNull FragmentActivity activity,
            @Nullable Factory factory) {
        Application application = checkApplication(activity);
        if (factory == null) {
            factory = ViewModelProvider.AndroidViewModelFactory.getInstance(application);
        }
        return new ViewModelProvider(ViewModelStores.of(activity), factory);
    }

    /**
     * {@link Factory} which may create {@link AndroidViewModel} and
     * {@link ViewModel}, which have an empty constructor.
     *
     * @deprecated Use {@link ViewModelProvider.AndroidViewModelFactory}
     */
    @SuppressWarnings("WeakerAccess")
    @Deprecated
    public static class DefaultFactory extends ViewModelProvider.AndroidViewModelFactory {
        /**
         * Creates a {@code AndroidViewModelFactory}
         *
         * @param application an application to pass in {@link AndroidViewModel}
         * @deprecated Use {@link ViewModelProvider.AndroidViewModelFactory} or
         * {@link ViewModelProvider.AndroidViewModelFactory#getInstance(Application)}.
         */
        @Deprecated
        public DefaultFactory(@NonNull Application application) {
            super(application);
        }
    }
}
```

### ViewModelProvider

```java
/**
 * An utility class that provides {@code ViewModels} for a scope.
 * <p>
 * Default {@code ViewModelProvider} for an {@code Activity} or a {@code Fragment} can be obtained
 * from {@link android.arch.lifecycle.ViewModelProviders} class.
 */
@SuppressWarnings("WeakerAccess")
public class ViewModelProvider {

    private static final String DEFAULT_KEY =
            "android.arch.lifecycle.ViewModelProvider.DefaultKey";

    /**
     * Implementations of {@code Factory} interface are responsible to instantiate ViewModels.
     */
    public interface Factory {
        /**
         * Creates a new instance of the given {@code Class}.
         * <p>
         *
         * @param modelClass a {@code Class} whose instance is requested
         * @param <T>        The type parameter for the ViewModel.
         * @return a newly created ViewModel
         */
        @NonNull
        <T extends ViewModel> T create(@NonNull Class<T> modelClass);
    }

    private final Factory mFactory;
    private final ViewModelStore mViewModelStore;

    /**
     * Creates {@code ViewModelProvider}, which will create {@code ViewModels} via the given
     * {@code Factory} and retain them in a store of the given {@code ViewModelStoreOwner}.
     *
     * @param owner   a {@code ViewModelStoreOwner} whose {@link ViewModelStore} will be used to
     *                retain {@code ViewModels}
     * @param factory a {@code Factory} which will be used to instantiate
     *                new {@code ViewModels}
     */
    public ViewModelProvider(@NonNull ViewModelStoreOwner owner, @NonNull Factory factory) {
        this(owner.getViewModelStore(), factory);
    }

    /**
     * Creates {@code ViewModelProvider}, which will create {@code ViewModels} via the given
     * {@code Factory} and retain them in the given {@code store}.
     *
     * @param store   {@code ViewModelStore} where ViewModels will be stored.
     * @param factory factory a {@code Factory} which will be used to instantiate
     *                new {@code ViewModels}
     */
    public ViewModelProvider(@NonNull ViewModelStore store, @NonNull Factory factory) {
        mFactory = factory;
        this.mViewModelStore = store;
    }

    /**
     * Returns an existing ViewModel or creates a new one in the scope (usually, a fragment or
     * an activity), associated with this {@code ViewModelProvider}.
     * <p>
     * The created ViewModel is associated with the given scope and will be retained
     * as long as the scope is alive (e.g. if it is an activity, until it is
     * finished or process is killed).
     *
     * @param modelClass The class of the ViewModel to create an instance of it if it is not
     *                   present.
     * @param <T>        The type parameter for the ViewModel.
     * @return A ViewModel that is an instance of the given type {@code T}.
     */
    @NonNull
    @MainThread
    public <T extends ViewModel> T get(@NonNull Class<T> modelClass) {
        String canonicalName = modelClass.getCanonicalName();
        if (canonicalName == null) {
            throw new IllegalArgumentException("Local and anonymous classes can not be ViewModels");
        }
        return get(DEFAULT_KEY + ":" + canonicalName, modelClass);
    }

    /**
     * Returns an existing ViewModel or creates a new one in the scope (usually, a fragment or
     * an activity), associated with this {@code ViewModelProvider}.
     * <p>
     * The created ViewModel is associated with the given scope and will be retained
     * as long as the scope is alive (e.g. if it is an activity, until it is
     * finished or process is killed).
     *
     * @param key        The key to use to identify the ViewModel.
     * @param modelClass The class of the ViewModel to create an instance of it if it is not
     *                   present.
     * @param <T>        The type parameter for the ViewModel.
     * @return A ViewModel that is an instance of the given type {@code T}.
     */
    @NonNull
    @MainThread
    public <T extends ViewModel> T get(@NonNull String key, @NonNull Class<T> modelClass) {
        ViewModel viewModel = mViewModelStore.get(key);

        if (modelClass.isInstance(viewModel)) {
            //noinspection unchecked
            return (T) viewModel;
        } else {
            //noinspection StatementWithEmptyBody
            if (viewModel != null) {
                // TODO: log a warning.
            }
        }

        viewModel = mFactory.create(modelClass);
        mViewModelStore.put(key, viewModel);
        //noinspection unchecked
        return (T) viewModel;
    }

    /**
     * Simple factory, which calls empty constructor on the give class.
     */
    public static class NewInstanceFactory implements Factory {

        @SuppressWarnings("ClassNewInstance")
        @NonNull
        @Override
        public <T extends ViewModel> T create(@NonNull Class<T> modelClass) {
            //noinspection TryWithIdenticalCatches
            try {
                return modelClass.newInstance();
            } catch (InstantiationException e) {
                throw new RuntimeException("Cannot create an instance of " + modelClass, e);
            } catch (IllegalAccessException e) {
                throw new RuntimeException("Cannot create an instance of " + modelClass, e);
            }
        }
    }

    /**
     * {@link Factory} which may create {@link AndroidViewModel} and
     * {@link ViewModel}, which have an empty constructor.
     */
    public static class AndroidViewModelFactory extends ViewModelProvider.NewInstanceFactory {

        private static AndroidViewModelFactory sInstance;

        /**
         * Retrieve a singleton instance of AndroidViewModelFactory.
         *
         * @param application an application to pass in {@link AndroidViewModel}
         * @return A valid {@link AndroidViewModelFactory}
         */
        @NonNull
        public static AndroidViewModelFactory getInstance(@NonNull Application application) {
            if (sInstance == null) {
                sInstance = new AndroidViewModelFactory(application);
            }
            return sInstance;
        }

        private Application mApplication;

        /**
         * Creates a {@code AndroidViewModelFactory}
         *
         * @param application an application to pass in {@link AndroidViewModel}
         */
        public AndroidViewModelFactory(@NonNull Application application) {
            mApplication = application;
        }

        @NonNull
        @Override
        public <T extends ViewModel> T create(@NonNull Class<T> modelClass) {
            if (AndroidViewModel.class.isAssignableFrom(modelClass)) {
                //noinspection TryWithIdenticalCatches
                try {
                    return modelClass.getConstructor(Application.class).newInstance(mApplication);
                } catch (NoSuchMethodException e) {
                    throw new RuntimeException("Cannot create an instance of " + modelClass, e);
                } catch (IllegalAccessException e) {
                    throw new RuntimeException("Cannot create an instance of " + modelClass, e);
                } catch (InstantiationException e) {
                    throw new RuntimeException("Cannot create an instance of " + modelClass, e);
                } catch (InvocationTargetException e) {
                    throw new RuntimeException("Cannot create an instance of " + modelClass, e);
                }
            }
            return super.create(modelClass);
        }
    }
}
```

一、Component接口中定义的方法，要么可以使用@Inject提供实例，要么可以在关联的Module中找到@Providers方法来提供实例对象，否则会报错：java.lang.String cannot be provided without an @Inject constructor or an @Provides-annotated method.

二、如果在Component中随便定义一个返回void的方法，会报错：

 An exception occurred: java.lang.IllegalArgumentException: component method cannot be void: aaaaa()

三、Module的方法可以注入到Component中去，也可以注入