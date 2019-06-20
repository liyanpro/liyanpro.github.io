---
title: Express项目结构总结
date: 2018-06-18 22:19:37
tags:
---
![](https://upload-images.jianshu.io/upload_images/4367237-3acf6f265b59daa8.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


站酷建站的后台管理系统是基于node.js平台的Express框架构成，这是一个极简、灵活的 web 应用开发框架，说白了就是完全由路由和中间件组成的web框架，一个express应用就是在调用各种中间件。

###项目结构
![建站后台目录结构图](https://upload-images.jianshu.io/upload_images/4367237-56f99d1967d30dc1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
bin：存放项目启动的执行文件
config：项目的配置文件，json格式，对应开发、测试到正式环境的配置文件各一份，内容包括环境名称、redis、es、mysql、上传的oss-path、域名等信息。
node_modules：项目需要依赖的模块，在该目录下执行npm install安装项目需要的模块。
public：静态资源文件夹
routes：路由文件，相当于mvc中的controller，默认创建的express项目包括index.js和user.js
views：视图文件，相当于mvc中的view。
package.json：项目的依赖配置与版本信息
zdiy.js：默认的名称为app.js，应用的核心配置文件，加载初始化依赖模块，应用的入口。

###集成权限认证中间件Passport

权限认证几乎是任何系统都存在的很重要的模块，也是实现比较繁琐和复杂的，在java中有Apache的Shiro和SpringSecurity等权限认证框架，node.js中自然用到passport。安装好passport后，在routes目录中创建passport.js文件，对passport进行配置。
* 策略
Passport使用所谓的策略来验证请求。策略包括验证用户名和密码，使用[OAuth](http://oauth.net/)委派身份验证或使用[OpenID的](http://openid.net/)联合身份验证。
在要求Passport对请求进行身份验证之前，必须配置应用程序使用的策略。
策略及其配置通过该`use()`功能提供。例如，以下使用`LocalStrategy`本地认证策略用于用户名/密码认证，通过表单提交给应用程序，登录表单通过该POST方法提交给服务器，使用 authenticate()与local战略将处理登录请求。

        var passport = require('passport');
        var LocalStrategy = require('passport-local').Strategy;
        var users = require('./users');

        passport.use(new LocalStrategy(function(username, password, cb) {
	      users.findByUsername(username, function(err, user) {
		  if (err) {
			return cb(err);
		  }
		  if (!user) {
			return cb(null, false, { message: '用户名不存在.' });
		  }
		  if (user.password != password) {
			return cb(null, false, { message: '密码错误.' });
		  }
		  return cb(null, user);
	      });
        }));
  这里有一个重要的概念称为验证回拨，目的是查找拥有一组凭证的用户，当Passport验证请求时，它将解析请求中包含的凭证。然后在这种情况下username，使用这些凭证作为参数调用验证回调password。如果凭证有效，则验证回调会调用done以向Passport提供经过验证的用户。

      return cb(null, user);

  如果凭证无效（例如，如果密码不正确），则done应该调用凭据 false而不是用户来指示验证失败。

      return cb(null, false, { message: '用户名不存在.' });
  路由index.js中配置的登录请求

      router.post('/login', function(req, res, next) {
	        passport.authenticate('local', function(err, user, info) {
		    if (err) {
			  return next(err);
		    }
		    if (!user) {
			    req.flash('errors', { msg:  info.message});
		        res.render('login', { title: '登录', msg: info.message});
			    return;
		    }
		    if (req.body.remember) {
		    	req.session.cookie.maxAge = 1000 * 60 * 3;
		    } else {
		    	req.session.cookie.expires = false;
		    }
		    req.session.displayName = user.displayName;
		    req.session.level = user.level;
		    req.login(user, function(err) {
            if (err) return next(err);
              req.flash('success', { msg: '登录成功！' });
              res.redirect('/web/index');
          });
	   })(req, res, next);
       });

* 中间件
在zdiy.js中需要配置passport中间件， `passport.initialize()` 用来初始化Passport。如果应用程序使用持久登录会话，则还必须使用`passport.session()`。启用会话支持完全是可选的，启用时，务必在`session() `之后使用。

      app.use(session({
	    secret: 'thisxulz.com',
	    resave: true,
	    saveUninitialized: true
      } )); // session secret
      app.use(passport.initialize());
      app.use(passport.session());



* 会话
在web应用程序中，对用户进行身份验证的凭据仅在登录请求期间传输。如果验证成功，会话将通过用户浏览器中设置的cookie建立和维护。
每个后续请求将不会包含凭据，而是标识会话的唯一Cookie。为了支持登录会话，Passport将序列化user.id和退出会话的实例。
    
        passport.serializeUser(function(user, cb) {
	      cb(null, user.id);
        });
        passport.deserializeUser(function(id, cb) {
	      users.findById(id, function(err, user) {
		    if (err) {
			  return cb(err);
		    }
		    cb(null, user);
	        });
        });
参考<http://www.passportjs.org/>
###功能开发实例
模版审核功能
* 添加view
新建template.ejs文件，编写html内容。
* 添加routes
添加页面的路由
          
       router.get('/web/templates',isLoggedIn, function(req, res, next) {
	         if(req.session.level >= 5 ){
		         res.render('templates', { title: '模版管理' ,displayName : 
                 req.session.displayName,imagePath:OSSPATH,env:env});
	         }else{
	  	       res.render('login', { title: '登录', msg: '您没有权限查看该模块，请 
                    联系管理员.'});
	         }
        });

  页面中调用的api也需要添加到路由

       router.post('/api/getTemplates', isLoggedIn, function(req,res) {
	     zcoolDo.getTemplates(req,res);
       });
* 自定义模块zcool-do.js中添加后台接口的业务代码
包括业务逻辑处理，查询数据库操作并返回相应数据

      zdoMysql.query('select count(*) as num' + sql).then(function(total){
		var start = (parseInt(req.body.pageNo) - 1) * 20;
		sql += ' order by template.template_id desc limit ' + start + ',20';
		zdoMysql.query('select * ' + sql).then(function(rows){
			if(rows && rows.length > 0){
				res.send({
					result : true,
					total : total[0].num,
					rows : rows
				});
			}else{
				res.send({
					result : false,
					msg : "没有数据"
				});
			}
	    })
      }).catch(function(e){
	    	logger.error("获取模版信息异常:" + e);
	    	res.send({
			result : false,
			msg : "数据库查询异常"
		});
      });
 注意，由于node.js的异步机制，在处理循环中的事件时不能按照同步的逻辑来写。比如在一个定时任务中，需要按照查询的结果集给用户发送通知短信，可以使用回调的方式来达到同步的效果。

          // 获取用户手机号码
	request(DOMAIN+"/api/getUserInfo.do?userid="+rows[i].user_id, function (error, response, body) {
		  if (!error && response.statusCode == 200) {	
			  var url = "";//短信发送请求URL  
			  logger.info("用户电话："+response.body.phonenum+" 发送内容："+content);	
			  request(url, function (err, response, body) {
					  if (err) {
						  console.log("过期用户套餐通知短信发送失败");
						  logger.error("过期用户套餐通知短信发送失败,phoneNum = " + err.errPhone);
					  }
					  else{
						  logger.info("用户id："+rows[i].user_id+"的短信发送成功,模版id:"+templateId);
						  sendMsg(++i,rows,templateId,isExceed,date);
					  }
					});
		  }else{
			  logger.info("获取用户手机号失败");
			  console.log("获取用户手机号失败");
		  }
		});

在给一个用户发送成功后回调当前函数sendMsg，在发送前判断i的值是否小于结果集的大小即可。
