#### 一. **IOT产品分类模块** (主要品类管理)

- **位置**: `src/views/eiot/category/`
- **功能**: 完整的IOT产品分类管理，支持树形结构

**文件结构**:

- `index.vue` - 分类列表页面
- `CategoryForm.vue` - 分类表单组件
- `src/api/eiot/category/index.ts` - API接口定义
## 二、动态路由数据加载流程

### 1. 登录后获取用户权限信息

当用户登录成功后，系统会调用 `setUserInfoAction` 方法（位于 `user.ts`）：

```
async setUserInfoAction() {
  if (!getAccessToken()) {
    this.resetState()
    return null
  }
  let userInfo = wsCache.get(CACHE_KEY.USER)
  if (!userInfo) {
    userInfo = await getInfo()  // 调用后端API获取权限信息
  }
  this.permissions = new Set(userInfo.permissions)
  this.roles = userInfo.roles
  this.user = userInfo.user
  this.isSetUser = true
  wsCache.set(CACHE_KEY.USER, userInfo)
  wsCache.set(CACHE_KEY.ROLE_ROUTERS, userInfo.menus)  // 关键：将菜单数据存入缓存
}
```

### 2. 后端API调用

`getInfo()` 函数（位于 `index.ts`）调用后端接口：

```
// 获取用户权限信息
export const getInfo = () => {
  return request.get({ url: '/system/auth/get-permission-info' })
}
```

这个接口返回的数据结构包含：

- `permissions`: 用户权限集合
- `roles`: 用户角色列表
- `user`: 用户基本信息
- `menus`: **菜单路由数据**（核心数据）

### 3. 动态路由生成

在权限存储模块（`permission.ts`）中，通过 `generateRoutes` 方法生成动态路由：

```
generateRoutes(): AppRouteRecordRaw[] {
  // 从缓存中获取菜单列表
  const menuList = wsCache.get(CACHE_KEY.ROLE_ROUTERS) as AppCustomRouteRecordRaw[]
  if (!menuList) return []
  
  // 调用路由助手生成路由映射
  const routeMap = generateRoute(menuList)
  
  // 添加404页面路由
  const routes = [...routeMap, ...notFoundRoute]
  
  // 设置动态路由
  this.addRouters = routes
  this.routers = [...remainingRouter, ...routes]
  
  return this.routers
}
```

### 4. 路由转换逻辑

路由助手文件（`routerHelper.ts`）中的 `generateRoute` 函数负责将后端返回的菜单数据转换为前端路由格式：

```
export const generateRoute = (routes: AppCustomRouteRecordRaw[]): AppRouteRecordRaw[] => {
  const res: AppRouteRecordRaw[] = []
  const modulesRoutesKeys = Object.keys(modules)
  
  for (const route of routes) {
    // 生成meta菜单元数据
    const meta = {
      title: route.name,
      icon: route.icon,
      hidden: !route.visible,
      noCache: !route.keepAlive,
      alwaysShow: route.children && route.children.length > 0
    }
    
    // 根据组件路径动态加载Vue组件
    const index = route?.component 
      ? modulesRoutesKeys.findIndex((ev) => ev.includes(route.component))
      : modulesRoutesKeys.findIndex((ev) => ev.includes(route.path))
    
    data.component = modules[modulesRoutesKeys[index]]
    
    // 递归处理子路由
    if (route.children) {
      data.children = generateRoute(route.children)
    }
    
    res.push(data as AppRouteRecordRaw)
  }
  return res
}
```

### 5. 数据流总结

整个动态路由加载的数据流如下：

1. **用户登录** → 获取token
2. **调用getInfo接口** → 获取用户权限和菜单数据
3. **存入缓存** → `CACHE_KEY.ROLE_ROUTERS` 存储菜单数据
4. **路由转换** → `generateRoute` 将后端菜单转换为前端路由格式
5. **路由合并** → 动态路由 + 基础路由 = 完整路由表
6. **路由安装** → 通过Vue Router安装到应用中

### 6. 技术特点

- **权限驱动**: 路由加载基于用户权限，不同用户看到不同的菜单
- **缓存机制**: 使用本地缓存避免重复请求
- **动态导入**: 使用Vite的 `import.meta.glob` 实现组件懒加载
- **类型安全**: 完整的TypeScript类型定义
- **模块化**: 路由生成逻辑与业务逻辑分离

这种设计确保了路由的动态性和安全性，同时提供了良好的用户体验和开发体验。