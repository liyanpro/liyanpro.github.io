---
title: SpringBoot集成Shiro权限管理
date: 2018-04-15 17:26:19
header-img: https://bucketyy.oss-cn-beijing.aliyuncs.com/image/post-bg-kuaidi.jpg
tags: 
    - shiro
    - springboot
    - 权限管理
---


最近给项目集成shiro权限管理，之前没有接触过，这几天也是查了一些资料，初步实现了登录验证和url的权限管理。

## 对Shiro的理解
  shiro是Apache的开源框架，是一种功能强大且易用的安全框架，主要有四种功能。

  * 认证——用户的身份识别，常用作用户的登录
  * 权限——访问的控制，对于每次访问都会检查是否具有访问权限
  * 会话管理——会话管理，用户session管理器，用户相关的时间敏感的状态
  * 密码加密——把JDK中复杂的密码加密方式进行封装，保护或隐藏数据防止被偷窥

网上找到架构图如下：
![](https://upload-images.jianshu.io/upload_images/4367237-5569f11139646687.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

shiro包含三个核心的组件：Subject,SecurityManager 和 Realms
* Subject是与程序交互的对象，一般称之为用户，Subject 实例都必须绑定到一个SecurityManager上。一个applicationCode与Subject 的交互，在运行时shiro会自动将次subject转化为与SecurityManager的特定subject的交互 。
* SecurityManager是shiro的核心，协调各个模块的运行，无需我们对其操作，我们只要操作subject就好，其他都交给SecurityManager去管理。
* Realms是应用程序与持久化数据之间交互的桥梁，需要我们自己编程去实现相应的功能。比如登录认证需要实现对doGetAuthenticationInfo方法的重写，权限认证需要实现对doGetAuthorizationInfo方法的重写

## 集成shiro
#### 添加依赖
  
    <dependency>
        <groupId>org.apache.shiro</groupId>
		<artifactId>shiro-spring</artifactId>
		<version>1.3.2</version>
    </dependency>
#### 权限与角色表结构
总共分四张表，用户角色表、角色表、角色权限表以及权限初始化表，用户与角色挂钩，角色与权限挂钩，每个用户可以担任多个角色，每个角色可以拥有多个权限，权限初始化表主要是对Shiro 拦截的请求做声明，提供访问页面与权限的关系。
角色表

    DROP TABLE IF EXISTS `role`;
    CREATE TABLE `role` (
       `id` bigint(64) DEFAULT NULL,
       `name` varchar(32) DEFAULT NULL COMMENT '角色名称'
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8;    
角色权限表

    DROP TABLE IF EXISTS `role_permission`;
    CREATE TABLE `role_permission` (
     `id` bigint(64) NOT NULL,
     `rid` bigint(64) DEFAULT NULL COMMENT '角色ID',
     `pname` varchar(255) DEFAULT NULL COMMENT '权限ID',
      PRIMARY KEY (`id`)
      ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
用户角色表

    DROP TABLE IF EXISTS `user_role`;
    CREATE TABLE `user_role` (
     `id` bigint(64) NOT NULL,
     `uid` bigint(64) DEFAULT NULL COMMENT '用户ID',
     `rid` bigint(64) DEFAULT NULL COMMENT '角色ID',
      PRIMARY KEY (`id`)
      ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
权限初始化表

    DROP TABLE IF EXISTS `permission_init`;
    CREATE TABLE `permission_init` (
     `id` bigint(11) NOT NULL,
     `url` varchar(255) DEFAULT NULL COMMENT '链接',
     `permission_init` varchar(255) DEFAULT NULL COMMENT '需要具备的权限',
     `sort` int(11) DEFAULT '0' COMMENT '排序',
      PRIMARY KEY (`id`)
      ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
权限初始化表用来存储访问链接与其需要具备的权限，在shiro的配置类中进行动态加载。

#### Shiro配置类

    /**
     * shiro配置
     * 
     * @title
     * @author liyan
     * @date 2018年5月23日 下午2:43:45
     */
     @Configuration
     public class ShiroConfig {
         @Autowired
	     private PermissionService permissionService;
	  @Bean
	  public ShiroFilterFactoryBean shiroFilter(SecurityManager sec urityManager) {
		ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
		shiroFilterFactoryBean.setSecurityManager(securityManager);
		// 设置 SecurityManager
		shiroFilterFactoryBean.setSecurityManager(securityManager);
		// 设置登录页
		shiroFilterFactoryBean.setLoginUrl("/login.html");
		// 登录成功后要跳转的链接
		shiroFilterFactoryBean.setSuccessUrl("/list.html");
		// 未授权界面;
		shiroFilterFactoryBean.setUnauthorizedUrl("/403.html");
		// 权限控制map.
		Map<String, String> filterChainDefinitionMap = new LinkedHashMap<String, String>();
		// 从数据库获取访问链接的初始化权限，对shiro拦截的请求做声明
		List<PermissionInit> permissionList = permissionService.getPermissionInit();
		for (PermissionInit permission : permissionList) {
			filterChainDefinitionMap.put(permission.getUrl(), permission.getPermissionInit());
		}
		shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
		return shiroFilterFactoryBean;
	}

	 @Bean
	 public SecurityManager securityManager() {
		DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
		// 设置realm.
		securityManager.setRealm(myShiroRealm());
		// 注入记住我管理器;
		securityManager.setRememberMeManager(rememberMeManager());
		return securityManager;
	}

	/**
	 * 权限认证realm
	 * 
	 * @return
	 */
	 @Bean
	 public ShiroRealm shiroRealms() {
		ShiroRealm shiroRealm = new ShiroRealm();
		return shiroRealm;
	}

	/**
	 * cookie管理对象;记住我功能
	 * 
	 * @return
	 */
	 public CookieRememberMeManager rememberMeManager() {
		CookieRememberMeManager cookieRememberMeManager = new CookieRememberMeManager();
		cookieRememberMeManager.setCookie(rememberMeCookie());
		// rememberMe cookie加密的密钥 建议每个项目都不一样 默认AES算法 密钥长度(128 256 512 位)
		cookieRememberMeManager.setCipherKey(Base64.decode("3AvVhmFLUs0KTA3Kprsdag=="));
		return cookieRememberMeManager;
	}

	/**
	 * cookie对象;
	 * 
	 * @return
	 */
	 public SimpleCookie rememberMeCookie() {
		// 这个参数是cookie的名称，对应前端的checkbox的name = rememberMe
		SimpleCookie simpleCookie = new SimpleCookie("rememberMe");
		// <!-- 记住我cookie生效时间30天 ,单位秒;-->
		simpleCookie.setMaxAge(2592000);
		return simpleCookie;
	  }
    }

在不添加cookie对象管理的时候，登录接口里如果remeberMe为true，调用会报异常。需要注意的一点是，我在permission_init表中初始化的页面链接没有加页面的后缀.html，比如字段url的值为/create，过滤器拦截一直失败，只有改成/create.html才成功拦截进入到realm中。ShiroRealm是需要我们自己继承AuthorizingRealm类，并且重写doGetAuthenticationInfo和doGetAuthorizationInfo方法。

     UsernamePasswordToken token = new UsernamePasswordToken(username, password, rememberMe);

#### 实现ShiroRealm
##### 登录认证

在Shiro中，最终的认证都是通过realm来获取应用程序的用户角色以及权限等信息，在对登录认证过程中，shiro会调用reaml的doGetAuthenticationInfo(AuthenticationToken authcToken)方法，验证通过后会返回一个封装了用户信息的SimpleAuthenticationInfo实例

    /**
	 * 登录验证: Authentication 是用来验证用户身份
	 * 
	 * @param token
	 * @return
	 * @throws AuthenticationException
	 */
	@Override
	protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authcToken)
			throws AuthenticationException {
		UsernamePasswordToken token = (UsernamePasswordToken) authcToken;
		String name = token.getUsername();
		String password = String.valueOf(token.getPassword());
		// 密码进行加密处理 明文为 password+name
		String paw = password + name;
		String pawDES = DesEncrypt.encryptBasedDes(paw);
		// 从数据库获取对应用户名密码的用户
		UserInfo userInfo = logionService.getUserInfo(name, pawDES);
		if (null == userInfo) {
			return null;
		} else {
			// 更新登录时间 last login time
			userInfo.setLastLoginTime(new Date());
			logionService.updateLoginTime(userInfo);
		}
		System.out.println("身份认证成功，登录用户：" + name);
		return new SimpleAuthenticationInfo(String.valueOf(userInfo.getUserId()), password, getName());
	}
##### 权限认证
访问的url只有在shiro的配置类里面添加了权限的filterChainDefinitions配置后，才会被拦截并跳转到重写的doGetAuthorizationInfo(PrincipalCollection principals)方法中，该方法获取当前用户的角色和拥有的权限信息，并封装到SimpleAuthorizationInfo的实例中，验证通过的允许访问，未经授权的会跳转到shiro配置类里面预先定义好的页面中，参考上面描述。

    /**
	 * 权限认证：Authorization用来进行权限验证
	 */
	@Override
	protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
		String userId = (String) SecurityUtils.getSubject().getPrincipal();
		SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
		Long userid = StringUtil.parseLong(userId);
		// 查询用户所属角色，放入到Authorization里。
		Set<String> roleSet = permissionService.getUserRole(userid);
		info.setRoles(roleSet);
		// 查询用户所拥有的权限（permission），放入到Authorization里。
		Set<String> permissionSet = permissionService.getUserPermission(userid);
		info.setStringPermissions(permissionSet);
		return info;
	}
      
#### 接口调用
在登录接口中将登录信息封装在UsernamePasswordToken中，再调用subject的login方法执行登录，这时会进入realm中进行登录信息的验证工作。

        UsernamePasswordToken token = new UsernamePasswordToken(username, password, rememberMe);
		Subject currentSubject=SecurityUtils.getSubject();
		currentSubject.login(token);
		currentSubject.getSession().setAttribute("name", username);
		serviceResult = new ServiceResult(HttpConstants.RESUTL_OK, "登录成功");