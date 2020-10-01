---
title: Vue-Router
date: 2020-04-28 19:59:32
tags: Vue-Router
excerpt: Summary of Vue-Router from RFC
categories:
- Vue
---

### vue-router
1. Dynamic Route Matching
Dynamic Route Matching ==>      { path:/user/:id , component:User}
both /user/foo and /user/bar  will map to the same route.
**No component reloaded=> No component life cycle called.**  
e.g.  from /user/foo to /user/bar     => more efficiently but have to use other methods to detect the params changes.

Two methods to detect&react to route params changes: 
(1) watch: { $route(to,from) { ... //react to route changes }  }
(2) beforeRouteUpdate(to , from , next ){ .. }        //always remember to call the next() function

All matched parts will turned to be the value of this.$route.params  ==>     { id:'foo'}         {id:'bar'}
*Allows multiple dynamic segments

Using asterisk * to `match all / catch 404 error` { path:'*' }       || { path:'/user-*'}
A param named pathMatch will be automatically added to  this.$route to tell the matching part.
* ROUTES RUN IN DEFINIED ORDER. (MATCHING PRIORITY)
* ALWAYS PUT THE ERROR MATCHING ROUTE AT THE END OF THE WHOLE APP.
e.g /user-halo      ==> and the value of 'this.$route.params.pathMatch' is 'halo' ;        *=>halo
(Internal: using path-to-regexp module, for more matching patterns, check out its docs.)
More:  Make the params optional by adding a questionmark ? => /optional/foo?  or       /optional/(foo/)?bar
Matched only if ids are numbers  => /foo/:id(\\d+)

2. Nested Routing 
Nested path that starts with '/' will be treated as the root path. Therefore, **there is no need to redeclare the root path.**
Not clear: { path:'/user/:id' , component:User, children:[ { path:'', component: Userhome }]}
When we visit /user/foo,  components User and Userhome will both be rendered  ==>User for /user/foo and Userhome for /user/ ??usage?

3. Programmatic Navigations
Declarative                                                      Programmatic
<router-link :to="..." />                                 this.$router.push( ... )
<router-link :to="..." replace />                         this.$route.replace( ... )

* DIFFERENCES:  'push' *pushes* a new entry into the history stack while 'replace' navigates without pushing a new history entry--it *replaces* the current entry.

Params: push/replace(location, onComplete? , onAbort?)
e.g this.$router.push({ path:'/home'})    push({name:'pathName' , params:{ userId:123}})   push({ path:'/pathName' , query:{plan:'private'}})
names  => params   //using post method?
path => query         //get method? Params are ignored if used in this case.
* SAME RULES APPLY TO <route-link :to="..." /> and replace(...) METHOD

onComplete/ onAbort : Called when the navigation either *successfully completed or aborted*.(Can be ommited)

4. Named Routes / Named Views 
**To identify a route / view with a name** 

Named views are needed when multiple views are shown at the same time.
(To show multiple views, multiple <router-view /> -s are needed)
A view without a name will be given `default` as its name.
```js
<!-- template -->
<router-view class="view one"></router-view>  //default
<router-view class="view two" name="a"></router-view> //a=>component Bar
<router-view class="view three" name="b"></router-view> //b=>component Baz

//vue
const router = new VueRouter({
  routes: [
    {
      path: '/',
      components: {
        default: Foo,
        a: Bar,
        b: Baz
      }
    }
  ]
})
```

5. Redirect & Alias
Property 'redirect' receives a(n) path / object or even a function:
```js
routes:[{
        path:'/a', redirect:'/b'
    },
    {
        path:'/c', redirect:{ name:'whatever' }
    },{
        path:'/d', redirect: (to)=> { ... }
    }
    ]
```

* Navigation Guards are not applied on the redirect routes.

- An alias of /a as /b means when the user visits /b, the URL remains /b, but it will be matched as if the user is visiting /a.
To reach the same route with different path ?

6. Passing props to Route Components
**To decouple $route with props**
```js
//before
const User = {
    template:`<div> {{$route.params.id}}</div>`
}
//after
const Client = {
    props:['id'],
    template:`<div>{{id}}</div>`
}
//by setting:
routes = [
    {
        path:'/user/:id' , component:A , props:true
    }, //Boolean mode
//MIND: you have to define the 'props' for each named view
    {
      path: '/user/:id',
      components: { default: User, sidebar: Sidebar },
      props: { default: true, sidebar: false }
    },
    { 
        path: '/promotion/from-newsletter', 
        component: Promotion, 
        props: { newsletterPopup: false } 
    }, //Object mode . To keep the prop as-is.(?)
    { 
        path: '/search', 
        component: SearchUser, 
        props: (route) => ({ query: route.query.q }) 
    } //Function mode.
    //e.g the URL /search?q=vue will pass object {query:vue} to the component SearchUser
]
```

7. Modes of vue-router
  * All three modes : Hash mode , HTML5 Histroy mode, Abstract mode
  * Using Hash mode by default. To change, add the statement : `mode:history`.
- Hash mode will `only change the value of hash`.
Process: $router.push() 
          -->HashHistory.push() 
          -->History.transitionTo() 
          -->History.updateRoute() 
          -->{app._route = route} 
          -->vm.render()
- HTML5 Histroy mode will change the request path,the value of location.pathname.
**Problem: users will get a 404 error if they access a non-exist page.**
To avoid that, add a catch-all route to show a 404 page:
```js
routes = [
    {
        path:* , component:ErrorPage
    }
]
```

8. Advanced: Navigation Guards
Usage: To guard navigations either by redirecting it or canceling it.
Three scopes: globally / per-route / in-component
Use `beforeRouteUpdate` or `watch:{$route(to,from){...}}` object to observe the changes of params and query(They will not trigger enter/leave guards)
#### Globally :router.beforeEach((to,from,next)=>{...}) / router.afterEach() / router.beforeResolve() 
8.1 beforeEach()  //No state detect
Timing: all navigations will be considered `pending` state before all hooks have been resovled (component hooks or what else?)
Params: (1) to /from : The target / current route , the route navigating to / navigated from . 
        (2) next : the function must be called to resolve the hook(?)
         (2.1) next(): move to the next hook.If no hooks are left , `confirms` the nav.
         (2.2) next('/') or next({path:'/'}): redirect to the root path defined in the routes.
         (2.3) next(false) :abort(deny) the navigation.
         (2.4) next(error) : The error will be passed to router.onError(error). A function defined by user. (vue2.4+)
Make sure the next() function is called only once,otherwise it will be considered as `BAD`.

8.2 afterEach((to,from)=>{ ... })
* Called after navigating away

8.3 beforeResolve()
* It will be called once any navigation is going to be resolved. (Only resovled/confirmed navs)

#### Per-Route Guard: beforeEnter()
```js
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: Foo,
      beforeEnter: (to, from, next) => {
        // ...
      }
    }
  ]
})
```

#### In-Component Guards
- beforeRouteEnter  `Called before the route that renders this component is confirmed`
                    **`this` does not refer to the component, which is not even created!**
                    **DIFFERS FROM THE OTHER TWO**
                    SOLUTION: By passing it a third params *next*.
                    ```js
                    beforeRouteEnter (to, from, next) {
                    next(vm => {
                    // access to component instance via `vm`
                    })
                    }
                    ```
- beforeRouteUpdate  `Called when the route changes , but the component is reused`
                    e.g:For a route with dynamic params `/foo/:id`, when we
                     navigate between `/foo/1` and `/foo/2`, the same `Foo` component instance
                     will be reused, and this hook will be called when that happens.
- beforeRouteLeave  `Called when the route is about to be navigated away from`
                    *Used to avoid leaving with not saving any edit info.*

The full flow of navigation (C+V , In-component level first , global guards follow)
1. Navigation triggered.
2. Call `beforeRouteLeave` guards in deactivated components.
3. Call global `beforeEach` guards.

4. Call beforeRouteUpdate guards in reused components.
5. Call beforeEnter in route configs.
6. Resolve async route components.
7. Call beforeRouteEnter in activated components.
8. Call global beforeResolve guards.
9. Navigation confirmed.
10. Call global afterEach hooks.
11. DOM updates triggered.
12. Call callbacks passed to next in beforeRouteEnter guards with instantiated instances.

Self-memorize: in=>glo=>in=>per=>in=>glo=>glo=>in

TO-DOS: transitions & data fetching & scroll behavior & lazy loading





