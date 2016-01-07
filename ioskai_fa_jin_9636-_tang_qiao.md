# iOS开发进阶-唐巧

- ARC使用
- `CoreFoundation`中不支持ARC，需要手动管理内存。
- 转换方式
    - `__bridge` 自己管理内存，不传递引用计数
    - `__bridge_retainded` 引用计数加1，但是释放还得要源对象
    - `__bridge_transfer` 传递引用计数，使用ARC管理内存
- GCD(Grand Central Dispatch)
    - 串行 - 并行队列
    - 同步 - 异步任务
- UIWindow
    - `makeKeyAndVisible` 
    - UIWindow 创建后会自动添加在屏幕上，只需设置其属性Hidden为NO就可显示
    - 常用于登陆界面，锁定界面，以及启动页等
- 动态下载系统的多种字体
    - 下载系统支持的中文字体到系统中 