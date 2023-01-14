---
layout: post
title: "Spring Security核心原理"
subtitle: "理解Spring Security如何保护你的应用"
date: 2020-11-04 17:02:00
author: "Echcz"
header-style: "text"
catalog: true
tags:
  - "Spring"
  - "认证与授权"
  - "安全"
---
{% raw %}
## Spring Security是什么

Spring Security是Spring项目组中用来提供安全认证服务的框架，其前身是Acegi Security。Spring Security的核心功能是认证(你是谁)与授权(你能够干什么)，其核心是一组过滤器链，项目启动后将会自动配置。

## 核心组件

在探讨Spring Security的执行流程之前，我们先看一下其核心组件。这些核心组件主要是解决如何存取用户信息以供其执行流程使用的问题。

### SecurityContext与SecurityContextHolder

`SecurityContext`，即安全上下文，它就是用于存储安全信息的对象了。它主要存储了当前与应用交互的用户详细信息，这个详细信息被表示为`Authentication`对象。

为了方便我们从全局任何位置方便的得到`SecurityContext`，Spring Security提供了`SecurityContext Holder`来作为全局统一的`SecurityContext`存取器。默认情况下，`SecurityContextHolder`使用`ThreadLocal`来存储`SecurityContext`，这样同一个线程就使用同样的安全上下文了。当然，为了应对不同的业务要求，`SecurityContextHolder`还提供了`MODE_INHERITABLETHREADLOCAL`与`MODE_GLOBAL`安全上下文存储策略。

### Authenication、UserDetails与UserDetailsService

`Authenication`表示前当用户的认证详情，其则主要包含了当前用户的主体信息(`principal`)、用户使用认证使用的凭证、是否被认证与被授与的权限表示(`GrantedAuthority`)列表。

。在认证前，需要创建一个(不完整的)`Authenication`以供认证，在认证通过后，会返回一个完整的`Authenication`以表示当前用户的认证详情。

虽然`Authenhication`是使用`Object`来标示`principal`的，但Spring Security中的认证大都返回一个`UserDetails`实例作为`principal`。`UserDetails`抽象了一些在安全环境下用户主体信息应该提供的方法接口，如获取用户名(`getUsername()`)。在应用的任何地方，我们可以使用如下代码来获取当前用户的名字:

``` java
Object principal = SecurityContextHolder.getContext().getAuthentication().getPrincipal();
if (principal instanceof UserDetails) {
String username = ((UserDetails)principal).getUsername();
} else {
String username = principal.toString();
}
```

与`UserDetails`相对应的，Spring Security还提供了`UserDetailsService`接口以获取`UserDetails`实例。`UserDetailsService`只有一个方法，其接收一个`String`类型的用户名参数，返回`UserDetails`，或在没有此用户名相对应的用户时抛出一个异常:

```java
UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
```

### Authentication与GrantedAuthority

当完成认证后，`Authentication`除了保存了用户的主体信息(`principal`)外，另外一个要保存的重要信息是当前用户拥有的权限表示(`authorities`)，这是一个`GrantedAuthority`对象的集合。通常，这个权限表示是角色，比如：`ROLE_ADMIN`、`ROLE_VIP`之类的。Spring Security能解析这些权限，并在授权(访问控制)时检查是`Authentication`否有相应的权限。

通常情况下，`GrantedAuthority`对象是应用范围的权限，而不是特定于给定的域对象。不要用某个`GrantedAuthority`对象来表示某个特定用户(的全部权限)，这样系统会依赖成千上万个这样的权限，最终导致内存耗尽(在最好的情况下，也会使得应用在认证上耗费很长的时间)。Spring Security有专门设计用于处理此类需求，但应该使用项目的域对象安全性来实现此功能。

## 认证(Authentication)

所谓认证，就是要解决"你是谁"的问题，其一般的流程如下：

1. 要求用户输入用户名(用户标识)和密码(认证凭证)
2. 系统验证用户名与密码是否正确，正确进入下一步，错误则回到第一步并提示相关错误
3. 获取用户的的安全上下文信息(包含用户的基本信息与其所拥有的权限等)
4. 以经过认证的方式进一步处理用户请求

这个流程在Spring Security中的代码示例如下：

``` java
public class AuthenticationExample {
private static AuthenticationManager am = new SampleAuthenticationManager();

public static void main(String[] args) throws Exception {
    BufferedReader in = new BufferedReader(new InputStreamReader(System.in));

    while(true) {
    System.out.println("Please enter your username:");
    String name = in.readLine();
    System.out.println("Please enter your password:");
    String password = in.readLine();
    try {
        Authentication request = new UsernamePasswordAuthenticationToken(name, password);
        Authentication result = am.authenticate(request);
        SecurityContextHolder.getContext().setAuthentication(result);
        break;
    } catch(AuthenticationException e) {
        System.out.println("Authentication failed: " + e.getMessage());
    }
    }
    System.out.println("Successfully authenticated. Security context contains: " +
            SecurityContextHolder.getContext().getAuthentication());
}
}

class SampleAuthenticationManager implements AuthenticationManager {
static final List<GrantedAuthority> AUTHORITIES = new ArrayList<GrantedAuthority>();

static {
    AUTHORITIES.add(new SimpleGrantedAuthority("ROLE_ADMIN"));
}

@Override
public Authentication authenticate(Authentication auth) throws AuthenticationException {
    if ("admin".equals(auth.getName()) && "123456".equals(auth.getCredentials())) {
    return new UsernamePasswordAuthenticationToken(auth.getName(),
        auth.getCredentials(), AUTHORITIES);
    }
    throw new BadCredentialsException("Bad Credentials");
}
}
```

这个例子很简单，他会认证用户名为`admin`密码为`123456`的用户，并给他`ROLE_ADMIN`权限。其中`UsernamePasswordAuthenticationToken`是Spring Secruity定义的一个`Authentication`的实现类。当使用其不带权限的参数构造器返回的实例是未经认证的实例，使用带权限参数的构造器返回的实例是经过认证的实例。

### AuthenticationManager、ProviderManager与AuthenticationProvider

仔细看上文的认证示例代码，你会发现一个我们还没提过的接口`AuthenticationManager`。它是Spring Security抽象出来的，用来处理认证请求的接口。其只有一个方法，接收一个未经认证的(不完整的)`Authentication`对象，认证通过后返回一个经过认证的(完整的)`Authentication`，如果认证失败则抛出一个错误：

``` java
Authentication authenticate(Authentication auth) throws AuthenticationException;
```

与之相对应的，Spring Security还定义一个`AuthenticationManger`的实现类`ProviderManager`。但`ProviderManager`自己也没有直接处理认证请求，而是将其委托给了一组`AuthenticationProvider`。当`ProviderManager`收到一个`Authentication`时，它会遍历其维护的一组`ProviderManager`实例，然后判断这个实例是否可以对这个`Authentication`进行认证，如果可以就将认证请求委托给这个实例。这样，我们就可以针对不同的`Authentication`使用不用的认证流程了，只需要定义其对应的`AuthenticationProvider`，然后将其注册到`ProviderManager`中即可。

### Web应用中的认证

一个典型的Web应用认证流程如下：

1. 用户请求访问某个被保护的链接
2. 服务端发现用户还未认证，于是返回一个响应，要求用户进行认证
3. 客户端重定向到登录页面，要求用户输入认证信息进行登录
4. 用户输入认证信息后执行登录
5. 服务器收到登录请求后，对认证信息进行认证。如果认证通过，执行下一步；如果认证失败，回到步骤2
6. 重试原始请求(即访问步骤1中的链接)

Spring Security定义一些类以处理上述步骤。主要有：`ExceptionTranslationFilter`、`AuthenticationEntryPoint`和上文提到过的`AuthenticationManager`及其实现类。

`ExceptionTranslationFilter`负责侦测所有抛出的Spring Security异常。其会在认证失败和授权失败(下文会讨论)时返回特定的(表示错误的)响应。

`AuthenticationEntryPoint`则用于处理步骤3，以让用户在需要认证时重定向到登录页。

#### 在请求之间存储SecurityContext

根据应用程序的类型，可能需要采用策略来在用户操作之间存储安全上下文。在典型的Web应用程序中，用户登录过后，会由其`session ID`标识。服务器在会话时间内缓存`principal`信息。在`Spring Security`中，在请求之间存储`SecurityContext`的工作由`SecurityContextPersistenceFilter`负责，默认将`SecurityContext`存储到`HttpSession`中，并为每个请求恢复`SecurityContext`到`SecurityContextHolder`中，并且当请求完成时还会清除`SecurityContextHolder`。出于安全目的，不应该直接从`HttpSession`中获取`SecurityContext`，而是使用`SecurityContextHolder`。

许多其他类型的应用(例如无状态的RESTful Web服务)不使用`HttpSession`，且为每一个请求都重新认证。但是，在`filter chain`中引入`SecurityContextPersistenceFilter`来确保每个请求之后都清除了`SecurityContextHolder`仍然很重要。

需要注意的是在单个session中接受并发请求的应用中，同一个`SecurityContext`实例会被线程共享。虽然使用了`ThreadLocal`，但每个请求从`HttpSession`中获取到的都是同一个实例。如果使用`SecurityContextHolder.getContext()`，并在返回的上下文对象上调用`setAuthentication()`方法，那么`Authentication`对象会在共享同一个`SecurityContext`实例的并发线程中修改。可以自定义`SecurityContextPersistenceFilter`的行为来创建为每个请求创建一个全新的`SecurityContext`，防止一个线程中的变动影响到其他线程。或者改动上下文的时候创建一个新的实例。`SecurityContextHolder.createEmptyContext()`方法始终返回一个全新的上下文实例。

## 授权(AccessDecision)

所谓授权，就是要解决"你能够干什么"的问题，其是建立在已通过认证的基础之上的。当用户执行某请求时，系统需要先经过认证，确认其所拥有的权限，然后将用户所拥有的权限与此请求所需要的权限进行匹配。如果匹配通过，就让用户执行原请求，如果匹配失败，就返回一个错误，告诉用户无权执行此请求。这个匹配权限，并根据匹配结果控制是否执行原请求的过程就是授权或访问控制。

### AccessDecisionManager

在Spring Security中，做出授权决定的就是`AccessDecisionManager`。它有一个方法，接收`Authentication`对象、安全对象(表示任何可以应用安全保护的对象，最常见的例子是方法执行和Web请求)和适用于该对象的安全元数据属性列表(如请求所需要的权限)以决定是否允许访问，并在拒绝后抛出异常：

``` java
void decide(Authentication authentication, Object object,
        Collection<ConfigAttribute> configAttributes) throws AccessDeniedException,
        InsufficientAuthenticationException;
```

### AOP与Web过滤器

Spring Security的安全控制是声明式的，这时你可能会问，Spring Security是在哪里使用b了`AccessDecisionManager`以保护应用的接口的。

如果熟悉AOP，你会知道这几种不同类型的advice：`before`、`after`、`throws`和`around`。around advice非常有用，因为它可以选择是否继续进行方法的调用，是否修改返回值，是否抛出异常。Spring Security为方法调用和web请求提供了around advice。Spring Security使用Spring标准的AOP支持来为方法执行进行around advice，通过标准的Web过滤器为Web请求实现around advice。

### AbstractSecurityInterceptor

Spring Security定义了`AbstractSecurityInterceptor`，为受保护的接口建构around advice。它为处理安全对象提供了一致的工作流程：

1. 查找与当前请求关联的"配置属性"
2. 将安全对象、`Authentication`和配置属性提交给到`AccessDecisionManager`以进行授权决策
3. 在调用时改变`Authentication`(可选)
4. 允许安全对象调用继续(如果访问被授权)
5. 调用返回后，如果有配置，就调用`AfterInvocationManager`。如果调用抛出异常，`AfterInvocationManager`不会执行。

### 配置属性

"配置属性"可以被认为是对于被`AbstractSecurityInterceptor`使用的类有特殊意义的字符串。在框架中由`ConfigAttribute`接口表示。取决于`AccessDecisionManager`实现的复杂程度，它可能是简单的角色名或更复杂的含义。`AbstractSecurityInterceptor`配置了一个`SecurityMetadataSource`，这个对象可以用来查找某个安全对象的属性。通常这个配置是对用户隐藏的。配置属性会作为被保护方法的注解或者被保护URL的访问属性加入。

### RunAsManager

如果`AccessDecisionManager`允许请求，`AbstractSecurityInterceptor`通常只会继续这个请求。话虽如此，但在某些极端情况下(例如某个服务层方法需要以不同的身份调用远程系统)，用户可能希望使用不同的`Authentication`替换`SecurityContext`中的`Authentication`，这由`AccessDecisionManager`调用`RunAsManager`处理。

### AfterInvocationManager

安全方法调用继续并返回，这可能意味着某个方法完成了或者某个过滤器链的继续，`AbstractSecurityInterceptor`获得最后的机会来处理调用。此阶段，`AbstractSecurityInterceptor`可能会修改返回对象。我们可能希望这种情况发生，因为无法在安全对象调用的途中进行授权决策。为了高度可拔插性，`AbstractSecurityInterceptor`会将控制权传递给`AfterInvocationManager`，以便在需要时修改返回对象。这个类甚至能完全替换掉返回对象，或者抛出异常，或者不以任何方式改变它。只有在调用成功后才会执行调用后检查。如果有任何的异常，这个检查会被跳过。
{% endraw %}