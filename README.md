
**说明**


    该文章是属于OverallAuth2\.0系列文章，每周更新一篇该系列文章（从0到1完成系统开发）。


    该系统文章，我会尽量说的非常详细，做到不管新手、老手都能看懂。


    说明：OverallAuth2\.0 是一个简单、易懂、功能强大的权限\+可视化流程管理系统。


友情提醒：本篇文章是属于系列文章，看该文章前，建议先看之前文章，可以更好理解项目结构。


qq群：801913255


**有兴趣的朋友，请关注我吧(\*^▽^\*)。**


**![](https://img2024.cnblogs.com/blog/1158526/202408/1158526-20240824140446786-404771438.png)**


**关注我，学不会你来打我**


**前言**


这篇文章有点长，内容丰富，如果你对该文章感兴趣，请耐心观看。


**一、什么是路由守卫，它的作用是什么**


**什么是路由守卫：**它是控制路由菜单访问的一种机制，当一个用户点击一个路由菜单时，那么路由守卫就会对其进行“保护”，常见的守卫方式有


beforeEach：路由菜单访问前守卫。


afterEach：路由菜单访问后守卫。


**路由守卫的作用：**了解什么是路由守卫后，其实我们大致可以得出它大致有以下作用。


　　1、身份认证：在进入模块之前，验证用户身份是否正确，列如：登录是否过期，用户是否登录等。


　　2、权限控制：控制用户、角色对应模块的访问权限。


　　3、日志记录：由于路由守卫能监控到用户对于模块访问前和访问后的动作，那么我们可以用来记录用户的访问日志等。


　　4、数据预加载：在很多时候，些许数据需要在我们访问页面前，加载完成。


　　5、路由动画：可以在路由访问前后，加载一个过渡动画，提高用户体验。


**二、路由守卫的使用**


在使用之前，我们需要安装状态管理库和状态持久化插件以及路由加载进度条。它可以共享程序中的一些状态。


　　1：安装npm install pinia  状态存储库


　　2：安装npm install pinia\-plugin\-persistedstate  状态持久化插件


　　3：安装 npm install nprogress  进度条插件


　　4：安装 npm install @types/nprogress


书接上一篇：[Vue3中菜单和路由的结合使用，实现菜单的动态切换](https://github.com)


创建一个路由文件index.ts，存放在指定文件夹下，由于我写的是从0到1搭建框架，我放在了以下目录中


![](https://img2024.cnblogs.com/blog/1158526/202411/1158526-20241111143751660-1998937052.png)


内容如下：




```
import { createRouter, createWebHashHistory, NavigationGuardNext, RouteLocationNormalized } from 'vue-router'
import { routes } from './module/base-routes'
import NProgress from 'nprogress'
import 'nprogress/nprogress.css'

NProgress.configure({ showSpinner: false })

const router = createRouter({
  history: createWebHashHistory(), //开发环境
  routes
})

/**
路由守卫，访问路由菜单前拦截
 * @param to 目标
 * @param from 来至 
 */
router.beforeEach((to: RouteLocationNormalized, from: RouteLocationNormalized, next: NavigationGuardNext) => {
  NProgress.start();
 if (to.meta.requireAuth) {
    next();
  } else if (to.matched.length == 0) {
    next({ path: '/panel' })
  } else {
    next();
  }
})

router.afterEach(() => {
  NProgress.done();
})

export default router
```


代码解释：


router.beforeEach：就是每次在访问路由前，都会进入的方法，在该方法中，我添加了一个进度条和路由访问后的拍断。

router.afterEach：就是每次在访问路由后，都会进入的方法，在该方法中，添加了一个进度条结束的方法。

然后我们在main中，全局注册路由和状态管理


![](https://img2024.cnblogs.com/blog/1158526/202411/1158526-20241111150741748-2014650247.png)


做好以上这些，路由守卫就完成。


如果，你是按照我的系列文章所搭建的前端框架，那么你要在以下2个文件中，做出改动（没有，请忽略）。


　　1、在HelloWorld.vue文件中把import router, { routes } from "../router/module/base\-routes";替换成import  { routes } from "../router/module/base\-routes"; 并加入import router from "../router/index";


　　2、在base\-routes.ts文件中删除以下代码




```
//创建路由，并且暴露出去
const router = createRouter({
  history: createWebHashHistory(), //开发环境
  //history:createWebHistory(), //正式环境
  routes
})
export default router
```


做好这些，我们启动项目，检查每次点击路由菜单，是否进入路由守护拦截中。


![](https://img2024.cnblogs.com/blog/1158526/202411/1158526-20241111151823031-1421135877.png)


明白的伙伴，请抓紧去填充你的内容吧。


**三、请求拦截、响应拦截**


我们OverallAuth2\.0使用的是Vue3\+.net8 WebApi创建的项目，所以我们会使用到后端接口。那么我们前端该如何和后端建立数据交互关系？建立关系会该如何处理返回信息？不要着急，耐心往下看。


 首先要安装组合式api请求插件axios


安装命令：npm install axios


然后按照下图，新建文件及文件夹


![](https://img2024.cnblogs.com/blog/1158526/202411/1158526-20241113141623058-2085406538.png)


http.ts文件内容如下




```
import axios, { AxiosResponse, InternalAxiosRequestConfig } from 'axios';

//声明模型参数
type TAxiosOption = {
    timeout: number;
    baseURL: string;
}

//配置赋值
const config: TAxiosOption = {
    timeout: 5000,
    baseURL: "https://localhost:44327/",    // 本地api接口地址
}

class Http {
    service;
    constructor(config: TAxiosOption) {
        this.service = axios.create(config)

        /* 请求拦截 */
        this.service.interceptors.request.use((config: InternalAxiosRequestConfig) => {
            //可以在这里做请求拦截处理   如：请求接口前，需要传入的token
            debugger;
            return config
        }, (error: any) => {
            return Promise.reject(error);
        })

        /* 响应拦截 */
        this.service.interceptors.response.use((response: AxiosResponse) => {
            debugger;
            switch (response.data.code) {
                case 200:
                    return response.data;
                case 500:
                    //这里面可以写错误提示，反馈给前端
                    return response.data;
                case 99991:
                    return response.data;
                case 99992:
                    return response.data;
                case 99998:
                    return response.data;
                default:
                    break;
            }
        }, (error: any) => {
            return Promise.reject(error)
        })
    }

    /* GET 方法 */
    get(url: string, params?: object, _object = {}): Promise {
        return this.service.get(url, { params, ..._object })
    }
    /* POST 方法 */
    post(url: string, params?: object, _object = {}): Promise {
        return this.service.post(url, params, _object)
    }
    /* PUT 方法 */
    put(url: string, params?: object, _object = {}): Promise {
        return this.service.put(url, params, _object)
    }
    /* DELETE 方法 */
    delete(url: string, params?: any, _object = {}): Promise {
        return this.service.delete(url, { params, ..._object })
    }
}

export default new Http(config)
```


以上代码关键点都有注释说明


user.ts中的内容如下




```
import Http from '../http';

export const TestAutofac = function () {
    return Http.get('/api/SysUser/TestAutofac');
}
```


这个就是我们之前搭建后端框架，演示示例的接口地址。


做完以上工作，整个响应拦截和请求拦截的基本代码编写完成，接下来就是测试。


**四、测试**


　　在用户界面添加以下代码（也可以新建vue页面）




```

  用户



```


点击用户菜单，测试请求拦截是否成功


![](https://img2024.cnblogs.com/blog/1158526/202411/1158526-20241113151422310-604547711.png)


拦截成功，但出现如下错误（跨域问题）


![](https://img2024.cnblogs.com/blog/1158526/202411/1158526-20241113151437572-1618363977.png)


上面问题是跨域问题导致，我们要在接口端配置跨域地址。


打开之前我们的后端框架，创建如下文件CrossDomainPlugIn.cs，路径和jwt鉴权，全局异常捕获插件放在同一个位置。




```
 /// 
 /// 跨域配置插件
 /// 
 public static class CrossDomainPlugIn
 {
     /// 
     /// 跨域
     /// 
     /// 
     public static void InitCors(this IServiceCollection services)
     {
         //允许一个或多个来源可以跨域
         services.AddCors(options =>
         {
             options.AddPolicy("Access-Control-Allow-Origin", policy =>
             {
                 var result = AppSettingsPlugIn.GetNode("CustomCorsPolicy:WhiteList").Split(',');
                 // 设定允许跨域的来源，有多个可以用','隔开
                 policy.WithOrigins(result)
                 .AllowAnyHeader()
                 .AllowAnyMethod()
                 .AllowCredentials();
             });
         });
     }
 }
```


然后再Program.cs文件中，添加


//跨域配置builder.Services.InitCors();和app.UseCors("Access\-Control\-Allow\-Origin");代码


appsettings.json配置中添加如下配置，地址是前端访问地址


 /\*跨越设置\*/ "AllowedHosts": "\*", "CustomCorsPolicy": { "WhiteList": "http://localhost:8080" },


启动后端接口，在测试下


![](https://img2024.cnblogs.com/blog/1158526/202411/1158526-20241113152405416-1530425982.png)


![](https://img2024.cnblogs.com/blog/1158526/202411/1158526-20241113152423446-1232166724.png)


以上就是本篇文章的全部内容，感谢耐心观看


****后端WebApi**预览地址：http://139\.155\.137\.144:8880/swagger/index.html**


**前端vue 预览地址：http://139\.155\.137\.144:8881**


**关注公众号：发送【权限】，获取前后端代码**


**有兴趣的朋友，请关注我微信公众号吧(\*^▽^\*)。**


**![](https://img2024.cnblogs.com/blog/1158526/202408/1158526-20240824140446786-404771438.png)**


关注我：一个全栈多端的宝藏博主，定时分享技术文章，不定时分享开源项目。关注我，带你认识不一样的程序世界


 


 本博客参考[slower加速器](https://jisuanqi.org)。转载请注明出处！
