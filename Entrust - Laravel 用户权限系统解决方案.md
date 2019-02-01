
Zizaco/Entrust 是 Laravel 下 用户权限系统 的解决方案, 配合 用户身份认证 扩展包 Zizaco/confide 使用, 可以快速搭建出一套具备高扩展性的用户系统.

https://github.com/Zizaco/entrust

## Confide, Entrust 和 Sentry

### 首先两个概念分清楚:

用户身份认证 Authentication - 处理用户登录, 退出, 注册, 找回密码, 重置密码, 用户邮箱认证 etc..

权限管理 Authorization - 负责 用户 与 权限, 用户组 三者之间的对应, 以及管理.

下面是这几个 Package 的简单区别:
```
Sentry = 用户身份认证 + 权限管理;

Zizaco/Entrust = 权限管理;

Zizaco/confide = 用户身份认证;
```

### 用户身份认证 和 权限管理 分开来做有什么好处呢?

分开的话可以更灵活, 有些项目因为特殊的业务逻辑, 无法使用 Confide 的 用户身份认证, 但是却需要用到 权限管理, 如: PHPHub .

Laravel-blog 就是一个简单的应用, 使用了 Confide 做 用户身份认证, Entrust 做 权限管理, 可以作为参考.

### 安装
1.composer.json
```
"zizaco/entrust": "1.8.*@dev"
```

```
composer require "zizaco/entrust: 1.8.*@dev"
```

2.install
```
composer update
```

3.provider
修改 app/config/app.php 文件, 在 providers 数组里面添加:
```
'Zizaco\Entrust\EntrustServiceProvider',
```

4.aliase
修改 app/config/app.php 文件, 在 aliases 数组里面添加:
```
'Entrust'    => 'Zizaco\Entrust\EntrustFacade',
```

5.系统配置
Entrust 会利用 config/auth.php 里面的值, 去决定使用那个 Model 和 用户表名.

6.生成 Migration
```
php artisan entrust:migration
```

会生成 <timestamp>_entrust_setup_tables.php Migration 文件, 检查没问题后:
```
php artisan migrate
```

7.Models
创建 app/models/Role.php 文件, 内容为以下:
```
<?php

use Zizaco\Entrust\EntrustRole;

class Role extends EntrustRole
{

}
```

创建 app/models/Permission.php 文件, 内容为以下:
```
<?php

use Zizaco\Entrust\EntrustPermission;

class Permission extends EntrustPermission
{

}
```

修改 app/models/User.php 文件, 添加 HasRole trait:
```
<?php

use Zizaco\Entrust\HasRole;

class User extends Eloquent /* or ConfideUser 'wink' */{
    use HasRole; // Add this trait to your user model

...
```

最后, 生成自动加载:
```
composer dump-autoload
```

### 基础概念
三个主要数据对象, 以及他们之间的关系:

User - 用户, 一个用户可以属于多个用户组, 不直接挂钩权限, 让用户组和权限绑定;

Roles - 用户组, 一个用户组可以拥有多个权限;

Permission - 权限;

四个数据库表说明:

安装的第 6 步会产生一个 Migration, 此文件定义了 4 张数据库表:

roles - 用户组信息表;

assigned_roles - 用户和用户组之间的对应关系;

permissions - 权限信息表;

permission_role - 权限和用户组之间的对应关系.

### 实例

接下来我们来创建用户组和权限, 并授权用户

创建用户组
```
$admin = new Role;
$admin->name = 'Admin';
$admin->save();

$owner = new Role;
$owner->name = 'Owner';
$owner->save();
```
创建权限
```
$manageUsers = new Permission;
$manageUsers->name = 'manage_users';
$manageUsers->display_name = 'Manage Users';
$manageUsers->save();

$managePosts = new Permission;
$managePosts->name = 'manage_posts';
$managePosts->display_name = 'Manage Posts';
$managePosts->save();
```
添加用户组权限
```
$owner->perms()->sync(array($managePosts->id, $manageUsers->id));
$admin->perms()->sync(array($managePosts->id));
```
添加用户到用户组
```
// 获取用户
$user = User::where('username','=','Zizaco')->first();

// 可以使用 Entrust 提供的便捷方法用户授权
// 注: 参数可以为 Role 对象, 数组, 或者 ID
$user->attachRole( $admin ); 

// 或者使用 Eloquent 自带的对象关系赋值
$user->roles()->attach( $admin->id ); // id only
```

### 使用

基本权限判断

判断用户是否属于某个用户组:
```
$user->hasRole("Owner");    // false
$user->hasRole("Admin");    // true
判断用户是否拥有某个权限 (通过用户组):

$user->can("manage_posts"); // true
$user->can("manage_users"); // false
使用 ability 方法同时判断多个权限和用户组

$user->ability(['Admin','Owner'], ['manage_posts','manage_users']);
// 或者
$user->ability('Admin,Owner', 'manage_posts,manage_users');
```

路由过滤

Entrust 还提供帮助方法, 用来做路由过滤:
```
// 有 `manage_posts` 权限的用户, 才能访问 `admin/posts` 开头的链接
Entrust::routeNeedsPermission( 'admin/post*', 'manage_posts' );

// 属于 `Owner` 用户组的人, 才能访问 `admin/advanced*` 开头的链接
Entrust::routeNeedsRole( 'admin/advanced*', 'Owner' );

// 可以在第二个选项传参数组, 当前用户需要符合所有传参的用户组或者权限, 
// 才能授权成功
Entrust::routeNeedsPermission( 'admin/post*', ['manage_posts','manage_comments'] );
Entrust::routeNeedsRole( 'admin/advanced*', ['Owner','Writer'] );
```